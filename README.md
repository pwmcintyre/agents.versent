# agents.versent

This repository holds company-specific agent configuration for Peter's work at Versent. It layers on top of the personal assistant config in `~/git/agents`.

## how it works

When you run `opencode` from this directory, it loads:

1. **Global personal context** — `~/.config/opencode/AGENTS.md` (symlink to `~/git/agents/AGENTS.home.md`)
2. **Company context** — `AGENTS.md` in this repo (picked up as the project-level rules file)
3. **Company skills** — `.opencode/skills/*/SKILL.md` in this repo (project-local, not active elsewhere)

## contents

- `AGENTS.md` — company context: Versent info, working norms, projects, tools
- `.opencode/skills/` — company-specific agent skills

## adding a skill

```sh
mkdir ~/git/agents.versent/.opencode/skills/<name>
# write ~/git/agents.versent/.opencode/skills/<name>/SKILL.md
```

Skills placed here are **project-local** — they only activate when opencode starts from this directory.

## working in this repo

When inside this directory your primary tasks are:
- Editing `AGENTS.md` to add or refine company context
- Adding or updating files in `.opencode/skills/`
- Committing changes to back them up
