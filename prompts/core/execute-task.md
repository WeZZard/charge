# Core: Execute Task

Execute a single task within a workflow.

## Input

- `task`: The task definition from manifest
- `instruction`: The task instruction content (loaded from file)
- `input_data`: The validated input data for this task
- `input_schema`: The input JSON Schema
- `output_schema`: The output JSON Schema

## Process

### Step 1: Prepare Execution Context

Load minimal context for this task:

- The task instruction (from `instructions/{task_id}.md`)
- The input data (built from previous task outputs)
- The output schema (to constrain the response)

Do NOT load:

- Other task instructions
- The full workflow manifest
- Results from unrelated tasks

This keeps context minimal to prevent window explosion.

### Step 2: Execute with Task Tool

**CRITICAL**: You MUST use the **Task tool** to spawn a sub-agent for executing this task. This ensures:

1. Each task runs in an isolated context (prevents context window explosion)
2. Only task-specific information is passed to the sub-agent
3. The main agent's context stays clean for orchestration

Invoke the Task tool:

```text
Task(
  description: "{task-name}",
  subagent_type: "general-purpose",
  prompt: "
    # Task: {task-name}

    {instruction content from file}

    ---

    ## Input Data

    {JSON formatted input data}

    ---

    ## Output Requirements

    1. Your output MUST be valid JSON matching this schema:

    {output schema}

    2. Write your output to: {workflow-path}/results/{task-id}.json

    3. After writing the file, respond with ONLY this status JSON:

    {\"status\": \"success\", \"output_path\": \"{workflow-path}/results/{task-id}.json\"}

    If you encounter an error, respond with:

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
   - Read the output file from `output_path`
   - Validate against the output schema
   - If valid, proceed to Step 4
   - If invalid, go to Step 5 (retry)

2. If `status` is `"failed"`:
   - Go to Step 5 (retry) with the error message

### Step 4: Update State

Update `state.json`:

- Add task_id to `completed_tasks`
- Add result reference to `results`
- Update `current_task` to next task (or null if done)

### Step 5: Handle Failure (Retry)

If validation fails or the sub-agent returns `"failed"` status:

1. Parse the error (validation error or sub-agent error)
2. **Resume** the same sub-agent using the stored agent ID:

```text
Task(
  resume: "{agent-id}",
  prompt: "
    Your previous attempt failed:
    - Error: {error message}
    - Details: {validation path or additional context}

    Please fix and try again. Write corrected output to the same path.
  "
)
```

3. Retry up to 2 times (each retry resumes the same agent)
4. If still failing, mark task as failed and pause workflow

### Step 6: Report Progress

Output to user:

```ascii
[{current}/{total}] {task-name}... done
```

Or on failure:

```ascii
[{current}/{total}] {task-name}... FAILED
Error: {error message}
```

## Important Notes

- **ALWAYS use the Task tool** to execute each task - this spawns an isolated sub-agent and prevents context explosion
- **Capture the agent ID** from the Task tool result for potential retries
- **Use resume for retries** - resumed agents preserve context, so retry prompts only need error feedback
- **Sub-agents write output to files** and return only status + path - keeps orchestrator context minimal
- Each task execution should be independent - don't carry over context from previous tasks
- Only pass data through the explicit schema-defined inputs
