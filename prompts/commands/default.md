# Command: /flowit (default)

Build a workflow from a user prompt AND execute it. This is the default command when `/flowit` is invoked without a subcommand.

## Input

- `prompt`: The user's request describing what they want to accomplish

## Workflow

### MANDATORY: Step 1. Build the Workflow

Follow all the steps in `prompts/commands/build.md`, getting the created wotkflow ID.

### MANDATORY: Step 2. Execute the Workflow

Once the workflow is built and approved, immediately execute it by following `prompts/commands/run.md` with the newly created workflow ID.

## Output

The combined output includes:

1. Workflow creation confirmation
2. Task-by-task execution progress
3. Final synthesized results

## Notes

- This is the most common usage pattern: build + run in one step
- Users who want to inspect or modify the workflow before running should use `/flowit:build` followed by `/flowit:run`
