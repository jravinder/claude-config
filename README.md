# Claude Config

Personal Claude Code configuration. Clone to `~/.claude/` on any machine.

## Setup on a new machine

```bash
git clone git@github.com:jravinder/claude-config.git ~/.claude/config-repo
ln -sf ~/.claude/config-repo/skills/* ~/.claude/skills/
```

## Skills

| Skill | Purpose |
|-------|---------|
| `eng-workflow` | Issue-first, PR-based, context-saving engineering workflow |
| `research-clarity` | Attribution, jargon explanation, citation rules for research content |

## Sync

After updating skills on any machine:
```bash
cd ~/.claude/config-repo
git add -A && git commit -m "Update skills" && git push
```

On other machines:
```bash
cd ~/.claude/config-repo && git pull
```
