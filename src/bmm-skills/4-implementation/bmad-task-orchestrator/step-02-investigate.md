# Step 2: Investigate

## RULES

- YOU MUST ALWAYS SPEAK OUTPUT in `{communication_language}`
- Do NOT proceed until INVESTIGATION.md is verified to exist and passes the quality gate
- The subagent runs the full SI skill — do not abbreviate or simplify it

## INSTRUCTIONS

### 1. Update State

Set `current_phase: investigate`, `investigate.status: in-progress` in 
the state file.

### 2. Spawn SI Investigation

Read the installed SI skill file at `{si_skill}`.

Spawn a subagent via the Agent tool with:
- The full SI SKILL.md content as instructions
- Task context: scope from intake, analysis folder path
- Pre-answered scope questions:
  - Root Jira ticket: {from intake}
  - Salesforce org access: {from config or user input at intake}
  - Scope boundaries: {from intake}
  - Output folder: `{analysis_folder}/`
  - Output filename: `INVESTIGATION.md`
- Instruction: "Proceed through the full investigation protocol. 
  Write INVESTIGATION.md to the specified folder. Do not ask 
  interactive questions — use the scope provided."

Display a status message to the user:
> "🔍 **Investigation running...** The SI skill is analyzing the 
> system. This may take several minutes."

### 3. Verify Output

After the subagent completes:
- Verify `{analysis_folder}/INVESTIGATION.md` exists
- Check that it contains: Jira Ticket Tree, Salesforce Object Inventory, 
  Metadata Dependency Matrix, and Coverage Summary sections
- If missing or incomplete: log the issue, retry once. If still 
  incomplete: HALT and escalate to user.

### 4. Quality Gate: Investigation Completeness

Read the Coverage Summary section of INVESTIGATION.md:
- If completeness checklist has unchecked items: warn user, list gaps
- If critical categories are missing (Jira, Apex, Security): HALT

### 5. Checkpoint

If `checkpoints.after_investigate` is ON:
- Present a brief summary of the investigation (scope covered, objects 
  found, key findings count)
- **HALT and wait for user approval** to proceed

### 6. Update State and Advance

Set `investigate.status: completed`, record artifact path and timestamp.

## NEXT

Read fully and follow `./step-03-analyze.md`
