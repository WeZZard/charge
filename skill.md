# Charge - Workflow Orchestration Skill

A pure natural language skill that decomposes user prompts into schema-bound tasks with file-based instruction offloading to prevent context window explosion.

## Commands

| Command | Description |
| ------- | ----------- |
| `/charge <prompt>` | Build workflow from prompt AND run it (default) |
| `/charge:build <prompt>` | Build workflow only (don't run) |
| `/charge:run <id>` | Run an existing workflow |
| `/charge:inspect <id>` | View workflow structure and status |
| `/charge:list` | List workflows in current project |
| `/charge:delete <id>` | Delete a workflow |

## Command Router

When this skill is invoked, route to the appropriate handler based on the command:

### Route: `/charge <prompt>` (no subcommand)

Load and follow: `prompts/commands/default.md`

This is the default flow that builds a workflow AND runs it.

### Route: `/charge:build <prompt>`

Load and follow: `prompts/commands/build.md`

Builds a workflow with plan-mode approval but does not execute.

### Route: `/charge:run <id>`

Load and follow: `prompts/commands/run.md`

Executes an existing workflow by ID.

### Route: `/charge:inspect <id>`

Load and follow: `prompts/commands/inspect.md`

Shows workflow structure, tasks, schemas, and execution status.

### Route: `/charge:list`

Load and follow: `prompts/commands/list.md`

Lists all workflows in the current project's `.charge/` directory.

### Route: `/charge:delete <id>`

Load and follow: `prompts/commands/delete.md`

Deletes a workflow and its artifacts.

## Storage Convention

All workflow artifacts are stored at:

```ascii
{project}/.charge/{YY-MM-DD}/{workflow-name}/
├── manifest.json          # Workflow definition
├── state.json             # Execution state
├── instructions/          # Per-task instruction files
├── schemas/               # JSON Schema files for task I/O
└── results/               # Task output files
```

## Core Principles

1. **Schema-Only Communication**: Tasks communicate exclusively via JSON Schema-validated data
2. **Lazy Loading**: Only load current task's instruction into context
3. **File Offloading**: Instructions, schemas, and results stored in files
4. **Plan Mode Approval**: User approves workflow structure before execution
5. **Retry on Failure**: Schema validation failures trigger retry with feedback

## Version

- Skill Version: 1.0
- Schema Version: 1.0
