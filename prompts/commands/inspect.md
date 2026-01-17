# Command: /flowit:inspect

View the structure and status of a workflow.

## Input

- `id`: The workflow identifier (workflow name or full path)

## Workflow

### Step 1: Locate the Workflow

1. Search for the workflow in `.flowit/` directory
2. If not found, report error and suggest `/flowit:list`

### Step 2: Load Workflow Data

Read from the workflow directory:

- `manifest.json` - workflow definition
- `state.json` - execution state

### Step 3: Present Workflow Information

Display the workflow details in this format:

```markdown
## Workflow: {name}

**ID**: {id}
**Location**: {path}
**Created**: {created timestamp}
**Status**: {pending | running | completed | failed}
**Schema Version**: {schema_version}

---

### Tasks ({count} total)

| # | Task | Status | Input Schema | Output Schema |
|---|------|--------|--------------|---------------|
| 1 | {name} | {status} | {schema file} | {schema file} |
| 2 | {name} | {status} | {schema file} | {schema file} |
| ... | ... | ... | ... | ... |

---

### Execution Flow

Type: {Sequential | DAG}
Order: {task_1} → {task_2} → {task_3} → ...

{If parallel groups exist:}
Parallel Groups:
  - Group 1: {task_a}, {task_b}
  - Group 2: {task_c}, {task_d}

---

### Results

{If completed tasks exist:}
Completed task results are in: {path}/results/

| Task | Result File | Size |
|------|-------------|------|
| {name} | results/{task_id}.json | {size} |
| ... | ... | ... |

{If no completed tasks:}
No tasks have been executed yet.

---

**Actions**:
- Run: `/flowit:run {id}`
- Delete: `/flowit:delete {id}`
```

## Notes

- This is a read-only command - it doesn't modify anything
- Show file sizes to help users understand output volumes
- Highlight failed tasks if the workflow status is "failed"
