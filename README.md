# ImmigrationOps

ImmigrationOps is a personal immigration preparation assistant that runs
locally on your machine. It helps you understand your options, organize
your documents, track your deadlines, and prepare your applications —
without replacing you as the decision-maker and without filing anything
on your behalf.

---

## What ImmigrationOps Does

It prepares. You decide. You act.

Specifically, it:

- **Understands your situation** progressively — learning who you are and
  what you are trying to do through natural conversation over time
- **Assesses your pathways** — evaluates which visa routes are available
  to you and scores them based on your specific profile
- **Analyses your documents** — tells you what you have, what you are
  missing, and how to get what you need
- **Calculates your timeline** — builds a realistic milestone schedule
  backward from your deadline using current, fetched processing times
- **Drafts your documents** — produces draft cover letters, personal
  statements, employer letters, and other application documents for you
  to review and personalize
- **Tracks your deadlines** — monitors your active cases and surfaces
  critical dates before they become crises
- **Stays current** — fetches official processing times, fees, and policy
  updates from government sources rather than using outdated training data
- **Responds to setbacks** — helps you organize your response when you
  receive an RFE, a refusal, or a notice of intent to deny
- **Verifies claims** — checks any immigration fact you encounter — from
  forums, other AI tools, or anywhere else — against official sources

---

## What ImmigrationOps Does Not Do

**This is not a lawyer and cannot give legal advice.**
ImmigrationOps can organize facts, explain published requirements, and help
you prepare. It cannot assess the legal implications of your specific history,
advise you on how to handle a prior refusal or overstay, or make eligibility
determinations. When a situation requires legal judgment, it will tell you
clearly and recommend you consult an immigration attorney.

**It does not file anything on your behalf.**
No government portals. No submissions. No automated form filings.
ImmigrationOps prepares everything. You review it, you decide, you submit.
This is intentional — your immigration applications require your direct
authorization and carry your name. They should be yours.

**Its accuracy depends on what you tell it.**
ImmigrationOps builds a picture of your situation over time through
conversation. The more accurately you describe your situation, the more
accurate and useful its guidance will be. It will tell you when it needs
more information to give you a reliable answer.

---

## The Core Philosophy

Immigration errors compound. A wrong move today can create barriers for years.
ImmigrationOps prioritizes accuracy over speed — it fetches current information
from official government sources rather than relying on potentially outdated
training data. It acknowledges what it does not know rather than guessing.
It is honest about complexity rather than giving comfortable but imprecise answers.

---

## Setup

See [docs/SETUP.md](docs/SETUP.md) for complete setup instructions.

**Quick start:**
1. Install your preferred AI CLI (Claude Code, Gemini CLI, or Codex CLI)
2. Open a terminal in this directory
3. Run your AI CLI
4. Start talking — ImmigrationOps will guide you from there

On first launch, ImmigrationOps will introduce itself and begin learning
about your situation. You do not need to fill in any forms or configure
anything. Just describe what you are trying to do.

---

## Your Data

Everything stays on your machine. Always.

ImmigrationOps stores your information in local files:
- `registry/registry.yml` — your profile
- `data/cases.md` — your active cases
- `knowledge/` — fetched official data
- `output/` — your drafted documents

None of this is transmitted to any server. When the agent fetches data
from official government websites, it reads public information — it does
not send your personal information anywhere.

Your data survives system updates. The files above are never overwritten
by updates to the system. See [docs/DATA_CONTRACT.md](docs/DATA_CONTRACT.md)
for the complete data contract.

---

## How the System Learns

ImmigrationOps builds its knowledge of your situation gradually through
conversation. It does not start with a form to fill — it starts with a
question: what are you trying to do?

Over the first few sessions, it learns:
- Where you are and what status you hold
- Where you are trying to go and what you want to achieve
- What your timeline looks like
- What documents you have and what you are missing

It learns passively — picking up information you mention in passing,
not interrogating you for details. The more you use it, the more useful
it becomes.

---

## How to Get the Most Out of It

**Be specific.** General questions get general answers. The more detail
you share about your actual situation, the more precisely ImmigrationOps
can help you.

**Correct it when it's wrong.** If ImmigrationOps has a date wrong or
misunderstood your situation, say so. It will update its records immediately.

**Check the source.** Every factual immigration claim ImmigrationOps makes
comes with an inline citation: `[VERIFIED | source.gov | date]`. If you
want to verify something independently, the source is always right there.

**Use it for complex documents.** Draft letters, personal statements, and
cover letters take significant time. ImmigrationOps can produce a draft
that you then personalize — this is often much faster than starting blank.

**Do not rely on it alone for complex situations.** If your situation
involves prior refusals, immigration violations, criminal history, or
other significant complexity, ImmigrationOps will tell you when you need
an immigration attorney. When it does: believe it and find one.

---

*Built to prepare. Not to decide. Not to file.*
