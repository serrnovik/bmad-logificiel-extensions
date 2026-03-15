# Sprint Automation Workflow

**Goal:** Run the BMAD story delivery loop autonomously until the sprint is complete or an explicit halt policy is triggered.

**Your Role:** Sprint conductor and orchestration layer.

- Own the sprint loop end to end.
- Use `sprint-status.yaml` as the source of truth.
- For Codex and Claude, you MUST use fresh delegated subagents for phase execution instead of doing phase work inline.
- Prefer delegation to dedicated SPA worker skills over directly doing story work in the conductor context.
- Inline phase execution is fallback-only, used only when subagent spawning is genuinely unavailable.
- Do not ask the user for routine routing choices while `automation_mode` is enabled.
- Stop only for explicit halt policies, missing prerequisites, or final sprint completion.

---

## INITIALIZATION

### Configuration Loading

Load config from `{project-root}/_bmad/bmm/config.yaml` and resolve:

- `project_name`
- `communication_language`
- `document_output_language`
- `implementation_artifacts`
- `planning_artifacts`
- `date`

### Paths

- `sprint_status` = `{implementation_artifacts}/sprint-status.yaml`
- `automation_report` = `{implementation_artifacts}/sprint-automation-report.md`

### Automation Policy

Use these defaults unless the host passes overrides:

```yaml
automation_mode: true
review_fix_mode: auto
run_sprint_review_on_complete: true
max_story_attempts: 3
max_consecutive_failures: 3
halt_on_new_dependency: true
halt_on_missing_config: true
commit_each_completed_story: true
push_commits: false
```

### Delegate Contract

The host adapter should provide:

- `delegate(skill_name, args)`
- `fallback_inline(skill_path, args)`

If `delegate(...)` is supported, use it.
If `delegate(...)` is not supported, load the referenced workflow and run it inline with the same inputs and policy.

### Delegation Model

For Codex and Claude, treat these as REQUIRED separate clean-context workers:

- `story-context-builder` -> `bmad-story-context-builder`
- `story-developer` -> `bmad-story-developer`
- `review-fixer` -> `bmad-review-fixer`
- `sprint-auditor` -> `bmad-sprint-auditor`

The conductor should route work and track state, but it should not carry full cross-story implementation context unless the host has no real subagent support.

### Codex / Claude Execution Rule

When running in Codex or Claude:

- Explicitly say `Use sub-agents for this task.`
- Name the worker to spawn.
- Pass only the minimum necessary story key, story path, project, and repo context.
- Wait for the worker result.
- Continue the loop yourself after the worker returns.
- Do not stop after a single phase unless a halt condition is triggered.

---

## EXECUTION

<workflow>

<step n="1" goal="Verify prerequisites">
  <action>Load {sprint_status} if it exists</action>
  <check if="{sprint_status} does NOT exist">
    <action>Delegate to `bmad-sprint-planning`</action>
    <action>Reload {sprint_status}</action>
  </check>

  <check if="{sprint_status} still does NOT exist">
    <output>🚫 Sprint automation cannot start because sprint-status.yaml could not be created.</output>
    <action>HALT</action>
  </check>
</step>

<step n="2" goal="Initialize loop state">
  <action>Set `consecutive_failures = 0`</action>
  <action>Set `story_attempts = {}`</action>
  <action>Set `halt_reasons = []`</action>
</step>

<step n="3" goal="Run autonomous sprint loop">
  <critical>Always re-read the COMPLETE sprint-status.yaml at the start of each loop iteration.</critical>
  <critical>Selection is deterministic by file order and status priority: review -> in-progress -> ready-for-dev -> backlog.</critical>
  <critical>This means the conductor resumes the earliest actionable story, not the most recently touched story.</critical>

  <action>Parse all development_status items in order</action>

  <check if="there is any story with status `review`">
    <action>Pick the first `review` story in file order</action>
    <action>Delegate to `bmad-review-fixer` using a fresh subagent with the story key and explicit story path</action>
    <action>Reload sprint-status.yaml</action>
    <action>GOTO step 4</action>
  </check>

  <check if="there is any story with status `in-progress`">
    <action>Pick the first `in-progress` story in file order</action>
    <action>Increment `story_attempts[story_key]`</action>
    <check if="story_attempts[story_key] > max_story_attempts">
      <action>Add halt reason: `Story exceeded retry budget: {story_key}`</action>
      <action>HALT</action>
    </check>
    <action>Delegate to `bmad-story-developer` using a fresh subagent with the story key and explicit story path</action>
    <action>Reload sprint-status.yaml</action>
    <action>GOTO step 4</action>
  </check>

  <check if="there is any story with status `ready-for-dev`">
    <action>Pick the first `ready-for-dev` story in file order</action>
    <action>Increment `story_attempts[story_key]`</action>
    <check if="story_attempts[story_key] > max_story_attempts">
      <action>Add halt reason: `Story exceeded retry budget: {story_key}`</action>
      <action>HALT</action>
    </check>
    <action>Delegate to `bmad-story-developer` using a fresh subagent with the story key and explicit story path</action>
    <action>Reload sprint-status.yaml</action>
    <action>GOTO step 4</action>
  </check>

  <check if="there is any story with status `backlog`">
    <action>Pick the first `backlog` story in file order</action>
    <action>Delegate to `bmad-story-context-builder` using a fresh subagent with the story key</action>
    <action>Reload sprint-status.yaml</action>
    <action>GOTO step 4</action>
  </check>

  <check if="no stories remain in `backlog`, `ready-for-dev`, `in-progress`, or `review`">
    <action>GOTO step 5</action>
  </check>
</step>

<step n="4" goal="Evaluate loop outcome and enforce halt policy">
  <action>Inspect the delegated workflow outcome</action>

  <check if="workflow completed successfully">
    <action>Set `consecutive_failures = 0`</action>
  </check>

  <check if="workflow halted because of new dependency and halt_on_new_dependency is true">
    <action>Add halt reason: `New dependency approval required`</action>
    <action>HALT</action>
  </check>

  <check if="workflow halted because of missing configuration and halt_on_missing_config is true">
    <action>Add halt reason: `Missing configuration`</action>
    <action>HALT</action>
  </check>

  <check if="workflow halted or failed for another reason">
    <action>Increment `consecutive_failures`</action>
    <check if="consecutive_failures >= max_consecutive_failures">
      <action>Add halt reason: `Too many consecutive failures`</action>
      <action>HALT</action>
    </check>
  </check>

  <check if="a story has transitioned to `done` and commit_each_completed_story is true">
    <action>Verify git repository exists</action>
    <action>Verify working tree changes correspond only to the completed story's work</action>
    <check if="unrelated dirty changes are present">
      <action>Add halt reason: `Working tree contains unrelated changes; cannot create per-story commit safely`</action>
      <action>HALT</action>
    </check>
    <action>Create one local git commit for the completed story</action>
    <action>Do NOT push the commit</action>
    <action>Use a concise story-scoped commit message derived from the story key and completed outcome</action>
  </check>

  <action>Return to step 3</action>
</step>

<step n="5" goal="Generate final sprint review">
  <check if="run_sprint_review_on_complete is true">
    <action>Delegate to `bmad-sprint-auditor` using a fresh subagent</action>
  </check>

  <output>✅ Sprint automation finished.</output>
</step>

</workflow>

---

## Notes For Host Adapters

### Claude / Codex

- Fresh-context subagents are REQUIRED, not optional.
- Pass the target story path or key whenever possible instead of relying on auto-discovery after the first loop decision.
- The conductor must remain thin and must not absorb implementation detail from multiple stories.

### Fallback Hosts

- When a host lacks native subagents, emulate delegation by loading the referenced workflow file and executing it inline.

### Required Behavior Overrides

To make this workflow truly headless, the underlying BMAD workflows should eventually accept these policy flags directly:

- `automation_mode=true`
- `review_fix_mode=auto`
- `skip_routing_questions=true`
- `story_path=<explicit path>` when already known

Without those flags, this conductor can still run, but it will rely on host-side prompt steering rather than first-class workflow inputs.
