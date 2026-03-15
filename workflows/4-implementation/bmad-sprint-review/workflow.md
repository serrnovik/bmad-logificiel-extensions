# Sprint Review Workflow

**Goal:** Generate a final sprint report from sprint tracking artifacts.

**Your Role:** Scrum Master performing end-of-sprint synthesis.

- Use `sprint-status.yaml` as the primary source of truth.
- Summarize completed work, unfinished work, blocked items, and recommended next actions.
- Keep the report factual and execution-oriented.

---

## INITIALIZATION

### Configuration Loading

Load config from `{project-root}/_bmad/bmm/config.yaml` and resolve:

- `project_name`
- `communication_language`
- `document_output_language`
- `implementation_artifacts`
- `date`

### Paths

- `sprint_status` = `{implementation_artifacts}/sprint-status.yaml`
- `output_file` = `{implementation_artifacts}/sprint-review.md`

---

## EXECUTION

<workflow>

<step n="1" goal="Load sprint tracking">
  <action>Load the COMPLETE {sprint_status}</action>
  <check if="{sprint_status} does NOT exist">
    <output>🚫 Cannot generate sprint review without sprint-status.yaml.</output>
    <action>HALT</action>
  </check>
</step>

<step n="2" goal="Analyze sprint outcome">
  <action>Parse all `development_status` entries in order</action>
  <action>Group stories into:
    - done
    - review
    - in-progress
    - ready-for-dev
    - backlog
  </action>
  <action>Group epics into:
    - done
    - in-progress
    - backlog
  </action>
</step>

<step n="3" goal="Write final report">
  <action>Write a markdown report to {output_file} including:
    - sprint summary
    - completed stories
    - unfinished stories
    - epic status summary
    - operational risks
    - recommended next actions
  </action>

  <output>✅ Sprint review written to {output_file}</output>
</step>

</workflow>
