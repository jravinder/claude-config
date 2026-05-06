# claude-config

Portable Claude Code setup. Skills, protocols, and workflows that travel
with me across machines and projects. Clone to `~/.claude/` on any new
box and the same agents start with the same playbook.

The interesting part isn't that skills exist — Claude Code ships with the
mechanism. The interesting part is **what's in them**: protocols for the
multi-agent, multi-session, multi-week reality that the default behaviors
don't address.

## Why this exists

A single Claude Code session in a fresh sandbox is great. The problems
start when:

- You run Claude *and* Codex *and* Gemini against the same repo and they
  step on each other's work.
- A session ends, a new one starts, and the agent has no idea what was
  killed yesterday.
- An agent ships something that bypassed the validation gates because
  "Sample 500 looked good locally."
- The only audit trail is whatever conversation history happens to still
  be in your terminal scrollback.

These skills are the antibody. Each one was earned on a real project —
not synthesized from a blog post — and rewritten until the failure mode
that motivated it stopped showing up.

## Skills

Six skills, ordered by how often I load them.

### [`eng-workflow`](skills/eng-workflow/SKILL.md) · per-session loop
Run at the start of every session, every repo. `git status` → find or
create an issue → save context every 30 min to `docs/worklog.md` → commit
on milestone → PR + hand-off. Stops the "I forgot what I was doing
yesterday" failure mode.

### [`agent-coordination`](skills/agent-coordination/SKILL.md) · multiple agents, one repo
The protocol for when more than one agent (Claude / Codex / Gemini /
human) touches the same codebase across sessions. Six in-repo files
(`CLAUDE.md`, `AGENTS.md`, `LOG.md`, `HANDOFF.md`,
`experiments/version_pages/<v>.md`, `wiki/decisions/<n>.md`) plus
GitHub primitives (issues, PRs, labels, commit trailers). Includes
copy-paste templates for each artifact and the cold-start sequence every
agent must run before its first edit.

### [`builder-principles`](skills/builder-principles/SKILL.md) · how I operate
Loaded by any agent before writing code, making architecture calls, or
drafting comms. Sets the tone — concise, opinionated, ship-or-revert —
and points to the right deeper skill based on what's being built.

### [`multi-agent`](skills/multi-agent/SKILL.md) · five roles in parallel
For time-boxed builds (hackathons, demos, sprints): split work across
five named roles — Builder / Researcher / Content / QA / Ops — so you
don't bottleneck on a single track. Complements `agent-coordination`
(that one is N agents over weeks; this one is 5 roles over hours).

### [`hackathon`](skills/hackathon/SKILL.md) · 4–48 hour clock
What changes when the window is days, not weeks. Lock the idea early,
freeze scope hard, ship submission materials before the last hour.

### [`research-clarity`](skills/research-clarity/SKILL.md) · content rules
Attribution rules (published research vs AI hypothesis vs human
decision), jargon explanation, citation discipline, and a hard ban on
the "AI-generated essay" voice. Used when building research-facing
pages, papers, or explainers.

## Setup

```bash
# 1. Clone alongside ~/.claude/, not into it (it has machine-local state)
git clone git@github.com:jravinder/claude-config.git ~/.claude/config-repo

# 2. Symlink every skill into the runtime location
mkdir -p ~/.claude/skills
for s in ~/.claude/config-repo/skills/*/; do
  ln -sfn "$s" ~/.claude/skills/"$(basename "$s")"
done

# 3. Verify Claude Code sees them
ls -la ~/.claude/skills/
```

To pull updates on any machine:

```bash
cd ~/.claude/config-repo && git pull
```

Symlinks (not copies) so a `git pull` is the only update step — no rsync,
no re-symlink.

## How it fits with the rest of the stack

```
┌─────────────────────────────────────────────────────────────────┐
│  Repo: <project>/CLAUDE.md            (project rules + data)    │
│  Repo: <project>/AGENTS.md            (multi-agent protocol)    │
│  Repo: <project>/LOG.md               (append-only journal)     │
│  Repo: <project>/HANDOFF.md           (open hand-offs)          │
│  Repo: <project>/experiments/         (RFCs before code lands)  │
│  Repo: <project>/wiki/decisions/      (cross-agent arbitration) │
│                                                                 │
│  GitHub: issues / PRs / labels        (durable shared memory)   │
│                                                                 │
│  This repo: ~/.claude/skills/         (cross-project workflow)  │
│                                                                 │
│  Local:  ~/.claude/projects/*/memory/ (private user context)    │
└─────────────────────────────────────────────────────────────────┘
```

In priority order when there's ambiguity:
1. **Repo `CLAUDE.md`** — authoritative for that project.
2. **Repo `AGENTS.md`** — collaboration protocol for that project.
3. **Skills here** — workflow rules that apply to every project.
4. **`~/.claude/projects/*/memory/`** — personal context, never committed.

Skills here are level 3: tools the agent loads *on top of* whatever the
current repo says. They never override a repo's rules.

## A real-world snippet

What the protocol actually looks like in a project, drawn from
`agent-coordination`:

```bash
# An agent starting a session, running through the cold-start sequence:
$ cat CLAUDE.md           # rules + constraints
$ cat AGENTS.md           # protocol + swimlanes + claim/release
$ tail -30 LOG.md         # what happened recently, who did it
$ cat HANDOFF.md          # is anything waiting on me?
$ gh issue list --state open
$ git log --oneline -20

# After a meaningful action, the same agent appends to the journal:
$ echo "$(date -u +'%Y-%m-%d %H:%M') · claude · scored · v3.7.1 baseline \
on Mac native — Sample 500: 1.0000, Medium 5K: 0.9740 · /tmp/v371-runs/" \
>> LOG.md

# Every commit ends with a trailer for per-agent audit:
git commit -m "$(cat <<'EOF'
fix: revert v3.9 → v3.7.1 production base

<body explaining what regressed and why>

Agent: claude
EOF
)"

# Per-agent audit later:
$ git log --grep "Agent: gemini" --oneline
```

## Adding a skill

1. **Find a pattern that's repeated across more than one project.** If it
   only matters in one repo, it belongs in that repo's `CLAUDE.md`, not
   here.
2. `mkdir skills/<name>/` and write `SKILL.md` with frontmatter:

   ```yaml
   ---
   name: <name>
   description: When to use this — concrete triggers an agent can match.
   ---

   # <Skill title>

   <body — keep it terse and concrete>
   ```
3. Test by deleting the symlink and re-running step 2 of Setup, then
   start a fresh Claude session and confirm the skill loads.
4. Commit, push, run `git pull` on other machines.

## What's deliberately *not* in here

- **Project-specific patterns** (those go in the project's `CLAUDE.md`).
- **Personal preferences** (those go in `~/.claude/projects/*/memory/`,
  never committed).
- **API keys, tokens, secrets** (those go in `~/.env`, gitignored).
- **One-off scripts** (those belong in the project that needed them).

## Conventions

- **Commit message style**: imperative subject, body explains *why*,
  trailer identifies the agent (`Agent: claude`).
- **Skill descriptions** read like a trigger ("Use this when…"), not a
  taxonomy.
- **No theory without a battle scar.** Every pattern earned on a real
  project; if I can't name the failure it prevents, it doesn't ship.
