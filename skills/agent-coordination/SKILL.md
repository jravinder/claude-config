---
name: agent-coordination
description: Use this skill when multiple AI agents (Claude / Codex / Gemini / etc.) work on the same repo across separate sessions or sandboxes and need durable shared state. Set up the in-repo files (AGENTS.md, LOG.md, HANDOFF.md, version_pages, wiki/decisions/) and use GitHub primitives (issues, PRs, comments, labels, commit trailers) as the coordination layer. Trigger when a project has more than one agent contributing, when agents keep losing context between sessions, when a session "starts cold" with no idea what the previous agent did, or when an agent silently shipped something that wasn't validated. Complements the per-role `multi-agent` skill — that one is "5 agents at the same time"; this one is "N agents over weeks, durable across crashes."
---

# Agent Coordination via GitHub

How multiple agents share state when sandboxes are ephemeral and context windows are private. The repo and the GitHub API are the only durable surfaces both agents can see.

## In-repo files (the protocol layer)

These six files live in the repo and are read at the start of every session before the first edit. Together they're the protocol + journal + baton + RFC system.

| File | Purpose | Read order |
|---|---|---|
| `CLAUDE.md` | Project rules, data reality, constraints. Authoritative. | 1 |
| `AGENTS.md` | Collaboration protocol. Cold-start checklist, swimlanes, claim/release rules, gates. | 2 |
| `LOG.md` | Append-only journal. One line per meaningful action. | 3 (last 30 lines) |
| `HANDOFF.md` | Open hand-offs. Empty below the divider = nothing waiting. | 4 |
| `experiments/version_pages/<v>.md` | RFC-before-code for any architectural change. Hypothesis + kill criterion + predicted result committed BEFORE the run. | 5 |
| `wiki/decisions/<n>-<topic>.md` | Where cross-agent disagreements get arbitrated. Both positions, impact, decision. | 6 |

Plus commit trailers — every commit ends with `Agent: <name>` so `git log --grep "Agent: <name>"` is the per-agent audit.

## GitHub primitives (the coordination layer)

GitHub gives you a coordination layer that survives sandbox crashes, branch deletes, and machine-local memory loss.

| Primitive | Best for | How agents use it |
|---|---|---|
| **Issues** | Long-running TODOs, named open problems | Each issue = one open question. Labels for owner (`agent:claude`, `agent:codex`), status (`probe`, `gated`, `dead-end`). Close only when verified, not when the patch lands. |
| **PRs** | Gated changesets | Every meaningful change goes through a PR with a description + the version_page link. No silent main commits. |
| **PR comments** | Async discussion | One agent opens a PR; another reviews; comments are the conversation. Survives across sessions. |
| **Issue comments** | Running notes during a long task | Append progress to the issue; later agents resume mid-task. Far better than session-only memory. |
| **Labels** | Routing | `gate:1`, `gate:2`, `held`, `shipped`, `rejected`, `agent:<name>`. |
| **Milestones** | Version bookkeeping | One milestone per shipped version; close when archived. |
| **GitHub Actions** | Automated checks | Lints, tests, smoke-runs. The harness can't be argued with by an agent. |

GitHub primitives win over in-repo files when:
- The state needs to survive a branch being deleted
- Multiple agents need to see the same async update without rebasing
- You want a permission boundary (issues are open to commenters; files require write)
- You want the audit to be queryable (`gh issue list --search "agent:..."`)

In-repo files win over GitHub primitives when:
- The state is the *current truth* an agent needs to see at HEAD (HANDOFF.md, AGENTS.md)
- The state belongs WITH the code (version_pages live next to the experiment)
- You want it to ride in a PR diff so reviewers see it changing

## Cold-start sequence (every agent, every session)

```
1. Read CLAUDE.md
2. Read AGENTS.md
3. tail -30 LOG.md       ← what the other agent (or you) just did
4. Read HANDOFF.md       ← is there work waiting?
5. gh issue list --state open
6. git log --oneline -20
7. Only THEN start work.
```

## Hot-end sequence (every agent, every meaningful action)

```
1. Append a line to LOG.md
   Format: YYYY-MM-DD HH:MM · agent · verb · target · ref
2. Commit with trailer:
       <subject>

       <body>

       Agent: <name>
3. If you're handing off, write a section in HANDOFF.md.
4. If you opened a question for the next agent, file an issue.
5. Decision-shaped change? Open wiki/decisions/<n>.md, link from the PR.
```

## Where this pattern breaks down (and how to mitigate)

The pattern works when agents follow it. It silently fails when they don't:

- **Agent ships a candidate that bypassed the gates.** The fix isn't a stronger AGENTS.md — it's pre-registered hypotheses with kill criteria committed *before* the run, so post-hoc rationalization isn't available. Make the version page mandatory and dated.
- **Agent goes silent for days.** No LOG.md entries, no commits, no HANDOFF.md update — but artifacts appear in the working tree. Mitigation: every long-running probe gets an issue with a label and the agent is required to comment on it daily.
- **Agent ignores the cold-start checklist.** Reads CLAUDE.md but skips LOG.md, ends up duplicating work the previous agent already killed. Mitigation: the cold-start sequence has to be the literal first thing in every agent's system prompt.
- **Audit log corroborates a lie.** A prompt-injected agent could write "task complete" to the journal even when the task regressed. Mitigation: trust the **external scoring artifact** (CI run, test harness, hidden test) — not the agent's claim. The journal records the agent's *claim*, not the truth.

## Templates

### `AGENTS.md` skeleton

```markdown
# AGENTS.md — Collaboration Protocol

Authoritative over everything in here is `CLAUDE.md`.

## Who's on the repo
| Agent | Identity | Commit trailer |
|---|---|---|
| <agent A> | <CLI/web> | `Agent: a` |
| <agent B> | <CLI/web> | `Agent: b` |

## Cold-start checklist
1. Read CLAUDE.md
2. Read AGENTS.md
3. tail -30 LOG.md
4. Read HANDOFF.md
5. gh issue list --state open
6. git log --oneline -20

## Default swimlanes
- <agent A> → <UI / docs / git workflow / ...>
- <agent B> → <numerical / tests / probes / ...>

If a task is unclaimed, either agent may pick it up.

## Task claim protocol
Use the TaskList tool. Set `owner: <agent>` and `status: in_progress` BEFORE editing.
Release with `status: pending` if you can't finish.

## Logging
Every meaningful action: one line in LOG.md.
Format: `YYYY-MM-DD HH:MM · agent · verb · target · ref`

## Disagreement protocol
Don't silently revert another agent's work. Open `wiki/decisions/<n>-<topic>.md`
with both positions; user arbitrates.

## What not to do
- Don't push to main / make public without user green-light.
- Don't submit to external grading / production without user green-light.
- Don't skip hooks (--no-verify) or change git identity.
- Don't rewrite the other agent's recent commits.
```

### `LOG.md` skeleton

```markdown
# LOG.md
Append-only journal. One line per significant action.
Format: YYYY-MM-DD HH:MM · agent · verb · target · ref
---

YYYY-MM-DD HH:MM · <agent> · <verb> · <target> · <ref>
YYYY-MM-DD HH:MM · <agent> · <verb> · <target> · <ref>
```

### `HANDOFF.md` skeleton

```markdown
# HANDOFF.md
Open hand-offs. Delete the section when you start the work.
---

## <short title>
**From:** <agent>
**To:** <agent> | either
**Opened:** YYYY-MM-DD HH:MM
**Status:** what's already done, one or two lines.
**Hand-off:** what the next agent should do, specifically.
**Gotchas:** stale files, hidden deps, failed attempts.
**Files touched so far:** list of paths.
```

### Version page (`experiments/version_pages/<v>.md`)

```markdown
# v<X> — <one-line headline>

**Base version:** <prior version with verified result>
**Author:** <agent>
**Status:** REGISTERED. No code in main. No external submission.

## Hypothesis (testable, falsifiable)

## Kill rule (committed BEFORE the run)
e.g. "If gate <N> does not improve over baseline by <margin>, stop."

## Predicted result band (forecast)
e.g. "<metric> = <value> ± <range>"

## Failure mode targeted by name
not "improves <thing>" — name the specific case or tier.

## Counter-evidence sought
what would convince you the change is wrong.

## Gates (in order, with required result)
1. Probe: <expected outcome>
2. <small scale>: <required result>
3. <medium scale>: <required result>
4. <large scale>: <required result>

## Hard stop rules
- No external compute until gate <N>.
- No external submission until gate <N>.
- No production-config edit until gate <N>.

## Records (filled as gates run)
| Gate | Status | Result |
|---|---|---|
| 1 | not run | — |

## Decision
Pending Gate 1.
```

### `wiki/decisions/<n>-<topic>.md`

```markdown
# Decision <n>: <topic>

**Opened:** YYYY-MM-DD by <agent>
**Status:** open | resolved

## Position A (from <agent>)
- claim
- reasoning
- impact if chosen

## Position B (from <agent>)
- claim
- reasoning
- impact if chosen

## User decision
<filled by user>
```

## Why this beats "let's just talk in chat"

- Every write is a commit. Sandboxes are disposable. Branches survive crashes.
- `git log --grep "Agent: <name>"` is a native audit trail.
- Issues outlive sessions. PR comments outlive sandboxes. Labels outlive IPs.
- New agents (or new humans) cold-start from the same surface.
- The user's role becomes "review and arbitrate," not "be the message bus."

## Why this works

GitHub is already a coordination layer. Issues outlive sessions, PRs gate
changes, commit history is the audit log, branches isolate experiments.
Pairing those primitives with a small set of in-repo protocol files gives
you durable shared state across agents without standing up new infra.

Every piece of this was earned on real projects — one agent silently shipped
a regression, another duplicated work that had already been killed, a third
went dark for days. The protocol is the antibody to each of those.
