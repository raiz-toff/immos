# Immos — Mode Index

Quick reference for all available modes.
Loaded only when needed — not at every session.

---

## Foundation Modes (always loaded)

### _shared.md
The agent's character, hard rules, tone, and behavioral constants.
Loaded every session alongside every other mode. Never skipped.
Contains: identity, 6 hard rules as behavioral scripts, confidence
tier enforcement, registry habit, tone guidance, session close protocol,
pattern awareness (active from session 3+).

### _profile.md
User customizations. Survives all system updates. User-owned.
Loaded every session. Agent appends to agent_notes only.
Contains: communication preferences, countries of interest,
visa categories, attorney info, alert preferences.

---

## Core Modes

### intake.md
**Purpose:** Progressive onboarding — builds the user registry through conversation.
**Triggers:** First session, `session_count = 0`, registry completeness < 30%.
**Loads:** `registry/registry.yml` (write), `modes/_shared.md`
**Writes:** All unlocked registry layers. Does not touch SENSITIVE layers.
**Key behavior:** One question at a time. Never interrogates. Non-blocking after session 1.

### gap.md
**Purpose:** Document gap analysis — what the user has vs. what they need.
**Triggers:** "what documents", "am I missing anything", "am I ready to apply", form numbers.
**Reads:** `evidence.md` first, then `registry`, `knowledge/sources.yml`
**Writes:** Document status updates to `data/cases.md`
**Key behavior:** Checks evidence.md before classifying any document. 3-tier output: REQUIRED / STRONGLY RECOMMENDED / SUPPORTING.

### timeline.md
**Purpose:** End-to-end timeline calculation and backward planning from a deadline.
**Triggers:** "how long", "deadline", "when should I start", "is there time", "my job starts".
**Reads:** `registry`, `knowledge/last_update.json`, processing times (fetched live)
**Writes:** Deadline entries to `data/cases.md`
**Key behavior:** Always fetches processing times — never estimates from memory. Backward plans from hard deadline.

### pathway.md
**Purpose:** Visa pathway assessment — scores and ranks available routes on 6 dimensions.
**Triggers:** "what visa", "what are my options", "can I get PR", "which route is best".
**Reads:** Full registry, `evidence.md`, knowledge files for target country.
**Key behavior:** Scores Eligibility (30%) / Timeline (20%) / Documents (15%) / Financial (15%) / Risk (10%) / Strategic (10%). Never claims eligibility — always "appears viable."

### draft.md
**Purpose:** Document drafting — cover letters, personal statements, employer letters, reference letters, explanatory letters, relationship letters.
**Triggers:** "write", "draft", "generate", any document type name.
**Reads:** `registry`, `evidence.md` (for real data to personalize drafts). **Writes:** `output/`
**Key behavior:** Reads evidence.md for real scores/names/dates. Uses [REQUIRED: ...] placeholders only for truly unknown data. Every draft has a review checklist. Never finalizes.

### setback.md
**Purpose:** Setback response preparation — more-evidence request, refusal,
procedural-fairness letter, NOID, or equivalent, in ANY country.
**Triggers:** "they asked for more documents", "I was refused", "procedural
fairness letter", "request for further information", "RFE", "NOID", "221(g)",
"they denied me", or any local setback term from an installed country pack.
**Reads:** `registry`, `data/cases.md`, country pack `terminology`. Writes
deadline immediately to `data/cases.md`.
**Key behavior:** Speaks the country's local term (from pack). Triage first
(type / deadline / exact text). Mandatory attorney triggers for NOID, fraud
allegations, criminal history, extraordinary-ability categories. Surfaces
deadline at every session until resolved.

### country.md
**Purpose:** Bootstrap a knowledge pack for any country with no pack yet.
**Triggers:** target/residence country has no `knowledge/countries/<code>/pack.yml`; `/immigrate country <name>`.
**Reads/Writes:** web-fetches the authority + source URLs; writes `pack.yml` at status draft. Never fills a pack from memory.
**Key behavior:** Confirm government domain → fetch each source URL → fill terminology (UNVERIFIED if unconfirmed) → report covered/not_covered in two lines.

### scan.md
**Purpose:** Portal scan — fetches and surfaces changes from official sources.
**Triggers:** "check if anything changed", "any news", "latest on my category", startup routine.
**Reads:** `knowledge/sources.yml`, `knowledge/last_update.json`, `modes/_profile.md`
**Writes:** `knowledge/*.yml`, `knowledge/changelog.md`, `knowledge/last_update.json`
**Key behavior:** Scans only relevant countries/domains. Surfaces only changes that affect user's cases. One-line report when nothing changed.

### batch.md
**Purpose:** Multi-pathway parallel comparison — side-by-side scoring table.
**Triggers:** "compare all my options", "rank my pathways", multiple pathways named simultaneously.
**Reads:** `registry`, `evidence.md`, knowledge files for all relevant countries.
**Key behavior:** Compact scoring framework (not full pathway.md). Table format for quick comparison. Links to pathway.md for deeper dives.

### alert.md
**Purpose:** Deadline alerts and rule change notifications.
**Triggers:** Startup (CRITICAL/URGENT detected), "what deadlines", "upcoming dates", "remind me".
**Reads:** `data/cases.md`
**Key behavior:** CRITICAL (≤7d) surfaces at top of EVERY session until resolved. URGENT (≤30d) surfaces at session open. MONITORING (>90d) tracked silently. Alerts persist until user confirms resolution.

### verify.md
**Purpose:** Fact verification against official sources.
**Triggers:** "is this still true", "someone told me", "I read that", "ChatGPT said", "verify this".
**Reads:** `knowledge/sources.yml`, fetches from official source before verdict.
**Key behavior:** 4 verdicts only: CONFIRMED / OUTDATED / INCORRECT / UNVERIFIABLE. Never verdicts on stale data — fetches first. Does not editorialize about other AI tools.

---

## Cherry Modes

### evidence.md
**Purpose:** Manages the user's evidence inventory — the document ammunition store.
**Triggers:** Any document mentioned or uploaded, "what evidence do I have", `/immigrate evidence`.
**Reads:** `evidence.md`, `registry` (for name consistency check). **Writes:** `evidence.md`, `data/cases.md` (expiry deadlines).
**Key behavior:** Step 1: categorize. Step 2: extract fields. Step 3: check expiry. Step 4: check name consistency. Step 5: write entry. Step 6: confirm to user. All other modes read this file.

### audit.md
**Purpose:** Pre-submission audit — 4 structured checks before filing.
**Triggers:** "am I ready to file", "audit", "final check", "pre-submission review", `/immigrate audit`.
**Reads:** `evidence.md`, `output/*`, `data/cases.md`, `registry` (all layers).
**Writes:** `reports/audit-[date].md`
**Key behavior:** 4 checks in sequence: Consistency / Expiry / Red Flags / Completeness. Deliberately skeptical. Binary verdicts. Identifies but does not solve legal issues.

### readiness.md
**Purpose:** Application readiness score 0–100 with exactly 3 specific next steps.
**Triggers:** "readiness", "how ready am I", "what should I do next", "progress", `/immigrate readiness`.
**Reads:** `registry`, `evidence.md`, `data/cases.md`, `output/`
**Key behavior:** 5 components: Registry (20) / Evidence (25) / Doc Quality (20) / Package (20) / Timeline (15). Always ends with 3 specific, actionable next steps — never vague. Tracks delta from prior score.

### attorney.md
**Purpose:** Attorney consultation briefing document — written for the lawyer, not the user.
**Triggers:** "attorney", "lawyer", "consultation", "briefing", `/immigrate attorney`.
**Reads:** `registry` (all layers), `evidence.md`, `data/cases.md`, `reports/*`. **Writes:** `output/attorney-brief-[date].md`
**Key behavior:** 5 sections: Client Profile / Situation Summary / Evidence Inventory / Concerns & Flags / Questions. Formal register. Never includes legal advice or probability of success.

### challenge.md
**Purpose:** Adversarial review — finds weaknesses before an immigration officer does.
**Triggers:** "challenge", "devil's advocate", "what would fail", "weak points", `/immigrate challenge`.
**Reads:** `evidence.md`, `registry`, `output/*`, `reports/*`, official officer guidance (fetched).
**Writes:** `reports/challenge-[date].md`
**Key behavior:** 4 angles: Weakest Evidence / Officer Red Flags / Dangerous Interview Questions / Conspicuously Absent Evidence. Always ends with the mandatory reassurance statement. Never predicts refusal.

---

## Data Files (read/write at runtime)

| File | Mode | What it tracks |
|------|------|---------------|
| `registry/registry.yml` | R/W | User profile — 7 layers |
| `evidence.md` | R/W | Document inventory |
| `data/cases.md` | R/W | Active cases, deadlines, scores |
| `knowledge/last_update.json` | R/W | Fetch timestamps |
| `knowledge/changelog.md` | Append | Knowledge diffs |
| `knowledge/*.yml` | R/W | Fetched official data |
| `output/` | Write | Drafted documents |
| `reports/` | Write | Audit, challenge, readiness reports |

---

## Discovery Chain

```
CLI startup
  → reads .claude/skills/immos/SKILL.md (frontmatter only, ~50 tokens)
  → task matches description triggers
  → full SKILL.md loads
  → instructions say: read AGENTS.md
  → AGENTS.md startup protocol runs
  → routes to specific mode file
  → mode fires in isolation alongside _shared.md + _profile.md
```
