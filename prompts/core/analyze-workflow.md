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

### Step 2: Identify Task Boundaries

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

### Step 3: Define Task Properties

For each task, specify:

```yaml
- id: task_01
  name: {kebab-case-name}
  description: {One sentence describing what this task does}
  input:
    - field_name: {description}
    - field_name: {description}
  output:
    - field_name: {description}
    - field_name: {description}
  depends_on: []  # List of task IDs that must complete first
```

### Step 4: Map Dependencies

Determine the data flow between tasks:
- Which task outputs feed into which task inputs?
- Are there tasks that can run in parallel (no shared dependencies)?
- What is the critical path through the workflow?

### Step 5: Validate Decomposition

Check that:
- All tasks are necessary (no redundant tasks)
- All outputs have a consumer (either another task or final output)
- The dependency graph has no cycles
- The first task can start with just the user prompt

## Output Format

Return a structured task list:

```json
{
  "workflow_name": "{derived-from-prompt}",
  "tasks": [
    {
      "id": "task_01",
      "name": "parse-requirements",
      "description": "Extract structured requirements from the user prompt",
      "input": [
        {"field": "raw_prompt", "type": "string", "description": "The original user request"}
      ],
      "output": [
        {"field": "requirements", "type": "array", "description": "List of structured requirements"},
        {"field": "constraints", "type": "array", "description": "Identified constraints"}
      ],
      "depends_on": []
    },
    {
      "id": "task_02",
      "name": "design-solution",
      "description": "Create a high-level design based on requirements",
      "input": [
        {"field": "requirements", "type": "array", "from": "task_01"}
      ],
      "output": [
        {"field": "design", "type": "object", "description": "Solution design"}
      ],
      "depends_on": ["task_01"]
    }
  ],
  "execution_order": ["task_01", "task_02"],
  "parallel_groups": []
}
```

## Examples

### Example 1: "Build a REST API with user authentication"

Tasks:
1. `parse-requirements` - Extract API requirements from prompt
2. `design-api-schema` - Create OpenAPI specification
3. `implement-endpoints` - Generate route handlers
4. `add-authentication` - Implement auth middleware
5. `generate-tests` - Create test files

### Example 2: "Refactor the payment module for better error handling"

Tasks:
1. `analyze-current-code` - Understand existing implementation
2. `identify-error-cases` - List all error scenarios
3. `design-error-strategy` - Define error handling approach
4. `implement-changes` - Apply the refactoring
5. `verify-behavior` - Ensure functionality is preserved
