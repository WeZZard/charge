# Command: /charge:delete

Delete a workflow and all its artifacts.

## Input

- `id`: The workflow identifier (workflow name or full path)

## Workflow

### Step 1: Locate the Workflow

1. Search for the workflow in `.charge/` directory
2. If not found, report error:

   ```markdown
   Workflow not found: {id}
   Run /charge:list to see available workflows.
   ```

### Step 2: Confirm Deletion

Ask for user confirmation before deleting:

```markdown
Delete workflow: {name}
Location: {path}

This will permanently remove:
- Manifest and state files
- {count} task instruction files
- {count} schema files
- {count} result files

Are you sure you want to delete this workflow? (yes/no)
```

### Step 3: Handle User Response

**If user confirms (yes/y):**

1. Delete the entire workflow directory
2. Report success:

   ```markdown
   Workflow deleted: {name}
   ```

**If user declines (no/n):**

1. Report cancellation:

   ```markdown
   Deletion cancelled.
   ```

### Step 4: Clean Up Empty Date Directories

After deleting a workflow:

1. Check if the parent date directory is now empty
2. If empty, delete the date directory too
3. Check if `.charge/` is now empty
4. If empty, optionally delete `.charge/` (or leave it for future workflows)

## Safety Notes

- Always require explicit confirmation before deletion
- Show exactly what will be deleted
- This action is irreversible - deleted workflows cannot be recovered
- Do NOT delete if the workflow is currently running
