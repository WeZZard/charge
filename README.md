# Charge

**Workflow orchestration for Claude Code** — Decompose complex prompts into schema-bound tasks with automatic context management.

## Why Charge?

Claude Code is powerful, but complex tasks can exhaust the context window. Charge solves this by:

- **Decomposing** prompts into discrete, focused tasks
- **Isolating** each task in its own sub-agent context
- **Validating** data flow through JSON Schema contracts
- **Persisting** results to files instead of memory

The result: reliable execution of multi-step workflows without context explosion.

## Installation

### As a Claude Code Plugin

```bash
# Add the marketplace (one-time setup)
/plugin marketplace add WeZZard/charge

# Install the skill
/plugin install charge
```

### Manual Installation

```bash
# Clone to your plugins directory
git clone https://github.com/WeZZard/charge.git ~/.claude/plugins/charge
```

## Quick Start

```
> /charge Build a REST API with user authentication

Analyzing prompt...

Proposed Workflow: build-rest-api-auth

  1. parse-requirements     Extract API requirements from prompt
  2. design-api-schema      Create OpenAPI specification
  3. implement-endpoints    Generate route handlers
  4. add-authentication     Implement auth middleware
  5. generate-tests         Create test files

Approve this workflow? [Y/n]

> Y

Executing workflow...
[1/5] parse-requirements... done
[2/5] design-api-schema... done
[3/5] implement-endpoints... done
[4/5] add-authentication... done
[5/5] generate-tests... done

Workflow complete!
```

## Commands

| Command | Description |
|---------|-------------|
| `/charge <prompt>` | Build and run a workflow |
| `/charge:build <prompt>` | Build workflow only (review before running) |
| `/charge:run <id>` | Run an existing workflow |
| `/charge:inspect <id>` | View workflow structure and status |
| `/charge:list` | List all workflows in project |
| `/charge:delete <id>` | Delete a workflow |

## How It Works

### 1. Prompt Analysis

Charge analyzes your prompt to identify:
- Primary goals and secondary objectives
- Task boundaries and dependencies
- Repetitive patterns that can be parallelized

### 2. Workflow Generation

Each task gets:
- **Instruction file** — Markdown guidance for the sub-agent
- **Input schema** — JSON Schema defining expected input
- **Output schema** — JSON Schema defining required output

### 3. Isolated Execution

Tasks run in separate sub-agent contexts:
```
Main Agent                    Sub-Agent (Task 1)
    │                              │
    ├─► Pass file paths ──────────►│
    │                              ├─► Read instruction
    │                              ├─► Read input data
    │                              ├─► Execute task
    │                              ├─► Write output file
    │◄── Return status ◄───────────┤
    │
    ├─► Pass file paths ──────────► Sub-Agent (Task 2)
    ...
```

The main agent passes **paths, not content** — sub-agents read their own files. This keeps the orchestrator's context minimal.

### 4. Template Tasks

For repetitive operations, Charge creates template tasks that iterate over collections:

```
> /charge Review all blog posts in the content/ directory

[1/3] discover-posts... done
[2/3] review-post [1/15]... done
[2/3] review-post [2/15]... done
...
[2/3] review-post [15/15]... done
[3/3] aggregate-feedback... done
```

Template tasks can run **sequentially** or **in parallel** depending on whether items are independent.

## Workflow Storage

Charge separates workflow definitions from execution results:

### Workflow Definitions
```
$HOME/.charge/workflows/{YYYY-MM-DD}-{workflow-name}/
├── manifest.json           # Workflow definition
├── instructions/           # Per-task instruction files
│   ├── task_01.md
│   └── task_02.md
└── schemas/                # JSON Schema contracts
    ├── task_01_input.json
    └── ...
```

### Execution Results
```
$HOME/.charge/sessions/{session_id}/{timestamp}-{workflow-name}/
├── state.json              # Execution state
└── results/                # Task outputs
    ├── task_01.json
    └── task_02.json
```

This separation enables:
- **Workflow reuse** — Run the same workflow multiple times
- **Session isolation** — Each Claude Code session has its own result space
- **History tracking** — Review past executions without collision

Use basic UNIX commands (`ls`, `rm -rf`) to manage sessions.

## Use Cases

**Code Generation**
```
/charge Build a GraphQL API for a todo app with TypeScript
```

**Refactoring**
```
/charge Refactor the payment module to use the strategy pattern
```

**Batch Processing**
```
/charge Analyze all Python files in src/ for security vulnerabilities
```

**Documentation**
```
/charge Generate API documentation from the OpenAPI spec
```

**Migration**
```
/charge Migrate all database models to the new ORM syntax
```

## Architecture

```
charge/
├── SKILL.md                    # Skill entry point
├── prompts/
│   ├── commands/               # Command handlers
│   │   ├── default.md          # /charge (build + run)
│   │   ├── build.md            # /charge:build
│   │   ├── run.md              # /charge:run
│   │   ├── inspect.md          # /charge:inspect
│   │   ├── list.md             # /charge:list
│   │   └── delete.md           # /charge:delete
│   ├── core/                   # Core orchestration
│   │   ├── analyze-workflow.md # Prompt decomposition
│   │   ├── generate-schemas.md # Schema generation
│   │   ├── build-flow.md       # Execution planning
│   │   ├── execute-task.md     # Task execution
│   │   └── synthesize.md       # Result aggregation
│   └── utilities/
│       └── inject-intermediate.md
├── templates/                  # File templates
└── examples/                   # Example workflows
```

## Design Principles

1. **Schema-Only Communication** — Tasks exchange data through validated JSON, not shared context
2. **Lazy Loading** — Only the current task's instruction is loaded
3. **File Offloading** — Instructions, schemas, and results live on disk
4. **Plan Mode Approval** — Review and approve before execution
5. **Graceful Retry** — Failed tasks retry with error feedback

## Context Management

Charge automatically manages context size:

| Condition | Action |
|-----------|--------|
| Single output > 4000 tokens | Chunk to file, pass reference |
| Cumulative results > 8000 tokens | Inject summarization task |
| Task count > 10 | Add checkpoint summarization |

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## License

MIT

## Version

- Skill Version: 1.0
- Schema Version: 1.0
