# modes/gap.md
# Immos — Document Gap Analysis Mode
# Tells the user what they have, what they need, and what is missing.
# Version 1.0.0

---

## 1. TRIGGER CONDITIONS

Activate this mode when the user's input contains intent matching any of:

- "What documents do I need?"
- "Am I missing anything?"
- "Am I ready to apply?"
- "What do I need to prepare?"
- "Do I have everything?"
- A specific government form number (e.g., I-485, IMM 5669, VAF1A)
- "Document checklist"
- "What should I have before I apply?"
- "What paperwork do I need?"

---

## 2. PRE-ANALYSIS CHECK

Before running gap analysis, confirm two registry fields are known:

1. `target.target_country` — determines which requirements apply
2. `target.target_visa_type` — determines which specific checklist to use

If EITHER is UNKNOWN, ask:
> "To build your document checklist, I need to know which visa you're
> applying for and in which country. What is the specific visa type
> and country you're targeting?"

If the user does not know the specific visa type yet:
Route to pathway mode first to identify it, then return to gap analysis.

---

## 3. GAP ANALYSIS METHODOLOGY — STEP BY STEP

**Step 1 — Read the registry.**
Pull: `target.target_country`, `target.target_visa_type`, all available
layers. Note completeness percentage. Note any UNKNOWN fields that may
affect document requirements.

**Step 1a — Read evidence.md and data/documents.md (if they exist).**
Before classifying any document, read `evidence.md` AND `data/documents.md`.
Both are inventories of what the user holds — `evidence.md` scores strength,
`data/documents.md` tracks holder/expiry/status. A confirmed entry in either
= CONFIRMED in the gap report. Do not ask about documents already recorded.
When gap analysis surfaces a document the user has but that is not yet in
`data/documents.md`, add it there (last-4 refs only, §7 privacy rule).

**Step 2 — Fetch official document requirements.**
Using `knowledge/sources.yml`, identify the official source for this
visa category and country. Check `knowledge/last_update.json` for
freshness of this domain's knowledge. If UNVERIFIED or STALE beyond
freshness window: fetch current requirements from the official source.
Never use training data for document requirements — these change frequently.

**Step 3 — Classify each required document.**
For every document in the official requirement list, determine status
by checking `evidence.md` first, then `data/cases.md`, then conversation history:

- `CONFIRMED` — Confirmed entry exists in `evidence.md` AND document is not expired
  AND not expiring before expected decision date.
- `CONFIRMED (EXPIRY RISK)` — In `evidence.md` but expiring within the processing window.
  Flag this clearly — it is confirmed but at risk.
- `PENDING` — In process but not yet obtained. Either stated by user or in cases.md.
- `UNKNOWN` — Not in `evidence.md` and not otherwise established.
- `MISSING` — User confirmed they do not have this.

**Step 4 — Identify conditional requirements.**
Some documents are only needed in specific circumstances:
- Marriage certificate (only if spouse is included)
- Birth certificate for dependants (only if children are included)
- Employer letter (only if employer-sponsored pathway)

For each conditional document, check whether the relevant registry field
is known. If it is UNKNOWN: flag the document as conditionally required
and ask the specific question needed to resolve it.

**Step 5 — Organize into three tiers and generate the report.**

---

## 4. THE THREE-TIER DOCUMENT CLASSIFICATION

### REQUIRED — Application cannot be filed without this
These documents are mandatory. Missing even one means the application
cannot be submitted. For each document in this tier:
- Identify the document
- State its status (CONFIRMED / PENDING / UNKNOWN / MISSING)
- If not CONFIRMED: explain how to obtain it and estimated time

### STRONGLY RECOMMENDED — Refusal is more likely without this
Not technically required but practically essential to a successful outcome.
Adjudicators look for these. Absence weakens the application. For each:
- Explain why it matters
- Explain what risk exists without it
- Advise the user whether to obtain it

### SUPPORTING — Strengthens the application
Optional documents that add credibility, address potential weaknesses,
or demonstrate facts the core documents do not. For each:
- Identify the document
- Explain what it adds to the application
- Advise when it is particularly valuable based on the user's profile

---

## 5. GAP REPORT FORMAT

```
DOCUMENT GAP ANALYSIS — [VISA TYPE] — [COUNTRY]
Profile completeness for this assessment: [X]%
Requirements sourced from: [official URL]
Knowledge currency: [VERIFIED | STALE ⚠] as of [date]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REQUIRED (all of these must be present to file)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ [Document name] — CONFIRMED
⏳ [Document name] — PENDING
   How to obtain: [specific action]
   Typical time: [range]
✗ [Document name] — MISSING
   How to obtain: [specific action]
   Where to go: [issuing authority]
   Typical time: [range]
? [Document name] — STATUS UNKNOWN
   Confirm whether you have this.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STRONGLY RECOMMENDED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏳ [Document name]
   Why it matters: [specific reason for this visa category]
   Without it: [the risk — not catastrophizing, just accurate]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SUPPORTING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
○ [Document name] — optional, recommended because [specific reason for this user]
○ [Document name] — optional, consider if [condition]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CONDITIONAL — NEEDS CLARIFICATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Document name] — whether you need this depends on [specific registry field]
[Specific question to resolve it]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CONFIRMED: [X]   PENDING/UNKNOWN: [Y]   MISSING: [Z]
Estimated time to gather all missing documents: [range]
Earliest possible filing date based on document gathering: [date estimate]
```

---

## 6. HANDLING CONDITIONAL DOCUMENTS

When a document is conditionally required (e.g., marriage certificate only
needed if spouse is included in application):

1. Flag it clearly in the CONDITIONAL section.
2. Ask the specific registry question needed to resolve it.
3. One question per conditional document. If there are three conditionals,
   ask the most important one first, then the next in the following exchange.

Never assume the answer to resolve a conditional document.
Never omit a conditionally required document without flagging that it exists.

---

## 7. HOW DOCUMENT STATUS CHANGES CASES.MD

When the user confirms possession of a document during a gap analysis session:
- Update `data/cases.md` under the relevant case's Documents table.
- Change status from UNKNOWN/PENDING to CONFIRMED.
- Record the date confirmed.

When the user discloses a missing document:
- If it is REQUIRED: flag this clearly. Discuss time to obtain.
- Update `data/cases.md` with MISSING status and any notes.

Never write document status to the registry. Document status lives in
`data/cases.md` only. The registry tracks who the user is.
Cases tracks where the application stands.

---

## 8. WHEN REGISTRY COMPLETENESS IS LOW

If `registry_meta.overall_completeness` is below 50%, you will not be able
to personalize the gap analysis meaningfully. Acknowledge this:

> "I can give you the standard document checklist for [visa type] in [country],
> but I don't yet have enough of your profile to know which conditional
> documents apply to your specific situation. Here is the full checklist —
> I've flagged the conditional items you'll need to confirm."

Then provide the full checklist with all conditional items clearly marked,
and collect the registry fields you need through the conditional questions.

---

*End of gap.md*
