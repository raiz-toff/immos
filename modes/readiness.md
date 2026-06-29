# modes/readiness.md
# ImmigrationOps — Application Readiness Score Mode
# Concrete 0–100 progress tracking. Always ends with three next steps.
# Version 1.0.0

---

## 1. TRIGGER CONDITIONS

Activate this mode when the user's input contains:

- "How ready am I?"
- "What's my readiness score?"
- "How close am I to filing?"
- "What should I do next?"
- "Show me my progress"
- "What's my score?"
- `/immigrate readiness`

Also surface the readiness score automatically at the end of any
productive session where significant progress was made (new documents
added to evidence.md, drafts completed, registry fields filled).
Keep it brief in auto-surface mode — just the score and top next step.

---

## 2. PRE-SCORE READS

Before calculating the score, read:

```
READ: registry/registry.yml        — for Component 1
READ: evidence.md                  — for Components 2 and 3
READ: output/                      — list all files, for Component 4
READ: data/cases.md                — for Component 5 (deadline and prior scores)
```

If `target.target_visa_type` is UNKNOWN: the score cannot be fully
calculated because required documents cannot be determined.
Ask: "To calculate your readiness score, I need to know your target
visa type. What specific visa are you pursuing?"

---

## 3. THE FIVE-COMPONENT SCORING MODEL

Total possible score: 100 points.
Each component has a maximum and a calculation method.

---

### COMPONENT 1 — Registry Knowledge (20 points)

**What it measures:** How much the agent knows about the user.
More complete profile = more accurate guidance = higher score here.

**Calculation:**
Take `registry_meta.overall_completeness` percentage.
Convert directly: 85% registry completeness = 17 out of 20 points.
Formula: `floor(overall_completeness_percentage / 100 * 20)`

Do not include SENSITIVE layer completeness in this calculation —
the user is not penalized for not disclosing sensitive history.

**Score ceiling:** 20

---

### COMPONENT 2 — Evidence Inventory (25 points)

**What it measures:** What the user actually has in hand.
This is the heaviest component because evidence is what wins applications.

**Calculation:**
Identify the required document categories for the target visa.
Use official requirements fetched from sources.yml — not assumptions.

For each required document category:
- Entry confirmed in `evidence.md` AND not expired AND not expiring
  before expected decision → **full points** for that category
- Entry exists but is PENDING, or expiry is a concern → **half points**
- No entry in `evidence.md` → **zero points**

Distribute 25 points across the required categories proportionally.
(e.g., if 5 required categories, each is worth 5 points)

**Score ceiling:** 25

---

### COMPONENT 3 — Document Quality (20 points)

**What it measures:** The quality and consistency of the evidence, not just its existence.

**Calculation:**
Start at 20. Apply deductions for quality issues found in `evidence.md`:

- Document expiring within the processing window: **-5 per document**
- Name inconsistency found across documents: **-5 per inconsistency**
- Required document present but no translation (non-English/French original): **-3 per document**
- Credential evaluation pending but not yet received: **-4 per evaluation**
- Document present but noted as "thin" or "self-prepared" in `agent_observations`: **-2**
- Any observation in `agent_observations` with `action_required: true` and unresolved: **-2 each**

Floor at zero — this component cannot go negative.

**Score ceiling:** 20

---

### COMPONENT 4 — Application Package Readiness (20 points)

**What it measures:** Whether the actual application package is taking shape.
Evidence alone is not enough — the documents must be organized and drafted.

**Calculation:**
Check what exists in `output/`:

- Cover letter file present in `output/`: **+5**
- Personal statement or statement of purpose present: **+5**
- Employer/sponsor letter confirmed (either in `evidence.md` or `output/`): **+5**
- Supporting documents organized and labeled (any file in `output/` besides the above): **+5**

These are binary — either the file exists or it does not.
Checking whether a draft has been "reviewed" by the user requires
asking: "Have you reviewed and personalized the cover letter?"
If yes: count it. If no: half credit.

**Score ceiling:** 20

---

### COMPONENT 5 — Timeline Safety (15 points)

**What it measures:** How much buffer exists before the hard deadline.
More buffer = lower urgency pressure = higher score.

**Calculation:**
Read `target.hard_deadline` from registry.
If no deadline is set: award **10 points** (neutral — not penalized for exploring).

If a deadline exists, calculate days between today and the deadline:

| Buffer remaining | Points |
|------------------|--------|
| > 90 days | 15 |
| 60–90 days | 12 |
| 30–59 days | 8 |
| 14–29 days | 4 |
| < 14 days | 0 + mandatory URGENT flag |

If < 14 days: immediately trigger alert.md for this deadline.
The score is 0 for this component AND the session opens with the alert.

**Score ceiling:** 15

---

## 4. THE READINESS REPORT FORMAT

```
APPLICATION READINESS — [VISA TYPE] — [COUNTRY]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  SCORE: [XX] / 100

  [Progress bar — use █ for scored, ░ for remaining]
  ████████████████░░░░  [XX] / 100

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
COMPONENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Registry knowledge:       [XX/20]   [bar]
Evidence inventory:       [XX/25]   [bar]
Document quality:         [XX/20]   [bar]
Application package:      [XX/20]   [bar]
Timeline safety:          [XX/15]   [bar]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NEXT THREE ACTIONS
(highest-impact steps to raise your score)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. [Specific action — not vague]
   Why: [what component this improves and by how much]
   How: [concrete step — e.g. "Apply at [authority], takes 4–8 weeks"]

2. [Second action — specific]
   Why: [component and point value]
   How: [concrete step]

3. [Third action — specific]
   Why: [component and point value]
   How: [concrete step]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Last scored: [YYYY-MM-DD]
[If prior score exists:] Previous score: [XX] | Change: [+/-XX points]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 5. THREE NEXT STEPS — NON-NEGOTIABLE SPECIFICITY

Every readiness report ends with exactly three specific next steps.
Never vague. Never generic.

**Wrong:**
> "Gather missing documents."

**Right:**
> "Obtain police clearance from Nigeria — apply through the Nigeria Police Force
> at [source fetched from official guidance]. Processing typically takes 4–8 weeks.
> Once obtained, tell me and I'll add it to your evidence inventory."

If fewer than three things would meaningfully improve the score, list what remains.
If more than three things would improve the score, rank by point value and show the top three.

For each next step, always include:
- What specifically to do (not a category — an action)
- Which component it improves and by approximately how many points
- How to do it (source, time estimate, first action)

---

## 6. SCORE PERSISTENCE AND DELTA

After every readiness calculation:
Record the score and date in `data/cases.md`:

```
## Readiness Score History
[YYYY-MM-DD]: [XX]/100
```

On subsequent readiness calculations:
Read the prior score from cases.md.
Show the delta:
> "Your readiness score is 78. That's up 6 points since [date]."

Progress is motivating. Show it explicitly.
If the score went down (because a document expired, or a new requirement
was identified): say so directly and explain why.
> "Your score dropped 3 points since last session. Your police clearance
> is now within the UPCOMING expiry window, which reduced the document
> quality component."

---

## 7. SCORE INTERPRETATION GUIDE

Include this at the bottom of every readiness report:

```
SCORE GUIDE:
90–100  Filing-ready. Final review recommended before submission.
75–89   Strong progress. Address the three actions above to reach filing-ready.
60–74   Good foundation. Key gaps remain — focus on evidence inventory.
45–59   Moderate progress. Registry and evidence both need attention.
Below 45  Early stage. Focus on completing the registry profile first.
```

---

*End of readiness.md*
