# Changelog

All notable changes to the ImmigrationOps harness are documented here.
This tracks system changes — mode updates, new countries, bug fixes, and
breaking changes. It is distinct from `knowledge/changelog.md` which
tracks changes to fetched government data.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versions follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

Changes staged for the next release.

---

## [1.1.0] — 2026-07-03

Gap-closure upgrade (six gaps G1–G8 from the upgrade guide). Zero new
dependencies — every mechanism is a contract the LLM follows, backed by the
Phase-5 eval set. See `docs/EVAL-RUN-2026-07-03.md`.

### Fixed
- **G1** — `Agents.md` → `AGENTS.md` (filesystem is case-sensitive here, so the
  canon file silently failed to load in AGENTS-aware harnesses). All in-repo
  references recased; DATA_CONTRACT path unified to `docs/DATA_CONTRACT.md`.

### Added
- **G2** — root wrappers `OPENCODE.md`, `QWEN.md`, `GROK.md`, `KIMI.md`.
- **G3** — AGENTS.md §9 replaced with the Style Contract (answer-first, hard
  length caps, Direct Question Protocol, terse session close); propagated via a
  `_shared.md` style pointer.
- **G4** — fast-track intake; completeness now labels (PROVISIONAL/WORKING/full)
  instead of blocking, with an assessment-weighted formula; registry Layer 8
  (communication & behavior) + per-dependent mini-registries; `data/documents.md`
  and `data/journal.md`; document-expiry deadline scan; journal continuity.
- **G5** — country packs: `knowledge/countries/<code>/pack.yml` (+`_TEMPLATE`),
  us/ca/uk migrated (status draft, terminology UNVERIFIED pending a fetch pass);
  thin `sources.yml` index; new `modes/country.md`; `rfe.md` → `setback.md`
  (any-country, pack-terminology-aware); de-Americanized routing.
- **G6** — fact-record schema; draft anti-hallucination contract
  (`[YOU:]`/`[CONFIRM:]`, mandatory consistency pass); attorney-referral trigger
  table (8 triggers); **RULE 6 — never assist misrepresentation** (now six hard
  rules); timeline hardening (backward-from-deadline on VERIFIED times only).
- **G7** — `docs/EVALS.md` (40 cases) + `docs/EVAL-RUN-2026-07-03.md`.
- **G8** — store-minimum privacy rule (last-4 only); `.gitignore` hardened for
  `data/documents.md`, `data/journal.md`, and `knowledge/countries/*/cache/`.

### Changed
- Canonicalized the freshness stack on five domain keys
  (`processing_times/fees/forms/rules_guidance/policy_updates`) across
  `last_update.json`, AGENTS.md §2, packs, and timeline staleness checks.

---

## [1.0.0] — 2026-06-29

### Added
- `AGENTS.md` — master agent brain with startup protocol, registry
  architecture, confidence tier system, mode routing, hard rules,
  data contract, and response standards
- `modes/_shared.md` — character file, loaded every session
- `modes/_profile.md` — user customization file, survives updates
- `modes/intake.md` — progressive onboarding mode
- `modes/gap.md` — document gap analysis mode
- `modes/timeline.md` — timeline calculation and deadline mode
- `modes/pathway.md` — visa pathway assessment and scoring mode
- `modes/draft.md` — document drafting mode
- `modes/rfe.md` — RFE and refusal response mode
- `modes/scan.md` — portal scan mode
- `modes/batch.md` — multi-pathway batch comparison mode
- `modes/alert.md` — deadline alert mode
- `modes/verify.md` — fact verification mode
- `modes/evidence.md` — evidence inventory management mode
- `modes/audit.md` — pre-submission audit mode (4-check system)
- `modes/readiness.md` — application readiness scoring mode (0–100)
- `modes/attorney.md` — attorney consultation briefing mode
- `modes/challenge.md` — adversarial application review mode
- `registry/registry.yml` — 7-layer user registry template
- `evidence.md` — user evidence inventory template
- `data/cases.md` — flat file case tracker
- `knowledge/sources.yml` — official source index
- `knowledge/last_update.json` — knowledge currency tracker
- `knowledge/changelog.md` — fetched data change log
- Countries supported: United States (us), Canada (ca), United Kingdom (uk)
- CLI registration: Claude Code, Antigravity, Gemini, Grok, Kimi, Qwen, OpenCode
- `docs/DATA_CONTRACT.md` — agent data permission contract
- `docs/SETUP.md` — installation and first launch guide
- `docs/ARCHITECTURE.md` — system architecture documentation
- `docs/RECOVERY.md` — recovery and reset procedures
- `docs/GLOBAL_INSTALL.md` — global skill installation guide
- `docs/LEGAL.md` — legal disclaimer, data privacy, EU AI Act, acceptable use
- `modes/status.md` — registry and system health overview mode
- `modes/_custom.md` — free-form user behavioral instructions, agent read-only
- `knowledge/sources.yml` — official source index with 4-attempt fallback chain
  per entry (primary URL, fallback URL, web search query, last_known_good cache)
- `CHANGELOG.md` — this file
- `VERSION` — single-file version number
- `.gitignore` — security boundary for all personal data files
- `.gitkeep` files in output/, reports/, data/, registry/ — preserve empty dirs
- CLI registration: `.agents/`, `.claude/`, `.antigravitycli/`, `.grok/`,
  `.kimi/`, `.qwen/`, `.opencode/` — skill auto-discovery across 7 CLIs

### Design decisions
- Progressive registry: need-driven asking only, not interrogation
- Self-healing startup: knowledge fetched from official sources on
  every session, never answered from training data alone
- Three confidence tiers: VERIFIED / STALE / UNVERIFIED gate all responses
- Five hard rules encoded in _shared.md and enforced across every mode
- Local-only: no data leaves the user's machine
- Fetch resilience: 4-attempt fallback hierarchy — never blocks on source failure
- Language-first: _profile.md language setting applied across every mode
- User escape hatch: _custom.md gives users free-form behavioral control
- New country safety: .gitignore updated before any knowledge files written

---

## How to update this file

**When updating a mode:** Add an entry under the next version number.
State which mode, what changed, and why.

**When adding a country:** Add an entry noting the country code,
the official sources added to knowledge/sources.yml, and
the knowledge/[code]/ directory created.

**When fixing a bug:** Add a `### Fixed` entry describing the bug
and the file(s) changed.

**When making a breaking change:** Add a `### Changed` or
`### Removed` entry and note any migration steps the user needs.

**Version numbering:**
- Patch (1.0.x): bug fixes, wording improvements, minor mode updates
- Minor (1.x.0): new modes, new countries, new features
- Major (x.0.0): breaking changes to registry schema, data contract,
  or startup protocol

---

## [Unreleased] template

Copy this block when starting a new version entry:

```
## [X.X.X] — YYYY-MM-DD

### Added
-

### Changed
-

### Fixed
-

### Removed
-

### Breaking changes
-
```
