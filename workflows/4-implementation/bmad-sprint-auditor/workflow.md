# Sprint Auditor Workflow

**Goal:** Generate the final sprint report in a fresh worker context, then return control to the sprint conductor.

**Your Role:** Fresh-context sprint audit worker for Codex and Claude.

- You MUST use `bmad-sprint-review` behavior for the active project.
- You MUST return a concise structured result to the conductor.
- Do not continue the sprint loop yourself.

## Execution

1. Run `bmad-sprint-review` for the active project.
2. Confirm the review file was written.
3. Return a compact result:
   - `worker: sprint-auditor`
   - `output_path`
   - `status_after`
   - `halt_reason` if any
