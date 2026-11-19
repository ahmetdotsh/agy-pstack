### Opening a PR

Invoked at the end of every other playbook.

Work from a git worktree (or isolated `Workspace: "share"` checkout) off trunk, and keep unrelated work out of it. Rebase into small, landable commits. Run **unslop** and `/no-comments` over the diff before commit.

Title: `type(scope): subject`, naming a real symbol when one carries the change. Body: a one-minute briefing under 40 lines with these `##` headings in order, dropping empty ones: `## Why`, `## What changed`, `## Scope`, `## Tradeoffs` (only rejected alternatives a reviewer would otherwise ask about), `## Blast Radius`, `## Verification`.

Stack follow-ups by base branch. Open PRs ready for review with `gh pr create` (never `--draft`). Opening a PR does not start a babysit.
