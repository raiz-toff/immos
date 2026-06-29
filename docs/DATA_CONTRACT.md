# DATA_CONTRACT.md
# ImmigrationOps — Explicit Data Permissions and Behavioral Contract
# Version 1.0.0
#
# This document governs exactly what the agent can and cannot touch.
# It is READ ONLY. No mode file, no user instruction, and no agent
# reasoning can override what is written here.
# If this document does not authorize an action, the agent does not take it.

---

## 1. PERMISSION MATRIX

Every file and directory in this system has an explicit permission level.
If a path is not listed here, the agent must ask the user before touching it.

### READ ONLY — the agent reads these but never modifies them

| Path | Reason |
|------|--------|
| `AGENTS.md` | Foundation layer. Immutable behavioral contract. |
| `CLAUDE.md` | CLI wrapper. System configuration only. |
| `GEMINI.md` | CLI wrapper. System configuration only. |
| `CODEX.md` | CLI wrapper. System configuration only. |
| `README.md` | User-facing documentation. Not modified at runtime. |
| `docs/DATA_CONTRACT.md` | This file. Never self-modifying. |
| `docs/SETUP.md` | Setup documentation. Not modified at runtime. |
| `docs/ARCHITECTURE.md` | Architecture documentation. Not modified at runtime. |
| `modes/*.md` | All mode instruction files except `_profile.md` and `_custom.md`. These are behavioral scripts, not data. |
| `knowledge/sources.yml` | Official source index. Updated only by explicit system update, not at runtime. |

**What READ ONLY means in practice:**
The agent loads these files into context and uses their content to drive behavior.
It does not append to them, overwrite them, delete lines from them, or create
modified versions. If a mode file appears to have an error, the agent notes it
in session context and continues — it does not "fix" the file.

---

### READ + WRITE ALLOWED — the agent reads and updates these during sessions

| Path | What the Agent May Write | Constraints |
|------|--------------------------|-------------|
| `registry/registry.yml` | Registry field updates as the agent learns from the user | Full metadata required on every write. See Section 3. |
| `modes/_profile.md` | User preference updates | Only appends to `agent_notes` freely. All other sections updated only when user explicitly states a preference. Never removes user-written content. |
| `modes/_custom.md` | Never modified by the agent | Read only at runtime. The user owns this file entirely. Agent applies what is written here but never touches the file itself. |
| `data/cases.md` | New cases, deadline updates, status changes, document status | Never deletes a case. Only changes status to COMPLETE or ON_HOLD. |
| `knowledge/last_update.json` | Fetch timestamps and status codes after successful or failed fetches | Only updates the specific domain that was fetched. Never resets other timestamps. |
| `knowledge/changelog.md` | Appends entries after knowledge updates | Append only. Never edits existing entries. |
| `knowledge/*.yml` | Domain knowledge files populated from official source fetches | Only overwrites when new data differs from existing. Records the diff in changelog.md before overwriting. |
| `reports/` | Analysis reports, pathway assessments, timeline documents | Creates new files with date-stamped names. Never overwrites existing reports. |
| `output/` | Drafted documents — letters, statements, templates | Creates new files. Never overwrites. User must explicitly request a replacement draft. |
| `.gitignore` | Adds new country folder entries when a new country is detected in the registry | Appends `knowledge/[code]/` under the "ADDING A NEW COUNTRY" section only. Never removes entries. Never modifies any other section. Must happen BEFORE creating the country knowledge directory. |

**What READ + WRITE ALLOWED means in practice:**
The agent may create, update, or append to these files during normal operation.
All writes require complete metadata. No silent overwrites of high-confidence data.
The user can inspect every file the agent has written at any time.

---

### NEVER TOUCH — absolute prohibitions

| Path / Action | Why |
|---------------|-----|
| `.git/` | Version control system. Agent actions are not git operations. |
| Any government portal, API, or submission endpoint | The agent is a preparation tool. It never acts as an applicant. |
| Any file not listed in this contract | Unknown files may be user data, system files, or sensitive material. Ask first. |
| Registry fields marked `UNDISCLOSED` | The user chose not to share this. That choice is permanent for the session and respected indefinitely. |
| HIGH confidence registry values — without explicit user confirmation | See Section 3, Conflict Protocol. |

**What NEVER TOUCH means in practice:**
Even if the user explicitly asks the agent to submit a form, interact with a
government portal, delete a file, or modify a `.git` directory, the agent
declines. The prohibition is categorical. There is no workaround.

---

## 2. PERMISSION LEVEL DEFINITIONS

### READ ONLY
The agent loads the file content into working context at the start of a session
or when the mode requires it. It uses the content to drive responses and behavior.
It does not write, append, delete, rename, or create derivative versions of the file.
If asked to modify a READ ONLY file, the agent declines and explains why.

### READ + WRITE ALLOWED
The agent may read, update, append, or create files in this category during
normal operation — but only in the ways specifically described in the permission
matrix above. Writes that fall outside the described behavior are treated as
NEVER TOUCH until the user explicitly authorizes them.

### NEVER TOUCH
The agent takes no action on these paths under any circumstance. If user
instructions or mode logic appears to require touching a NEVER TOUCH path,
the agent stops, declines, and explains the constraint. It does not find
workarounds. It does not attempt the action "just this once."

---

## 3. REGISTRY WRITE RULES

### 3.1 — Mandatory Metadata on Every Write

Every registry field write must include all four metadata properties.
A write that is missing any of the four is incomplete and must not be committed.

```yaml
field_name:
  value: [the data]
  confidence: [HIGH | PARTIAL | LOW | UNKNOWN | SENSITIVE | UNDISCLOSED]
  source: [user_stated | document_parsed | inferred | system_derived | assumed]
  updated: [YYYY-MM-DD ISO 8601 date]
```

The agent never writes a field with only the `value` property populated.
The agent never writes `updated: ~` on a new write — the date is always the
current session date.

### 3.2 — Conflict Protocol

When new incoming data conflicts with an existing field that carries
`confidence: HIGH`, the agent must NOT silently overwrite the existing value.

The conflict is recorded using the following structure:

```yaml
field_name:
  value: [existing HIGH confidence value — unchanged]
  confidence: HIGH
  source: [existing source]
  updated: [existing date]
  conflict:
    incoming_value: [the new value]
    incoming_source: [source of new value]
    incoming_date: [date new value was encountered]
    resolution: PENDING
    note: [brief explanation of the discrepancy]
```

The agent then surfaces the conflict to the user at the next natural
conversational moment — not immediately if mid-topic, but not suppressed
for more than one exchange. It asks the user to confirm which value is
correct before resolving the conflict and updating the field.

Until the conflict is resolved, the agent uses the existing HIGH confidence
value in all calculations and assessments, noting that a discrepancy exists.

### 3.3 — The UNDISCLOSED Boundary

When a user is asked about a registry field and explicitly declines to answer
— by saying "I'd rather not say," "skip that," "not relevant," "I don't want
to share that," or any equivalent — the field is written as:

```yaml
field_name:
  value: ~
  confidence: UNDISCLOSED
  source: user_stated
  updated: [date of declination]
```

Once a field is marked UNDISCLOSED:
- The agent does not re-ask about this field during the current session
- The agent does not infer around the boundary
- The agent acknowledges the gap in any assessment that would require this field
- The field remains UNDISCLOSED until the user proactively offers the information

The UNDISCLOSED marking is a user choice, not a data gap. It is treated
with the same finality as a document that explicitly states the information
is not available.

---

## 4. SENSITIVE DATA HANDLING

### 4.1 — Default Sensitivity

The following registry layers are SENSITIVE by default:

| Layer | Fields | Default State |
|-------|--------|---------------|
| Layer 3 — History | prior_visas, travel_history, prior_refusals, overstays, deportations_removals | SENSITIVE until unlocked |
| Layer 7 — Legal | criminal_history_any_country, pending_legal_matters, prior_immigration_violations | SENSITIVE until unlocked |

All fields in these layers initialize with `confidence: SENSITIVE`.

### 4.2 — What SENSITIVE Means Operationally

A SENSITIVE field is one the agent does not surface, reference, probe, or ask
about without meeting the conditions in Section 4.3.

In practice:
- The agent does not mention these topics in conversation
- The agent does not include these fields in completeness calculations that
  surface to the user
- The agent does not ask about prior refusals, criminal history, overstays,
  or deportations until the unlock conditions are met
- If a pathway assessment technically requires this data, the agent flags that
  it cannot give a fully accurate assessment without information it has not
  yet asked for, and explains when it will

### 4.3 — Unlock Conditions

A SENSITIVE layer is unlocked when ALL of the following are true:
1. The user has completed at least 3 meaningful sessions (session_count ≥ 3), OR
   the user raises the sensitive topic themselves
2. A specific pathway assessment makes the data necessary — not merely useful
3. The agent explains why it needs the information before asking
4. The agent explicitly states that data stays local before asking
5. The agent makes clear the user can decline

When a SENSITIVE layer is unlocked, it is recorded in:
```yaml
registry_meta:
  sensitive_layers_unlocked: [history, legal]
```

### 4.4 — SENSITIVE Data Is Never Written to Reports or Output

Data from SENSITIVE layers is never included in:
- Reports written to `reports/`
- Drafted documents written to `output/`
- Any file that leaves the `registry/` directory

Unless the user explicitly requests it and understands the implication.

---

## 5. LOCAL-ONLY GUARANTEE

### 5.1 — What This Means

All data in this system — registry entries, case records, knowledge files,
drafted documents, and reports — exists only on the user's local machine.

The agent does not:
- Transmit any user data to external servers
- Store data in cloud memory or external databases
- Include registry contents in web search queries
- Log or cache user information in any system outside the project directory

### 5.2 — What This Means Per File Type

| File Type | Local Guarantee |
|-----------|-----------------|
| `registry/registry.yml` | Never transmitted. Read locally, written locally. |
| `data/cases.md` | Never transmitted. Read locally, written locally. |
| `knowledge/*.yml` | Content fetched from official URLs (read-only fetch). No user data sent. |
| `output/*.md` | Generated locally. Not automatically sent anywhere. |
| `reports/*.md` | Generated locally. Not automatically sent anywhere. |

### 5.3 — What System Updates Do and Do Not Modify

A system update may replace the following files:
- `modes/*.md` (all mode files except `_profile.md`)
- `docs/*.md`
- `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `CODEX.md`, `README.md`

A system update NEVER modifies or replaces:
- `registry/registry.yml` — this is user data
- `modes/_profile.md` — this is user customization
- `data/cases.md` — this is user case records
- `knowledge/*.yml` — these are fetched knowledge files specific to this user
- `knowledge/last_update.json` — fetch history
- `knowledge/changelog.md` — knowledge diff log
- `output/` — all drafted documents
- `reports/` — all generated reports

User data survives system updates intact, always.

---

## 6. UPDATE BOUNDARY SUMMARY

| File / Directory | Survives System Update | Reason |
|-----------------|----------------------|--------|
| `registry/registry.yml` | YES — always preserved | User data |
| `modes/_profile.md` | YES — always preserved | User customization |
| `data/cases.md` | YES — always preserved | User case records |
| `knowledge/` (entire directory) | YES — always preserved | User-specific fetched data |
| `output/` (entire directory) | YES — always preserved | User-generated documents |
| `reports/` (entire directory) | YES — always preserved | User-generated reports |
| `modes/*.md` (except `_profile.md`) | MAY BE UPDATED | System behavior files |
| `AGENTS.md` | MAY BE UPDATED | Foundation layer |
| `docs/*.md` | MAY BE UPDATED | Documentation |
| `CLAUDE.md`, `GEMINI.md`, `CODEX.md` | MAY BE UPDATED | CLI wrappers |

---

*End of DATA_CONTRACT.md — Version 1.0.0*
*This document is READ ONLY at runtime. Violations of this contract are not permitted under any circumstance.*
