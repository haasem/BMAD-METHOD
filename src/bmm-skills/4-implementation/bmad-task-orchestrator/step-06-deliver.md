# Step 6: Deliver

## RULES

- YOU MUST ALWAYS SPEAK OUTPUT in `{communication_language}`
- Never push code or write to external systems without explicit user approval
- The delivery summary must be self-contained — someone reading only 
  this file should understand what was done, why, and what's left

## INSTRUCTIONS

### 1. Update State

Set `current_phase: deliver`, `deliver.status: in-progress` in state file.

### 2. Compile Delivery Summary

Write `{analysis_folder}/DELIVERY.md` with the following structure:

```markdown
# Delivery Summary: {task_id}

## Task
{task description from intake}

## What Was Done
{concise summary of the work performed across all phases}

## Artifacts Produced
| Artifact | Location | Purpose |
|----------|----------|---------|
{list every artifact from every phase with file paths}

## Key Findings
{top findings from investigation and reviews that shaped the solution}

## Implementation
{what was built, files changed, approach taken}
{if no implementation: note this was analysis-only}

## QA Results
{QA iteration count, final quality gate result}
{if no QA: note this was analysis/proposal only}

## Deferred Work
{items from DEFERRED.md, if any}
{items that were explicitly out of scope}

## Open Items Requiring Team Confirmation
{absence claims that need team verification}
{dependencies on people or systems the orchestrator couldn't resolve}

## Recommendations
{any follow-up actions recommended}
```

### 3. Update Sprint Status (if applicable)

If `{implementation_artifacts}/sprint-status.yaml` exists and the task 
corresponds to a tracked story:
- Update the story status to `done`
- Update `last_updated` timestamp

### 4. Present to User

Display the delivery summary to the user. Offer:

1. **Commit and push** — if implementation was done, offer to create 
   a commit with a descriptive message and push (requires user approval 
   per `before_external_write` checkpoint)
2. **Create PR** — offer to create a pull request with the delivery 
   summary as description (requires user approval)
3. **Update Jira** — offer to update the Jira ticket status or add a 
   comment (requires user approval per external write rules)
4. **Done** — just mark complete, no external actions

All external write actions require explicit user approval regardless 
of checkpoint configuration.

### 5. Update State

Set `deliver.status: completed`, `current_phase: complete`.
Record final timestamp.

Display:
> "✅ **Task complete: {task_id}**. All artifacts are in 
> `{analysis_folder}/`. Delivery summary: `DELIVERY.md`."
