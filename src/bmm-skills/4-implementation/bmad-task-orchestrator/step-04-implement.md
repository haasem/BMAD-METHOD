# Step 4: Implement

## RULES

- YOU MUST ALWAYS SPEAK OUTPUT in `{communication_language}`
- The implementation subagent does the actual coding — you coordinate
- If the subagent HALTs, surface the reason to the user immediately

## INSTRUCTIONS

### 1. Update State

Set `current_phase: implement`, `implement.status: in-progress` in state file.

### 2. Route Decision

Evaluate the approved proposal to determine implementation approach:

**Quick Dev route** (default for most tasks):
- Single cohesive feature or change
- Can be specified and implemented in one pass
- Use `{quick_dev_skill}`

**Dev Story route** (for larger work):
- Multiple stories or epics needed
- Requires sprint planning structure
- Use the dev story pipeline (create-story → dev-story)

Present the routing decision to the user:
> "Implementation approach: {Quick Dev / Dev Story pipeline}. 
> Reason: {why this route}."

If the user disagrees, switch routes.

### 3a. Quick Dev Route

Read the Quick Dev workflow at `{quick_dev_skill}/workflow.md`.

Spawn a subagent via the Agent tool with:
- The full Quick Dev workflow content and all step files it references
- Input: the approved PROPOSAL.md as the user intent
- Analysis folder for output
- Instruction: "Implement the following proposal using the Quick Dev 
  workflow. The proposal has been reviewed and approved. Treat it as 
  your specification input. Write implementation artifacts to 
  `{implementation_artifacts}/`. When the review step (step-04) 
  produces findings, respond with the finding classifications so the 
  orchestrator can handle the QA loop."

The Quick Dev skill includes its own plan → implement → review cycle. 
The orchestrator delegates the full cycle but monitors the output.

After completion:
- Check if the subagent reported any unresolved findings
- Record the baseline_commit and spec_file from the subagent output
- If the subagent HALTed: surface the HALT reason and ask user how 
  to proceed

### 3b. Dev Story Route

For larger work requiring multiple stories:

1. Check if epics/stories already exist in `{planning_artifacts}/`
2. If not, inform user that story creation is needed first and spawn 
   the create-epics-and-stories skill
3. For each story, spawn dev-story as a subagent sequentially
4. Track progress in sprint-status.yaml if it exists

This route is more complex and may span multiple sessions. Update 
the state file after each story completion so the orchestrator can 
resume.

### 4. Checkpoint

If `checkpoints.after_implement` is ON:
- Present: what was built, files changed, test results
- **HALT and wait for user approval** before QA

### 5. Update State and Advance

Set `implement.status: completed`, record artifacts and timestamps.

## NEXT

Read fully and follow `./step-05-qa.md`
