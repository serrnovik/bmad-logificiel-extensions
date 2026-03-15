# BMad Logificiel Extensions

External BMAD module repository for Logificiel-specific automation extensions.

## Included

- `bmad-sprint-automation`
  Runs the implementation loop autonomously after sprint planning.
- `bmad-sprint-review`
  Produces a final sprint report from `sprint-status.yaml`.

## Current Behavior

- Each completed story is expected to end in its own local git commit.
- Commits should be created after the story reaches `done`.
- Commits should not be pushed automatically.
- The workflow is designed around clean-context delegation for:
  - `create-story`
  - `dev-story`
  - `code-review`
  - `sprint-review`

If the host cannot spawn subagents or fresh sessions, the workflow falls back to inline execution, which is less clean for long-running sprints.

## Design Goals

- Survive BMAD updates by shipping as a separate module repo.
- Avoid patching installed `_bmad` files.
- Prefer additive workflows over replacing core BMAD skills.
- Work in Claude, Codex, Cursor, and similar hosts through a host-neutral workflow contract.
- Require one local, non-pushed commit per completed story.

## Repository Layout

```text
module.yaml
module-help.csv
workflows/
  4-implementation/
    bmad-sprint-automation/
    bmad-sprint-review/
```

## Local Installation

Install this module into a project with BMAD already present:

```bash
npx bmad-method install \
  --directory /path/to/project \
  --custom-content /Users/sno/Data/snogit/bmad-logificiel-extensions \
  --tools none
```

If BMAD is already installed in the target project, you can also run:

```bash
npx bmad-method install \
  --directory /path/to/project \
  --action update \
  --custom-content /Users/sno/Data/snogit/bmad-logificiel-extensions
```

BMAD will cache the custom module source under `_bmad/_config/custom`, which makes it survive future BMAD updates.

## Later Publication

When the module is stable:

1. Publish this repo.
2. Add an entry to BMAD-METHOD `tools/cli/external-official-modules.yaml`.
3. Point `module-definition` to `module.yaml`.

That upgrades the module from local custom content to first-class external module installation.
