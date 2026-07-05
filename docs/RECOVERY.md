# Recovery and Reset Procedures

This document covers what to do when things go wrong.
Every recovery procedure is designed to be conservative —
preserve what is recoverable, reset only what is broken.

---

## Quick Reference

| Problem | Recovery |
|---------|----------|
| Registry field is wrong | [Correct a field](#correct-a-registry-field) |
| HIGH confidence field overwritten | [Restore from git](#restore-an-overwritten-field) |
| Registry corrupted or unreadable | [Reset the registry](#reset-the-registry) |
| cases.md has duplicate or stale entries | [Clean cases.md](#clean-casesmd) |
| Knowledge fetch failing | [Force re-fetch](#force-a-knowledge-re-fetch) |
| Knowledge source URL broken | [Update a source URL](#update-a-broken-source-url) |
| evidence.md has wrong data | [Correct evidence](#correct-evidence-entries) |
| Full fresh start needed | [Factory reset](#factory-reset) |
| Start fresh but keep evidence | [Soft reset](#soft-reset-keep-evidence) |

---

## Correct a Registry Field

**When to use:** A field in the registry has the wrong value,
wrong confidence level, or wrong source type.

**How:**
Tell the agent directly in conversation.

"My visa expiry is actually March 2027, not January. Please correct
the registry entry for current_status.visa_expiry."

The agent will:
1. Find the current value and confidence level
2. Ask you to confirm the correction
3. Update the field with the new value, `source: user_stated`,
   and updated timestamp
4. If the existing confidence was HIGH, it records the conflict
   in a `conflict:` sub-field rather than silently overwriting

You can also edit `registry/registry.yml` directly in any text
editor. The format is YAML — change the `value:` field, update
`confidence:` to match how you know this fact, and update
`updated:` to today's date.

---

## Restore an Overwritten Field

**When to use:** A HIGH confidence field was overwritten incorrectly
and you want to restore the previous value.

**If you are using git:**

```bash
# See what the field looked like before
git log -p registry/registry.yml | grep -A 10 "field_name"

# Restore the entire registry file to a previous commit
git checkout [commit_hash] -- registry/registry.yml
```

**If you are not using git:**

The agent records conflicts in a `conflict:` sub-field when it
detects that a new value contradicts an existing HIGH confidence
value. Check `registry/registry.yml` for any `conflict:` blocks —
the previous value is preserved there.

Manually edit the field back to the correct value and remove the
`conflict:` block once resolved.

**Prevention:** Turn on git tracking for your immos folder.
Even though `registry/registry.yml` is gitignored (it should never
be pushed), git still tracks LOCAL changes — you can always revert
to any previous local state. Run `git init` in the project root
if you have not already.

---

## Reset the Registry

**When to use:** The registry file is corrupted, unreadable,
or so inaccurate that starting the profile fresh is cleaner.

**Option A — Full reset (lose all registry data):**

```bash
cp registry/registry.yml registry/registry.yml.bak
cp registry/registry.yml registry/registry-$(date +%Y%m%d).bak
```

Then replace `registry/registry.yml` with the clean template.
The clean template is in git:

```bash
git checkout HEAD -- registry/registry.yml
```

**Option B — Reset one layer only:**

Open `registry/registry.yml` in a text editor.
Find the layer you want to reset (e.g. `financial:`).
Set all `value:` fields under that layer back to `~`.
Set all `confidence:` fields back to `UNKNOWN`.
Set all `source:` and `updated:` fields back to `~`.
Set `layer_completeness:` back to `0%`.

The agent rebuilds any layer through normal conversation.

**After a registry reset:**
The agent will detect low completeness on next launch and
activate intake mode to rebuild what it needs. This is normal.
Your evidence.md and cases.md are not affected by a registry reset.

---

## Clean cases.md

**When to use:** cases.md has duplicate entries, stale cases,
or entries in the wrong format.

**Duplicate entries:**
Open `data/cases.md` in a text editor.
Find the duplicate case blocks (same case_id or same pathway).
Delete one. Keep the one with the most recent and accurate data.

**Stale cases:**
Change the `status:` field from `ACTIVE` to `COMPLETE` or `ON_HOLD`.
Never delete a case — status changes preserve the history.

**Wrong format:**
Tell the agent: "Please review cases.md and fix any formatting issues."
The agent will read the file, identify malformed entries, and
rewrite them in the correct schema format.

**Nuclear option — clear all cases:**

```bash
cp data/cases.md data/cases-$(date +%Y%m%d).bak
```

Then clear the file back to the initialized empty state with just
the schema comment header. The agent will rebuild cases from
scratch as you describe your situation.

---

## Force a Knowledge Re-fetch

**When to use:** Knowledge files are stale, fetch-failed, or
you want to force a fresh pull from official sources regardless
of the last_update.json timestamps.

**Tell the agent:**
"Force a full knowledge refresh for [country code]."

The agent will reset the timestamps for that country in
`knowledge/last_update.json` to the epoch value and run
the full fetch routine.

**Manual reset:**
Open `knowledge/last_update.json`.
Find the country block you want to refresh.
Change all `fetched_at` values to `"1970-01-01T00:00:00Z"`.
Change all `status` values to `"NEVER_FETCHED"`.
Restart your agent session. The startup protocol will detect
the epoch timestamp and trigger a fresh fetch automatically.

**Reset all countries:**

```bash
cp knowledge/last_update.json knowledge/last_update.json.bak
```

Then manually set all `fetched_at` values to the epoch.

---

## Update a Broken Source URL

**When to use:** A government website has restructured its URLs
and the agent's fetch routine is failing for a specific domain.

The agent tries three sources before giving up (primary URL →
fallback URL → web search query). If all three fail, it uses
the `last_known_good` cached data. If you are seeing persistent
failures, the primary and fallback URLs may both be stale.

1. Verify the correct current URL by visiting the official site
   directly and finding the page the agent was trying to reach.

2. Open `knowledge/sources.yml`.

3. Find the entry for this country and domain.

4. Update the `url:` field with the correct current URL.

5. Update the `fallback_url:` if it is also broken.

6. Force a re-fetch (see above) to verify the new URL works.

7. Add an entry to `CHANGELOG.md` noting the URL change and date.

**If the official source has moved permanently:**
Also update the `fallback_query:` field — this is the web search
query the agent uses when both primary and fallback URLs fail.
A good fallback query: `"site:gov.uk skilled worker visa processing times 2026"`

---

## Correct Evidence Entries

**When to use:** An entry in evidence.md has wrong dates,
wrong expiry, or wrong status.

**Tell the agent:**
"My IELTS score date is wrong — it was taken on [date], not [wrong date].
Please correct the evidence entry."

**Or edit directly:**
Open `evidence.md` in any text editor.
Find the entry under the relevant category.
Correct the field values.
Save the file.

The agent reads evidence.md fresh at the start of every mode
that uses it, so corrections take effect immediately on the
next session.

**If an expiry date was wrong:**
After correcting it, also check `data/cases.md` for any deadline
entries that were created from the wrong expiry. Update those too.

---

## Soft Reset — Keep Evidence

**When to use:** You want to start your immigration journey fresh
(new target, new country, changed goal) but want to keep your
evidence inventory because the documents themselves haven't changed.

This preserves:
- `evidence.md` — all your documents
- `data/cases.md` — your case history (mark old cases ON_HOLD)
- `CHANGELOG.md` — system history
- `knowledge/` — fetched government data

This resets:
- `registry/registry.yml` — rebuilt from scratch through intake
- `modes/_profile.md` — optionally reset preferences

**Steps:**

1. Back up everything first:
   ```bash
   cp registry/registry.yml registry/registry-$(date +%Y%m%d).bak
   ```

2. Reset the registry to the clean template (see Reset the Registry).

3. In `data/cases.md`, change status of all ACTIVE cases to ON_HOLD.

4. Open a new agent session. Intake mode will activate.
   Tell the agent your new goal and let it rebuild your profile.

5. As the agent asks questions, your evidence.md data will
   automatically feed into gap analysis and pathway scoring —
   even with a fresh registry.

---

## Factory Reset

**When to use:** Complete fresh start. Remove all personal data.
This is what you do before sharing the project or before uninstalling.

```bash
# 1. Back up everything (optional but recommended)
mkdir -p ../immos-backup-$(date +%Y%m%d)
cp registry/registry.yml ../immos-backup-$(date +%Y%m%d)/
cp evidence.md ../immos-backup-$(date +%Y%m%d)/
cp data/cases.md ../immos-backup-$(date +%Y%m%d)/
cp -r reports/ ../immos-backup-$(date +%Y%m%d)/
cp -r output/ ../immos-backup-$(date +%Y%m%d)/

# 2. Remove all personal data
rm registry/registry.yml
rm evidence.md
rm data/cases.md
rm -rf reports/*.md
rm -rf output/*
rm -rf knowledge/us/ knowledge/ca/ knowledge/uk/
rm knowledge/last_update.json
rm knowledge/changelog.md

# 3. Restore clean templates
git checkout HEAD -- registry/registry.yml
git checkout HEAD -- knowledge/last_update.json
```

After a factory reset, the next session starts completely fresh —
intake mode activates, the startup routine detects NEVER_FETCHED,
and it fetches current government data before the first response.

---

## Verify Recovery Worked

After any recovery procedure, run this check:

Tell the agent: `/immigrate status`

The status mode will read the registry, knowledge files, and cases
and report what it finds. If recovery was successful, you will see:

- Registry layer completeness values (may be 0% after a full reset)
- Knowledge currency status (NEVER_FETCHED on first session after reset)
- Active cases count (0 after factory reset)
- No error or parse failure messages

If the agent reports a parse error on `registry/registry.yml`,
the YAML formatting is broken. Open the file and look for
unclosed strings, wrong indentation, or invalid characters.

---

## Getting Help

If none of these procedures resolve the issue, the fastest path
is to open the relevant file directly in a text editor. All
Immos data files are plain text — YAML or Markdown.
They can be read and edited without any special tools.

If the agent itself is behaving unexpectedly (routing to wrong modes,
ignoring registry data, giving responses that contradict _shared.md),
the issue is almost always one of these:

1. A mode file was accidentally modified — restore from git
2. `_shared.md` was modified incorrectly — restore from git
3. The registry has a YAML parse error — fix the formatting
4. The session context is corrupted — start a new session

A new agent session always starts fresh. If behavior corrects
itself at the start of a new session, the issue was session-level,
not file-level.
