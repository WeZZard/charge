# Task: Design API Schema

## Purpose

Create the API schema including endpoints and data models based on requirements.

## Input Contract

**Schema**: `../schemas/task_02_input.json`

You will receive:

- `requirements` (array): Structured requirements from task_01

## Output Contract

**Schema**: `../schemas/task_02_output.json`

You must produce:

- `api_schema` (object): Complete API schema with endpoints and models

## Instructions

1. Review the requirements
2. Design RESTful endpoints for each requirement
3. Define data models based on entities
4. Specify HTTP methods, paths, and response schemas

## Rules

1. Your output MUST be valid JSON matching the output schema exactly
2. Use RESTful conventions (GET, POST, PUT, DELETE)
3. Include request/response schemas for each endpoint

## Example Output

```json
{
  "api_schema": {
    "basePath": "/api/v1",
    "endpoints": [
      {
        "method": "GET",
        "path": "/todos",
        "description": "List all todos",
        "response": { "type": "array", "items": "Todo" }
      },
      {
        "method": "POST",
        "path": "/todos",
        "description": "Create a new todo",
        "request": { "title": "string" },
        "response": "Todo"
      }
    ],
    "models": {
      "Todo": {
        "id": "string",
        "title": "string",
        "completed": "boolean",
        "createdAt": "string"
      }
    }
  }
}
```
