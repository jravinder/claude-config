# Engineering Workflow

Standard workflow for all projects. Every session, every repo.

## Before starting any work

1. **Check status first.** Before doing anything:
   - `git status` — any uncommitted work?
   - `gh pr list --state open` — any open PRs in this repo?
   - `git log --oneline -5` — what changed recently? Did Codex or another agent make changes?
   - Read CLAUDE.md if it exists — any updated instructions?

2. **Find or create an issue.** Every piece of work ties to an issue:
   - `gh issue list` — find a relevant open issue
   - If none exists, create one before starting
   - Reference the issue number in commits and PRs

## During work

3. **Save context every 30 minutes or at meaningful milestones.** Append to `docs/worklog.md`:

   ```markdown
   ## YYYY-MM-DD HH:MM — [Issue #N] Brief description

   **Done:**
   - What was accomplished

   **Learned:**
   - Any findings, surprises, or decisions

   **Next:**
   - What should happen next

   **Blockers:**
   - Anything blocking progress (if any)
   ```

4. **Commit frequently.** Small, meaningful commits. Reference the issue:
   ```
   Fix JSON parsing in experiment runner (#29)
   ```

5. **Don't overwrite other work.** If Codex or another agent modified a file:
   - Read the current state first
   - Understand what changed and why
   - Build on top of it, don't revert it

## After meaningful work

6. **Create a PR.** Never push directly to main. PR description includes:
   - What changed
   - Why (link to issue)
   - How to test

7. **Update docs.** If the work changes how things work:
   - Update CLAUDE.md
   - Update relevant docs/
   - Close or update the issue

8. **Write findings.** If research or experiments produced new knowledge:
   - Write to `docs/findings/` with date-stamped files
   - Reference in the issue

## Handoff protocol

When ending a session or handing off:

1. Commit all work to a branch (even if incomplete)
2. Append status to `docs/worklog.md`
3. List any running background tasks
4. Note which issues are in progress vs done
5. Push the branch
