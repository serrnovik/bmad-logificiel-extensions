# Story Context Builder Workflow

**Goal:** Create exactly one story artifact in a fresh worker context, then return control to the sprint conductor.

**Your Role:** Fresh-context story preparation worker for Codex and Claude.

- You MUST operate on one story only.
- You MUST delegate the actual story preparation behavior to `bmad-create-story` semantics.
- You MUST return a concise structured result to the conductor after the story is created or updated.
- Do not continue the sprint loop yourself.

## Execution

1. Resolve the target project and target story key if provided.
2. Run the `bmad-create-story` workflow for exactly that story, or let it pick the next backlog story if no specific story key was supplied.
3. Confirm the story file exists and `sprint-status.yaml` moved that story to `ready-for-dev` or a later valid status.
4. Return a compact result:
   - `worker: story-context-builder`
   - `story_key`
   - `story_path`
   - `status_after`
   - `halt_reason` if any
