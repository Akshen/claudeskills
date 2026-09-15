# claude-skills

Personal library of Claude Code skills — portable across machines. Each skill lives in `skills/<skill-name>/` and follows the standard `SKILL.md` format Claude Code expects.

## Skills in this repo

- **robust-coding-practices** — Default engineering bar for all coding tasks: code quality, error handling, security, testing, performance, self-review checklist.
- **code-review** — Structured review mode for evaluating existing code/diffs/PRs (find and flag issues, don't silently rewrite).

## Setup on a new machine

1. Clone this repo:
```bash
   git clone https://github.com/<your-username>/claude-skills.git ~/claude-skills
```

2. Make sure Claude Code's skills directory exists:
```bash
   mkdir -p ~/.claude/skills
```

3. Copy each skill folder into place:
```bash
   cp -r ~/claude-skills/skills/robust-coding-practices ~/.claude/skills/
   cp -r ~/claude-skills/skills/code-review ~/.claude/skills/
```

4. Verify Claude Code sees them:
```bash
   ls ~/.claude/skills
```
   You should see both folders listed.

## Keeping skills in sync

Since skills are copied (not symlinked), edits made directly in `~/.claude/skills/` won't automatically flow back into this repo. Workflow:

- **To edit a skill**: edit the file inside `~/claude-skills/skills/<skill-name>/SKILL.md`, then re-copy it to `~/.claude/skills/<skill-name>/` to pick up the change locally, then commit + push from `~/claude-skills`.
- **To pull updates made on another machine**: `git pull` inside `~/claude-skills`, then re-run the `cp -r` commands above for any skill that changed.

## Adding a new skill

```bash
mkdir -p ~/claude-skills/skills/<new-skill-name>
# write SKILL.md inside it, following the format of existing skills
cp -r ~/claude-skills/skills/<new-skill-name> ~/.claude/skills/
cd ~/claude-skills && git add . && git commit -m "Add <new-skill-name> skill" && git push
```
