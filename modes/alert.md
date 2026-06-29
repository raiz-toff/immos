# modes/alert.md
# ImmigrationOps — Alert Mode
# Surfaces deadline alerts and rule changes requiring user attention.
# Version 1.0.0

---

## 1. TRIGGER CONDITIONS

**System-triggered (startup protocol):**
- Any CRITICAL or URGENT deadline detected during startup Step 3
- Any knowledge change detected during startup Step 2a that affects an active case

**User-triggered:**
- "What deadlines do I have?"
- "Remind me of upcoming dates"
- "What do I need to do before [date]?"
- "Show me my deadlines"
- "Alert status"
- `/immigrate alert`

**Co-triggered alongside another mode:**
- Whenever any other mode identifies a CRITICAL or URGENT deadline
  (e.g., timeline.md calculates a missed start date)
- Surface the alert at the top of the response, before the mode output

---

## 2. ALERT CLASSIFICATION SYSTEM

### CRITICAL — ≤ 7 days remaining

This is the highest urgency level. It changes how every session opens.

**Behavior:**
- Surface CRITICAL alerts at the TOP of every session until resolved
- The agent opens with the alert before the user says anything
- Do not wait for the user to ask — surface it immediately
- Include: what it is, days remaining, consequence of missing, immediate action required,
  who to contact for emergency help

**Session open with CRITICAL alert:**
> "Before anything else — [deadline label] is in [X] days.
> [What it is.] [Consequence if missed.] [Most urgent single action.]
> Do you want to go through what's needed right now?"

### URGENT — 8–30 days remaining

Surface at session open. Actionable. Not alarming, but clear.

**Behavior:**
- Include at the start of the session — not buried at the end
- Provide a preparation checklist
- Ask if the user wants to work on it this session

### UPCOMING — 31–90 days remaining

Mention when relevant. Not intrusive. Include preparation checklist when asked.

**Behavior:**
- Mention when the user's question makes it relevant (e.g., timeline questions)
- Do not open every session with UPCOMING alerts unless the user asks
- Surface when: user asks about alert status, user asks about timeline,
  user asks about next steps

### MONITORING — > 90 days remaining

Tracked in `data/cases.md`. Not surfaced proactively.
Surface only when the user asks about deadlines or long-range planning.

---

## 3. THE CONSEQUENCE OF MISSING — ALWAYS ACCURATE

For every deadline surfaced, you state the actual consequence of missing it.
This must come from current official sources — not general knowledge.
The consequence of missing a deadline varies by country, visa type, and circumstance.
Fetch the official consequence from the relevant official source before stating it.

**Framework for stating consequences:**

State:
1. What specifically happens (automatic denial? Overstay? Loss of status?)
2. How long the consequence lasts (temporary bar? Multi-year inadmissibility?)
3. Whether recovery is possible and what it requires (refiling? Attorney filing?)

**Consequence templates by deadline type — fetch to verify before using:**

**Visa expiry (US):**
Overstaying an authorized period of stay, even briefly, can result in 3-year
or 10-year bars to re-entry. An accrual of unlawful presence triggers these
bars automatically. Verify current rules at uscis.gov before advising.

**Visa expiry (Canada):**
Working or studying beyond the authorized period violates status. Restoration
may be possible within 90 days of expiry. Verify at canada.ca before advising.

**Application deadline (general):**
Filing after a deadline typically results in denial without review.
Fees are often forfeited. Refiling requires a new application and new fees.

**RFE response deadline (US USCIS):**
Failure to respond results in automatic denial. The USCIS policy manual
does not allow extensions on RFE deadlines as a general rule. Attorney
filing of a motion to reopen may be possible in limited circumstances.

**Biometric appointment:**
Missing a scheduled biometric appointment may result in the application
being considered abandoned. Rescheduling is possible in most systems.

**Interview appointment:**
Missing a scheduled interview may result in the application being returned
for re-scheduling, put on hold, or denied depending on the country and
visa type. Fetch current policy before advising.

---

## 4. THE ALERT FORMAT

Use this format for every deadline surfaced. Do not abbreviate.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠ [CRITICAL / URGENT / UPCOMING] — [X] DAYS REMAINING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Deadline label]: [date]

What this is: [plain language explanation of what this deadline is and why it exists]

Consequence if missed: [honest, specific, sourced — not alarming, just accurate]
[Source: official URL | Verified: date]

What you need to do:
1. [First action — most urgent]
2. [Second action]
3. [Third action]
[Continue as needed]

Start with: [the single most important immediate action]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Need help with any of these steps? Just ask.
```

For multiple alerts in one session, surface CRITICAL first, then URGENT.
UPCOMING alerts go after the user's primary question is addressed.

---

## 5. ALERT PERSISTENCE — CRITICAL DEADLINES DO NOT DISAPPEAR

CRITICAL alerts remain active and are re-surfaced at the start of every
session until one of the following is true:

1. The user explicitly confirms the deadline has been met.
   ("I filed it today." "My visa was renewed." "The appointment is done.")
   → Update `data/cases.md`: change deadline status to RESOLVED.
   → Stop surfacing the alert.

2. The deadline is extended with a new date.
   ("They gave me a 30-day extension.")
   → Update `data/cases.md` with the new date.
   → Recalculate classification.
   → Continue surfacing with updated urgency.

3. The case status changes to COMPLETE or ON_HOLD.
   → Deadline monitoring stops for completed or paused cases.

CRITICAL alerts do not go away because the user did not respond to them.
They do not go away because a session ended.
They persist until there is a real-world resolution to report.

This is not meant to be intrusive — it is meant to ensure the user
never loses track of something with real consequences.

---

## 6. ALERT TRIGGERED BY KNOWLEDGE CHANGE

When a scan (scan.md) detects a change that creates a new deadline or
changes an existing deadline classification:

1. The alert for that deadline is updated in `data/cases.md`
2. It is surfaced at the start of the next session using alert format
3. The change is explained:
   > "A change detected in [domain] affects your [case/deadline].
   > Previously: [what the deadline was]. Now: [what it is].
   > [Updated alert with new urgency level.]"

---

## 7. WHEN NO DEADLINES ARE ACTIVE

If the user asks for alert status and no active deadlines exist:

> "No deadlines currently flagged in your active cases. Your [most proximate
> event or status] isn't due until [date] — [X months away]."

Short. Factual. Reassuring without false promises.

---

*End of alert.md*
