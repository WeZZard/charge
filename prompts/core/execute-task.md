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

### Step 2: Execute with Schema Constraint

Present the task to Claude with this structure:

```markdown
# Task: {task_name}

{instruction content from file}

---

## Input Data

{JSON formatted input data}

---

## Required Output Format

Your response MUST be valid JSON matching this schema:

{output schema}

Respond with ONLY the JSON output. No explanations, no markdown code blocks, just raw JSON.
```

### Step 3: Validate Output

After receiving the task output:

1. Parse the response as JSON
2. Validate against the output schema
3. If valid, proceed to Step 4
4. If invalid, go to Step 5 (retry)

### Step 4: Persist Result

Write the validated output to `results/{task_id}.json`:

```json
{
  "task_id": "{task_id}",
  "executed_at": "{ISO-timestamp}",
  "output": { /* the validated output */ }
}
```

Update `state.json`:

- Add task_id to `completed_tasks`
- Add result reference to `results`
- Update `current_task` to next task (or null if done)

### Step 5: Handle Validation Failure (Retry)

If output validation fails:

1. Parse the validation error
2. Re-invoke the task with error context:

```markdown
# Task: {task_name}

{instruction content}

---

## Input Data

{input data}

---

## Previous Attempt Failed

Your previous output failed validation:
- Error: {validation error message}
- Path: {JSON path to invalid field}
- Expected: {expected type/format}
- Got: {actual value}

Please fix the output and try again.

---

## Required Output Format

{output schema}

Respond with ONLY valid JSON.
```

3. Retry up to 2 times
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

- Each task execution should be independent - don't carry over context from previous tasks
- Only pass data through the explicit schema-defined inputs
- The instruction file contains task-specific guidance; the schema defines the contract
- Retry mechanism helps recover from minor output format issues
