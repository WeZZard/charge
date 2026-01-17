# Task: Parse Requirements

## Purpose

Extract structured requirements from the user's prompt for building a REST API.

## Input Contract

**Schema**: `../schemas/task_01_input.json`

You will receive:

- `raw_prompt` (string): The original user request describing what API to build

## Output Contract

**Schema**: `../schemas/task_01_output.json`

You must produce:

- `requirements` (array): List of structured requirements with id, description, and priority
- `entities` (array): Data entities/models needed for the API
- `constraints` (array): Any constraints or limitations identified

## Instructions

1. Read the user's prompt carefully
2. Identify all explicit requirements (what they asked for)
3. Infer implicit requirements (common needs for this type of API)
4. Extract data entities that will need to be modeled
5. Note any constraints mentioned

## Rules

1. Your output MUST be valid JSON matching the output schema exactly
2. Do NOT include any text outside the JSON structure
3. Do NOT wrap the JSON in markdown code blocks
4. All required fields must be present
5. Each requirement must have a unique ID (REQ-001, REQ-002, etc.)

## Example

### Input

```json
{
  "raw_prompt": "Build a simple REST API with CRUD operations for a todo list"
}
```

### Expected Output

```json
{
  "requirements": [
    {
      "id": "REQ-001",
      "description": "Create new todo items",
      "priority": "must"
    },
    {
      "id": "REQ-002",
      "description": "Read/list todo items",
      "priority": "must"
    },
    {
      "id": "REQ-003",
      "description": "Update existing todo items",
      "priority": "must"
    },
    {
      "id": "REQ-004",
      "description": "Delete todo items",
      "priority": "must"
    }
  ],
  "entities": [
    {
      "name": "Todo",
      "fields": ["id", "title", "completed", "createdAt"]
    }
  ],
  "constraints": []
}
```
