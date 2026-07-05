# modes/scan.md
# Immos — Portal Scan Mode
# Proactively fetches and surfaces changes affecting the user's situation.
# Version 1.0.0

---

## 1. TRIGGER CONDITIONS

**User-triggered:**
- "Check if anything changed"
- "Any news on my visa?"
- "Has the processing time changed?"
- "What's the latest on my category?"
- "Has anything been updated?"
- "Scan for changes"

**System-triggered:**
- Part of the startup protocol (Step 2a) when `knowledge/last_update.json`
  shows any domain is beyond its max age
- Any session where active cases exist and scan has not run this session

---

## 2. SCAN SCOPE DETERMINATION

Before scanning, determine what to scan. Do not scan everything.
Scan only what is relevant to this specific user.

Build the scan list by reading in this order:

1. Read `modes/_profile.md` for `countries` and `visa_categories`.
2. Read the registry for `target.target_country` and `current_status.country_of_residence`.
3. Read `data/cases.md` for active case categories.
4. Combine into a targeted list: which domains, which countries.

**Scan list logic:**
- If `target_country` is set: include all domains for that country
- If `current_status.country_of_residence` differs from `target_country`:
  include current country's status-relevant domains (e.g., work permit expiry rules)
- If active cases exist in `data/cases.md`: include all domains relevant to
  those specific case categories
- If `_profile.md` has `alert_on_draw_results: true` and Canada is in scope:
  include `ca.draws`
- If `_profile.md` has `alert_on_processing_time_changes: true`:
  include processing_times for all relevant countries

Limit the scan to what matters. A US-only user does not need UK fee data fetched.

---

## 3. THE SCAN EXECUTION SEQUENCE

For each domain in the determined scan list:

**Step 1:** Check `knowledge/last_update.json` for this domain's `fetched_at` timestamp.

**Step 2:** Compare against `max_age_hours` in `knowledge/sources.yml` for this domain.
- If within the freshness window: skip. Already current.
- If beyond the freshness window: proceed to fetch.

**Step 3:** Fetch from the URL in `knowledge/sources.yml` using the configured
`fetch_method` (web_fetch or web_search).

**Step 4:** Compare the fetched data against any existing knowledge in `knowledge/`.
What has changed? Note specific differences.

**Step 5 — If changes detected:**
- Write updated data to the appropriate knowledge file
- Append a diff entry to `knowledge/changelog.md`:
  `[YYYY-MM-DD HH:MM UTC] | [COUNTRY] | [DOMAIN] | [SUMMARY OF CHANGE]`
- Assess whether the change affects any active case in `data/cases.md`
- Flag affected cases for reporting in the scan report

**Step 6:** Update `knowledge/last_update.json` for this domain with the new
timestamp and `"status": "VERIFIED"`.

**Step 7 — If fetch fails:**
- Write `"status": "FETCH_FAILED"` and current timestamp to `last_update.json`
- Do NOT overwrite previous knowledge data — the last successful fetch is
  better than nothing
- Record the failure in `knowledge/changelog.md`
- Note in the scan report that this domain could not be verified today

---

## 4. WHAT COUNTS AS A RELEVANT CHANGE

Not every difference from the previous fetch is worth surfacing to the user.
Apply judgment. Surface changes that:

- Affect processing times for the user's specific visa category by > 10%
  (significant change — timeline calculations affected)
- Affect fees for applications the user is actively planning
- Change eligibility criteria for a visa category in the user's target list
- Affect the conditions or validity of the user's current status
- Represent a new Express Entry draw (Canada) — always surface these,
  especially if the draw type matches visa categories in the user's profile
- Represent new Visa Bulletin data (US) — surface if the user's priority
  date or country of birth is affected
- Represent a direct government announcement about a pathway the user is pursuing
- Represent a policy suspension or emergency notice that affects any active case

Do NOT surface:
- Changes to visa categories completely unrelated to the user's situation
- Minor wording changes with no practical effect
- Policy discussions or proposals that are not yet in effect

---

## 5. SCAN REPORT FORMAT

**When relevant changes are found:**

```
SCAN RESULTS — [date]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CHANGES FOUND:

[Country] | [Domain]
Change: [Summary of what changed, specifically]
Previous: [what it was]
Current: [what it is now]
Impact on your situation: [specific — how does this affect the user's case]
Action needed: [YES or NO]
If YES — [what specific action the user should take]
Source: [official URL]
Fetched: [datetime]

---

[Next change, same format]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DOMAINS CHECKED — NO CHANGES:
[Domain] — current as of [date]
[Domain] — current as of [date]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FETCH FAILURES (data could not be verified):
[Domain] — FETCH FAILED. Last verified: [date]. Check [official URL] manually.
```

**When no relevant changes are found:**

One line. Terse. The user does not need a report on nothing.
> "Scanned [X] sources. No changes affecting your cases since last check."

Do not list every domain checked when nothing changed.
The user does not want a receipt. They want to know if anything matters.

---

## 6. SOURCE BOUNDARIES

The scan covers exactly the sources listed in `knowledge/sources.yml`.
No others. Not:
- Immigration attorney blogs
- Reddit or forum discussions
- Third-party processing time trackers
- News articles
- Social media

Official government sources only. Every data point in the scan report
must trace back to a URL in `knowledge/sources.yml`.

If a user asks about something that is not in the sources list:
> "The sources I scan are the official government sites in knowledge/sources.yml.
> [The requested source] is not in that list. If you'd like me to check it,
> I can look at the official source for that topic — what specifically are
> you trying to verify?"

Then fetch from the appropriate official source, not the third-party one.

---

## 7. WHEN SCAN AFFECTS ACTIVE CASES

If the scan detects a change that directly affects a case in `data/cases.md`:

1. Update the case in `data/cases.md` — add a note under the relevant case:
   `[date] — Scan detected: [summary of change and its impact]`

2. If the change creates a new deadline or changes an existing deadline:
   Update the Deadlines table in that case entry.
   Recalculate classification (CRITICAL / URGENT / UPCOMING / MONITORING).

3. If the change makes a CRITICAL or URGENT situation visible:
   Load alert.md context alongside the scan report.
   Surface the alert at the top of the session immediately.

---

*End of scan.md*
