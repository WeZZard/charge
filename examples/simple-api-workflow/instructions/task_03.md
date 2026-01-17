# Task: Generate Code

## Purpose

Generate the API implementation code based on the API schema.

## Input Contract

**Schema**: `../schemas/task_03_input.json`

You will receive:

- `api_schema` (object): The API schema from task_02

## Output Contract

**Schema**: `../schemas/task_03_output.json`

You must produce:

- `files` (array): List of generated code files with path and content

## Instructions

1. Review the API schema
2. Generate implementation code for each endpoint
3. Include data models
4. Add basic error handling
5. Use a simple in-memory store for data persistence

## Rules

1. Your output MUST be valid JSON matching the output schema exactly
2. Generate complete, runnable code
3. Include all necessary imports
4. Use consistent code style

## Example Output

```json
{
  "files": [
    {
      "path": "app.js",
      "content": "const express = require('express');\nconst app = express();\n..."
    },
    {
      "path": "routes/todos.js",
      "content": "const router = require('express').Router();\n..."
    }
  ]
}
```
