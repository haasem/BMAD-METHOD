how should I start a project and how should I continue an existing project so that the correct order of agents is ensured and they can benefit from each other?1:41 PMGood question to nail down before you start using the new pipeline on real client work. Here's the complete picture.

Starting a New Project
Step 1 — Install
bashcd /path/to/new-client-project
~/install-bmad-salesforce.sh
Step 2 — Create your CLAUDE.md
Before any agent runs, create a CLAUDE.md in the project root encoding client-specific knowledge that no agent can discover on its own:
markdown# Client: [Client Name]
## Salesforce Org
- Sandbox URL: 
- API version: 
- Naming conventions: 
- CPQ version and edition: 
- Key integration partners: 

## Project Scope
- In-scope objects: 
- Out-of-scope: 
- Known constraints: 

## Standards
- Deployment process: 
- Branch naming: 
- Test coverage minimum:
This is read every single turn by every agent. It's the cheapest leverage you have.
Step 3 — Run Mary: Domain or Technical Research
Before any planning or analysis, ground the project in current knowledge:

[DR] if you need industry/regulatory context
[TR] if you need technical feasibility or architecture options
Both if the project is complex

These produce research documents in your planning artifacts folder that feed everything downstream.
Step 4 — Run Mary: Document Project [DP]
If this is a brownfield Salesforce org (which it almost always is), run Document Project first to produce a baseline understanding of the existing system. This becomes foundational context for all subsequent work.
Step 5 — Run Mary: Salesforce Investigation [SI]
For each specific task, feature, or issue you're working on, run SI with the relevant Jira root ticket. This produces INVESTIGATION.md scoped to that task.
Step 6 — Party Mode
Party Mode now automatically loads INVESTIGATION.md. All agents are grounded in the same evidence base from the first message.
Step 7 — Salesforce Adversarial Review
After Party Mode produces a proposal, run the Salesforce adversarial review passing both the proposal and INVESTIGATION.md. Iterate on the proposal until coverage gaps are resolved.
Step 7b — Claim Verification
Run Claim Verification [CV] on any analysis or proposal output before handing it to stakeholders. This is not optional. Analysis agents systematically produce confident statements that do not survive scrutiny — incorrect action items, fabricated estimates, wrong data-access-path claims. The only reliable way to find them is to re-examine every claim against the actual source material. CV does what you would do if you questioned every sentence in the document.
Step 7c — Independent Verification (high-stakes deliverables)
For deliverables going to clients under your professional name, run Independent Verification [IV]. This is a second analyst who re-investigates key claims using DIFFERENT methods than the original analyst — different queries, different angles, inverse checks. This catches errors that survive adversarial review because every prior step shares the same blind spots (same evidence, queried the same way). The second analyst does NOT read INVESTIGATION.md before their own verification — they approach fresh and only cross-reference afterward. Skip this for internal working documents; use it for anything a client or stakeholder will see.
Step 8 — Advanced Elicitation
Now run Advanced Elicitation — on a proposal that has already been grounded and adversarially reviewed, this is genuinely productive rather than patching upstream gaps.

Continuing an Existing Project
The key difference from starting fresh is that context already exists and needs to be loaded, not created.
Before each working session:

Check what exists — scan your planning artifacts folder:

ls _bmad-output/
Know what research, investigation files, and proposals are already there before invoking any agent.

Update CLAUDE.md if anything has changed — new decisions made, scope changes, constraints discovered. Takes two minutes and saves significant drift across sessions.
For a new task on an existing project — always run [SI] fresh for that task. Don't reuse a previous INVESTIGATION.md from a different ticket — Salesforce orgs change and a stale investigation is worse than no investigation because it creates false confidence.
For continuing an in-progress task — if you're continuing work from a previous session on the same ticket, check whether the existing INVESTIGATION.md is still current. If any deployments have happened since it was created, re-run [SI] or at minimum ask Mary to validate it.


The Complete Order — Quick Reference
NEW PROJECT
─────────────────────────────────────────────
install → CLAUDE.md → [DR]/[TR] → [DP] → [SI] → Party Mode
→ SF Adversarial Review → [CV] Claim Verification
→ [IV] Independent Verification (if client-facing) → Advanced Elicitation → Build

EXISTING PROJECT — NEW TASK
─────────────────────────────────────────────
Update CLAUDE.md → [SI] (fresh for this ticket) → Party Mode
→ SF Adversarial Review → [CV] Claim Verification
→ [IV] Independent Verification (if client-facing) → Advanced Elicitation → Build

EXISTING PROJECT — CONTINUING SAME TASK
─────────────────────────────────────────────
Check INVESTIGATION.md currency → update if needed
→ Party Mode → SF Adversarial Review → [CV] Claim Verification
→ [IV] if client-facing → continue Build

QUICK TASK (bug fix, config change, small story)
─────────────────────────────────────────────
[SI] (scoped tight) → Party Mode → SF Adversarial Review → [CV] → Build
Skip [DR]/[TR]/[DP]/[IV] unless something unexpected surfaces

One Rule to Enforce Above All Others
Never run Party Mode without a current INVESTIGATION.md for the task in scope. The Investigation Gate will warn you, but it won't stop you if you say yes. The gate is a guardrail, not a lock. The discipline has to come from you.
Everything else in the pipeline is additive — you can skip [DR] on a simple bug fix, you can skip [DP] if the org is well-documented. But [SI] before Party Mode is the one step that cannot be skipped without directly causing the problem you've been experiencing.