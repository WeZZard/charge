# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Charge is a Claude Code skill for workflow orchestration. It decomposes complex user prompts into discrete, schema-bound tasks with file-based instruction offloading to prevent context window explosion.

**This is a pure prompt-based project** - there is no traditional build, test, or lint system. All functionality is implemented through markdown prompt files and JSON schema definitions.

## Usage

Charge is invoked as a Claude Code skill:

| Command | Description |
|---------|-------------|
| `/charge <prompt>` | Build and run a workflow (default) |
| `/charge:build <prompt>` | Build workflow only, with approval flow |
| `/charge:run <id>` | Run an existing workflow |
| `/charge:inspect <id>` | View workflow structure and status |
| `/charge:list` | List all workflows in project |
| `/charge:delete <id>` | Delete a workflow |

Workflows are stored at: `{project}/.charge/{YYYY-MM-DD}/{workflow-name}/`

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

## Prompt Authoring Rules

This section defines conventions for placeholders, variables, and examples in prompt files.

### Placeholder Syntax

Charge uses distinct placeholder syntaxes for different purposes:

| Syntax | Purpose | Example |
| ------ | ------- | ------- |
| `{placeholder}` | Runtime substitution (filled during execution) | `{workflow-name}`, `{task-id}` |
| `{{placeholder}}` | Handlebars template rendering | `{{name}}`, `{{#each items}}` |
| `$.path` | JSONPath for data mapping between tasks | `$.requirements` |
| `<placeholder>` | User input in command documentation | `<prompt>`, `<id>` |

### JSON Type Placeholders

When documenting JSON data formats, use these type placeholders instead of mock data:

| Syntax | Purpose | Example |
| ------ | ------- | ------- |
| `"[string]"` | String value placeholder | `"name": "[string]"` |
| `[number]` | Number value placeholder | `"count": [number]` |
| `true\|false` | Boolean value placeholder | `"enabled": true\|false` |
| `"value1\|value2\|..."` | Enum/union string values | `"status": "pending\|running\|completed"` |

Example JSON format specification (correct - no mock data):

```json
{
  "id": "[string]",
  "name": "[string]",
  "task_count": [number],
  "is_parallel": true|false,
  "status": "pending|running|completed|failed"
}
```

### Placeholder Naming Conventions

1. **Self-explanatory names**: Descriptive enough to understand without context
   - Good: `{workflow-name}`, `{task-count}`, `{ISO-timestamp}`
   - Bad: `{name}`, `{count}`, `{ts}`

2. **Context-specific prefixes**: When generic concepts appear in multiple contexts
   - `{task-count}` not `{count}` when counting tasks
   - `{schema-file-count}` not `{count}` when counting schema files

3. **Case conventions**:
   - Runtime placeholders: `kebab-case` (e.g., `{workflow-name}`)
   - Handlebars variables: `snake_case` (e.g., `{{task_id}}`)
   - JSONPath: `snake_case` matching JSON field names

4. **Inline documentation**: Add parenthetical when clarification needed

   ```markdown
   Created: {created-timestamp} (ISO 8601 format)
   ```

### Variable Documentation

Every variable must be explained via one of:

1. **Inline** - parenthetical or comment
2. **Preceding section** - in `## Input` or table header
3. **Self-explanatory naming** - clear enough to need no explanation

### Mock Data Rules

**PROHIBITED in**:

- JSON Schema definitions (`templates/*.json`, generated `schemas/*.json`)
- Data exchange format specifications
- Output format templates (use placeholders instead)

**ALLOWED in**:

- `## Example` or `## Examples` sections (clearly marked)
- Files in `examples/` directory
- Tutorial/README content

### Handlebars Template Variables

The template `templates/task-instruction.md` uses these variables:

| Variable | Type | Description |
| -------- | ---- | ----------- |
| `{{name}}` | string | Task name (kebab-case) |
| `{{description}}` | string | One-sentence task description |
| `{{id}}` | string | Task ID (format: `task_XX`) |
| `{{input_fields}}` | array | Input field definitions |
| `{{output_fields}}` | array | Output field definitions |
| `{{task_instructions}}` | string | Task-specific guidance |
| `{{additional_rules}}` | array | Optional extra rules |
| `{{example_input}}` | string | Example input JSON |
| `{{example_output}}` | string | Example output JSON |

Field objects contain: `{{field}}`, `{{type}}`, `{{description}}`
