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

Present the proposed workflow to the user with the **ExitPlanMode** tool. Include a visual diagram with bounding box.

**Format:**

```markdown
## Proposed Workflow: {workflow-name}

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   ┌─────────────────┐                                       │
│   │ 1. {task-name}  │                                       │
│   └────────┬────────┘                                       │
│            │                                                │
│            ▼                                                │
│   ┌─────────────────┐                                       │
│   │ 2. {task-name}  │                                       │
│   └────────┬────────┘                                       │
│            │                                                │
│            ▼                                                │
│   ┌─────────────────┐                                       │
│   │ 3. {task-name}  │                                       │
│   └─────────────────┘                                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘

### Tasks

| # | Task | Description |
|---|------|-------------|
| 1 | {task-name} | {brief description} |
| 2 | {task-name} | {brief description} |
| 3 | {task-name} | {brief description} |

---
Approve this workflow?
```

**Diagram Rules:**

1. **Always use bounding box** - Even for 1-step workflows, wrap in outer box
2. **Sequential flow** - Use `│` and `▼` arrows between tasks
3. **Parallel tasks** - Show side-by-side with horizontal connection:
   ```
   │            ┌─────────────────┐   ┌─────────────────┐
   ├───────────►│ 2a. {task}      │   │ 2b. {task}      │◄────┤
   │            └────────┬────────┘   └────────┬────────┘     │
   │                     └──────────┬──────────┘              │
   │                                ▼                         │
   ```
4. **Template tasks** - Show with iteration indicator:
   ```
   │   ┌─────────────────────────┐                            │
   │   │ 2. {task-name} [×N]     │  ◄── iterates over items   │
   │   └─────────────────────────┘                            │
   ```

**Single-Step Workflow Example:**

```markdown
## Proposed Workflow: analyze-code

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   ┌─────────────────────────┐                               │
│   │ 1. analyze-codebase     │                               │
│   └─────────────────────────┘                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Phase 5: Handle User Response

**If user approves:**

1. Generate a workflow ID: `{workflow-name}` (derived from prompt, kebab-case)
2. Create the workflow directory at `$HOME/.charge/workflows/{YYYY-MM-DD}-{workflow-name}/`
3. Write workflow artifacts (NO `state.json` or `results/` - those are created at execution time):
   - `manifest.json` using `templates/manifest.json` structure
   - Task instruction files in `instructions/` using `templates/task-instruction.md`
   - Schema files in `schemas/`
4. Report success:

   ```ascii
   Workflow created: {workflow-name}
   Location: $HOME/.charge/workflows/{YYYY-MM-DD}-{workflow-name}/

   Run with: /charge:run {workflow-name}
   ```

5. You MUST NOT execute the plan when it was approved.

**If user provides feedback:**

1. Incorporate the feedback into the workflow analysis
2. Revise the task list, schemas, or flow as needed
3. Present the revised plan for approval (loop back to Phase 4)
4. Continue until user approves

### Phase 6: Persist the Workflow

You MUST persist workflow to `$HOME/.charge/workflows/{YYYY-MM-DD}-{workflow-name}/`

**DO NOT** create `state.json` or `results/` directory during build - these are created during execution in the sessions directory.

## Important Notes

- Do NOT execute the workflow after building - that's what `/charge:run` is for
- The bare `/charge` command calls this build flow AND then executes; this command only builds
- Always persist the approved workflow to disk before reporting success
- Use the current date (YYYY-MM-DD) as a prefix in the workflow directory name
- Workflow directory format: `$HOME/.charge/workflows/{YYYY-MM-DD}-{workflow-name}/`
