# Command: /charge:run

Execute an existing workflow by ID.

## Input

- `id`: The workflow identifier (workflow name or full path)

## Workflow

### Step 1: Locate the Workflow

1. Search for the workflow in `.charge/workflows/` directory
2. If `id` is a full path, use it directly
3. If `id` is just a name, search by matching the workflow name suffix (ignoring date prefix)
4. Load `manifest.json` from the workflow directory

If workflow not found, report error:

```markdown
Workflow not found: {id}
Run /charge:list to see available workflows.
```

### Step 2: Get Session ID and Create Execution Directory

1. Get the Claude Code parent process info via bash:
   ```bash
   ps -p $PPID -o lstart,pid | tail -1
   ```
2. Parse the output to construct session ID: `{YYYY-MM-DD-hh-mm-ss}-{PPID}`
   - Example output: `Sun Jan 18 20:34:14 2026 50622`
   - Session ID: `2026-01-18-20-34-14-50622`
3. Generate execution timestamp: `{YYYY-MM-DD-hh-mm-ss}` (current UTC time)
4. Create execution directory: `.charge/sessions/{session_id}/{execution_timestamp}-{workflow-name}/`
5. Create `results/` subdirectory
6. Initialize `state.json` with:
   ```json
   {
     "workflow_ref": "workflows/{YYYY-MM-DD}-{workflow-name}",
     "session_id": "{session_id}",
     "execution_id": "{execution_timestamp}-{workflow-name}",
     "status": "running",
     "current_task": null,
     "completed_tasks": [],
     "failed_tasks": [],
     "results": {},
     "started_at": "{ISO-timestamp}",
     "completed_at": null,
     "error": null
   }
   ```

Store both paths for use throughout execution:
- `workflow_path`: `.charge/workflows/{YYYY-MM-DD}-{workflow-name}/`
- `execution_path`: `.charge/sessions/{session_id}/{execution_timestamp}-{workflow-name}/`

### Step 3: Execute Tasks

For each task in the execution order defined in `manifest.json`:

1. Read the task definition from `manifest.json`
2. Check if `is_template` is `true`
   - If yes → Follow **Step 4 (Execute Template Task)**
   - If no → Follow **Step 5 (Execute Regular Task)**
3. After task completion, proceed to the next task in execution order

### Step 4: Execute Template Task

Template tasks iterate over a collection and execute once per item.

**4.1 Resolve the Collection**

- Read `iteration.over` from the task definition
- For `static_list`: use items from `manifest.json`'s `static_items` array
- For JSONPath (e.g., `$.tasks.task_01.output.posts`): resolve against completed task results
- If resolution fails, pause and report the error

**4.2 Determine Iteration Strategy**

- Read `iteration.strategy` from the task definition
- `sequential`: Execute one item at a time, wait for completion before next
- `parallel`: Execute all items concurrently using multiple Task tool calls (batch size: 5)

**4.3 Execute for Each Item**

For each item in the resolved collection (with index `i` starting at 1):

1. **Determine Input Source Path**
   - DO NOT read the file content
   - Only construct the full path to the source file
   - For static items (like blog posts): resolve relative path to absolute path

2. **Execute the Task**
   - Follow `prompts/core/execute-task.md`
   - Pass only: workflow path, task ID, input source path, item index, total items
   - The sub-agent will read all files itself

3. **Report Item Progress**
   ```
   [{task_current}/{task_total}] {task-name} [{item_current}/{item_total}]... done
   ```

**4.4 Handle Parallel Execution**

When `iteration.strategy` is `parallel`:

- Spawn multiple Task tool calls in a single message (up to 5 concurrent)
- Wait for all to complete before spawning next batch
- If any fail, retry failed items up to 2 times each

**4.5 Aggregate Template Results**

After all items complete:

- Collect all item result file paths from `{execution_path}/results/`
- Create an aggregated result at `{execution_path}/results/{task_id}.json` containing:
  ```json
  {
    "items": [
      {"index": 1, "result_path": "results/{task_id}_item_01.json"},
      {"index": 2, "result_path": "results/{task_id}_item_02.json"}
    ],
    "total_count": [number],
    "success_count": [number],
    "failed_count": [number]
  }
  ```
- Update `{execution_path}/state.json` with the aggregated result path
- Return to Step 3 for the next task

### Step 5: Execute Regular Task

For non-template tasks:

**5.1 Determine File Paths**

DO NOT read any files. Only determine paths:

**Workflow paths** (read-only, from workflow directory):
- Instruction file: `{workflow_path}/instructions/{task_id}.md`
- Input schema: `{workflow_path}/schemas/{task_id}_input.json`
- Output schema: `{workflow_path}/schemas/{task_id}_output.json`

**Execution paths** (read-write, from execution directory):
- Output file: `{execution_path}/results/{task_id}.json`
- Dependency results: `{execution_path}/results/{dep_task_id}.json` for each dependency

**5.2 Execute Task**

- Follow `prompts/core/execute-task.md`
- Pass: workflow path, execution path, task ID, list of dependency result paths
- The sub-agent will read all files itself

**5.3 Handle Response**

- On success: update `{execution_path}/state.json` with completed task
- On failure: retry up to 2 times, then pause for user guidance

**5.4 Report Progress**

```markdown
[{current}/{total}] {task-name}... done
```

Return to Step 3 for the next task.

### Step 6: Synthesize Results

After all tasks complete:

1. Follow `prompts/core/synthesize.md` to combine task outputs from `{execution_path}/results/`
2. Present the final result to the user
3. Update `{execution_path}/state.json` status to "completed"

## Output Format

```markdown
Executing workflow: {workflow-name}
Workflow: {workflow_path}
Session: {execution_path}

[1/3] discover-items... done
[2/3] process-item [1/10]... done
[2/3] process-item [2/10]... done
...
[2/3] process-item [10/10]... done
[3/3] aggregate-results... done

Workflow complete.
Results saved to: {execution_path}/results/

---
{Synthesized final output}
```

## Error Handling

- **Task execution failure**: Retry with error context, then pause for user input
- **Missing dependencies**: Report which predecessor task outputs are missing
- **Template item failure**: Continue with remaining items, report failures at end
- **Collection resolution failure**: If `iteration.over` cannot be resolved, pause and report the error

## Key Principle: Main Agent Passes Paths, Not Content

The main orchestrator agent should NEVER:
- Read instruction file content
- Read source file content (blog posts, code files, etc.)
- Read schema file content
- Pass content inline in Task prompts

The sub-agent is responsible for reading all necessary files.

## Template Task Examples

### Example 1: Sequential Processing

Manifest defines:
```json
{
  "id": "task_02",
  "name": "process-file",
  "is_template": true,
  "iteration": {
    "over": "$.tasks.task_01.output.files",
    "item_alias": "file",
    "strategy": "sequential"
  }
}
```

Execution:
```
[2/3] process-file [1/5]... done
[2/3] process-file [2/5]... done
[2/3] process-file [3/5]... done
[2/3] process-file [4/5]... done
[2/3] process-file [5/5]... done
```

### Example 2: Static List Processing

Manifest defines:
```json
{
  "id": "task_01",
  "name": "review-post",
  "is_template": true,
  "iteration": {
    "over": "static_list",
    "item_alias": "post_path",
    "strategy": "sequential"
  }
}
```

With `static_items` in manifest:
```json
{
  "static_items": [
    "2019-02-05-hello-world.md",
    "2019-03-01-aop/index.md",
    ...
  ]
}
```

Execution:
```
[1/2] review-post [1/20]... done
[1/2] review-post [2/20]... done
...
```

### Example 3: Parallel Processing

Manifest defines:
```json
{
  "id": "task_02",
  "name": "analyze-endpoint",
  "is_template": true,
  "iteration": {
    "over": "$.tasks.task_01.output.endpoints",
    "item_alias": "endpoint",
    "strategy": "parallel"
  }
}
```

Execution (spawn 5 Task agents in single message):
```
[2/3] analyze-endpoint [1-5/20]... done (parallel batch)
[2/3] analyze-endpoint [6-10/20]... done (parallel batch)
[2/3] analyze-endpoint [11-15/20]... done (parallel batch)
[2/3] analyze-endpoint [16-20/20]... done (parallel batch)
```
