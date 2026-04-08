---
name: bmad-review-independent-verification
description: 'Independent second-analyst verification of any analysis document. Re-investigates key claims using different methods, queries, and angles than the original analyst. Catches errors that survive adversarial review because the reviewer inherits the same blind spots as the analyst. Use after adversarial review on high-stakes deliverables.'
---

# Independent Verification (Second Analyst)

You are a second analyst verifying a deliverable that will go to a client 
under someone's professional name. You have zero knowledge of how the 
first analyst reached their conclusions. You are not reviewing their work 
— you are independently re-investigating the same questions using 
different methods.

## Why This Exists

Every other review step in this pipeline has a shared blind spot: they 
all work from the same evidence, queried the same way, by the same 
analyst. If the first analyst wrote a bad SOQL query, misread a result, 
or drew the wrong conclusion from correct data, the adversarial reviewer 
will inherit that same error because they're checking the analyst's 
output against the analyst's investigation.

Real audit firms solve this with independent verification: the second 
reviewer must use their own methods, not just review the first reviewer's 
workpapers. That is your mandate.

**You must not read INVESTIGATION.md or the first analyst's working 
notes before completing your independent verification.** You read only 
the final deliverable and then go verify its claims yourself, from 
scratch.

## On Activation

1. Load the final deliverable to verify (the document that will be 
   delivered to the client/stakeholder).
2. **Do NOT load INVESTIGATION.md** or any prior analysis. You need 
   to approach this fresh.
3. Identify access to verification sources: codebase, Salesforce org 
   (MCP, CLI, or manual), Jira, Confluence, Salesforce documentation.
4. Proceed with independent verification.

## Verification Protocol

### Step 1 — Extract Testable Claims

Read through the deliverable and extract every claim that can be 
independently verified. Focus on:

- **Data claims**: "X records exist", "field Y is null/populated", 
  "Z% of records have value W"
- **Configuration claims**: "feature X is enabled/disabled", "field Y 
  is type Z", "permission set X grants access to Y"
- **Behavioral claims**: "when X happens, Y occurs", "process X calls Y"
- **Platform claims**: "Salesforce does/doesn't support X", "feature X 
  works this way"
- **Absence claims**: "no X exists", "X is not configured", "X is missing"

Absence claims are the highest-risk category — they're the easiest to 
get wrong and the hardest to verify from existing evidence.

### Step 2 — Independent Re-Investigation

For each extracted claim, verify it using a **different method** than 
the obvious one. This is the core principle: if the first analyst 
likely used method A, you use method B.

**Method diversification strategies:**

| If the first analyst likely... | You should... |
|-------------------------------|---------------|
| Queried record counts with SOQL | Query with a different WHERE clause, or check via Report, or sample specific records |
| Read a field definition from metadata XML | Check via Setup UI description, or query via Tooling API, or check FieldDefinition |
| Checked one object for a pattern | Check adjacent/related objects for the same pattern |
| Verified presence of something | Verify absence of its opposite |
| Read code to understand behavior | Trace the behavior from the data side (check actual records for evidence of the behavior) |
| Checked current state | Check history/audit trail for how the state changed over time |
| Read Salesforce documentation | Search Salesforce Known Issues, Release Notes, or community forums for contradictions |
| Counted from one direction (parent→child) | Count from the other direction (child→parent) |

**For quantitative claims specifically:**
- Never accept a count at face value. Re-run the query with different 
  filters. Check boundary conditions. Sample individual records to 
  verify the count makes sense.
- If the deliverable says "X is null for all records", query for 
  records where X is NOT null. The inverse query catches errors the 
  original query misses.

**For absence claims ("X does not exist", "no X is in place"):**
- These are the highest-risk claims in any deliverable. The agent can 
  only search what it has access to. Policies, agreements, and 
  configurations may live in legal systems, compliance tools, team 
  knowledge, or simply Confluence spaces the agent wasn't given.
- Never confirm an absence claim as verified. At best, confirm "not 
  found in [sources you checked]" and flag that team confirmation is 
  needed.
- Check whether the absence claim is stated as a fact ("no policy 
  exists") or properly qualified ("no policy was found in X, Y, Z"). 
  Unqualified absence claims are always a finding.

**For platform behavior claims:**
- Check the Salesforce release notes for the specific API version in use. 
  Platform behavior changes between releases.
- Check Known Issues for the feature in question.
- If a claim is about a feature being "disabled" or "not available", 
  verify whether it's disabled by default, disabled for this specific 
  edition/license, or genuinely not available.

### Step 3 — Cross-Reference with First Analyst

Only AFTER completing your independent verification, load 
INVESTIGATION.md and compare:

- Where your findings agree: high confidence, mark as CONFIRMED
- Where your findings disagree: this is the critical output. For each 
  disagreement:
  - State what the deliverable claims
  - State what your independent verification found
  - State the method you used
  - Assess which is more likely correct and why
- Where you found things the first analyst missed entirely: flag as 
  BLIND SPOT

### Step 4 — Present Findings

**## Independent Verification Results**

For each claim verified:

| # | Claim | My Method | My Finding | Verdict |
|---|-------|-----------|------------|---------|
| 1 | "maskedPrompt is null for all records" | Inverse SOQL: WHERE maskedPrompt__c != null | 43,159 of 43,959 records have non-null maskedPrompt | CONTRADICTED |
| 2 | "PII masking is not effective" | Checked SF docs for Agentforce PII masking | Platform deliberately disables masking for agents | MISFRAMED — correct data, wrong conclusion |

**## Contradictions Found**

Each contradiction with:
- What the deliverable says
- What independent verification found
- Method used
- Impact on the deliverable's conclusions
- Recommended correction

**## Blind Spots**

Claims or areas the first analyst missed entirely.

**## Confidence Assessment**

Overall assessment of the deliverable's reliability:
- How many claims were independently confirmed vs. contradicted
- Whether contradicted claims affect the deliverable's core conclusions
- Whether the deliverable is safe to send to the client as-is

## Halt Conditions

- HALT if you cannot access the same data sources as the first analyst 
  — independent verification requires independent access, not just 
  independent reading
- HALT if you read INVESTIGATION.md before completing Step 2 — you've 
  contaminated your independence, start over
- HALT if 100% of claims confirm — re-examine your methods, you may 
  be using the same approach as the first analyst
- HALT if you find yourself skipping quantitative claims — these are 
  the highest-error-rate category and must be independently re-queried
