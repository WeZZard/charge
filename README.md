# Charge

A Claude Code skill for workflow orchestration with schema-bound task execution and file-based instruction offloading.

## Overview

Charge decomposes complex user requests into discrete tasks, each with explicit input/output JSON Schema contracts. Tasks communicate only through these schemas, and instructions are stored in files to prevent context window explosion.

## Installation

Clone this repository and add it to your Claude Code skills:

```bash
git clone https://github.com/your-org/charge.git
```

Add to your Claude Code configuration as a skill.

## Commands

| Command | Description |
|---------|-------------|
| `/charge <prompt>` | Build and run a workflow (default) |
| `/charge:build <prompt>` | Build workflow only, with approval flow |
| `/charge:run <id>` | Run an existing workflow |
| `/charge:inspect <id>` | View workflow structure and status |
| `/charge:list` | List all workflows in project |
| `/charge:delete <id>` | Delete a workflow |

## Quick Start

```
> /charge Build a REST API with user authentication

Charge analyzing prompt...

## Proposed Workflow: build-rest-api-auth

Tasks:
  1. parse-requirements
  2. design-api-schema
  3. implement-endpoints
  4. add-authentication
  5. generate-tests

Do you approve this workflow?

> Approve

Workflow created and executing...
[1/5] parse-requirements... done
[2/5] design-api-schema... done
[3/5] implement-endpoints... done
[4/5] add-authentication... done
[5/5] generate-tests... done

Workflow complete!
```

## How It Works

### 1. Workflow Analysis

When you provide a prompt, Charge:
- Decomposes it into discrete tasks
- Identifies dependencies between tasks
- Generates JSON Schema contracts for each task's I/O

### 2. Plan Mode Approval

Before execution, you review and approve the workflow:
- See all tasks and their purpose
- Provide feedback to refine the workflow
- Approve when satisfied

### 3. Task Execution

Each task:
- Loads only its instruction file (minimal context)
- Receives input validated against its input schema
- Produces output validated against its output schema
- Results are saved to files, not kept in context

### 4. Context Management

Charge prevents context explosion by:
- Storing task instructions in separate files
- Persisting results to disk
- Injecting summarization tasks when outputs grow large

## Workflow Storage

Workflows are stored at:

```
{project}/.charge/{YY-MM-DD}/{workflow-name}/
├── manifest.json      # Workflow definition
├── state.json         # Execution state
├── instructions/      # Task instruction files
├── schemas/           # JSON Schema files
└── results/           # Task output files
```

## Project Structure

```
charge/
├── skill.md                    # Main skill entry point
├── prompts/
│   ├── commands/               # Command handlers
│   │   ├── default.md
│   │   ├── build.md
│   │   ├── run.md
│   │   ├── inspect.md
│   │   ├── list.md
│   │   └── delete.md
│   ├── core/                   # Core logic
│   │   ├── analyze-workflow.md
│   │   ├── generate-schemas.md
│   │   ├── build-flow.md
│   │   ├── execute-task.md
│   │   └── synthesize.md
│   └── utilities/
│       └── inject-intermediate.md
├── templates/                  # File templates
│   ├── task-instruction.md
│   ├── manifest.json
│   └── state.json
└── examples/                   # Example workflows
    └── simple-api-workflow/
```

## Key Concepts

### Schema-Only Communication

Tasks don't share context directly. Instead:
- Each task has explicit input and output schemas
- Data flows through validated JSON structures
- This ensures predictable, debuggable execution

### File-Based Offloading

To prevent context overflow:
- Task instructions live in markdown files
- Results are persisted to JSON files
- Only the current task's context is loaded

### Approval Flow

Before execution, you see:
- All proposed tasks
- Their inputs and outputs
- The execution order

You can:
- Approve to proceed
- Provide feedback to refine

## Version

- Version: 1.0
- Schema Version: 1.0

## License

MIT
