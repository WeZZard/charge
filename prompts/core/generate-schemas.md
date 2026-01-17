# Core: Generate Schemas

Create JSON Schema definitions for each task's input and output.

## Input

- `tasks`: The task list from analyze-workflow (with input/output field definitions)

## Process

### Step 1: Schema Design Principles

Follow these principles when generating schemas:

1. **Explicit over implicit**: Every field should be explicitly defined
2. **Strict validation**: Use `"additionalProperties": false` to catch unexpected fields
3. **Required fields**: Mark essential fields as required
4. **Descriptive**: Include descriptions for complex fields
5. **Type safety**: Use appropriate types (string, number, boolean, array, object)

### Step 2: Generate Input Schemas

For each task, create an input schema based on its declared inputs:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "{task_id}_input",
  "type": "object",
  "properties": {
    "{field_name}": {
      "type": "{type}",
      "description": "{description}"
    }
  },
  "required": ["{required_fields}"],
  "additionalProperties": false
}
```

### Step 3: Generate Output Schemas

For each task, create an output schema based on its declared outputs:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "{task_id}_output",
  "type": "object",
  "properties": {
    "{field_name}": {
      "type": "{type}",
      "description": "{description}"
    }
  },
  "required": ["{required_fields}"],
  "additionalProperties": false
}
```

### Step 4: Define Field Mappings

For each task input that comes from a previous task's output, define the mapping:

```json
{
  "task_id": "task_02",
  "input_field": "requirements",
  "source_task": "task_01",
  "source_path": "$.requirements"
}
```

### Step 5: Validate Schema Consistency

Ensure:
- Output types match input types where they're connected
- All referenced fields exist in their source schemas
- No orphaned outputs (outputs that nothing consumes, except final task)

## Output Format

Return schemas organized by task:

```json
{
  "schemas": {
    "{task-id}": {
      "input": {"$comment": "JSON Schema object"},
      "output": {"$comment": "JSON Schema object"}
    }
  },
  "mappings": [
    {
      "target_task": "[string]",
      "target_field": "[string]",
      "source_task": "[string]",
      "source_path": "[string]"
    }
  ]
}
```

## Common Schema Patterns

### String with constraints
```json
{
  "type": "string",
  "minLength": 1,
  "maxLength": 1000
}
```

### Array of objects
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "id": { "type": "string" },
      "value": { "type": "string" }
    },
    "required": ["id", "value"]
  }
}
```

### Enum values
```json
{
  "type": "string",
  "enum": ["option1", "option2", "option3"]
}
```

### Optional field with default
```json
{
  "type": "string",
  "default": "default_value"
}
```

## Notes

- Keep schemas focused - don't over-specify
- Use `$ref` for repeated structures if the same shape appears multiple times
- Schemas should be strict enough to validate but flexible enough to not break on minor variations
