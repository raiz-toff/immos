# ARCHITECTURE.md — ImmigrationOps Technical Architecture
# Version 1.0.0

---

## Overview

ImmigrationOps is a file-based, agent-driven immigration assistance system.
It is not a web application, not a database, and not a service. It is a
structured directory of instruction files, user data files, and knowledge
files that an AI agent reads and writes during each session.

The architecture has five layers:

```
┌─────────────────────────────────────────────────────┐
│ 5. OUTPUT LAYER                                     │
│    output/ reports/  — drafts, reports, documents   │
├─────────────────────────────────────────────────────┤
│ 4. KNOWLEDGE LAYER                                  │
│    knowledge/  — fetched official data, changelog   │
├─────────────────────────────────────────────────────┤
│ 3. DATA LAYER                                       │
│    registry/ data/  — user profile, active cases    │
├─────────────────────────────────────────────────────┤
│ 2. MODE LAYER                                       │
│    modes/  — behavioral scripts per mode            │
├─────────────────────────────────────────────────────┤
│ 1. FOUNDATION LAYER                                 │
│    AGENTS.md  — master behavioral contract          │
└─────────────────────────────────────────────────────┘
```

The foundation layer is immutable during runtime.
Every layer above it is progressively more specific and more writeable.

---

## Layer 1 — Foundation

**File:** `AGENTS.md`

The master contract. Every AI CLI that runs this system reads `AGENTS.md`
and every behavior in the system derives from it. It defines:

- The agent's identity and philosophy
- The startup protocol (5 steps, run on every session)
- The 7-layer user registry schema and field structure
- The progressive learning protocol
- The knowledge and confidence tier system
- The mode routing table
- The data contract (what can be touched)
- The 5 hard rules that override everything else
- Response standards, tone, and session close protocol

`CLAUDE.md`, `GEMINI.md`, and `CODEX.md` each import `AGENTS.md` and add
CLI-specific command syntax and notes. The foundation is shared.

---

## Layer 2 — Mode Layer

**Directory:** `modes/`

Mode files are behavioral scripts — instructions to the agent for how to
behave when in a specific operating context. They are READ ONLY at runtime.

**Always loaded:**
- `_shared.md` — the agent's core identity, hard rules in behavioral script form,
  tone guidance, and session close protocol. This file runs on every session.
- `_profile.md` — user customizations (the only mode file the agent writes to).
  This file personalizes behavior: communication preferences, countries of interest,
  attorney information, alert settings.

**Mode files (loaded one at a time alongside _shared and _profile):**

| File | Mode | Trigger |
|------|------|---------|
| `intake.md` | Onboarding | First session, incomplete registry |
| `gap.md` | Document gap analysis | "What do I need?" queries |
| `timeline.md` | Timeline calculation | "How long will this take?" queries |
| `pathway.md` | Pathway assessment | "What are my options?" queries |
| `draft.md` | Document drafting | "Write me a letter" queries |
| `rfe.md` | RFE/refusal response | "I got an RFE" / denial notices |
| `scan.md` | Portal scan | "Check for changes" / startup scan |
| `batch.md` | Multi-pathway comparison | "Compare all my options" |
| `alert.md` | Deadline alerts | CRITICAL/URGENT deadlines / "What deadlines?" |
| `verify.md` | Fact verification | "Is this still true?" queries |

Only one mode file is loaded at a time. This prevents context bloat
and keeps each mode's logic clean and focused.

---

## Layer 3 — Data Layer

**Files:** `registry/registry.yml`, `data/cases.md`

### registry/registry.yml

The user registry is the agent's persistent memory. It is a 7-layer YAML file
that stores everything the agent has learned about the user across all sessions.

Every field uses exactly four properties:
```yaml
field_name:
  value: ~
  confidence: UNKNOWN
  source: ~
  updated: YYYY-MM-DD
```

The 7 layers are:
1. Identity — who the user is fundamentally
2. Current Status — where they are legally and physically right now
3. History — prior visa history, refusals, overstays (SENSITIVE by default)
4. Target — what they are trying to achieve and where
5. Family — family composition and immigration situations
6. Financial — ability to meet visa financial requirements
7. Legal — criminal history and legal matters (SENSITIVE by default)

Plus `registry_meta` which tracks session count, completeness, and unlock state.

The registry drives everything. Before any substantive response, the agent
reads the registry to personalize its answer. After every session, the agent
updates any fields it learned from conversation.

### data/cases.md

A flat Markdown file used as a simple case database. One block per active
immigration case. Each case block includes:
- Case ID, pathway, and status
- Next action and when it is due
- Document status table (CONFIRMED / PENDING / MISSING)
- Deadline table with classification and consequences
- Free-form notes

Cases are never deleted. They transition to COMPLETE or ON_HOLD.

---

## Layer 4 — Knowledge Layer

**Directory:** `knowledge/`

| File | Purpose |
|------|---------|
| `sources.yml` | Index of official source URLs by country and domain. READ ONLY. |
| `last_update.json` | Fetch timestamps and status for every domain. Updated after every fetch. |
| `changelog.md` | Append-only log of all knowledge changes detected. |
| `[domain].yml` | Domain knowledge files written from official source fetches. Created at runtime. |

**How the knowledge system works:**

1. At startup, the agent reads `last_update.json` to check which domains
   are stale (beyond their `max_age_hours` window from `sources.yml`).
2. Stale domains are fetched from the URL in `sources.yml`.
3. Fetched data is compared against the existing domain file.
4. If changes are detected: the domain file is updated, the diff is recorded
   in `changelog.md`, and `last_update.json` is updated.
5. If a fetch fails: the agent marks it as `FETCH_FAILED`, keeps the previous
   data, and notes the failure.

The knowledge layer has three confidence states:
- **VERIFIED** — fetched within the freshness window
- **STALE** — fetched, but older than the freshness window (used cautiously)
- **UNVERIFIED** — never fetched, or fetch failed (never used as fact)

---

## Layer 5 — Output Layer

**Directories:** `output/`, `reports/`

The agent writes to these directories when generating documents or reports.
These directories are never automatically overwritten — each new file gets
a unique name with a date stamp.

`output/` — drafted documents (letters, statements, templates)
`reports/` — analysis reports (pathway assessments, gap analyses)

Files in these directories are the agent's work products for the user.
They are always DRAFT documents — the user reviews and finalizes.

---

## How the Startup Protocol Works End-to-End

Every session follows this sequence:

```
Launch
  │
  ├─ 1. READ registry/registry.yml
  │     Calculate completeness per layer
  │     Identify critical gaps
  │     Note SENSITIVE layer unlock state
  │
  ├─ 2. READ knowledge/last_update.json
  │     Compare each domain timestamp against max_age_hours
  │     If stale → Step 2a
  │
  ├─ 2a. KNOWLEDGE UPDATE (if needed)
  │      For each stale domain:
  │        Fetch from sources.yml URL
  │        Compare against existing data
  │        Write diff to changelog.md
  │        Update last_update.json
  │
  ├─ 3. READ data/cases.md
  │     Load all ACTIVE cases
  │     Calculate days remaining for each deadline
  │     Classify: CRITICAL / URGENT / UPCOMING / MONITORING
  │
  ├─ 4. BUILD SESSION CONTEXT
  │     What is most important to surface today?
  │     Priority: CRITICAL > knowledge changes > registry gaps > URGENT
  │
  └─ 5. OPEN SESSION
        CRITICAL deadline → lead with it
        Nothing urgent   → greet and wait
```

The user never sees this happening. It is invisible preparation.

---

## How the Registry Drives Agent Behavior

The registry is consulted before every substantive response.

1. Agent reads the registry fields most relevant to the current question.
2. Fields with HIGH or PARTIAL confidence → used in the response directly.
3. Fields with LOW confidence → used with explicit caveat.
4. Fields with UNKNOWN → if needed, the agent asks for them (in context, once).
5. Fields with SENSITIVE confidence → not surfaced or probed unless unlocked.
6. Fields with UNDISCLOSED → never re-asked this session.

This means responses become more precise and personal as the registry fills.
A session 1 pathway assessment is general. A session 10 pathway assessment
is specific to the user's exact profile, employer, timeline, and visa history.

---

## How Progressive Learning Accumulates

The agent uses two mechanisms to build the registry:

**Active collection:** When the agent needs a specific field to answer
accurately, it asks for it — in context, with a reason, once per session.

**Passive collection:** The agent listens during normal conversation and
extracts registry-mappable information the user mentions incidentally.
If the user says "my work permit expires in March 2027" in passing,
the agent records `current_status.visa_expiry: 2027-03-XX` without
announcing it.

Over 3–5 sessions, a typical user will have Priority 1 and 2 fields
filled without having formally answered any "profile questions." The
registry fills through use.

---

## How the Confidence Tier System Gates Responses

The confidence tier system prevents the agent from presenting uncertain
information as fact. It applies to both registry values and knowledge data.

**Registry values:**
Each field carries a confidence level that tells the agent how much to trust it.
HIGH → use freely | PARTIAL → use with caveat | LOW → caveat and seek confirmation |
UNKNOWN → do not use, ask if needed | SENSITIVE → do not surface |
UNDISCLOSED → do not re-ask

**Knowledge data:**
Each domain's knowledge is tagged with a confidence tier based on fetch freshness.
VERIFIED → answer with confidence + citation |
STALE → answer with warning + citation |
UNVERIFIED → fetch before answering; if fetch fails, do not answer from memory

---

## How Modes Are Isolated

Each mode file loads alongside `_shared.md` and `_profile.md` only.
No other mode files are loaded.

This isolation has two benefits:
1. **Context efficiency** — the agent's working context is not bloated with
   irrelevant mode logic. A document gap analysis does not load timeline
   calculation instructions.
2. **Clean behavioral scope** — each mode's logic is focused and self-contained.
   Modes cross-reference each other through explicit handoffs, not through
   shared loading.

The only mode loaded universally is `_shared.md` (core identity) and
`_profile.md` (user customizations). Everything else is loaded by need.

---

## How to Add a New Mode

1. Create the mode file in `modes/[modename].md`.
2. Define: trigger conditions, methodology, report format, registry interaction.
3. Write the file as behavioral instructions — use "you" not "the agent."
4. Verify the file does not violate any of the 5 hard rules from `AGENTS.md`.
5. Add the mode to the routing table in `AGENTS.md` Section 6.
6. Add the slash command to `CLAUDE.md`, `GEMINI.md`, and `CODEX.md`.
7. Document the mode in this file under the Mode Layer section.

Modes should be self-contained. If a mode needs to reference another mode's
behavior, use explicit language: "See timeline.md protocol for deadline
classification." Do not duplicate logic — reference it.

---

## How to Add a New Country

1. Add the country to `knowledge/sources.yml` with all relevant domains
   and official URLs. Follow the existing country structure exactly.
2. Add the country to `knowledge/last_update.json` with all domains
   initialized to `1970-01-01T00:00:00Z` / `NEVER_FETCHED`.
3. Update `modes/_profile.md` instructions to include the new country code.
4. Update the sources table in `AGENTS.md` Section 5.
5. Test the fetch sequence: set the country in `_profile.md`, launch a
   session, and verify the agent fetches current knowledge for that country.

Each country's sources should be official government immigration authorities.
No third-party sources. No unofficial aggregators.

---

## Design Decisions and Rationale

**Why local files, not a database?**
Immigration data is sensitive personal information. A file-based system
gives the user direct visibility and control over their data. There are no
server-side data stores, no cloud sync, no databases to breach. The user
can open `registry/registry.yml` in a text editor and read everything the
system knows about them.

**Why YAML for the registry?**
YAML is human-readable and writable. Users who want to correct or inspect
their registry can do so directly without special tools. The agent can write
clean YAML without complex serialization logic.

**Why Markdown for cases.md?**
Cases are narrative — they describe ongoing situations with notes, tables,
and updates. Markdown is flexible enough to handle the variety of information
a case record needs. It is also readable by the user without any parsing.

**Why mode files as instruction text rather than structured logic?**
The running agent is an LLM, not a rule engine. LLMs follow natural language
instructions well. Behavioral instruction files are more maintainable and
auditable than formal rules, and they allow for nuance that rigid logic
cannot express — for example, "ask about this sensitively" is easy to
express in prose and nearly impossible in formal logic.

**Why the five hard rules?**
The hard rules exist because the consequences of violating them are severe:
accidentally filing an application, giving overconfident eligibility assessments,
or silently overwriting correct data could cause real harm. The rules are
explicit and inviolable so they cannot be circumvented by user pressure
or by mode-specific instructions that might seem to supersede them.

**Why epoch timestamps in last_update.json?**
Initializing all domains to epoch (1970-01-01) ensures that the very first
session forces a full knowledge fetch from all official sources. This means
the system starts with verified current data rather than relying on any
pre-loaded information that could be outdated from the moment of setup.

---

*End of ARCHITECTURE.md*
