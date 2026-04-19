# Multi-Agent Build

How to run parallel agents so you don't bottleneck on one track.

## The 5 Agents

### Agent 1: Builder
Owns core product code. Runs silently, only interrupts when blocked.
"Build X feature. Use the existing stack. Don't refactor. Ship it."

### Agent 2: Researcher
Owns user pain points, competitive landscape, data sources.
Runs before and during the build.
"Find the top 5 complaints about X. Be specific, cite sources."

### Agent 3: Content
Owns README, website copy, demo script, submission materials.
Runs parallel to builder. Doesn't need working code to write the story.

### Agent 4: QA / Demo Prep
Owns screenshots, demo video script, edge case testing.
Runs in the last 90 minutes.
"Take screenshots. Write a 2-min demo script. Find 3 things that could break."

### Agent 5: Ops
Owns deployment, git hygiene, service health.
Runs on demand.

## Orchestration

```
Phase 1:  Researcher runs first — locks the idea
Phase 2:  Builder + Content run in parallel
Phase 3:  QA/Demo takes over
On demand: Ops deploys on your command
```

**Rule:** Never let the builder touch copy. Never let the content agent touch code. Context contamination kills both.

## Your Role

You are the product manager:
- Lock the idea after research
- Unblock agents when stuck
- Make the call on ambiguity
- Review the demo before shipping

## Lessons Learned

- One agent owns git pushes. Others write files, one agent commits.
- Config files (API keys, endpoints) created on Day 0 before anyone writes code.
- Submission forms filled with placeholders on Day 0, updated at end.
- Demo video scripted and recorded before the last hour, not during it.
- Serve from repo directly. No copying files to secondary locations.
