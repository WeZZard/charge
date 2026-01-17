# Core: Build Flow

Determine the execution order and create the workflow manifest.

## Input

- `tasks`: The task list with dependencies
- `schemas`: The generated JSON Schemas
- `mappings`: The field mappings between tasks

## Process

### Step 1: Topological Sort

Order tasks based on their dependencies:

1. Find all tasks with no dependencies (these can start first)
2. For each remaining task, ensure all its dependencies come before it
3. Detect cycles (if any task depends on itself through a chain, error out)

### Step 2: Identify Parallel Groups

Tasks can run in parallel if:

- They have no dependency on each other
- All their dependencies are already completed

Group tasks into parallel execution groups:

```ascii
Sequential: task_01 → task_02 → task_03
Parallel:   task_01 → [task_02, task_03] → task_04
```

### Step 3: Generate Workflow Name

Derive a kebab-case name from the original prompt:

- Extract key nouns and verbs
- Combine into a concise identifier
- Example: "Build a REST API with auth" → `build-rest-api-auth`

### Step 4: Create Manifest

Build the `manifest.json` structure:

```json
{
  "schema_version": "[string]",
  "id": "[string]",
  "name": "[string]",
  "created": "[string]",
  "source_prompt": "[string]",

  "tasks": [
    {
      "id": "[string]",
      "name": "[string]",
      "description": "[string]",
      "instruction_file": "[string]",
      "input_schema": "[string]",
      "output_schema": "[string]",
      "depends_on": ["[string]"]
    }
  ],

  "flow": {
    "type": "sequential|dag",
    "order": ["[string]"],
    "parallel_groups": [["[string]"]]
  },

  "mappings": [
    {
      "target_task": "[string]",
      "target_field": "[string]",
      "source_task": "[string]",
      "source_path": "[string]"
    }
  ]
}
```

### Step 5: Create State File

Initialize `state.json`:

```json
{
  "workflow_id": "[string]",
  "status": "pending|running|completed|failed",
  "current_task": "[string]|null",
  "completed_tasks": ["[string]"],
  "failed_tasks": ["[string]"],
  "results": {},
  "started_at": "[string]|null",
  "completed_at": "[string]|null",
  "error": "[string]|null"
}
```

## Output

The complete manifest and state structures, ready to be written to disk.

## Flow Type Decision

Use `"type": "sequential"` when:

- All tasks depend on the previous one
- Simple linear execution

Use `"type": "dag"` when:

- Some tasks can run in parallel
- Complex dependency graph

## Example Flows

### Sequential

```markdown
task_01 → task_02 → task_03 → task_04
```

```json
{
  "type": "sequential",
  "order": ["task_01", "task_02", "task_03", "task_04"],
  "parallel_groups": []
}
```

### Parallel (diamond pattern)

```ascii
         ┌─ task_02 ─┐
task_01 ─┤           ├─ task_04
         └─ task_03 ─┘
```

```json
{
  "type": "dag",
  "order": ["task_01", "task_02", "task_03", "task_04"],
  "parallel_groups": [["task_02", "task_03"]]
}
```
