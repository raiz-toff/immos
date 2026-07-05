---
name: immos
description: >-
  Immigration preparation assistant. Activate for visa pathway
  assessment, document gap analysis, timeline calculation, document
  drafting, RFE response, pre-submission audit, readiness scoring,
  attorney briefing, and adversarial review. Trigger: "what visa",
  "documents I need", "how long will this take", "I got a refusal",
  "am I ready to apply", "audit my application", "readiness score".
  Data fetched from USCIS, IRCC, and Home Office — never from
  training data. User data stays local.
metadata:
  author: immos
  version: 1.0.0
---

# Immos — Claude Code

See canonical skill definition at:
`.agents/skills/immos/SKILL.md`

On activation: read `AGENTS.md`, run startup protocol,
load `modes/_shared.md` and `modes/_profile.md`, route
to the appropriate mode based on user input.

Slash command: `/immigrate [input or subcommand]`
Full command list: `.agents/skills/immos/SKILL.md`
