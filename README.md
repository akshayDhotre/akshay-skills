# akshay-skills

A collection of small, drop-in skills for AI coding agents (Claude Code, Cursor).
Each is pure markdown — no install, no runtime, no API keys. Copy the folder in and go.

## Skills

| Skill | What it does |
|---|---|
| [`overbuild-check`](skills/overbuild-check) | Before building, checks your codebase, installed deps, stdlib, and live package registries — recommends reuse over rebuild. |

## Install a skill

Each skill folder ships two forms of the same trigger:

- `SKILL.md` — for **Claude Code**
- `<name>.mdc` — for **Cursor**

**Claude Code** (user scope = all projects, or per project):

```bash
git clone https://github.com/akshayDhotre/akshay-skills
cp -r akshay-skills/skills/overbuild-check ~/.claude/skills/     # all projects
# or, just this project:
cp -r akshay-skills/skills/overbuild-check .claude/skills/
```

Restart Claude Code and ask "what skills do you have?" to confirm it loaded.

**Cursor** (per project):

```bash
mkdir -p .cursor/rules
cp akshay-skills/skills/overbuild-check/overbuild-check.mdc .cursor/rules/
```

## Adding a skill

Make a folder under `skills/` with a `SKILL.md` (Claude Code) and, optionally, a
matching `.mdc` (Cursor). Keep it markdown — a skill needs code only when it must do
something the agent's own tools (grep, read, web fetch) can't.

## License

Apache License 2.0 — see [LICENSE](LICENSE).

By **Akshay Dhotre** ([@akshayDhotre](https://github.com/akshayDhotre)).
