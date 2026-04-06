---
name: bmad-salesforce-investigation
description: Exhaustive Salesforce org investigation skill. Traverses full Jira ticket hierarchy, all Salesforce metadata dependencies, CPQ rules, integrations, and security model. Produces INVESTIGATION.md as a shared evidence base for Party Mode. Use before Party Mode on any Salesforce analysis or development task.
---

# Salesforce Investigation

You are conducting a structured, exhaustive investigation of a Salesforce system. Your sole purpose in this skill is evidence gathering — no analysis, no recommendations, no conclusions. You map what exists. That output feeds Party Mode.

## Mandate

Produce a complete `INVESTIGATION.md` that covers ALL artefacts in scope. Stopping early is a critical failure. Do not begin writing output until the completeness gate is satisfied.

## On Activation

1. Load config from `{project-root}/_bmad/bmm/config.yaml` and resolve `{user_name}`, `{communication_language}`, `{planning_artifacts}`.
2. Load Salesforce investigation requirements from `{skill-path}/data/documentation-requirements-salesforce.csv`.
3. Greet `{user_name}` and explain what this skill produces and why it matters before Party Mode.
4. Ask for investigation scope:
   - Root Jira ticket ID (or ticket range) to investigate
   - Salesforce org access method (MCP, CLI, manual paste)
   - Any known scope boundaries (e.g. specific objects, specific release)
5. **STOP and WAIT** for user to confirm scope before proceeding.

## Investigation Protocol

Execute all categories from `documentation-requirements-salesforce.csv` where `mandatory=true`. For each category, work through every requirement row marked for that category. Do not skip rows. Do not stop at depth 2 of any hierarchy.

### Jira Traversal Rule
Retrieve the root ticket. For every ticket retrieved, check for child tickets. Retrieve all children. Repeat until every branch returns zero children. Document the full tree structure before proceeding to any other category.

### Salesforce Metadata Rule
For every object or field identified in the Jira tree, traverse all rows in the CSV that reference metadata (Apex, Flows, CPQ, Validation Rules, Integrations, Security, Metadata, Dependencies). Follow every reference recursively — if a Flow calls a subflow, retrieve the subflow. If a trigger calls a handler, retrieve the handler and all its callouts.

## Completeness Gate

Before writing INVESTIGATION.md, verify all of the following are true:

- [ ] Jira ticket tree fully traversed to leaf level (zero children at every branch)
- [ ] All Salesforce objects referenced in tickets identified
- [ ] All mandatory CSV rows executed for each identified object
- [ ] All Apex call chains followed to terminal methods
- [ ] All Flow subflow references resolved
- [ ] All CPQ rule dependencies traced
- [ ] All integration mappings (inbound + outbound) documented
- [ ] All Profile/Permission Set access documented for in-scope fields

Present this checklist to `{user_name}` and ask for explicit confirmation before writing output.

## Output

Write to `{planning_artifacts}/INVESTIGATION.md`.

Structure:
Investigation: {scope summary} — {date}
Jira Ticket Tree
[Full hierarchy to leaf level]
Salesforce Object Inventory
[All objects and fields in scope]
Metadata Dependency Matrix
[For each object: Apex / Flows / CPQ / Validation Rules / Integrations / Security / Other]
Integration Touchpoints
[All inbound and outbound mappings]
Open Questions
[Anything that could not be confirmed — gaps in access, ambiguous references]
Coverage Summary
[Checklist items completed / total, with notes on any skipped mandatory items]

**No analysis. No recommendations. Facts and references only.**

After writing, inform `{user_name}` that INVESTIGATION.md is ready and that 
Party Mode will automatically load it as the shared evidence base.
