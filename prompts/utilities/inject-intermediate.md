# Utility: Inject Intermediate Tasks

Monitor context growth and inject summarization tasks when needed.

## Purpose

Long workflows can accumulate large amounts of data in task outputs, leading to context window explosion. This utility detects when outputs are growing too large and injects intermediate summarization tasks to compress the context.

## Triggers

Inject an intermediate task when:

| Condition | Threshold | Action |
|-----------|-----------|--------|
| Single task output size | > 4000 tokens | Chunk output to file, pass reference |
| Cumulative result size | > 8000 tokens | Inject summarization task |
| Task count in workflow | > 10 tasks | Add checkpoint summarization |

## Injection Process

### Step 1: Detect Growth

After each task completes, check:
1. Size of the task's output (in tokens or characters)
2. Cumulative size of all results so far
3. Number of tasks completed

### Step 2: Decide on Intervention

If thresholds are exceeded:

**For large single output:**
- Don't pass the full output to the next task
- Instead, write to a file and pass a reference
- The next task loads from file as needed

**For cumulative growth:**
- Insert a `summarize-progress` task
- This task reads all outputs and produces a condensed summary
- Subsequent tasks use the summary instead of raw outputs

### Step 3: Generate Intermediate Task

Create a summarization task:

```json
{
  "id": "[string]",
  "name": "[string]",
  "description": "[string]",
  "instruction_file": "[string]",
  "input_schema": "[string]",
  "output_schema": "[string]",
  "depends_on": ["[string]"],
  "injected": true|false
}
```

Summarization instruction:

```markdown
# Task: Summarize Progress

## Purpose

Condense the outputs from previous tasks into a compact summary that preserves essential information while reducing size.

## Input

You will receive outputs from multiple preceding tasks.

## Output

Produce a condensed summary that:
- Preserves all critical information needed by subsequent tasks
- Removes redundant or verbose content
- Maintains structural integrity of important data
- Keeps the output under 2000 tokens

## Guidelines

- Focus on facts and data, not explanations
- Use bullet points for lists
- Compress verbose descriptions
- Keep code snippets if they're referenced later
- Remove intermediate reasoning
```

### Step 4: Update Flow

Insert the summarization task into the execution order:
1. Update manifest with new task
2. Adjust dependencies: subsequent tasks depend on summarization
3. Update mappings: subsequent tasks receive summary instead of raw outputs

## Output Reference Pattern

For chunked outputs, use this reference pattern:

```json
{
  "type": "[string]",
  "path": "[string]",
  "summary": "[string]",
  "key_fields": ["[string]"]
}
```

Tasks receiving a reference should:
1. Check if input is a reference type
2. Load from file if needed
3. Process only the relevant portions

## Notes

- Intermediate tasks are marked with `"injected": true` in manifest
- Users can see these in `/charge:inspect` output
- Summarization preserves correctness over compression ratio
- When in doubt, keep more information rather than less
