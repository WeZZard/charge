# Command: /charge:build

Build a workflow from a user prompt without executing it. Uses a plan-mode approval flow.

## Input

- `prompt`: The user's request describing what they want to accomplish

## Workflow

### Phase 1: Preflight

**YOU CANNOT BUILD A FLOW IN PLAN MODE.**

Ensure the Claude Code is not in plan mode.
When Claude Code is in plan mode, you MUST reject user's request and prompt the following notice:

> Plan mode detected.
>
> You cannot build a flow in plan mode.

Call **EnterPlanMode**.

### Phase 1: Analyze the Prompt

Read and apply the instructions in `prompts/core/analyze-workflow.md` to decompose the user's prompt into discrete tasks.

For each task, identify:

- Task name (kebab-case)
- Brief description
- Input requirements (what data it needs)
- Output specification (what it produces)
- Dependencies (which tasks must complete first)

### Phase 2: Generate Schemas

Read and apply `prompts/core/generate-schemas.md` to create JSON Schema definitions for each task's input and output.

### Phase 3: Build Execution Flow

Read and apply `prompts/core/build-flow.md` to determine:

- Execution order (topological sort based on dependencies)
- Parallel execution groups (tasks that can run concurrently)

### Phase 4: Present Plan for Approval

Present the proposed workflow to the user with the **ExitPlanMode** tool in this format:

```markdown
## Proposed Workflow: {workflow-name}

Tasks:
  1. {task-name}
     Input: {brief description of input}
     Output: {brief description of output}

  2. {task-name}
     Input: {brief description of input}
     Output: {brief description of output}

  ... (continue for all tasks)

Execution: {Sequential | Parallel groups: [...]}

---
Do you approve this workflow? You can:
- Approve to proceed
- Provide feedback to refine the workflow
```

### Phase 5: Handle User Response

**If user approves:**

1. Generate a workflow ID: `{workflow-name}` (derived from prompt, kebab-case)
2. Create the workflow directory at `.charge/{YYYY-MM-DD}/{workflow-name}/`
3. Write all artifacts:
   - `manifest.json` using `templates/manifest.json` structure
   - `state.json` using `templates/state.json` structure
   - Task instruction files in `instructions/` using `templates/task-instruction.md`
   - Schema files in `schemas/`
4. Report success:

   ```ascii
   Workflow created: {workflow-name}
   Location: .charge/{YYYY-MM-DD}/{workflow-name}/

   Run with: /charge:run {workflow-name}
   ```

5. You MUST NOT execute the plan when it was approved.

**If user provides feedback:**

1. Incorporate the feedback into the workflow analysis
2. Revise the task list, schemas, or flow as needed
3. Present the revised plan for approval (loop back to Phase 4)
4. Continue until user approves

### Phase 6: Persist the Workflow

You MUST persist workflow to `.charge/{YYYY-MM-DD}/{workflow-name}/`

## Important Notes

- Do NOT execute the workflow after building - that's what `/charge:run` is for
- The bare `/charge` command calls this build flow AND then executes; this command only builds
- Always persist the approved workflow to disk before reporting success
- Use the current date (YYYY-MM-DD format) for the workflow directory
