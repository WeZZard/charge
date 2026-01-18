# Core: Analyze Workflow

Decompose a user prompt into discrete, executable tasks.

## Input

- `prompt`: The user's natural language request
- `context` (optional): Additional context about the project or requirements

## Process

### Step 1: Understand the Intent

Read the user's prompt and identify:

- The primary goal (what they want to achieve)
- Secondary objectives (implicit requirements)
- Constraints (limitations, preferences)
- Domain context (technology, framework, etc.)

### Step 2: Identify Repetitive Patterns

**Before decomposing into tasks**, scan the prompt for iteration signals that indicate repetitive operations.

> Recognize these semantic patterns regardless of the prompt's language.

| Signal Type | Recognition Approach | Indicates |
|-------------|---------------------|-----------|
| Plural nouns | Nouns referring to collections, groups, or multiple items | Collection iteration |
| Iterator keywords | Words indicating per-item processing or repetition | Explicit iteration |
| Quantifiers | Words expressing totality, universality, or selection | Collection scope |
| Collection references | Phrases describing sets, lists, or containers of items | Data source |
| Enumerated items | Explicit listing of specific items to process | Static item list |

For each identified pattern, determine:

1. **Collection source**: What items are being iterated over?
2. **Operation**: What action applies to each item?
3. **Cardinality type**:
   - **Static**: Items known at design time (e.g., "update files A, B, and C")
   - **Dynamic**: Items discovered at runtime (e.g., "process all .json files in the directory")

**Output**: A list of repetitive patterns to inform task decomposition:

```yaml
repetitive_patterns:
  - collection: {what is being iterated}
    operation: {action applied to each item}
    cardinality: static|dynamic
    source: {where items come from - prompt literal or task output}
```

### Step 3: Identify Task Boundaries

Break down the goal into discrete tasks. A good task:

- Has a single, clear purpose
- Can be described in one sentence
- Has identifiable inputs and outputs
- Is neither too granular nor too broad

Guidelines for task granularity:

- Each task should represent a logical step in the workflow
- Tasks should be independent enough to have their own instruction file
- Avoid tasks that are just "validate" or "check" - those are part of execution
- Don't over-decompose: 3-10 tasks is typical for most workflows

**Handling repetitive patterns** (from Step 2):

- For each repetitive pattern, create a **template task** rather than duplicating tasks
- Template tasks are marked with `is_template: true` and execute once per item in the collection
- If the collection is dynamic (unknown at design time), create a **discovery task** first to produce the item list
- Consider whether repetitive tasks can run in parallel or must be sequential

Task structure for repetitive operations:

```text
[discovery task] → [template task (iterates)] → [aggregation task]
```

### Step 4: Define Task Properties

For each task, specify:

```yaml
- id: task_01
  name: {kebab-case-name}
  description: {One sentence describing what this task does}
  complexity: light|moderate|heavy
  output_size: minimal|standard|extensive|collection
  is_template: true|false
  iteration:  # Only for template tasks
    over: {collection reference - task output path or prompt literal}
    item_alias: {name for current item, e.g., "file", "table"}
    strategy: sequential|parallel
  input:
    - field_name: {description}
    - field_name: {description}
  output:
    - field_name: {description}
    - field_name: {description}
  depends_on: []  # List of task IDs that must complete first
```

For non-template tasks, omit `is_template` and `iteration` fields.

#### Iteration Strategy Selection

**Choose `parallel` when:**
- Items are independent (no shared state)
- Order doesn't matter for correctness
- Operations are read-only or analysis-focused

**Choose `sequential` when:**
- Items depend on previous item's result
- Order matters (numbered steps, migrations)
- Write operations that could conflict

**Default to `parallel`** unless a sequential condition applies. Parallel execution significantly improves workflow performance.

#### Complexity Classification

Classify each task's processing effort:

| Level | Indicators |
|-------|------------|
| `light` | Single-step operation, parsing, extraction, simple formatting |
| `moderate` | Multi-step logic, analysis, design decisions, structured generation |
| `heavy` | Extensive code generation, multi-file output, complex reasoning chains |

#### Output Size Classification

Estimate the relative size of task output:

| Level | Indicators |
|-------|------------|
| `minimal` | 1-3 short fields, simple values |
| `standard` | Typical structured object, moderate detail |
| `extensive` | Long text blocks, code files, detailed analysis |
| `collection` | Array outputs, lists of items, aggregated data |

> Template tasks that iterate over collections typically produce `collection` output size.

### Step 5: Map Dependencies

Determine the data flow between tasks:

- Which task outputs feed into which task inputs?
- Are there tasks that can run in parallel (no shared dependencies)?
- What is the critical path through the workflow?

### Step 6: Validate Decomposition

Check that:

- All tasks are necessary (no redundant tasks)
- All outputs have a consumer (either another task or final output)
- The dependency graph has no cycles
- The first task can start with just the user prompt

**Additional validation for template tasks:**

- Template tasks must have `iteration.over` referencing a valid collection
- The `iteration.over` source must be either a prompt literal (static) or a task output (dynamic)
- Dynamic collections require a preceding discovery task that outputs the item list
- Template tasks should not be the final task - an aggregation task should follow to combine results

## Output Format

Return a structured task list:

```json
{
  "workflow_name": "[string]",
  "repetitive_patterns": [
    {
      "collection": "[string]",
      "operation": "[string]",
      "cardinality": "static|dynamic",
      "source": "[string]"
    }
  ],
  "tasks": [
    {
      "id": "[string]",
      "name": "[string]",
      "description": "[string]",
      "complexity": "light|moderate|heavy",
      "output_size": "minimal|standard|extensive|collection",
      "is_template": true|false,
      "iteration": {
        "over": "[string]",
        "item_alias": "[string]",
        "strategy": "sequential|parallel"
      },
      "input": [
        {"field": "[string]", "type": "[string]", "description": "[string]"}
      ],
      "output": [
        {"field": "[string]", "type": "[string]", "description": "[string]"}
      ],
      "depends_on": ["[string]"]
    }
  ],
  "execution_order": ["[string]"],
  "parallel_groups": [["[string]"]]
}
```

Notes:

- `is_template` and `iteration` are only present for template tasks
- `repetitive_patterns` may be empty if no repetitive patterns were identified
- `complexity` and `output_size` are required for all tasks

## Examples

### Example 1: "Build a REST API with user authentication"

Tasks:

1. `parse-requirements` - Extract API requirements from prompt
   - complexity: `light`, output_size: `standard`
2. `design-api-schema` - Create OpenAPI specification
   - complexity: `moderate`, output_size: `extensive`
3. `implement-endpoints` - Generate route handlers
   - complexity: `heavy`, output_size: `extensive`
4. `add-authentication` - Implement auth middleware
   - complexity: `moderate`, output_size: `extensive`
5. `generate-tests` - Create test files
   - complexity: `moderate`, output_size: `extensive`

### Example 2: "Refactor the payment module for better error handling"

Tasks:

1. `analyze-current-code` - Understand existing implementation
   - complexity: `moderate`, output_size: `standard`
2. `identify-error-cases` - List all error scenarios
   - complexity: `light`, output_size: `collection`
3. `design-error-strategy` - Define error handling approach
   - complexity: `moderate`, output_size: `standard`
4. `implement-changes` - Apply the refactoring
   - complexity: `heavy`, output_size: `extensive`
5. `verify-behavior` - Ensure functionality is preserved
   - complexity: `light`, output_size: `minimal`

### Example 3: "Migrate all database tables to the new schema format"

**Identified repetitive patterns:**

```yaml
repetitive_patterns:
  - collection: database tables
    operation: migrate to new schema format
    cardinality: dynamic
    source: task_01 output (tables[])
```

**Tasks:**

1. `discover-tables` - List all tables in the database
   - complexity: `light`, output_size: `collection`
   - Output: `tables[]` (array of table names)
2. `analyze-table` - Analyze a single table's current schema (**template**)
   - complexity: `moderate`, output_size: `standard`
   - `is_template: true`
   - `iteration.over: $.tasks[0].output.tables`
   - `iteration.item_alias: table`
   - `iteration.strategy: parallel`
3. `generate-migration` - Generate migration script for a single table (**template**)
   - complexity: `heavy`, output_size: `extensive`
   - `is_template: true`
   - `iteration.over: $.tasks[0].output.tables`
   - `iteration.item_alias: table`
   - `iteration.strategy: sequential`
4. `apply-migrations` - Execute all migration scripts and report results
   - complexity: `moderate`, output_size: `standard`
