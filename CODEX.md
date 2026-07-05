# IMMOS — Codex / OpenAI Agents
# Version 1.0.0

Read `AGENTS.md` for all project instructions, behavioral rules,
registry architecture, confidence tier system, startup protocol,
mode routing, hard rules, and response standards.
Everything in `AGENTS.md` applies fully here.

---

## Quick Commands

When invoking via the OpenAI Codex CLI or Assistants API:

```
immigrate [question or input]     → auto-routes to the correct mode
immigrate intake                  → runs onboarding / registry building
immigrate gap                     → runs document gap analysis
immigrate timeline                → runs timeline calculation
immigrate pathway                 → runs visa pathway assessment
immigrate draft [type]            → runs document drafting
immigrate rfe                     → runs RFE / refusal response
immigrate scan                    → runs portal scan for changes
immigrate batch                   → runs multi-pathway comparison
immigrate alert                   → checks and surfaces all deadlines
immigrate verify [claim]          → fact-checks a specific claim
immigrate status                  → shows registry completeness summary
immigrate evidence                → manage and review evidence inventory
immigrate audit                   → run pre-submission audit (4-check review)
immigrate readiness               → show readiness score and top 3 next steps
immigrate attorney                → generate attorney consultation brief
immigrate challenge               → adversarial review — find weaknesses before filing
```

---

## On Every Launch

Before responding to the user, run the full startup protocol from
`AGENTS.md` Section 2. In order:

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
`AGENTS.md` Section 6.

---

## Codex / OpenAI-Specific Notes

When using the file_search or retrieval tools:
- Read from the paths specified in the permission matrix in `docs/DATA_CONTRACT.md`
- Use `knowledge/sources.yml` as the URL index for web fetching tasks
- Do not retrieve from unofficial sources (forums, blogs, third-party sites)

When using code_interpreter or similar execution tools:
- Use them for date arithmetic (timeline calculations, days-remaining)
- Use them for registry completeness calculations
- Do NOT use them to interact with government portals

When using function_calls for file operations:
- Respect the READ ONLY / READ+WRITE / NEVER TOUCH distinctions in DATA_CONTRACT.md
- Every registry write must include all four metadata fields
- Never overwrite HIGH confidence values without user confirmation

Citation format:
`[VERIFIED | source.gov | YYYY-MM-DD]`

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
