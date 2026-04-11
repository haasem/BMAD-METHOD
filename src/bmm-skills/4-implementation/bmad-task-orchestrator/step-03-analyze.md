# Step 3: Analyze

## RULES

- YOU MUST ALWAYS SPEAK OUTPUT in `{communication_language}`
- Run reviews SEQUENTIALLY, not in parallel — each review may inform the next
- Independent Verification must NOT receive INVESTIGATION.md (per skill rules)
- Quality gate evaluation happens AFTER all reviews complete

## INSTRUCTIONS

### 1. Update State

Set `current_phase: analyze`, `analyze.status: in-progress` in state file.

### 2. Spawn Party Mode for Proposal

Read the installed Party Mode skill at `{party_mode_skill}`.

Spawn a subagent via the Agent tool with:
- The Party Mode SKILL.md content
- The investigation file: `{analysis_folder}/INVESTIGATION.md`
- Instruction: "Analyze the investigation and produce a concrete, 
  actionable proposal for: {task_scope}. Write the consolidated 
  proposal to `{analysis_folder}/PROPOSAL.md`. Include specific 
  recommendations with reasoning, not just observations."
- The orchestrator drives the discussion by providing these as 
  sequential "user messages" in the prompt:
  1. "Based on the investigation, what are the key technical decisions 
     and trade-offs for this task?"
  2. "Produce a concrete implementation approach with specific steps."
  3. "Consolidate into a final proposal document."

After completion, verify `{analysis_folder}/PROPOSAL.md` exists.

Update state: record artifact, note skill used.

### 3. Spawn Adversarial Review

Read the installed adversarial review skill at `{adversarial_review_skill}`.

Spawn a subagent with:
- The skill SKILL.md content
- Input: INVESTIGATION.md and PROPOSAL.md paths
- Instruction: "Review this proposal against the investigation. 
  Write findings to `{analysis_folder}/REVIEW-adversarial.md`."

After completion, verify review file exists. Parse finding counts:
- Count CRITICAL, HIGH, MEDIUM findings
- Count UNGROUNDED and WRONG claims from claim verification section

Update state: record artifact.

### 4. Spawn Claim Verification

Read the installed claim verification skill at `{claim_verification_skill}`.

Spawn a subagent with:
- The skill SKILL.md content
- Input: PROPOSAL.md and source materials (codebase, INVESTIGATION.md)
- Instruction: "Verify every claim in the proposal against source 
  material. Write findings to `{analysis_folder}/REVIEW-claims.md`."

After completion, parse verification results:
- Count IMPRECISE, UNGROUNDED, WRONG, MISCLASSIFIED, INCOMPLETE verdicts

Update state: record artifact.

### 5. Spawn Independent Verification (conditional)

Only run if the task was marked as client-facing at intake.

Read the installed IV skill at `{independent_verification_skill}`.

Spawn a subagent with:
- The skill SKILL.md content
- Input: PROPOSAL.md ONLY — do NOT include INVESTIGATION.md
- Instruction: "Independently verify the key claims in this proposal. 
  You must NOT read INVESTIGATION.md before completing your own 
  verification. Write findings to 
  `{analysis_folder}/REVIEW-independent.md`."

After completion, parse findings: count CONTRADICTED, MISFRAMED items.

Update state: record artifact.

### 6. Quality Gate: Analysis Review

Aggregate findings across all review artifacts:

| Metric | Source | Threshold |
|--------|--------|-----------|
| CRITICAL findings | Adversarial Review | 0 (any = halt) |
| HIGH findings | Adversarial Review | >3 = halt |
| UNGROUNDED claims | Claim Verification | >5 = halt |
| WRONG claims | Claim Verification | >0 = halt |
| CONTRADICTED claims | Independent Verification | >2 = halt |

**If quality gate FAILS:**
- Present consolidated findings to user with counts per category
- Offer options:
  1. Revise proposal and re-run reviews (loop back to Party Mode)
  2. Accept findings and proceed with documented caveats
  3. Halt entirely
- If user chooses option 1: loop back to the Party Mode step, 
  incrementing an analyze_iteration counter. Halt if >3 iterations.

**If quality gate PASSES:**
- Present a brief summary: "Analysis complete. Proposal reviewed by 
  {N} reviewers. {findings summary}. Ready to proceed."

### 7. Checkpoint

If `checkpoints.after_analyze` is ON:
- Present: proposal summary, review findings summary, quality gate result
- **HALT and wait for user approval** to proceed

If task type is "analysis + proposal", skip to step-06-deliver.md.

### 8. Update State and Advance

Set `analyze.status: completed`, record all artifacts and timestamps.

## NEXT

- If task type is "analysis + proposal": read fully and follow `./step-06-deliver.md`
- If task type is "full end-to-end": read fully and follow `./step-04-implement.md`
