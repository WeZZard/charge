# Task: {{name}}

## Purpose

{{description}}

## Input Contract

**Schema**: `../schemas/{{id}}_input.json`

You will receive:

{{#each input_fields}}
- `{{field}}` ({{type}}): {{description}}
{{/each}}

## Output Contract

**Schema**: `../schemas/{{id}}_output.json`

You must produce:

{{#each output_fields}}
- `{{field}}` ({{type}}): {{description}}
{{/each}}

## Instructions

{{task_instructions}}

## Rules

1. Your output MUST be valid JSON matching the output schema exactly
2. Do NOT include any text outside the JSON structure
3. Do NOT wrap the JSON in markdown code blocks
4. All required fields must be present
5. Field types must match the schema specifications

{{#if additional_rules}}
## Additional Rules

{{#each additional_rules}}
- {{this}}
{{/each}}
{{/if}}

## Example

### Input

```json
{{example_input}}
```

### Expected Output

```json
{{example_output}}
```
