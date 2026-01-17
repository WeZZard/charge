# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FlowIt is a Claude Code skill for workflow orchestration. It decomposes complex user prompts into discrete, schema-bound tasks with file-based instruction offloading to prevent context window explosion.

**This is a pure prompt-based project** - there is no traditional build, test, or lint system. All functionality is implemented through markdown prompt files and JSON schema definitions.

## Usage

FlowIt is invoked as a Claude Code skill:

| Command | Description |
|---------|-------------|
| `/flowit <prompt>` | Build and run a workflow (default) |
| `/flowit:build <prompt>` | Build workflow only, with approval flow |
| `/flowit:run <id>` | Run an existing workflow |
| `/flowit:inspect <id>` | View workflow structure and status |
| `/flowit:list` | List all workflows in project |
| `/flowit:delete <id>` | Delete a workflow |

Workflows are stored at: `{project}/.flowit/{YY-MM-DD}/{workflow-name}/`

## Architecture

### Command Flow

```
skill.md → prompts/commands/*.md → prompts/core/*.md → workflow execution
```

- `skill.md` - Entry point and command router
- `prompts/commands/` - User-facing command handlers
- `prompts/core/` - Core orchestration logic (analyze, build, execute, synthesize)
- `prompts/utilities/` - Helper prompts (context injection)
- `templates/` - JSON and markdown templates for generated workflows

### Core Design Principles

1. **Schema-Only Communication**: Tasks communicate exclusively via JSON Schema-validated data, not shared context
2. **Lazy Loading**: Only load current task's instruction into context
3. **File Offloading**: Instructions, schemas, and results stored in files to prevent context explosion
4. **Plan Mode Approval**: User approves workflow structure before execution begins

### Workflow Lifecycle

1. **Analyze** (`analyze-workflow.md`) - Decompose prompt into tasks with dependencies
2. **Generate Schemas** (`generate-schemas.md`) - Create JSON Schema contracts for each task I/O
3. **Build Flow** (`build-flow.md`) - Determine execution order (sequential or DAG)
4. **Execute** (`execute-task.md`) - Run tasks one at a time with minimal context
5. **Synthesize** (`synthesize.md`) - Combine outputs for final presentation

### Data Structures

**Manifest** (`manifest.json`): Workflow definition with tasks, flow order, and data mappings between tasks

**State** (`state.json`): Runtime execution state tracking completed/failed tasks and results

**Task Definition**: Each task has an instruction file (`instructions/{task_id}.md`) and input/output schemas (`schemas/{task_id}_input.json`, `schemas/{task_id}_output.json`)

### Context Management

- Single task output > 4000 tokens: Chunk to file, pass reference
- Cumulative results > 8000 tokens: Inject summarization task
- Task count > 10: Add checkpoint summarization
