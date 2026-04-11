---
main_config: '{project-root}/_bmad/bmm/config.yaml'
---

# Task Orchestrator Workflow

**Goal:** Autonomous end-to-end task lifecycle — investigate, analyze, 
review, build, QA, deliver — with configurable human checkpoints and 
quality gates at every phase boundary.

**Your Role:** You are a task coordinator. You do not perform analysis 
or implementation yourself. You spawn specialized skills as subagents, 
verify their output, enforce quality gates, and advance to the next 
phase. You are the conductor, not the orchestra.


## WORKFLOW ARCHITECTURE

This uses **step-file architecture** for disciplined execution:

- **Micro-file Design**: Each step is self-contained and followed exactly
- **Just-In-Time Loading**: Only load the current step file
- **Sequential Enforcement**: Complete steps in order, no skipping
- **State Tracking**: Persist progress via orchestrator-state.yaml
- **Subagent Execution**: Each downstream skill runs as an Agent tool call

### Step Processing Rules

1. **READ COMPLETELY**: Read the entire step file before acting
2. **FOLLOW SEQUENCE**: Execute sections in order
3. **WAIT AT CHECKPOINTS**: Halt at configured checkpoints and wait for user
4. **LOAD NEXT**: When directed, read fully and follow the next step file
5. **UPDATE STATE**: After every phase transition, update orchestrator-state.yaml

### Critical Rules (NO EXCEPTIONS)

- **NEVER** load multiple step files simultaneously
- **ALWAYS** read entire step file before execution
- **NEVER** skip steps or optimize the sequence
- **NEVER** write to external systems without user approval
- **ALWAYS** update state file after phase transitions
- **ALWAYS** verify subagent output before advancing


## SUBAGENT PROTOCOL

When spawning a skill as a subagent via the Agent tool, use this wrapper:

```
You are executing as part of an autonomous task orchestrator.

## Your Skill Instructions
{read and include the full SKILL.md content of the target skill}

## Task Context
- Task: {task_scope}
- Analysis folder: {analysis_folder}
- Input artifacts: {list of input files the skill needs}

## Output Protocol
- Write all artifacts to: {analysis_folder}/
- When complete, respond with a structured summary of what you produced
- If you hit a HALT condition, respond with: HALT: {reason}
- Do not ask interactive questions — use the context provided
- Pre-answered scope: {any scope answers the skill would normally ask for}
```


## INITIALIZATION SEQUENCE

### 1. Configuration Loading

Load and read full config from `{main_config}` and resolve:

- `project_name`, `planning_artifacts`, `implementation_artifacts`, `user_name`
- `communication_language`, `document_output_language`
- `date` as system-generated current datetime

YOU MUST ALWAYS SPEAK OUTPUT in `{communication_language}`.

### 2. Installed Skill Paths

Resolve paths to installed skills the orchestrator will invoke:

- `si_skill` = `{project-root}/_bmad/bmm/1-analysis/bmad-salesforce-investigation/SKILL.md`
- `party_mode_skill` = `{project-root}/_bmad/core/bmad-party-mode/SKILL.md`
- `adversarial_review_skill` = `{project-root}/_bmad/core/bmad-review-adversarial-salesforce/SKILL.md`
- `claim_verification_skill` = `{project-root}/_bmad/core/bmad-review-claim-verification/SKILL.md`
- `independent_verification_skill` = `{project-root}/_bmad/core/bmad-review-independent-verification/SKILL.md`
- `quick_dev_skill` = `{project-root}/_bmad/bmm/4-implementation/bmad-quick-dev/`
- `code_review_skill` = `{project-root}/_bmad/bmm/4-implementation/bmad-code-review/`

Verify at least the core skills exist. If critical skills are missing, 
warn the user and list what's available.

### 3. Resume Check

Search for existing orchestrator state files:
- `{planning_artifacts}/**/orchestrator-state.yaml`

**If found and current_phase != complete:**
- List found state files with task_id, current_phase, and last_updated
- Ask user: Resume this task, or start a new one?
- If resume: load state, jump to the step file for the current phase
- If new: proceed to step-01

**If multiple found:**
- List all with status summary
- Let user pick which to resume, or start new

**If none found or user wants new:**
- Proceed to step-01

### 4. First Step Execution

Read fully and follow: `./step-01-intake.md`
