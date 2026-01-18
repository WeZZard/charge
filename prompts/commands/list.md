# Command: /charge:list

List all workflows in the current project.

## Input

None required.

## Workflow

### Step 1: Find Workflows

1. Check if `.charge/` directory exists in the current project
2. If not, report:

   ```markdown
   No workflows found in this project.
   Create one with: /charge <your prompt>
   ```

### Step 2: Scan Workflow Directories

For each date directory in `.charge/`:

1. List all workflow subdirectories
2. For each workflow, read `manifest.json` and `state.json`
3. Collect: name, created date, status, task count

### Step 3: Present Workflow List

Display workflows sorted by date (most recent first):

#### Example

```markdown
## Workflows in {project-name}

| Date | Workflow | Tasks | Status |
|------|----------|-------|--------|
| 26-01-17 | build-rest-api-auth | 5 | completed |
| 26-01-17 | refactor-payment-module | 3 | pending |
| 26-01-16 | add-user-dashboard | 7 | failed |
| 26-01-15 | setup-database | 4 | completed |

Total: {count} workflows

---

**Actions**:
- Inspect: `/charge:inspect <workflow-name>`
- Run: `/charge:run <workflow-name>`
- Delete: `/charge:delete <workflow-name>`
```

### Step 4: Handle Empty Results

If `.charge/` exists but contains no workflows:

```markdown
No workflows found in this project.
Create one with: /charge <your prompt>
```

## Notes

- Group by date for easy temporal navigation
- Show status with visual indicators if possible
- Keep the list concise - users can use `/charge:inspect` for details
