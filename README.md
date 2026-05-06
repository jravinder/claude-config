# claude-config

My portable Claude Code setup. Skills, workflows, and protocols I keep
across machines and projects. Clone to `~/.claude/` on any new box and
the same agents start with the same playbook.

## Skills

| Skill | What it covers |
|---|---|
| **[agent-coordination](skills/agent-coordination/SKILL.md)** | Coordinating multiple AI agents (Claude / Codex / Gemini) on one repo via in-repo files (`AGENTS.md`, `LOG.md`, `HANDOFF.md`, `version_pages/`) and GitHub primitives (issues, PRs, labels, commit trailers). |
| **[builder-principles](skills/builder-principles/SKILL.md)** | How I operate when building. Read this before writing code or making architecture calls. |
| **[eng-workflow](skills/eng-workflow/SKILL.md)** | Standard per-session loop for any project — status check, issue link, work-log, commit cadence, PR + handoff. |
| **[hackathon](skills/hackathon/SKILL.md)** | What changes when the clock is 4–48 hours instead of weeks. |
| **[multi-agent](skills/multi-agent/SKILL.md)** | Parallel agents in five named roles (Builder / Researcher / Content / QA / Ops) for time-boxed builds. |
| **[research-clarity](skills/research-clarity/SKILL.md)** | Content rules for research-facing pages: attribution, jargon, citations, no slop. |

## Setup

```bash
# Clone into ~/.claude/ on a new machine
git clone git@github.com:jravinder/claude-config.git ~/.claude/config-repo

# Symlink every skill into the runtime location
for s in ~/.claude/config-repo/skills/*/; do
  ln -sfn "$s" ~/.claude/skills/"$(basename "$s")"
done

# Pull updates anywhere
cd ~/.claude/config-repo && git pull
```

## How it fits

The full Claude Code stack I use has four surfaces, in priority order:

1. **`CLAUDE.md` (repo-local)** — project rules, data reality, constraints. Authoritative for that repo.
2. **Skills (this repo)** — workflow rules and patterns that apply to *every* project. Public.
3. **GitHub (issues, PRs, AGENTS.md)** — coordination across sessions and across agents. See `agent-coordination`.
4. **`~/.claude/projects/*/memory/`** — personal context (preferences, project state, feedback). Never committed.

Skills here are level 2: tools the agent loads on top of whatever the
current repo says.

## Adding a skill

1. Find a pattern that's repeated across more than one project.
2. `mkdir skills/<name>/` and write `SKILL.md` with frontmatter:
   ```yaml
   ---
   name: <name>
   description: When to use this — concrete triggers an agent can match.
   ---
   ```
3. Commit, push, symlink on other machines (or re-run the loop above).

If a pattern only ever applies to one repo, it belongs in that repo's
`CLAUDE.md`, not here.

## Why this exists

I run multiple AI agents (Claude, Codex, Gemini) against the same projects
across sessions and machines. Without a shared protocol they each cold-start
into chaos — duplicate work, silent regressions, no audit trail. These skills
are the playbook that lets a fresh session pick up where the last one left
off, and lets two agents work the same repo without stepping on each other.

Every pattern here was earned on real projects, not theory.
