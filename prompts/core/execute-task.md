# Core: Execute Task

Execute a single task within a workflow by delegating to a sub-agent.

## Input

- `task`: The task definition from manifest
- `workflow_path`: Absolute path to the workflow directory
- `input_source`: For template tasks, the file path or reference to process
- `item_index` (optional): For template tasks, the current item index
- `total_items` (optional): For template tasks, total number of items

## Process

### Step 1: Prepare Minimal Context

The main agent should NOT read:

- Instruction file content
- Input data content (blog posts, source files, etc.)
- Output schema content

Instead, only determine **file paths** to pass to the sub-agent.

**Key paths:**
- Instruction: `{workflow_path}/instructions/{task_id}.md`
- Input schema: `{workflow_path}/schemas/{task_id}_input.json`
- Output schema: `{workflow_path}/schemas/{task_id}_output.json`
- Output file: `{workflow_path}/results/{task_id}.json`

### Step 2: Execute with Task Tool

**CRITICAL**: The sub-agent reads all files itself. The main agent only passes paths.

**For regular tasks**, invoke the Task tool:

```text
Task(
  description: "{task-name}",
  subagent_type: "general-purpose",
  prompt: "
    # Charge Task Execution

    Workflow: {workflow-path}
    Task ID: {task-id}

    ## Step 1: Read Instructions
    Read the instruction file at:
    {workflow-path}/instructions/{task-id}.md

    ## Step 2: Read Input
    Read previous task results from:
    {workflow-path}/results/

    Relevant result files: {list of dependency result files}

    ## Step 3: Execute
    Follow the instructions to process the input data.

    ## Step 4: Write Output
    Write your JSON output to:
    {workflow-path}/results/{task-id}.json

    Your output must match the schema at:
    {workflow-path}/schemas/{task-id}_output.json

    ## Step 5: Report Status
    After writing the file, respond with ONLY:

    {\"status\": \"success\", \"output_path\": \"{workflow-path}/results/{task-id}.json\"}

    If you encounter an error:

    {\"status\": \"failed\", \"error\": \"[brief error description]\"}
  "
)
```

**For template task iterations**, adjust the prompt:

```text
Task(
  description: "{task-name} [{item-index}/{total-items}]",
  subagent_type: "general-purpose",
  prompt: "
    # Charge Task Execution

    Workflow: {workflow-path}
    Task ID: {task-id}
    Item: {item-index} of {total-items}

    ## Step 1: Read Instructions
    Read the instruction file at:
    {workflow-path}/instructions/{task-id}.md

    ## Step 2: Read Input
    Read the source file at:
    {input-source-path}

    ## Step 3: Execute
    Follow the instructions to process the input data.

    ## Step 4: Write Output
    Write your JSON output to:
    {workflow-path}/results/{task-id}_item_{item-index:02d}.json

    Your output must match the schema at:
    {workflow-path}/schemas/{task-id}_output.json

    ## Step 5: Report Status
    After writing the file, respond with ONLY:

    {\"status\": \"success\", \"output_path\": \"{workflow-path}/results/{task-id}_item_{item-index:02d}.json\"}

    If you encounter an error:

    {\"status\": \"failed\", \"error\": \"[brief error description]\"}
  "
)
```

### Step 3: Process Sub-Agent Response

The Task tool result includes:

1. **Agent ID**: Capture and store `agentId: {agent-id}` for potential retries
2. **Status object**: The sub-agent's response

```json
{
  "status": "success|failed",
  "output_path": "[string]",
  "error": "[string]"
}
```

Process the status:

1. If `status` is `"success"`:
   - Proceed to Step 4 (do NOT read the output file in main agent)
   - Trust the sub-agent's validation

2. If `status` is `"failed"`:
   - Go to Step 5 (retry) with the error message

### Step 4: Update State

Update `state.json`:

- Add task_id to `completed_tasks`
- Add result path to `results`
- Update `current_task` to next task (or null if done)

**For template task iterations**:

- Do NOT update `completed_tasks` until ALL items are processed
- Track individual item completions in `iteration_state`
- Only mark the template task as complete after all items finish

### Step 5: Handle Failure (Retry)

If the sub-agent returns `"failed"` status:

1. Parse the error message
2. **Resume** the same sub-agent using the stored agent ID:

```text
Task(
  resume: "{agent-id}",
  prompt: "
    Your previous attempt failed:
    - Error: {error message}

    Please fix and try again. Write corrected output to the same path.
  "
)
```

3. Retry up to 2 times (each retry resumes the same agent)
4. If still failing, mark task as failed and pause workflow

**For template task iterations**:

- Failed items should not block other items from processing
- Track failed items separately
- After all items attempted, report which items failed
- Allow user to choose: retry failed items, skip them, or abort

### Step 6: Report Progress

**For regular tasks**:

```
[{current}/{total}] {task-name}... done
```

Or on failure:

```
[{current}/{total}] {task-name}... FAILED
Error: {error message}
```

**For template task iterations**:

```
[{task-current}/{task-total}] {task-name} [{item-current}/{item-total}]... done
```

Or on failure:

```
[{task-current}/{task-total}] {task-name} [{item-current}/{item-total}]... FAILED
Error: {error message}
(Continuing with remaining items...)
```

## Important Notes

- **Main agent passes PATHS, not CONTENT** - sub-agents read their own files
- **ALWAYS use the Task tool** - spawns isolated sub-agent, prevents context explosion
- **Capture the agent ID** from Task tool result for potential retries
- **Use resume for retries** - resumed agents preserve context
- **Sub-agents write output to files** and return only status + path
- Each task execution is independent - don't carry over context
- **For template tasks**: Each item is fully independent
