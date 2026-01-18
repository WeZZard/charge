# Command: /charge:run

Execute an existing workflow by ID.

## Input

- `id`: The workflow identifier (workflow name or full path)

## Workflow

### Step 1: Locate the Workflow

1. Search for the workflow in `$HOME/.charge/workflows/` directory
2. If `id` is a full path, use it directly
3. If `id` is just a name, search by matching the workflow name suffix (ignoring date prefix)
4. Load `manifest.json` from the workflow directory

If workflow not found, report error:

```markdown
Workflow not found: {id}
Run /charge:list to see available workflows.
```

### Step 2: Initialize Execution

1. **Compute session ID** with a single bash command:
   ```bash
   SESSION_ID=$(LANG=C ps -p $PPID -o lstart= | awk 'BEGIN{m["Jan"]="01";m["Feb"]="02";m["Mar"]="03";m["Apr"]="04";m["May"]="05";m["Jun"]="06";m["Jul"]="07";m["Aug"]="08";m["Sep"]="09";m["Oct"]="10";m["Nov"]="11";m["Dec"]="12"}{gsub(/:/,"-",$4);printf "%s-%s-%02d-%s",$5,m[$2],$3,$4}')-$PPID
   ```
   - Example result: `2026-01-18-20-34-14-50622`

2. **Create execution environment** by following `prompts/core/persist-execution.md` with operation `init`:
   - workflow_path: `{workflow_path}`
   - workflow_name: `{workflow-name}`
   - session_id: `{SESSION_ID}`

   This creates directories and initializes `state.json`. Returns:
   - `execution_path`: Path to the execution directory
   - `state_path`: Path to state.json

Store both paths for use throughout execution:
- `workflow_path`: `$HOME/.charge/workflows/{YYYY-MM-DD}-{workflow-name}/`
- `execution_path`: (returned from persist-execution)

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

**4.3 Execute Items Based on Strategy**

For each item, determine input source path first:
- DO NOT read the file content
- Only construct the full path to the source file
- For static items (like blog posts): resolve relative path to absolute path

**If `iteration.strategy` is `sequential`:**

For each item in the collection (index `i` starting at 1):
1. Execute the task via `prompts/core/execute-task.md`
2. Pass only: workflow path, task ID, input source path, item index, total items
3. Wait for completion
4. Report progress: `[{task}/{total}] {name} [{i}/{items}]... done`
5. Proceed to next item

**If `iteration.strategy` is `parallel`:**

1. Batch items into groups of 5
2. For each batch:
   - **Spawn ALL Task tool calls in a SINGLE message** (this is critical for parallelism)
   - Each Task call follows `prompts/core/execute-task.md`
   - Wait for all tasks in the batch to complete
   - Report batch progress: `[{task}/{total}] {name} [{start}-{end}/{items}]... done`
3. Proceed to next batch
4. If any fail, retry failed items up to 2 times each

**MANDATORY**: For parallel execution, you MUST invoke multiple Task tools in ONE message.
Sequential invocation defeats the purpose of parallel strategy.

**4.5 Aggregate Template Results**

After all items complete:

1. Collect all item result file paths from `{execution_path}/results/`

2. Create an aggregated result at `{execution_path}/results/{task_id}.json` containing:
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

3. Follow `prompts/core/persist-execution.md` with operation `task_complete`:
   - execution_path: `{execution_path}`
   - task_id: `{task_id}`
   - result_path: `results/{task_id}.json`
   - next_task: (next task ID or null)

4. Return to Step 3 for the next task

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

On success:
- Follow `prompts/core/persist-execution.md` with operation `task_complete`:
  - execution_path: `{execution_path}`
  - task_id: `{task_id}`
  - result_path: `results/{task_id}.json`
  - next_task: (next task ID or null)

On failure:
- Retry up to 2 times
- If still failing, follow `prompts/core/persist-execution.md` with operation `task_failed`:
  - execution_path: `{execution_path}`
  - task_id: `{task_id}`
  - error: `{error message}`
- Pause for user guidance

**5.4 Report Progress**

```markdown
[{current}/{total}] {task-name}... done
```

Return to Step 3 for the next task.

### Step 6: Synthesize Results

After all tasks complete:

1. Follow `prompts/core/synthesize.md` to combine task outputs from `{execution_path}/results/`
2. Present the final result to the user
3. Follow `prompts/core/persist-execution.md` with operation `finalize`:
   - execution_path: `{execution_path}`

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
