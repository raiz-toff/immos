# modes/status.md
# Status Mode — Registry and System Health Overview
#
# Activated by: /immigrate status
# Always loaded with: modes/_shared.md, modes/_profile.md

---

## Trigger Conditions

Activate this mode when the user:
- Runs `/immigrate status`
- Asks "what do you know about me"
- Asks "show me my profile"
- Asks "how complete is my registry"
- Asks "what's my current situation"
- Asks "system status" or "health check"
- Asks "what cases do I have active"

---

## What This Mode Does

Renders a clean, scannable snapshot of four things:

1. Registry completeness — what the agent knows about the user
2. Knowledge currency — how fresh the government data is
3. Active cases — what is being tracked right now
4. System health — any issues the user should know about

This is a READ-ONLY mode. It updates nothing.
It writes nothing. It fetches nothing.
It reads the current state and reports it honestly.

---

## Execution Protocol

### Step 1 — Read all inputs

```
READ: registry/registry.yml         → completeness per layer
READ: knowledge/last_update.json    → currency per domain
READ: data/cases.md                 → active cases and deadlines
READ: evidence.md                   → evidence inventory count
READ: modes/_profile.md             → user language preference
```

### Step 2 — Calculate completeness per layer

For each of the 7 registry layers, count:
- Fields with `confidence: HIGH` or `PARTIAL` → contributing
- Fields with `confidence: UNKNOWN` → gap
- Fields with `confidence: SENSITIVE` → not yet unlocked
- Fields with `confidence: UNDISCLOSED` → user declined

Layer completeness = contributing fields / total fields × 100

Overall completeness = average across all 7 layers

### Step 3 — Assess knowledge currency

For each domain in last_update.json:

| Status | Condition |
|--------|-----------|
| CURRENT | fetched within freshness window |
| STALE | beyond freshness window but ≤ 30 days |
| UNVERIFIED | > 30 days or NEVER_FETCHED |
| FETCH_FAILED | last attempt returned an error |

Group by country. Report per country.

### Step 4 — Summarize active cases

From cases.md, extract all cases with `status: ACTIVE`.
For each: pathway, days until next deadline, next action.
Sort by urgency (CRITICAL first, then URGENT, then UPCOMING).

### Step 5 — Flag system health issues

Check for:
- Any FETCH_FAILED domains in last_update.json → flag
- Any CRITICAL deadlines (≤ 7 days) → flag prominently
- Any registry conflicts (`conflict:` sub-field present) → flag
- evidence.md expiry dates within 90 days → flag
- registry_meta.session_count is 0 → note (first session)

---

## Output Format

Render this exact format. Clean. Scannable. No padding.

```
IMMOS STATUS
─────────────────────────────────────────

REGISTRY  [overall%]% complete

  Identity          ████████░░  [X/5 fields]
  Current status    █████░░░░░  [X/6 fields]
  History           ██░░░░░░░░  [X/5 fields]  [SENSITIVE if not unlocked]
  Target            ████░░░░░░  [X/5 fields]
  Family            ░░░░░░░░░░  [X/4 fields]
  Financial         ░░░░░░░░░░  [X/4 fields]
  Legal             ░░░░░░░░░░  [X/3 fields]  [SENSITIVE]

  Evidence inventory: [count] documents logged

─────────────────────────────────────────

KNOWLEDGE  Last fetched from official sources

  United States
    Processing times    [CURRENT | STALE | UNVERIFIED | FETCH_FAILED]  [date]
    Fees               [CURRENT | STALE | UNVERIFIED | FETCH_FAILED]  [date]
    Forms              [CURRENT | STALE | UNVERIFIED | FETCH_FAILED]  [date]
    Policy notices     [CURRENT | STALE | UNVERIFIED | FETCH_FAILED]  [date]

  Canada
    Processing times    [status]  [date]
    Fees               [status]  [date]
    Express Entry      [status]  [date]
    Policy notices     [status]  [date]

  United Kingdom
    Processing times    [status]  [date]
    Fees               [status]  [date]
    Salary thresholds  [status]  [date]
    Policy notices     [status]  [date]

─────────────────────────────────────────

ACTIVE CASES  [count]

  [CASE_ID] — [PATHWAY]
  Next: [next_action] by [date] ([X] days)  [CRITICAL/URGENT/UPCOMING]

  [If no active cases:]
  No active cases. Start one with /immigrate pathway.

─────────────────────────────────────────

[ALERTS — only show if any exist]

  ⚠ CRITICAL  [X] days — [deadline label]
  ⚑ FETCH_FAILED  [country] [domain] — source unavailable
  ⚑ CONFLICT  [field] — unresolved value conflict in registry
  ⚑ EXPIRING  [document] in evidence.md expires [date]
  ⚑ STALE KNOWLEDGE  [count] domains need refresh

─────────────────────────────────────────
```

---

## Progress Bar Rendering

Use text-based progress bars. Each bar is 10 characters.

| Completeness | Bar |
|---|---|
| 0% | `░░░░░░░░░░` |
| 10% | `█░░░░░░░░░` |
| 20% | `██░░░░░░░░` |
| 30% | `███░░░░░░░` |
| 40% | `████░░░░░░` |
| 50% | `█████░░░░░` |
| 60% | `██████░░░░` |
| 70% | `███████░░░` |
| 80% | `████████░░` |
| 90% | `█████████░` |
| 100% | `██████████` |

Round to nearest 10% for the bar. Show the exact field count as text.

---

## Knowledge Status Labels

CURRENT → no annotation needed
STALE → show date and add `⚠` prefix
UNVERIFIED → show `--` for date and `⚑ needs refresh`
FETCH_FAILED → show `⚑ FETCH_FAILED` in warning tone
NEVER_FETCHED → show `--` and note first session will fetch

---

## What This Mode Never Does

Never fetches data — it reads last_update.json, not live sources.
Never updates the registry — read only.
Never routes to another mode automatically.
Never offers to "fix" anything — it reports. The user decides what to do.

If the user wants to act on something in the status report:
- "Let's fill in that gap" → route to intake mode
- "Fetch that knowledge" → route to scan mode
- "Tell me about that case" → route to appropriate case mode
- "Run an audit" → route to audit mode

The status report ends with one line suggesting the next action
based on the most important thing found. If there is a CRITICAL
deadline: "Run /immigrate alert to review your deadline."
If registry is under 30%: "Run /immigrate intake to build your profile."
If knowledge is NEVER_FETCHED: "Your first scan will run on next session."
If everything looks good: "Everything looks current. What would you like to work on?"

---

## For First Session (session_count = 0)

If `registry_meta.session_count` is 0:

Adapt the output:
- Registry will show 0% across all layers — that is expected
- Knowledge will show NEVER_FETCHED — will update this session
- Active cases will be empty — expected

Open the status display with:
"Here is your system at first launch. Everything starts at zero
and builds as we work together."

Close with:
"To get started, try /immigrate intake — it will ask you a few
questions to understand your immigration situation."
