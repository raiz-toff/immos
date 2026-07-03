# SETUP.md — ImmigrationOps Technical Setup Guide
# Version 1.0.0

---

## Prerequisites

You need one of the following AI CLI tools installed on your machine.
ImmigrationOps works with all three:

### Option A — Claude Code (Recommended)
Claude Code is Anthropic's official CLI. It has strong file read/write
capabilities and integrates well with this system's slash commands.

**Install:** Follow the instructions at [claude.ai/code](https://claude.ai/code)
You will need an Anthropic account and API access.

### Option B — Gemini CLI
Google's Gemini CLI.

**Install:** Follow the instructions at [ai.google.dev/gemini-api/docs/gemini-cli](https://ai.google.dev/gemini-api/docs/gemini-cli)

### Option C — Codex CLI (OpenAI)
OpenAI's Codex or assistant-based CLI.

**Install:** Follow the instructions at [platform.openai.com](https://platform.openai.com)

---

## Installation

ImmigrationOps is a directory-based system. There is nothing to install
beyond cloning or copying the project folder to your machine.

**Step 1:** Ensure the project folder is on your local machine.
You should have this file at:
```
immigration-ops/docs/SETUP.md
```

**Step 2:** Verify the directory structure is complete.
Run this in your terminal from the project folder:
```bash
ls -la
```
You should see: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `CODEX.md`, `README.md`,
and the directories `modes/`, `registry/`, `knowledge/`, `data/`, `output/`,
`reports/`, `docs/`.

**Step 3:** Install your AI CLI if you have not already (see Prerequisites).

---

## Configuration

ImmigrationOps does not require configuration before first use.

**Optional pre-configuration:**
If you already know your countries of interest and visa categories,
you can open `modes/_profile.md` and fill in:

```yaml
countries: [us]        # or [ca], [uk], or [us, ca, uk]
visa_categories: [h1b] # the specific visa type you are pursuing
```

This tells the system which knowledge to fetch on startup. If you
do not set this, the system will learn your countries of interest
through conversation.

**API Keys:**
Your AI CLI requires its own API key/credentials — configure those
in the CLI's own settings, not here. ImmigrationOps does not handle
API keys.

---

## First Launch

**Claude Code:**
```bash
cd /path/to/immigration-ops
claude
```

**Gemini CLI:**
```bash
cd /path/to/immigration-ops
gemini
```

**Codex / OpenAI:**
```bash
cd /path/to/immigration-ops
codex
```

On first launch, the system will:

1. **Read its configuration files** — this is silent and takes a moment
2. **Check knowledge currency** — on first launch, all knowledge is flagged
   as `NEVER_FETCHED`. The agent will begin fetching current data from
   official government sources for your relevant countries. This may take
   30–60 seconds as it retrieves processing times, fees, and policy updates.
3. **Open intake mode** — since your registry is empty, the agent will
   introduce itself and begin asking about your situation.

**What to expect:**

The agent will greet you with something like:
> "I'm here to help you understand your immigration options, organize
> what you need, and keep track of your progress. To get started —
> what are you trying to do?"

Answer naturally. You do not need to format your response in any
particular way. The agent will take it from there.

---

## Troubleshooting

### Problem: "I don't see any output after launching"
Check that you launched the CLI from inside the `immigration-ops` directory.
The CLI needs to be able to read `AGENTS.md` and the other project files.

### Problem: "The agent doesn't seem to know my country/visa"
If `modes/_profile.md` has empty `countries: []`, the agent will ask
about your situation before fetching knowledge. This is expected behavior.
Tell the agent your target country and visa type and it will update the profile.

### Problem: "Knowledge fetch seems to be failing"
Some official government sites can be slow or temporarily unavailable.
If a fetch fails, the agent will note it and continue with cached data
(if any exists) or acknowledge the gap. Retry in the next session.
Check your internet connection and that you are not behind a firewall
that blocks the relevant government domains.

### Problem: "The agent asked me something I already told it"
This may mean the registry write from that session did not persist.
Check `registry/registry.yml` to see if the field was written.
If it is empty, tell the agent again — it will re-record it.

### Problem: "I made a mistake in my profile and need to correct it"
Tell the agent directly: "My visa expiry is actually [date], not [date]."
It will handle the conflict protocol and update the registry.
You can also edit `registry/registry.yml` directly if you are comfortable
with YAML format.

### Problem: "Something seems corrupted or wrong with the system"
See the reset procedure below.

---

## How to Reset

**Reset your profile only (keeps system files intact):**
Delete the contents of `registry/registry.yml` below the header comments
and restore the null/UNKNOWN initialization values, OR delete the file
and create a fresh one from the template structure in `registry/registry.yml`.

**Reset knowledge (forces fresh fetch on next launch):**
Edit `knowledge/last_update.json` and set all `fetched_at` values back to
`"1970-01-01T00:00:00Z"` and all `status` values to `"NEVER_FETCHED"`.
The agent will refetch everything on next launch.

**Reset cases (clear all tracked cases):**
Edit `data/cases.md` and delete all case blocks (keep the header schema comment).

**Full reset (start completely fresh):**
Delete all files in `registry/`, `knowledge/`, `data/`, `output/`, `reports/`
and re-run the builder agent on `Builder_agent.md` to recreate the templates.
Your conversation history (in the AI CLI) is separate from these files.

**Note:** You should not need to reset the system under normal circumstances.
The agent is designed to handle corrections and updates gracefully through conversation.

---

## Next Steps After Setup

Once you have had your first session:

1. **Continue the conversation** — add more sessions to build a richer profile
2. **Check your generated files** — look at `registry/registry.yml` to see
   what the agent has learned about you
3. **Review active cases** — if you have an active application, open
   `data/cases.md` to see tracked cases and deadlines
4. **Customize your profile** — open `modes/_profile.md` to see and adjust
   your communication preferences

---

*End of SETUP.md*
