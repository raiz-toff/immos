# IMMIGRATION-OPS — Claude Code
# Version 1.0.0

Read `Agents.md` for all project instructions, behavioral rules,
registry architecture, confidence tier system, startup protocol,
mode routing, hard rules, and response standards.
Everything in `Agents.md` applies fully here.

---

## Quick Commands

```
/immigrate [question or input]     → auto-routes to the correct mode
/immigrate intake                  → runs onboarding / registry building
/immigrate gap                     → runs document gap analysis
/immigrate timeline                → runs timeline calculation
/immigrate pathway                 → runs visa pathway assessment
/immigrate draft [type]            → runs document drafting
/immigrate rfe                     → runs RFE / refusal response
/immigrate scan                    → runs portal scan for changes
/immigrate batch                   → runs multi-pathway comparison
/immigrate alert                   → checks and surfaces all deadlines
/immigrate verify [claim]          → fact-checks a specific claim
/immigrate status                  → shows registry completeness summary
/immigrate evidence               → manage and review evidence inventory
/immigrate audit                  → run pre-submission audit (4-check review)
/immigrate readiness              → show readiness score and top 3 next steps
/immigrate attorney               → generate attorney consultation brief
/immigrate challenge              → adversarial review — find weaknesses before filing
```

---

## On Every Launch

Before responding to the user, run the full startup protocol from
`Agents.md` Section 2. In order:

1. Read `registry/registry.yml` — load profile, calculate completeness
2. Read `knowledge/last_update.json` — check freshness of all domains
3. If any domain is stale: run the knowledge update routine silently
4. Read `data/cases.md` — load active cases, classify deadlines
5. Build session context: know what matters today before the user speaks
6. Open the session appropriately:
   - If CRITICAL deadline: surface it immediately
   - If nothing urgent: greet simply and wait

Do this silently. The user should not feel the startup happening.

---

## Mode Loading Rules

Load only the relevant mode file alongside `modes/_shared.md` and
`modes/_profile.md`. Never load two full mode contexts at once.
Mode files are in `modes/`. Route based on the trigger table in
`Agents.md` Section 6.

---

## Skills Location

`.claude/skills/immigration-ops/SKILL.md`
(Create this path if extending the system with custom Claude Code skills.)

---

## Data Files

```
registry/registry.yml      — user profile (read + write)
data/cases.md              — active cases and deadlines (read + write)
knowledge/                 — fetched official knowledge (read + write)
modes/_profile.md          — user customizations (read + write)
output/                    — drafted documents (write)
reports/                   — generated reports (write)
```

Everything else is READ ONLY during normal sessions.
See `docs/DATA_CONTRACT.md` for the full permission matrix.
