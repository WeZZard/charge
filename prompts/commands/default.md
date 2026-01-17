# Command: /flowit (default)

Build a workflow from a user prompt AND execute it. This is the default command when `/flowit` is invoked without a subcommand.

## Input

- `prompt`: The user's request describing what they want to accomplish

## Workflow

### Step 1: Build the Workflow

Follow all the steps in `prompts/commands/build.md`:
1. Analyze the prompt
2. Generate schemas
3. Build execution flow
4. Present plan for approval
5. Handle user response (approval or refinement loop)
6. Persist workflow to `.flowit/{YY-MM-DD}/{workflow-name}/`

### Step 2: Execute the Workflow

Once the workflow is built and approved, immediately execute it by following `prompts/commands/run.md` with the newly created workflow ID.

## Output

The combined output includes:
1. Workflow creation confirmation
2. Task-by-task execution progress
3. Final synthesized results

## Example Flow

```
> /flowit Build a REST API with user authentication

FlowIt analyzing prompt...

## Proposed Workflow: build-rest-api-auth

Tasks:
  1. parse-requirements
     Input: raw prompt
     Output: structured requirements

  2. design-api-schema
     Input: requirements
     Output: OpenAPI spec

  3. implement-endpoints
     Input: API schema
     Output: route handlers

  4. add-authentication
     Input: endpoints
     Output: auth middleware

  5. generate-tests
     Input: all outputs
     Output: test files

Execution: Sequential (1 → 2 → 3 → 4 → 5)

---
Do you approve this workflow?

> Approve

Workflow created: build-rest-api-auth
Location: .flowit/26-01-17/build-rest-api-auth/

Executing workflow...

[1/5] parse-requirements... done
[2/5] design-api-schema... done
[3/5] implement-endpoints... done
[4/5] add-authentication... done
[5/5] generate-tests... done

Workflow complete.
Results saved to: .flowit/26-01-17/build-rest-api-auth/results/

{Final synthesized output presented to user}
```

## Notes

- This is the most common usage pattern: build + run in one step
- Users who want to inspect or modify the workflow before running should use `/flowit:build` followed by `/flowit:run`
