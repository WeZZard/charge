# Command: /charge:list

List all workflows in the current project.

## Input

None required.

## Workflow

### Step 1: Find Workflows

1. Check if `$HOME/.charge/workflows/` directory exists in the current project
2. If not, report:

   ```markdown
   No workflows found in this project.
   Create one with: /charge <your prompt>
   ```

### Step 2: Scan Workflow Directories

For each workflow directory in `$HOME/.charge/workflows/`:

1. Parse directory name to extract date and workflow name (format: `{YYYY-MM-DD}-{workflow-name}`)
2. Read `manifest.json` to get task count
3. Collect: name, created date, task count

### Step 3: Present Workflow List

Display workflows sorted by date (most recent first):

#### Example

```markdown
## Workflows in {project-name}

| Date | Workflow | Tasks |
|------|----------|-------|
| 26-01-17 | build-rest-api-auth | 5 |
| 26-01-17 | refactor-payment-module | 3 |
| 26-01-16 | add-user-dashboard | 7 |
| 26-01-15 | setup-database | 4 |

Total: {count} workflows

---

**Actions**:
- Inspect: `/charge:inspect <workflow-name>`
- Run: `/charge:run <workflow-name>`
- Delete: `/charge:delete <workflow-name>`

**Sessions**: Use `ls $HOME/.charge/sessions/` to view execution history.
```

### Step 4: Handle Empty Results

If `$HOME/.charge/workflows/` exists but contains no workflows:

```markdown
No workflows found in this project.
Create one with: /charge <your prompt>
```

## Notes

- Workflows are sorted by creation date (from directory name prefix)
- Status is not shown here since execution state is stored separately in sessions
- Use `/charge:inspect` to view workflow details
- Use `ls $HOME/.charge/sessions/` to view execution history
