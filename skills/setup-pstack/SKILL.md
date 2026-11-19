---
name: setup-pstack
description: Configure which Antigravity model tiers and swarm models pstack uses per role, and optionally generate project verification skills. Detects available swarm models and writes ~/.gemini/config/rules/pstack-models.md. Use for /setup-pstack, "configure pstack models", or changing pstack's model choices.
---

# Setup pstack

Write `~/.gemini/config/rules/pstack-models.md`, an always-on Antigravity rule that configures pstack's per-role model tiers (`invoke_subagent` / `run_workflow`) and optional multi-model panel seats (`/swarm`).

## Steps

### 1. Detect available models

Call `swarm{action:"models", all:true}` to list the live models available to your Antigravity account (along with quota status). Never write a swarm model name you have not confirmed in that output.

Antigravity uses two model layers:

- **Subagent / workflow tiers** (for `invoke_subagent` and `run_workflow`): `inherit` (your active conversation model, default), `pro`, `flash`, `flash_lite`.
- **Swarm models** (for `/swarm`, `/arena`, `/interrogate` multi-model panels): `inherit` (omit `model` on `swarm{action:"add"}`, default) or any exact model name returned by `swarm{action:"models", all:true}`.

### 2. Load current state

If `~/.gemini/config/rules/pstack-models.md` already exists, read it and treat its `# budget` line and role lines as the current choices. Otherwise start from the defaults in step 5. Drop any retired role lines not listed in step 5 and note what you dropped.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Use `ask_question` with these three options (name the current budget if the rule exists; otherwise note that `balanced` matches the defaults):

- `(Recommended) balanced — inherit active model for code/judgment, flash for bulk sweeps`
- `max — pro tier for code/judgment, multi-model panels in /swarm when available`
- `lean — flash tier for delegates and sweeps, single-model inherit panels`

**(b) Build the working table:**

- `balanced`: code/judgment roles = `inherit`; routine/bulk sweeps = `flash`; `swarm workers` = `inherit`; `arena runners` / `interrogate reviewers` = `inherit, inherit, inherit` (or up to 3 distinct models if the user picks multi-model); `arena cross-judge pool` = `inherit`.
- `max`: code/judgment roles = `pro`; routine/bulk sweeps = `flash`; `swarm workers` = `inherit`; `arena runners` / `interrogate reviewers` = up to 3 distinct models from `swarm{action:"models", all:true}` (preferring diverse families if available, else `inherit`); `arena cross-judge pool` = first available distinct model from `swarm{action:"models", all:true}` (else `inherit`).
- `lean`: code/judgment roles = `flash`; routine/bulk sweeps = `flash_lite`; `swarm workers` = `inherit`; `arena runners` / `interrogate reviewers` = `inherit, inherit`; `arena cross-judge pool` = `inherit`.

On a re-run, preserve any role the user explicitly customized unless they ask to reset it.

**(c) Show the roles and confirm.** Show the full role table and ask via `ask_question` whether to write it as-is or customize specific roles. For panel roles (`arena runners`, `interrogate reviewers`), explain that the list length sets the number of parallel workers spawned.

### 4. Validate

- Every `invoke_subagent` / `run_workflow` role must be one of `inherit`, `pro`, `flash`, `flash_lite`.
- Every `swarm` / panel entry must be `inherit` or an exact model name confirmed by `swarm{action:"models", all:true}`.
- If a chosen model is unavailable or out of quota, flag it and ask again.

### 5. Write the rule

Write `~/.gemini/config/rules/pstack-models.md` overwriting the entire file so re-runs stay idempotent:

```markdown
---
trigger: always_on
---
# pstack model configuration (Antigravity native).
# Delete any role line to fall back to the skill default.
# Subagent / workflow tiers: inherit | pro | flash | flash_lite
# Swarm / panel values: inherit (omit model) or an exact model name from swarm{action:"models", all:true}
# budget: balanced
feature, refactoring: inherit
bug-fix: inherit
perf-issue: inherit
hillclimb: inherit
judgment and prose: inherit
routine: flash
swarm workers: inherit
arena runners: inherit, inherit, inherit
arena cross-judge pool: inherit
interrogate reviewers: inherit, inherit, inherit
```

### 6. Confirm

Tell the user that [pstack-models.md](file:///~/.gemini/config/rules/pstack-models.md) was written (resolve the real absolute path) and applies automatically in new conversations. Re-running `/setup-pstack` updates it.

### 7. Offer a verification skill (optional)

Check whether the active workspace has a scripted way to drive the real app (an `.agents/skills/verify-*/SKILL.md` skill or an existing harness). If not, offer once via `ask_question`: "Want a project-local verification skill in `.agents/skills/verify-<app>/` so agents can drive the app like a user and prove changes work?" On yes, run `/create-verification-skill`. On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the active workspace already has a formal verification skill (model checker, proof, or verifier). If not, offer once: "Want a formal verification skill to prove a key invariant holds across every case in scope?" On yes, run `/create-formal-verification-skill`. On no, move on without pushing.
