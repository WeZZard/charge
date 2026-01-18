# Core: Synthesize

Combine task outputs into a coherent final result for the user.

## Input

- `manifest`: The workflow manifest
- `results`: All task result files (from `results/` directory)
- `original_prompt`: The user's original request

## Process

### Step 1: Load All Results

Read each result file from `results/{task_id}.json` and extract the output data.

### Step 2: Understand the Original Goal

Review the original prompt to understand:

- What the user asked for
- What format they likely expect (code, text, structured data)
- What level of detail is appropriate

### Step 3: Combine Results

Different synthesis strategies based on workflow type:

**For code generation workflows:**

- Combine code outputs into complete, runnable files
- Ensure imports and dependencies are resolved
- Present files in a logical order

**For analysis workflows:**

- Summarize findings from each task
- Highlight key insights
- Present recommendations

**For transformation workflows:**

- Show the final transformed output
- Optionally show intermediate steps
- Validate the transformation meets requirements

### Step 4: Format the Output

Present the synthesized result in a user-friendly format:

```markdown
## Workflow Complete: {workflow-name}

{Summary of what was accomplished}

---

### Results

{Main output content - code, analysis, etc.}

---

### Task Summary

| Task | Status | Key Output |
|------|--------|------------|
| {task_name} | Done | {brief description of output} |
| {task_name} | Done | {brief description of output} |

---

### Files Created

{If the workflow created files, list them}

### Next Steps

{Suggest what the user might want to do next}
```

### Step 5: Handle Partial Completion

If some tasks failed:

```
## Workflow Partially Complete: {workflow-name}

{Number} of {total} tasks completed.

### Completed

| Task | Output |
|------|--------|
| {task_name} | {brief output} |

### Failed

| Task | Error |
|------|-------|
| {task_name} | {error message} |

### Available Results

{Present what was completed}

### To Continue

Run `/charge:run {workflow-name}` to retry failed tasks.
```

## Output Formats

### Code Output

```markdown
## Generated Code

### `{filename}`

\`\`\`{language}
{code content}
\`\`\`

### `{filename}`

\`\`\`{language}
{code content}
\`\`\`
```

### Analysis Output

```markdown
## Analysis Results

### Key Findings

1. {Finding 1}
2. {Finding 2}

### Recommendations

- {Recommendation 1}
- {Recommendation 2}

### Details

{Detailed analysis}
```

### Data Output

```markdown
## Processed Data

\`\`\`json
{structured data output}
\`\`\`
```

## Notes

- The synthesis should feel like a natural conclusion to the workflow
- Don't just dump all task outputs - create a cohesive narrative
- Highlight the most important results
- Make it easy for the user to find what they need
