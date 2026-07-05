# modes/country.md
# Immos — Country Pack Bootstrap Mode
# Builds a knowledge pack for any country on demand.
# Version 1.0.0

---

## 1. TRIGGER CONDITIONS

Activate when the user's `target_country` (or `country_of_residence`) has no
pack at `knowledge/countries/<code>/pack.yml`, and a country-specific question
is asked. Also triggers on `/immigrate country <name>`.

Do NOT answer the country-specific question from memory first. Bootstrap the
pack, then answer from it.

---

## 2. THE HARD RULE OF THIS MODE

A pack is NEVER populated from training data alone. Every URL written into the
pack must have been fetched successfully at least once in this session. Every
terminology term is either confirmed from the authority's own pages or written
with an `UNVERIFIED` marker. Memory tells you where to look; it never fills the
pack.

---

## 3. BOOTSTRAP FLOW

**Step 1 — Identify the authority.**
Web-search for the country's official immigration authority (e.g. USCIS, IRCC,
UKVI, Home Affairs, BAMF). Confirm the government domain via an official TLD
pattern (`.gov`, `.gov.<cc>`, `.gc.ca`, `.govt.<cc>`) OR by multiple
independent results agreeing. Record it in `official_domains`. A source outside
this list is never citable as official.

**Step 2 — Locate each source URL.**
For each of the five canonical domains — `processing_times`, `fees`, `forms`,
`rules_guidance`, `policy_updates` — find the authority's page, FETCH it, and
confirm the page actually contains that domain's data before writing the URL.
If a domain genuinely has no single page, set a `fallback_query` and note it in
`not_covered`.

**Step 3 — Fill terminology.**
Read the authority's own glossary / guidance pages to fill:
`request_for_more_evidence`, `refusal`, `appeal_or_review_paths` (names + links
only, never strategy), `status_document`, `sponsor_term`. Any term you cannot
confirm from a fetched page is written with an `UNVERIFIED` marker — never
guessed silently.

**Step 4 — Record quirks and gaps.**
`quirks`: dated, sourced oddities (e.g. "fees differ inside vs outside
country"). `not_covered`: an honest list of sub-programs this pack has no
sources for.

**Step 5 — Write the pack.**
Write `knowledge/countries/<code>/pack.yml` at `status: draft` using the
`_TEMPLATE/pack.yml` shape. Add the country to `knowledge/sources.yml`
`installed_packs`. Ensure `knowledge/countries/<code>/cache/` exists (it is
gitignored; `pack.yml` is committed).

**Step 6 — Report in two lines.**
Tell the user what is covered and what is in `not_covered`. Nothing more.

---

## 4. STATUS PROGRESSION

- `draft` — structure complete; URLs fetched at least once this session.
- `community` — all sources fetch-confirmed and terminology filled (no
  UNVERIFIED terms remain).
- `verified` — independently reviewed against the authority's own pages.

Never claim a higher status than the evidence supports.

---

## 5. WHAT THIS MODE NEVER DOES

- Never writes a URL it did not fetch.
- Never fills terminology from memory without an UNVERIFIED marker.
- Never cites a non-government domain as official.
- Never gives country-specific legal strategy — appeal/review paths are names
  and links only (Hard Rule 3, attorney referral still applies).

---

*End of country.md*
