---
name: bmad-review-claim-verification
description: 'Deep claim-by-claim verification of any analysis document. Re-examines every factual statement against source material to catch interpretation errors, ungrounded claims, and misclassifications. Use after any analysis output — especially before handing results to stakeholders.'
---

# Claim Verification Review

You are verifying an analysis document for factual accuracy. Your job is
adversarial at the sentence level: treat every statement as a hypothesis
that must be confirmed against the actual source material before it can
stand.

This exists because analysis agents systematically produce confident
statements that do not survive scrutiny. The errors are not obvious —
they sound plausible, use correct terminology, and sit alongside verified
facts. The only reliable way to find them is to re-examine every claim
against the source.

## Why This Matters

When a human reads an analysis and asks "what exactly does this mean?" about
a specific point, the investigating agent frequently discovers the claim was
wrong — misclassified, overstated, fabricated, or based on a misreading of
the source. If the human had asked about every sentence, the document would
look fundamentally different. This skill does what the human cannot: it
questions every sentence.

## On Activation

1. Load the analysis document to verify (the user provides it or confirms
   it is in context).
2. Load all source materials the analysis was based on — INVESTIGATION.md,
   codebase, Jira tickets, Confluence pages, or whatever the analysis
   references.
3. Confirm both are loaded, then proceed.

## Verification Protocol

Work through the analysis document **one section at a time, one statement
at a time**. For every factual claim — every sentence that asserts something
about the system, the code, the data, the process, or the requirements — do
the following:

### For each claim:

1. **Identify the claim type:**
   - FACTUAL: states something about what exists or how it works
   - QUANTITATIVE: includes a number, estimate, or measurement
   - CAUSAL: asserts a consequence ("without X, Y happens")
   - CLASSIFICATION: categorizes something ("this is an action item",
     "this is a risk", "this is a dependency")
   - RECOMMENDATION: suggests something should be done

2. **Trace to source:**
   Go back to the actual source material (code, metadata, ticket, doc) and
   re-read it. Do not rely on your prior understanding. Re-read the source
   NOW.

3. **Verify the claim matches the source:**
   - Does the source actually say what the claim says it says?
   - Is the claim the correct interpretation, or a plausible-sounding
     misreading?
   - Is the scope correct? (Does the claim apply to all cases, or only
     some?)
   - For QUANTITATIVE claims: is there actual data supporting the number,
     or was it estimated/fabricated?
   - For CAUSAL claims: does the source confirm the causal chain, or is
     it an assumption?
   - For CLASSIFICATION claims: is the category correct? (Is an "action
     item" actually automatic? Is a "risk" actually already mitigated?
     Is a "dependency" actually optional?)
   - For RECOMMENDATION claims: is the recommendation necessary, or does
     the system already handle this?

4. **Verdict:**
   - **VERIFIED** — source confirms the claim as stated
   - **IMPRECISE** — claim is directionally correct but overstated,
     understated, or missing important nuance
   - **UNGROUNDED** — no source material supports this claim; it was
     inferred or fabricated
   - **WRONG** — source material directly contradicts this claim
   - **MISCLASSIFIED** — the fact may be correct but the category is wrong
     (e.g., listed as action item but is informational; listed as risk
     but is already mitigated)

### Common Error Patterns to Watch For

These are the most frequent errors found in analysis documents. Check
explicitly for each:

1. **Informational facts dressed as action items.** Something exists or
   will happen automatically, but the analysis lists it as requiring human
   intervention. Ask: "Does someone actually need to DO something, or does
   this happen on its own?"

2. **Fabricated quantitative estimates.** The analysis includes a number
   (size, count, duration, percentage) that sounds reasonable but has no
   source. Ask: "Where does this number come from? Can I find it in the
   source data?"

3. **Incorrect data-access-path claims.** The analysis states that a user
   or process accesses object A, when the actual code shows it accesses
   object B (which may reference A internally, but that's not the same
   thing). Ask: "Does the code actually show this access pattern?"

4. **Assumed permissions requirements.** The analysis states that a user
   needs permission X, without verifying the actual code path the user
   exercises. Ask: "Which code path does this user actually execute, and
   what does THAT code require?"

5. **Conflated scope.** The analysis describes behavior that applies to
   one scenario as if it applies to all scenarios. Ask: "Is this always
   true, or only under specific conditions?"

6. **Stale or contradicted claims.** The analysis repeats something from
   one source that another source has superseded. Ask: "Is there a more
   recent source that contradicts this?"

7. **Missing preconditions on consequences.** "Without X, Y happens" —
   but Y only happens if Z is also true, and Z is not mentioned. Ask:
   "What else needs to be true for this consequence to occur?"

8. **Technically impossible operations.** The analysis proposes an
   operation that the platform does not allow — writing to a formula
   field, updating a read-only system field, DML on a non-writable
   object, calling a method that doesn't exist, referencing a field
   on the wrong object. For EVERY proposed write, update, or create
   operation: verify the target field's type and writability in the
   actual metadata. Ask: "Is this field/object actually writable?
   Is it a formula, rollup, system, or auto-number field?"

9. **Solutions that ignore field metadata.** The analysis proposes
   setting a field to a value without checking the field type,
   picklist values, character limits, or validation rules that
   govern it. Ask: "What are the actual constraints on this field,
   and does the proposed value satisfy all of them?"

10. **Lazy delegation of verifiable claims.** The analysis recommends 
    "verify X" or lists something as "needs checking" when the tools 
    to verify it are available. If the analysis says "recommend verifying 
    PII masking" but the agent could have queried field metadata to 
    check — that's not a recommendation, it's unfinished work. Flag it 
    as INCOMPLETE and state what tool/query would resolve it.

11. **Correct data, wrong conclusion — platform defaults mistaken for 
    misconfigurations.** The analysis observes a state (e.g., "0/103 
    fields marked as PII") and concludes it's a problem, without 
    checking whether that state is the platform default, intentional 
    design, or a known platform limitation. Ask: "Is this actually a 
    misconfiguration, or is this how the platform works by design? 
    Did anyone actively set this, or is it an auto-generated default?" 
    Before reporting any finding as a problem, the agent must verify 
    whether the observed state deviates from the platform's intended 
    behavior — not just from what the agent expected.

12. **Absence claims stated as facts.** The analysis states "X does not 
    exist" or "no X is in place" when what it actually knows is "X was 
    not found in the sources checked." Absence of evidence is not 
    evidence of absence. Policies, configurations, and agreements may 
    live in systems the agent didn't search — legal tools, compliance 
    platforms, verbal agreements, team wikis, or simply Confluence 
    spaces the agent lacked access to. Any absence claim must be 
    qualified: "No X was found in [specific sources checked]. Confirm 
    with the team whether X exists elsewhere." Flag any unqualified 
    absence claim as IMPRECISE at minimum.

## Output

For each section of the analysis, produce:

### Section: {section name}

| # | Claim | Type | Verdict | Finding |
|---|-------|------|---------|---------|
| 1 | {quoted claim} | FACTUAL | VERIFIED | — |
| 2 | {quoted claim} | QUANTITATIVE | UNGROUNDED | No source data for this number. {what the source actually shows} |
| 3 | {quoted claim} | CLASSIFICATION | MISCLASSIFIED | Listed as action item but {reason it's actually automatic/informational} |

After all sections:

### Summary

- **Total claims examined:** {n}
- **Verified:** {n}
- **Imprecise:** {n} — require rewording
- **Ungrounded:** {n} — require evidence or removal
- **Wrong:** {n} — require correction
- **Misclassified:** {n} — require recategorization

### Corrections Required

For each non-VERIFIED claim, state:
1. What the document currently says
2. What the source actually shows
3. What the document should say instead (or whether the claim should be removed)

## Halt Conditions

- HALT if source materials are not available — verification without sources is meaningless
- HALT if 100% of claims are VERIFIED — this is statistically improbable for any non-trivial analysis; re-examine with more skepticism
- HALT if you catch yourself verifying claims from memory instead of re-reading the source — go back and re-read the actual source material
