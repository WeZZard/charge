# Command: /flowit:run

Execute an existing workflow by ID.

## Input

- `id`: The workflow identifier (workflow name or full path)

## Workflow

### Step 1: Locate the Workflow

1. Search for the workflow in `.flowit/` directory
2. If `id` is a full path, use it directly
3. If `id` is just a name, search in date-ordered directories (most recent first)
4. Load `manifest.json` from the workflow directory

If workflow not found, report error:

```markdown
Workflow not found: {id}
Run /flowit:list to see available workflows.
```

### Step 2: Load Workflow State

1. Read `state.json` to check current status
2. If status is "completed", ask user if they want to re-run
3. If status is "failed", show the failure point and ask to continue or restart

### Step 3: Execute Tasks

For each task in the execution order defined in `manifest.json`:

1. **Load Task Context**
   - Read the task's instruction file from `instructions/{task_id}.md`
   - Read the task's input schema from `schemas/{task_id}_input.json`
   - Read the task's output schema from `schemas/{task_id}_output.json`

2. **Build Task Input**
   - For the first task: use the original prompt/context
   - For subsequent tasks: map outputs from completed tasks using the mappings in manifest

3. **Execute Task**
   - Follow the instructions in `prompts/core/execute-task.md`
   - The task instruction file contains the specific guidance
   - Constrain output to match the output schema

4. **Validate Output**
   - Validate the task output against its output schema
   - If validation fails, retry with error feedback (max 2 retries)
   - If still failing after retries, pause and ask user for guidance

5. **Persist Result**
   - Write the validated output to `results/{task_id}.json`
   - Update `state.json` with completed task

6. **Report Progress**

   ```markdown
   [{current}/{total}] {task-name}... done
   ```

### Step 4: Synthesize Results

After all tasks complete:

1. Follow `prompts/core/synthesize.md` to combine task outputs
2. Present the final result to the user
3. Update `state.json` status to "completed"

## Output Format

```markdown
Executing workflow: {workflow-name}
Location: {workflow-path}

[1/5] parse-requirements... done
[2/5] design-api-schema... done
[3/5] implement-endpoints... done
[4/5] add-authentication... done
[5/5] generate-tests... done

Workflow complete.
Results saved to: {workflow-path}/results/

---
{Synthesized final output}
```

## Error Handling

- **Task execution failure**: Retry with error context, then pause for user input
- **Schema validation failure**: Retry with validation error details
- **Missing dependencies**: Report which predecessor task outputs are missing
