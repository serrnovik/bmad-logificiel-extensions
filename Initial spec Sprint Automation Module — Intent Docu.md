# Sprint Automation Module — Intent Document

**Purpose:** Document intent for a BMAD external module that automates the create-story → dev-story → code-review cycle until the sprint is complete.

**Status:** Intent / design doc for future implementation. Add to private repo for tracking.

---

## Goal

Automate the full story cycle so that, once sprint planning is done, the system can run:

1. **Create Story (CS)** — Prepare next backlog story for implementation
2. **Dev Story (DS)** — Implement the story
3. **Code Review (CR)** — Validate and fix or create action items
4. **Loop** — If CR marks `in-progress`, re-run DS; if `done`, run CS for next story
5. **Stop** — When no more `backlog` or `ready-for-dev` stories exist

With minimal or no human intervention per story.

---

## Delivery Model: External Module (Plugin)

- Implement as an **external BMAD module**, not in the main BMAD-METHOD repo
- Use the existing extension mechanism (`external-official-modules.yaml`, module structure)
- Module depends on `bmm`; reuses `bmad-create-story`, `bmad-dev-story`, `bmad-code-review`
- Keeps main codebase unchanged; installable via `npx bmad-method install` or community registry

---

## What Already Supports Automation

| Component     | Support                                                       |
| ------------- | ------------------------------------------------------------- |
| Create Story  | Auto-discovers next `backlog` story from `sprint-status.yaml` |
| Dev Story     | Auto-discovers first `ready-for-dev` story                    |
| Code Review   | Can "fix automatically" (option 1) for HIGH/MEDIUM issues     |
| Sprint status | Each workflow updates `sprint-status.yaml`                    |

---

## Blockers to Address

| Blocker                                                | Approach                                                                              |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| User prompts (`<ask>`) in workflows                    | Add `automation_mode` or similar config to skip prompts when next step is unambiguous |
| Code review choice (fix vs. action items)              | Default to "fix automatically" in automation mode                                     |
| HALT conditions (new deps, 3 failures, missing config) | Define policy: fail-fast, retry, or escalate                                          |
| No orchestration                                       | New workflow: `bmad-sprint-automation` that runs CS → DS → CR in sequence             |
| Session/context limits                                 | Handle long stories; may need chunking or continuation strategy                       |

---

## Proposed Module Structure

```
bmad-sprint-automation/
├── src/
│   ├── module.yaml              # code: spa (or similar), depends on bmm
│   ├── module-help.csv          # Register Sprint Automation workflow
│   └── workflows/
│       └── 4-implementation/
│           └── bmad-sprint-automation/
│               ├── SKILL.md
│               └── workflow.md  # Orchestrator logic
```

---

## Orchestrator Logic (High Level)

1. Load `sprint-status.yaml` and config
2. **Loop** until sprint complete:
   - If first `backlog` story exists → run create-story (or delegate)
   - If first `ready-for-dev` story exists → run dev-story
   - If first `review` story exists → run code-review (auto-fix mode)
   - If code-review returns `in-progress` → run dev-story again
   - If code-review returns `done` → continue to next story
3. **Exit** when no `backlog` or `ready-for-dev` stories remain

---

## Distribution Options

- **Official:** Add to `external-official-modules.yaml` in BMAD-METHOD
- **Community:** Publish to npm, document for manual install
- **Private:** Install from local path or private repo URL

---

## Dependencies

- BMAD Method with BMM module installed
- `sprint-status.yaml` from `bmad-sprint-planning`
- Platform support for workflow chaining / subagent invocation (varies by IDE)

---

## References

- BMAD workflow map: `docs/reference/workflow-map.md`
- Module extension: `tools/cli/external-official-modules.yaml`
- Story cycle workflows: `src/bmm/workflows/4-implementation/` (create-story, dev-story, code-review)
