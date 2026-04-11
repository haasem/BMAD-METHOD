# Step 1: Intake

## RULES

- YOU MUST ALWAYS SPEAK OUTPUT in `{communication_language}`
- Do NOT proceed past this step until user confirms scope AND checkpoint configuration
- Create the analysis folder and state file before advancing

## INSTRUCTIONS

### 1. Gather Task Scope

Ask the user for:
- **Task description**: What needs to be done? (Jira ticket ID, feature 
  description, bug report, or existing artifact to work from)
- **Task type**: investigation-only, analysis + proposal, or full 
  end-to-end (investigate → build → deliver)
- **Deliverable audience**: internal working document or client-facing 
  deliverable? (This determines whether Independent Verification runs)

If the user provides a Jira ticket ID, fetch the ticket summary to 
confirm scope. If they provide an existing artifact (e.g., a previous 
INVESTIGATION.md), note it as a starting point.

### 2. Determine Phase Sequence

Based on task type, set the active phases:

| Task Type | Phases |
|-----------|--------|
| investigation-only | intake → investigate → deliver |
| analysis + proposal | intake → investigate → analyze → deliver |
| full end-to-end | intake → investigate → analyze → implement → qa → deliver |

Mark skipped phases as `skipped` in the state file.

If the user has an existing investigation file they want to build on, 
the investigate phase can be skipped — set it to `skipped` and record 
the existing file path.

### 3. Set Up Analysis Folder

- Propose a folder name based on the scope: `{ticket-id}_{short-topic}` 
  (e.g., `CRM-9940_Rechnungsanzeige`)
- List existing analysis folders in `{planning_artifacts}/` so the user 
  sees what already exists
- Confirm folder name with user
- Create the folder: `{planning_artifacts}/{folder_name}/`
- Store as `{analysis_folder}`

### 4. Configure Checkpoints

Present the default checkpoint configuration:

```
Checkpoints (pause for your approval):
  after_investigate:  OFF  — investigation is factual, low risk
  after_analyze:      ON   — proposal + review define what gets built
  after_implement:    OFF  — QA will catch issues
  after_qa_pass:      ON   — confirm before delivery
  before_ext_write:   ON   — always (Jira, git push, deploy)
```

Ask: "Would you like to adjust any checkpoints? You can turn any 
on or off. Default is conservative — pauses before major decisions."

Record the final checkpoint configuration.

### 5. Configure Quality Gates

Present the default quality gate thresholds:

```
Quality gates (auto-halt if exceeded):
  Review: halt on any CRITICAL, or >3 HIGH, or >5 UNGROUNDED claims
  QA:     halt on any intent_gap, or >3 QA iterations, or >2 HIGH after fix
```

Ask if user wants to adjust. Most users should keep defaults.

### 6. Write Initial State File

Write `{analysis_folder}/orchestrator-state.yaml` with:
- task_id, created timestamp, analysis_folder path
- current_phase: first active phase (usually "investigate")
- All phases with their initial status (pending or skipped)
- Checkpoint configuration
- Quality gate thresholds

### 7. Confirm and Advance

Present a summary:
```
Task: {task description}
Folder: {analysis_folder}
Phases: {list of active phases}
Checkpoints: {which are ON}
Quality gates: {thresholds}
```

**HALT and wait for user confirmation.**

## NEXT

After user confirms, update state file with `intake.status = completed` 
and advance to the first active phase:
- If investigate is active: read fully and follow `./step-02-investigate.md`
- If investigate is skipped: read fully and follow `./step-03-analyze.md`
