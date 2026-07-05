# Global Skill Installation

By default, Immos is a project-level skill — it only
activates when you open your CLI in the immos directory.

To make it available globally (from any directory on your machine),
copy or symlink the skill folder to your CLI's global skills location.

---

## Claude Code (global)

```bash
mkdir -p ~/.claude/skills/
cp -r .agents/skills/immos ~/.claude/skills/
```

## Antigravity CLI (global)

```bash
mkdir -p ~/.antigravitycli/skills/
cp -r .agents/skills/immos ~/.antigravitycli/skills/
```

## Gemini CLI (global)

```bash
mkdir -p ~/.gemini/skills/
cp -r .agents/skills/immos ~/.gemini/skills/
```

## Grok Build CLI (global)

```bash
mkdir -p ~/.grok/skills/
cp -r .agents/skills/immos ~/.grok/skills/
```

## Kimi (global)

```bash
mkdir -p ~/.kimi/skills/
cp -r .agents/skills/immos ~/.kimi/skills/
```

## Qwen CLI (global)

```bash
mkdir -p ~/.qwen/skills/
cp -r .agents/skills/immos ~/.qwen/skills/
```

---

## Or symlink (Unix/Mac) — stays in sync automatically

A symlink means you only maintain one copy. Changes to the skill
automatically apply everywhere.

```bash
# Claude Code
ln -s $(pwd)/.agents/skills/immos ~/.claude/skills/immos

# Gemini CLI
ln -s $(pwd)/.agents/skills/immos ~/.gemini/skills/immos

# Antigravity CLI
ln -s $(pwd)/.agents/skills/immos ~/.antigravitycli/skills/immos
```

After copying or symlinking, restart your CLI session.
The skill will be discovered automatically on next launch.

---

## Verify installation

In your CLI, ask: "What skills do you have available?"
Immos should appear in the list.

Or simply try:
```
/immigrate
```

---

## Important note on global installation

Global installation means the skill activates from any directory.
Your registry and evidence files still live in the immos
project folder — the skill reads them from that location.

If you move your project folder:
1. Update any symlinks to point to the new location
2. Or reinstall the skill from the new location with `cp -r`

The skill itself contains no user data. Moving the project folder
does not lose any of your registry, evidence, or case data —
those files travel with the folder.
