# Command: /charge:inspect

View the structure of a workflow definition.

## Input

- `id`: The workflow identifier (workflow name or full path)

## Workflow

### Step 1: Locate the Workflow

1. Search for the workflow in `.charge/workflows/` directory
2. If `id` is just a name, search by matching the workflow name suffix (ignoring date prefix)
3. If not found, report error and suggest `/charge:list`

### Step 2: Load Workflow Data

Read from the workflow directory:

- `manifest.json` - workflow definition

### Step 3: Present Workflow Information

Display the workflow details in this format:

```markdown
## Workflow: {name}

**Location**: .charge/workflows/{YYYY-MM-DD}-{name}/
**Created**: {created timestamp from directory name}
**Schema Version**: {schema_version}

---

### Tasks ({count} total)

| # | Task | Description | Complexity | Output Size |
|---|------|-------------|------------|-------------|
| 1 | {name} | {description} | {complexity} | {output_size} |
| 2 | {name} | {description} | {complexity} | {output_size} |
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

### Files

| Directory | Contents |
|-----------|----------|
| instructions/ | {count} task instruction files |
| schemas/ | {count} schema files |

---

**Actions**:
- Run: `/charge:run {name}`
- Delete: `/charge:delete {name}`

**Execution History**: Use `ls .charge/sessions/` to view past executions.
```

## Notes

- This is a read-only command - it doesn't modify anything
- Workflow definitions do not include execution state (that's in sessions)
- Use `ls .charge/sessions/{session_id}/` to find execution results
