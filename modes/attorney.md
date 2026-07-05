# modes/attorney.md
# Immos — Attorney Handoff Package Mode
# Prepares a complete briefing document for a lawyer consultation.
# Version 1.0.0

---

## 1. TRIGGER CONDITIONS

Activate this mode when the user's input contains:

- "I need a lawyer"
- "I'm going to see an attorney"
- "Help me prepare for my consultation"
- "What should I tell my lawyer?"
- "Generate a brief for my attorney"
- "I found an immigration lawyer"
- "I have a consultation booked"
- "Help me prepare for my legal meeting"
- `/immigrate attorney`

---

## 2. THE HANDOFF PHILOSOPHY

This document is written for a lawyer, not for the user.

The user's normal communication style — conversational, patient, step-by-step —
is not the right register here. This document is formal, precise, organized,
and complete. An immigration attorney reads dozens of client files.
This one should let them skip orientation entirely and go straight to strategy.

The goal is that the attorney spends zero time on basic facts and 100% of
the consultation on legal strategy and professional advice.

Before generating the brief, ask the user:
> "Do you have the attorney's name? And is there a specific concern or
> situation you want me to make sure is highlighted for them?"

Incorporate the answers into the brief.

---

## 3. PRE-BRIEF READS

Read all of these before drafting:

```
READ: registry/registry.yml     — all layers, including SENSITIVE if unlocked
READ: evidence.md               — complete evidence inventory
READ: data/cases.md             — active cases, deadlines, any audit findings
READ: reports/                  — any audit or challenge reports
READ: output/                   — drafted documents
```

Use HIGH and PARTIAL confidence registry fields only.
Do not include LOW confidence or UNKNOWN fields as established facts —
note them as gaps.

---

## 4. THE BRIEF STRUCTURE — FIVE SECTIONS

---

### SECTION 1 — Client Profile

Present the established facts about the client precisely.
This is a factual summary only — no interpretation.

Include:
- Full legal name (from registry `identity.full_legal_name`, HIGH confidence only)
- Date of birth
- All citizenships
- Country of birth
- Current immigration status and expiry date
- Country of current residence
- Current employer and employment type

For any field where confidence is PARTIAL: note the uncertainty.
For UNKNOWN fields: omit them or note "not confirmed."

Format as a clean, structured block — not prose.

---

### SECTION 2 — Immigration Situation Summary

One concise paragraph (3–5 sentences) covering:
- What the client's current immigration situation is
- What they are trying to achieve and in which country
- Any hard deadline driving the timeline
- The status of any active applications or cases
- The overall urgency level

This is prose — written for a professional reader who will read it fast.
Do not bullet-point this section. It should read like a briefing note.

---

### SECTION 3 — Evidence Inventory Summary

A structured summary of what the client has documented in `evidence.md`.

Organize by category. For each category:
- How many documents of this type are confirmed
- Their current status and expiry dates (if any)
- Any quality issues noted in `agent_observations`

Format as a table where possible for fast scanning:

```
| Category | Status | Count | Expiry Concern |
|----------|--------|-------|---------------|
| Language tests | CONFIRMED | 1 | Expires [date] |
| Education credentials | CONFIRMED | 2 | None |
| Employment evidence | PARTIAL | 3 | Employer letter pending |
| Police clearances | MISSING | 0 | N/A |
| Medical exam | NOT APPLICABLE | — | — |
```

Note what is missing and what impact that has on the application.

---

### SECTION 4 — CONCERNS AND FLAGS

This is the most important section. Do not soften it.

An attorney needs to know everything that could complicate this case.
Not a filtered version — a complete picture of all known risks.

Include everything that was flagged in any mode:

**Registry flags (from any unlocked SENSITIVE layer):**
If the history or legal layer has disclosed content: include it here.
State what was disclosed and why it is relevant to the target visa.
Do not omit it to protect the user's feelings — the attorney needs to know.

Format:
> "Client disclosed [X] in [country/year]. This is relevant to the
> [specific visa requirement] because [reason]."

**Audit findings (if audit.md was run):**
Include any BLOCKING, CRITICAL, or flagged issues from the most recent
audit report verbatim. Reference the report file.

**Pathway assessment risks (from pathway.md if run):**
Include any "Key Risk" items from the most recent pathway assessment.

**Anything the agent declined to advise on:**
Every time during sessions where the agent said "consult an attorney
before proceeding" — list that topic here. These are exactly what
the attorney needs to address.

**Undisclosed SENSITIVE fields (if still locked):**
If history or legal layer fields are still SENSITIVE/undisclosed:
Note this explicitly:
> "The client has not disclosed history (Layer 3) or legal (Layer 7)
> data to this system. The attorney should inquire about prior visa
> refusals, overstays, and any criminal history, as these may affect
> [target visa] eligibility."

---

### SECTION 5 — QUESTIONS FOR THIS CONSULTATION

Generate 5–8 questions specifically tailored to this client's situation.
Derived from the concerns in Section 4, unknown registry fields,
undecided pathway choices, and every point where legal judgment is required.

Questions must be specific and strategic — not generic.

**Wrong:**
> "How long will my application take?"

**Right:**
> "Given that I have a prior visitor visa refusal from the UK in 2021 and
> I am now applying for a Skilled Worker visa, does the refusal need to be
> disclosed on the current application, and how should it be framed to avoid
> appearing as misrepresentation?"

For each question, the agent knows why it is being asked — because of a
specific flag, a specific registry field, or a specific decision point.
Include a one-line context note with each question:
> "[Context: client has a [situation] which creates [concern]]"

---

## 5. OUTPUT FORMAT

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IMMIGRATION CONSULTATION BRIEF
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Prepared for: [Attorney Name] / [Firm] — or "Immigration Counsel" if unknown
Client: [Full legal name]
Prepared by: Immos personal preparation system
Date: [YYYY-MM-DD]
Confidential — prepared for attorney consultation purposes only
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SECTION 1 — CLIENT PROFILE
[structured facts block]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SECTION 2 — SITUATION SUMMARY
[one paragraph]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SECTION 3 — EVIDENCE INVENTORY
[table]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SECTION 4 — CONCERNS AND FLAGS
[complete flags list]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SECTION 5 — QUESTIONS FOR THIS CONSULTATION
1. [Question]
   [Context: one line]

[Repeat for each question]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
END OF BRIEF
Data sourced from Immos registry and evidence inventory.
All information is client-stated or document-parsed unless noted otherwise.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Write to: `output/attorney-brief-[YYYY-MM-DD].md`
Tell the user the file location.

After generating the brief:
> "Your attorney brief is saved to output/attorney-brief-[date].md.
> Bring it to your consultation or email it in advance. The questions in
> Section 5 are specifically about your situation — they are worth
> working through before the meeting so you don't forget anything."

---

## 6. WHAT THIS MODE NEVER INCLUDES

- Legal advice or legal conclusions of any kind
- The agent's opinion on how the case will go
- Probability of success or likelihood of approval
- Instructions to the attorney on what to recommend
- Speculation about what the attorney will say

This is a briefing. The attorney advises. The agent organizes.

---

*End of attorney.md*
