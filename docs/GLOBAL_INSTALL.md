# Global Skill Installation

By default, ImmigrationOps is a project-level skill — it only
activates when you open your CLI in the immigration-ops directory.

To make it available globally (from any directory on your machine),
copy or symlink the skill folder to your CLI's global skills location.

---

## Claude Code (global)

```bash
mkdir -p ~/.claude/skills/
cp -r .agents/skills/immigration-ops ~/.claude/skills/
```

## Antigravity CLI (global)

```bash
mkdir -p ~/.antigravitycli/skills/
cp -r .agents/skills/immigration-ops ~/.antigravitycli/skills/
```

## Gemini CLI (global)

```bash
mkdir -p ~/.gemini/skills/
cp -r .agents/skills/immigration-ops ~/.gemini/skills/
```

## Grok Build CLI (global)

```bash
mkdir -p ~/.grok/skills/
cp -r .agents/skills/immigration-ops ~/.grok/skills/
```

## Kimi (global)

```bash
mkdir -p ~/.kimi/skills/
cp -r .agents/skills/immigration-ops ~/.kimi/skills/
```

## Qwen CLI (global)

```bash
mkdir -p ~/.qwen/skills/
cp -r .agents/skills/immigration-ops ~/.qwen/skills/
```

---

## Or symlink (Unix/Mac) — stays in sync automatically

A symlink means you only maintain one copy. Changes to the skill
automatically apply everywhere.

```bash
# Claude Code
ln -s $(pwd)/.agents/skills/immigration-ops ~/.claude/skills/immigration-ops

# Gemini CLI
ln -s $(pwd)/.agents/skills/immigration-ops ~/.gemini/skills/immigration-ops

# Antigravity CLI
ln -s $(pwd)/.agents/skills/immigration-ops ~/.antigravitycli/skills/immigration-ops
```

After copying or symlinking, restart your CLI session.
The skill will be discovered automatically on next launch.

---

## Verify installation

In your CLI, ask: "What skills do you have available?"
Immigration-Ops should appear in the list.

Or simply try:
```
/immigrate
```

---

## Important note on global installation

Global installation means the skill activates from any directory.
Your registry and evidence files still live in the immigration-ops
project folder — the skill reads them from that location.

If you move your project folder:
1. Update any symlinks to point to the new location
2. Or reinstall the skill from the new location with `cp -r`

The skill itself contains no user data. Moving the project folder
does not lose any of your registry, evidence, or case data —
those files travel with the folder.
