# Command: /charge:delete

Delete a workflow definition.

## Input

- `id`: The workflow identifier (workflow name)

## Workflow

### Step 1: Locate the Workflow

1. Search for the workflow in `.charge/workflows/` directory
2. If `id` is just a name, search by matching the workflow name suffix (ignoring date prefix)
3. If not found, report error:

   ```markdown
   Workflow not found: {id}
   Run /charge:list to see available workflows.
   ```

### Step 2: Confirm Deletion

Ask for user confirmation before deleting:

```markdown
Delete workflow: {name}
Location: .charge/workflows/{YYYY-MM-DD}-{name}/

This will permanently remove:
- Manifest file
- {count} task instruction files
- {count} schema files

Note: Execution history in .charge/sessions/ will NOT be deleted.
To delete sessions, use: rm -rf .charge/sessions/{session_id}

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

### Step 4: Clean Up Empty Directories

After deleting a workflow:

1. Check if `.charge/workflows/` is now empty
2. If empty, delete the workflows directory
3. Check if `.charge/` has no other content (no workflows or sessions)
4. If empty, delete `.charge/` (or leave it for future use)

## Safety Notes

- Always require explicit confirmation before deletion
- Show exactly what will be deleted
- This action only deletes the workflow definition, not execution history
- To delete sessions/executions, use UNIX commands:
  - `rm -rf .charge/sessions/{session_id}` - delete entire session
  - `rm -rf .charge/sessions/{session_id}/{execution}` - delete specific execution
