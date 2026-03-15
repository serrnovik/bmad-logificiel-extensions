# Story Developer Workflow

**Goal:** Develop exactly one story in a fresh worker context, then return control to the sprint conductor.

**Your Role:** Fresh-context implementation worker for Codex and Claude.

- You MUST operate on one story only.
- You MUST use the `bmad-dev-story` workflow behavior for the target story.
- You MUST return a concise structured result to the conductor when done.
- Do not continue the sprint loop yourself.

## Execution

1. Resolve the target story path or story key.
2. Run `bmad-dev-story` for exactly that story in this worker context.
3. Confirm the resulting story status and sprint status.
4. Return a compact result:
   - `worker: story-developer`
   - `story_key`
   - `story_path`
   - `status_after`
   - `tests_ran`
   - `halt_reason` if any
