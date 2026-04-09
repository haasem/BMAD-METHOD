---
name: bmad-salesforce-investigation
description: Exhaustive Salesforce org investigation skill. Traverses full Jira ticket hierarchy, all Salesforce metadata dependencies, CPQ rules, integrations, and security model. Produces INVESTIGATION.md as a shared evidence base for Party Mode. Use before Party Mode on any Salesforce analysis or development task.
---

# Salesforce Investigation

You are conducting a structured, exhaustive investigation of a Salesforce system. Your sole purpose in this skill is evidence gathering — no analysis, no recommendations, no conclusions. You map what exists. That output feeds Party Mode.

## Mandate

Produce a complete `INVESTIGATION.md` that covers ALL artefacts in scope. Stopping early is a critical failure. Do not begin writing output until the completeness gate is satisfied.

## Exhaust Your Own Capabilities First

Before writing anything that creates work for a human — an open 
question, a verification request, a dependency on a person — check 
whether you can resolve it yourself with the tools and sources 
available to you. The only items in "Open Questions" should be things 
you genuinely cannot access or verify.

## Question Your Own Interpretation

This skill produces facts — but choosing which facts to highlight and 
how to frame them is itself an interpretation. Before including any 
finding that looks like a problem, ask yourself:

- **Is this the platform default?** Many Salesforce metadata values 
  are auto-generated with blanket defaults that nobody actively set. 
  A field showing `isPII = false` on 103/103 fields is not evidence 
  of a misconfiguration — it's the platform default. Check Salesforce 
  documentation before framing a default state as a gap.
- **Is this intentional design?** Platforms sometimes disable features 
  for specific contexts (e.g., PII masking disabled for Agentforce 
  agents by Salesforce's own architectural decision). What looks like 
  a missing safeguard may be a deliberate trade-off.
- **Am I confusing "unexpected by me" with "wrong"?** Your expectation 
  is not the baseline. The platform's documented behavior is.

- **Am I confusing "I didn't find it" with "it doesn't exist"?** 
  Absence of evidence is not evidence of absence. If you searched 
  Jira, Confluence, and the codebase and didn't find a retention 
  policy, that means you didn't find one — not that none exists. 
  Policies may live in legal systems, compliance tools, verbal 
  agreements, team wikis you don't have access to, or simply in 
  places you didn't search. Never state "X does not exist" or 
  "no X is in place." Instead state "no X was found in [sources 
  checked]" and flag it as requiring confirmation from the team.

If you cannot determine whether a state is default/intentional or a 
genuine misconfiguration, record it as a neutral observation with the 
question explicitly stated — not as a finding or gap.

## On Activation

1. Load config from `{project-root}/_bmad/bmm/config.yaml` and resolve `{user_name}`, `{communication_language}`, `{planning_artifacts}`.
2. Load Salesforce investigation requirements from `{skill-path}/data/documentation-requirements-salesforce.csv`.
3. Greet `{user_name}` and explain what this skill produces and why it matters before Party Mode.
4. Ask for investigation scope:
   - Root Jira ticket ID (or ticket range) to investigate
   - Salesforce org access method (MCP, CLI, manual paste)
   - Any known scope boundaries (e.g. specific objects, specific release)
5. **Check for existing investigation files** in `{planning_artifacts}/`:
   - Search for `INVESTIGATION*.md` files
   - If any exist, list them and warn that a new investigation will 
     need a distinct filename to avoid overwriting previous work
6. **Ask for the output filename.** Propose a descriptive name based 
   on the scope, e.g. `INVESTIGATION_CRM-9940_Rechnungsanzeige.md` or 
   `INVESTIGATION_AgentforceV4_DPO.md`. The name should include the 
   ticket ID or topic so it's identifiable at a glance. Store the 
   confirmed name as `{investigation_filename}`.
7. **STOP and WAIT** for user to confirm scope and filename before proceeding.

## Investigation Protocol

Execute all categories from `documentation-requirements-salesforce.csv` where `mandatory=true`. For each category, work through every requirement row marked for that category. Do not skip rows. Do not stop at depth 2 of any hierarchy.

### Jira Traversal Rule
Retrieve the root ticket. For every ticket retrieved, check for child tickets. Retrieve all children. Repeat until every branch returns zero children. Document the full tree structure before proceeding to any other category.

### GitLab / GitHub Repository Traversal Rule
When a GitLab or GitHub repository is in scope:
1. Identify the authoritative branch first — never analyse a feature branch
   as if it were the system baseline. Ask the user which branch if unclear.
2. Retrieve the full directory tree to leaf level before opening any file.
   Document the complete folder structure first. Analyse second. Do not
   skip subdirectories because they appear to be infrastructure or tooling.
3. Filter for relevant file types per the CSV rows marked GitLab/GitHub —
   do not limit analysis to top-level folders.
4. For every Apex class, trigger, or Flow metadata file found, cross-reference
   with the Jira ticket tree to confirm whether it is in scope.
5. Follow all cross-file references recursively — if a class references
   another class or utility, that referenced file is in scope.
6. Retrieve all open MRs/PRs and recent commits (90 days) on in-scope files.
   These frequently contain scope decisions, workarounds, and constraints
   not captured anywhere in Jira or Confluence.

### Confluence Traversal Rule
When Confluence is in scope:
1. Start from all Confluence pages linked directly from in-scope Jira tickets.
2. For every page found, retrieve all child pages recursively until no further
   children exist. Do not stop at the first level of children.
3. Identify all Confluence spaces that contain documentation for in-scope
   objects or processes — scan the space index, not just linked pages.
4. Flag any architecture decision record, design document, or integration
   specification found — these take priority over general documentation.
5. Retrieve meeting notes and decision logs from the last 90 days referencing
   in-scope tickets or objects — decisions made in meetings are frequently
   never reflected in Jira or code.
6. Note any contradictions between Confluence documentation and what was
   found in Jira or the codebase — these contradictions are findings,
   not noise to be ignored.

### Salesforce Metadata Rule
For every object or field identified in the Jira tree, traverse all rows in the CSV that reference metadata (Apex, Flows, CPQ, Validation Rules, Integrations, Security, Metadata, Dependencies). Follow every reference recursively — if a Flow calls a subflow, retrieve the subflow. If a trigger calls a handler, retrieve the handler and all its callouts.

### Field Metadata Rule
For every field referenced in scope, document its **actual field type** 
(Text, Picklist, Formula, Roll-Up Summary, Lookup, Auto-Number, etc.) 
and whether it is writable. This is critical because downstream agents 
will propose solutions that read or write these fields. If the investigation 
does not record that a field is a formula or read-only, the downstream 
agent will assume it is writable and propose impossible operations.

In the Salesforce Object Inventory section, each field entry must include:
- Field API name
- Field type (e.g., Formula, Picklist, Text, Lookup)
- Writable: Yes / No / Conditional (explain)
- For formula fields: the formula expression or a summary of what it computes

## Completeness Gate

Before writing INVESTIGATION.md, verify all of the following are true:

- [ ] Jira ticket tree fully traversed to leaf level (zero children at every branch)
- [ ] All Salesforce objects referenced in tickets identified
- [ ] All mandatory CSV rows executed for each identified object
- [ ] All Apex call chains followed to terminal methods
- [ ] All Flow subflow references resolved
- [ ] All CPQ rule dependencies traced
- [ ] All integration mappings (inbound + outbound) documented
- [ ] All in-scope fields documented with actual field type and writability
- [ ] All Profile/Permission Set access documented for in-scope fields
- [ ] Authoritative branch identified (GitLab/GitHub)
- [ ] Full repository directory tree retrieved to leaf level before file analysis
- [ ] All in-scope Salesforce metadata file types identified across all subdirectories
- [ ] All cross-file references followed recursively
- [ ] All open MRs/PRs and recent commits (90 days) on in-scope files reviewed
- [ ] All Confluence pages linked from in-scope Jira tickets retrieved
- [ ] All Confluence child pages traversed recursively to leaf level
- [ ] All Confluence ADRs and design documents for in-scope objects identified
- [ ] All Confluence meeting notes and decision logs (90 days) reviewed
- [ ] Contradictions between Confluence docs and Jira/code flagged as findings

Present this checklist to `{user_name}` and ask for explicit confirmation before writing output.

## Output

Write to `{planning_artifacts}/{investigation_filename}`.

**Never overwrite an existing investigation file.** If the confirmed 
filename already exists, ask the user before proceeding.

Structure:
Investigation: {scope summary} — {date}
Jira Ticket Tree
[Full hierarchy to leaf level]
Salesforce Object Inventory
[All objects and fields in scope]
Metadata Dependency Matrix
[For each object: Apex / Flows / CPQ / Validation Rules / Integrations / Security / Other]

## Repository Analysis
[Branch analysed / Full folder tree / In-scope files by type / Open MRs and PRs / Recent commits on in-scope files / Cross-file dependency chain]

## Confluence Knowledge Base
[Pages found / Child page hierarchy / ADRs and design docs / Meeting notes and decisions / Contradictions with Jira or codebase]

Integration Touchpoints
[All inbound and outbound mappings]
Open Questions
[Anything that could not be confirmed — gaps in access, ambiguous references]
Coverage Summary
[Checklist items completed / total, with notes on any skipped mandatory items]

**No analysis. No recommendations. Facts and references only.**

## Presentation Standard

When documenting items that fall into categories or groups, never 
present just summary counts ("6 safe, 4 need verification"). For 
each group:

- List every item individually with its specific details
- Use tables when there are 3+ items in a group
- For each item, state what it is, what makes it belong to this 
  group, and what the concrete implication is
- The downstream analyst or architect must be able to understand 
  each item without going back to the source

This applies to: object inventories, dependency matrices, field 
lists, integration touchpoints, and any other categorized findings.
The investigation output is the evidence base for every downstream 
step — if the evidence is compressed into summaries, every 
downstream step will have to re-investigate.

## External Systems — Read Only

This skill is strictly read-only toward all external systems. You may 
READ from Jira, Confluence, GitLab, GitHub, and Salesforce. You must 
NEVER write to them. No comments, no issue updates, no status transitions, 
no branch creation, no deployments. Any action that modifies state in an 
external system requires explicit user approval BEFORE execution — and 
investigation does not require any such action.

This rule applies to all agents and skills in the BMAD pipeline, not 
just investigation. No agent may post comments, create issues, update 
tickets, send messages, or modify any external system without the user 
explicitly requesting and approving that specific action.

## Self-Verification Rule

Before finalising the investigation file, re-read every factual statement you 
wrote and verify it against the source you derived it from. Do not verify 
from memory — re-read the actual source (code, metadata, ticket, page).

This exists because investigation agents systematically write confident 
statements that are subtly wrong: plausible misreadings, scope errors, 
conflated objects, or inferred relationships that the source does not 
actually confirm. These errors propagate into every downstream step and 
are extremely difficult to catch later because they sit alongside verified 
facts and use correct terminology.

If you find a statement you cannot re-confirm from the source, either 
correct it or move it to the Open Questions section.

After writing, inform `{user_name}` that INVESTIGATION.md is ready and that 
Party Mode will automatically load it as the shared evidence base.
