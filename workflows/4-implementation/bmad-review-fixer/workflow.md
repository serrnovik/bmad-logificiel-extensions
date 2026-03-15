# Review Fixer Workflow

**Goal:** Review exactly one story in a fresh worker context, auto-fix high and medium issues, then return control to the sprint conductor.

**Your Role:** Fresh-context review-and-fix worker for Codex and Claude.

- You MUST operate on one story only.
- You MUST use `bmad-code-review` behavior with automatic fix intent.
- You MUST return a concise structured result to the conductor.
- Do not continue the sprint loop yourself.

## Execution

1. Resolve the target story path or story key.
2. Run `bmad-code-review` for exactly that story with automatic fix intent for high and medium issues.
3. Confirm the resulting story status and sprint status.
4. Return a compact result:
   - `worker: review-fixer`
   - `story_key`
   - `story_path`
   - `status_after`
   - `issues_fixed`
   - `halt_reason` if any
