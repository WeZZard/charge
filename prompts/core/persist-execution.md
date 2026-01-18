# Core: Persist Execution

Handle all execution-time persistence operations to keep the main orchestration context clean.

## Operations

### Operation: `init`

Create execution directories and initialize state.json.

**Input**:
- `workflow_path`: Path to the workflow directory
- `workflow_name`: Name of the workflow (without date prefix)
- `session_id`: Computed session ID

**Process**:

1. Generate execution timestamp: `{YYYY-MM-DD-hh-mm-ss}` (current UTC time)

2. Create execution directory and results subdirectory:
   ```bash
   mkdir -p $HOME/.charge/sessions/{session_id}/{execution_timestamp}-{workflow_name}/results
   ```

3. Initialize `state.json` at `{execution_path}/state.json`:
   ```json
   {
     "workflow_ref": "{workflow_path}",
     "session_id": "{session_id}",
     "execution_id": "{execution_timestamp}-{workflow_name}",
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

**Output**:
```json
{
  "execution_path": "$HOME/.charge/sessions/{session_id}/{execution_id}",
  "state_path": "$HOME/.charge/sessions/{session_id}/{execution_id}/state.json"
}
```

---

### Operation: `task_complete`

Update state.json after a task completes successfully.

**Input**:
- `execution_path`: Path to the execution directory
- `task_id`: ID of the completed task
- `result_path`: Relative path to the result file (e.g., `results/task_01.json`)
- `next_task`: ID of the next task (or null if done)

**Process**:

1. Read current `{execution_path}/state.json`

2. Update fields:
   - Add `task_id` to `completed_tasks` array
   - Set `results[task_id]` to `result_path`
   - Set `current_task` to `next_task`

3. Write updated state.json

**Output**:
```json
{
  "success": true,
  "state_path": "{execution_path}/state.json"
}
```

---

### Operation: `task_failed`

Update state.json when a task fails after all retries.

**Input**:
- `execution_path`: Path to the execution directory
- `task_id`: ID of the failed task
- `error`: Error message

**Process**:

1. Read current `{execution_path}/state.json`

2. Update fields:
   - Add `task_id` to `failed_tasks` array
   - Set `error` to the error message
   - Set `status` to `"paused"`

3. Write updated state.json

**Output**:
```json
{
  "success": true,
  "state_path": "{execution_path}/state.json"
}
```

---

### Operation: `finalize`

Mark execution as completed after all tasks finish.

**Input**:
- `execution_path`: Path to the execution directory

**Process**:

1. Read current `{execution_path}/state.json`

2. Update fields:
   - Set `status` to `"completed"`
   - Set `completed_at` to current ISO timestamp
   - Set `current_task` to null

3. Write updated state.json

**Output**:
```json
{
  "success": true,
  "state_path": "{execution_path}/state.json"
}
```

---

## Invocation Pattern

From `run.md`, invoke this task directly (not via Task tool):

```markdown
Follow `prompts/core/persist-execution.md` with operation `init`:
- workflow_path: {workflow_path}
- workflow_name: {workflow-name}
- session_id: {SESSION_ID}
```

The persist operations are lightweight file I/O that don't require sub-agent isolation.
