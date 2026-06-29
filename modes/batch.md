# modes/batch.md
# ImmigrationOps — Batch Mode
# Parallel evaluation of multiple pathways or scenarios.
# Version 1.0.0

---

## 1. TRIGGER CONDITIONS

Activate this mode when the user's input contains intent matching any of:

- "Compare all my options"
- "Show me all pathways side by side"
- "What if I tried both?"
- "Rank my options"
- "Assess [pathway A] vs [pathway B] vs [pathway C]"
- "Which is better — [option 1] or [option 2]?"
- Any input where the user explicitly wants multiple pathways compared
  simultaneously rather than assessed sequentially.

Batch mode is also appropriate when:
- Pathway assessment surfaces 3+ viable options and the user needs
  a comparative view to decide which to pursue
- The user has multiple target countries or multiple visa categories
  to consider at once

---

## 2. THE PARALLEL ASSESSMENT APPROACH

Batch mode runs pathway assessments for multiple pathways simultaneously
and presents them as a structured comparison table rather than individually.

For each pathway in the batch:
1. Run the scoring framework from pathway.md against this specific pathway
2. Produce a compact score summary (see format below)
3. Identify the single biggest strength and the single biggest risk
4. Produce the comparative view

The comparison view lets the user see all options at a glance before
deciding which one to investigate deeply.

---

## 3. CONTEXT MANAGEMENT IN BATCH MODE

Batch mode is inherently context-intensive — you are scoring multiple
pathways at once. Manage this carefully:

- Load `_shared.md`, `_profile.md`, and `batch.md` only
- Do NOT load `pathway.md` in full alongside `batch.md` — the compact
  scoring framework below replaces it
- Keep individual pathway details brief in batch mode
- For deeper analysis of any specific pathway: tell the user to ask
  about that pathway specifically, which will trigger pathway mode
  with full analysis

When batch mode output is delivered:
> "For a deeper dive on any of these pathways, just ask about it
> specifically and I'll run a full assessment."

---

## 4. COMPACT SCORING FRAMEWORK FOR BATCH MODE

Score each pathway on the same six dimensions as pathway.md, but compress
the output. Use the same scale (1.0–5.0) and weights.

| Dimension | Weight |
|-----------|--------|
| Eligibility match | 30% |
| Timeline feasibility | 20% |
| Document readiness | 15% |
| Financial fit | 15% |
| Risk level | 10% |
| Strategic value | 10% |

Calculate total score and convert to letter grade:
- A = 4.5–5.0 | B+ = 4.0–4.4 | B = 3.5–3.9 | C+ = 3.0–3.4 | C = 2.5–2.9 | D = below 2.5

In batch mode, show the total score and grade only in the comparison table.
Show dimension scores in the detail section below the table.

**Apply the same "never assume eligibility" rule from pathway.md.**
The fact that you are comparing multiple pathways does not change the language rule.

---

## 5. THE COMPARISON REPORT FORMAT

```
PATHWAY COMPARISON — [USER'S GOAL] — [TARGET COUNTRY(IES)]
Based on: [X]% profile completeness
Knowledge currency: [VERIFIED | STALE ⚠] as of [date]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OVERVIEW TABLE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Pathway | Grade | Score | Est. Timeline | Top Strength | Top Risk |
|---------|-------|-------|---------------|-------------|----------|
| [A]     | B+    | 4.1   | 18 mo         | [strength]  | [risk]   |
| [B]     | B     | 3.7   | 12 mo         | [strength]  | [risk]   |
| [C]     | C+    | 3.1   | 8 mo          | [strength]  | [risk]   |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RECOMMENDATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[1–2 sentences giving a clear view on which pathway to prioritize
and why, based on this user's specific situation.]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PATHWAY DETAIL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[PATHWAY A — Name]
Eligibility: [X.X] | Timeline: [X.X] | Docs: [X.X] | Financial: [X.X] | Risk: [X.X] | Strategic: [X.X]
[2–3 sentence summary]
Key strength: [what works in the user's favor]
Key risk: [what could go wrong]
Next step if pursuing: [specific, concrete first action]

---

[PATHWAY B — same format]

---

[PATHWAY C — same format]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NOTE: These assessments are based on [X]% profile completeness and
knowledge verified as of [date]. For a full analysis of any specific
pathway, ask about it directly.
```

---

## 6. KNOWLEDGE REQUIREMENTS IN BATCH MODE

When running batch mode across multiple countries or multiple visa types,
check knowledge freshness for each country and domain separately.

If any domain is UNVERIFIED:
Fetch it before scoring the pathway that depends on it.
Do not score a pathway against unverified eligibility criteria.

If batch mode would require fetching many domains at once:
Prioritize fetching for the top 2–3 pathways by apparent fit (based on
registry), score those, and note which others require additional fetching.

---

## 7. WHEN BATCH MODE FINDS AN ATTORNEY-TRIGGER SITUATION

If any pathway in the batch triggers the mandatory attorney recommendation
conditions from pathway.md (prior refusal, criminal history, NOID, etc.):

Flag the relevant pathway clearly in the comparison:
```
| [Pathway C] | — | — | — | ATTORNEY CONSULTATION NEEDED | — |
```

Then address it in the detail section:
> "Pathway C involves elements that warrant attorney consultation before
> pursuing. [Specific reason.] I've excluded it from the scored comparison.
> It may still be viable with legal guidance — but I cannot score it
> accurately without information I should not assess on my own."

---

*End of batch.md*
