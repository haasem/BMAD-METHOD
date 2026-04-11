# Step 5: QA

## RULES

- YOU MUST ALWAYS SPEAK OUTPUT in `{communication_language}`
- Maximum QA iterations: read from state file `qa.max_iterations` (default 3)
- intent_gap findings ALWAYS halt — never auto-fix
- Track every iteration in `qa_iterations` array in state file

## INSTRUCTIONS

### 1. Update State

Set `current_phase: qa`, `qa.status: in-progress` in state file.

### 2. Construct Review Input

Determine what to review:
- If Quick Dev route: use the diff since `{baseline_commit}` and the 
  spec file from step-04
- If Dev Story route: use the story implementation diff and story file

### 3. Spawn Code Review

Read the installed code review skill at `{code_review_skill}`.

Spawn a subagent via the Agent tool with:
- The code review skill content (SKILL.md + step files)
- Input: the implementation diff and spec/story file
- Instruction: "Review this implementation. Classify all findings as: 
  intent_gap, bad_spec, patch, defer, or reject. Respond with the 
  full classification list. Write the review report to 
  `{analysis_folder}/REVIEW-qa-{iteration}.md`."

### 4. Parse and Classify Findings

From the code review output, extract findings by classification:

| Classification | Action |
|---------------|--------|
| **intent_gap** | HALT unconditionally. Present to user. Cannot auto-fix. |
| **bad_spec** | Auto-retry: revert code, fix spec, re-implement, re-review |
| **patch** | Auto-fix: apply the fix, re-review |
| **defer** | Log to deferred work file, do not block |
| **reject** | Drop silently |

### 5. Handle Findings

**If intent_gap found:**
- Present the intent_gap findings to the user
- HALT. The user must clarify intent before continuing.
- After user resolves: loop back to step-04 (re-implement with 
  clarified intent), then re-enter this step.

**If bad_spec found (no intent_gap):**
- Increment `qa.iteration` in state file
- Check iteration count against `qa.max_iterations`
- If exceeded: HALT, present all iterations summary, escalate to user
- Otherwise: revert code changes, apply spec corrections (extract 
  KEEP instructions for what worked well), re-implement by spawning 
  implementation subagent again, then re-enter this step at section 3

**If only patch/defer/reject:**
- Auto-fix all patch findings
- Log defer findings to `{analysis_folder}/DEFERRED.md`
- Drop reject findings
- Re-run code review on the patched code (one final pass)

**If no findings (clean pass):**
- QA passes

### 6. Record QA Iteration

After each iteration, append to `qa_iterations` in state file:
```yaml
- iteration: {n}
  findings_count: {total}
  finding_types: {intent_gap: N, bad_spec: N, patch: N, defer: N, reject: N}
  action: "{auto-retry | halt | pass}"
  resolved: [list of resolved finding IDs]
```

### 7. Quality Gate: QA

**QA PASSES when:**
- Code review returns zero intent_gap, bad_spec, or patch findings
- Only defer/reject remain (or nothing)

**QA FAILS (halt) when:**
- Any intent_gap finding exists
- `qa.iteration` exceeds `qa.max_iterations`
- After auto-fix pass, new HIGH findings appear that weren't in 
  the original review

On halt: present full QA history (all iterations, what was found, 
what was fixed, what remains) and ask user how to proceed.

### 8. Checkpoint

If `checkpoints.after_qa_pass` is ON and QA passed:
- Present: QA summary, iterations count, files changed, deferred items
- **HALT and wait for user approval** before delivery

### 9. Update State and Advance

Set `qa.status: completed`, record quality gate result and timestamp.

## NEXT

Read fully and follow `./step-06-deliver.md`
