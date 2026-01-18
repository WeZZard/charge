# Task: {{name}}

## Purpose

{{description}}

## Input Contract

**Schema**: `../schemas/{{id}}_input.json`

You will receive:

{{#each input_fields}}
- `{{field}}` ({{type}}): {{description}}
{{/each}}

## Reading Input

Your input source will be provided in the task execution prompt as a file path.
**You must read the file yourself** using the Read tool before processing.

For template tasks iterating over items:
- The execution prompt specifies which file to read
- Read the file content, then process according to the instructions below

For regular tasks with dependencies:
- Read previous task results from the `results/` directory
- The execution prompt lists which result files to read

## Output Contract

**Schema**: `../schemas/{{id}}_output.json`

You must produce:

{{#each output_fields}}
- `{{field}}` ({{type}}): {{description}}
{{/each}}

## Instructions

{{task_instructions}}

## Rules

1. **Read input files first** - use the Read tool to load source content
2. Your output MUST be valid JSON matching the output schema exactly
3. Do NOT include any text outside the JSON structure
4. Do NOT wrap the JSON in markdown code blocks
5. All required fields must be present
6. Field types must match the schema specifications

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
