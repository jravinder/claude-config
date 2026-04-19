# Claude Config

How I work with Claude Code across all projects. Open source, clone to any machine.

## Hierarchy

```
Level 1: Global (this repo → ~/.claude/skills/)
  Skills and workflow rules that apply to every project.
  Public. Open source. Synced across machines.

Level 2: Per-repo (each repo's CLAUDE.md)
  Project-specific context: what it does, patterns, commands.
  Lives in the repo. Public or private depending on the repo.

Level 3: Coordination (GitHub Issues)
  Cross-repo work tracked via issues.
  Every piece of work ties to an issue.

Level 4: Memory (~/.claude/projects/*/memory/)
  Personal context: user preferences, project state, feedback.
  Private. Never committed. Machine-local.
```

## Skills

### eng-workflow
Engineering process for any project. Check status before starting, tie work to issues, save context every 30 min, commit frequently, create PRs, handoff protocol.

### research-clarity
Content rules for research projects. Attribution (who said what: published research vs AI hypothesis vs human decision), jargon explanation, citation format.

## Setup

```bash
# Clone to any machine
git clone git@github.com:jravinder/claude-config.git ~/.claude/config-repo

# Symlink skills
ln -sf ~/.claude/config-repo/skills/eng-workflow ~/.claude/skills/eng-workflow
ln -sf ~/.claude/config-repo/skills/research-clarity ~/.claude/skills/research-clarity
```

## Sync

After updating skills:
```bash
cd ~/.claude/config-repo
git add -A && git commit -m "Update skills" && git push
```

On other machines:
```bash
cd ~/.claude/config-repo && git pull
```

## What goes where

| What | Where | Visibility |
|------|-------|------------|
| Workflow rules, skills | This repo (`claude-config`) | Public |
| Project instructions | Each repo's `CLAUDE.md` | Matches repo |
| Work coordination | GitHub Issues on each repo | Matches repo |
| Personal memory, preferences | `~/.claude/projects/*/memory/` | Private, never committed |
| API keys, tokens | `~/.env` files, gitignored | Private, never committed |
