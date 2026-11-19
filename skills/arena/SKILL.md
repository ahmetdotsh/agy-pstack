---
name: arena
description: Spawn N parallel candidates at the same task via native /swarm or run_workflow, pick a base against a concrete rubric, and graft the strongest parts of the losers into it. Use for /arena, 'arena this', 'throw it in the arena', or when one attempt at a non-trivial artifact would lock in the wrong shape.
---

# Arena

Track one phase per item in a task checklist (Frame, Fan out, Cross-judge, Pick, Graft, Verify) before launching anything.

## Phase A: Frame

Every candidate gets the same prompt, so the prompt is the contract.

1. State the artifact each candidate is producing.
2. Derive the rubric, 3–6 concrete gradeable criteria for what success looks like on *this* task. It is the picker's tool in Phase D. Candidates only see the task.
3. Read the `arena runners` line from `~/.gemini/config/rules/pstack-models.md` (default: 3 candidates on `inherit`, i.e., your active model with independent runs, or distinct swarm models if configured via `/setup-pstack`). Spawn more for multiple design directions, and repeat one model when the work is generation-bound rather than judgment-sensitive.
4. Give each candidate its own isolated workspace so candidates never collide, per the **separate-before-serializing-shared-state** principle skill:
   - For repository code changes, use **`workspace: "share"`** (or **`workspace: "branch"`**) on native `/swarm` workers (or `run_workflow` `agent(..., workspace="share")`).
   - For standalone artifacts or design docs, give each candidate its own scratch path under `<appDataDir>/brain/<conversation-id>/scratch/arena-<slug>/candidate-<n>/`.

## Phase B: Fan out

Launch all N candidates in parallel using **native `/swarm`** (preferred when candidates explore, iterate, or run distinct configured models) or **`run_workflow`** `parallel(...)` (preferred for structured artifact generation):

- **Via native `/swarm`**: call `swarm{action:"title", title:"arena: <slug>"}`, then `swarm{action:"add", name:"candidate-<n>", workspace:"share", brief:...}` (omitting `model` for `inherit`, or passing the configured model from `pstack-models.md`). Each brief starts with `Read <poteto-mode-path>/worker.md before anything else.` and gives the task, shared grounding paths, output location, and instructions to produce the artifact plus a short rationale naming alternatives considered and rejected, posting `@coordinator [done] <summary>` when finished.
- **Via `run_workflow`**: run `parallel(candidates, lambda c: agent(..., workspace="share", schema=...))` returning each candidate's output path and rationale.

If a candidate fails or drops out, proceed with N-1 and note the dropout.

## Phase C: Cross-judge

After all Phase B candidates complete, spawn one read-only judge to score each candidate by neutral path label (`candidate-1`, `candidate-2`, ...) on every rubric criterion and recommend a base:

- Check the `arena cross-judge pool` line in `~/.gemini/config/rules/pstack-models.md`.
- If a distinct model is configured for `/swarm`, add a read-only judge worker on that model (`workspace: "inherit"`). Otherwise invoke a fresh `research` subagent (`invoke_subagent(TypeName="research", Role="Arena Cross-Judge", Prompt=...)`) so the judge sees only the candidate outputs and rubric with no author bias.

## Phase D: Pick a base

Read every candidate end to end before picking. Score each against the rubric criterion by criterion, not on holistic feel, and compare with the cross-judge's verdict. Agreement on the base confirms the pick. Disagreement means one of you is biased or the rubric was ambiguous. Read both rationales before deciding.

Pick the base a future maintainer can extend most easily without breaking invariants. When two feel tied, prefer the cleaner boundary or smaller API, per the **laziness-protocol** principle skill.

## Phase E: Graft

Walk each losing candidate once more for what is worth porting into the base, usually one or two things per candidate, not most of it. Fold each graft in by hand, per the **redesign-from-first-principles** principle skill, so the result stays coherent under one mental model. Don't paste mechanically.

If the candidates converge on one shape, that is strong agreement. Note it and ship the consensus shape. No graft is needed. If they wildly diverge, Phase A was under-specified. Reframe and re-run rather than averaging the divergence.

Land the chosen base (merge/cherry-pick from its shared workspace or copy the synthesized artifact) before killing any swarm workers.

## Phase F: Verify

Verify the synthesized artifact like any other output, per the **prove-it-works** principle skill. If verification finds a problem the arena missed, either Phase A was wrong (re-frame and re-run) or a candidate caught it and you missed the graft (go back to Phase E). Don't paper over it.

## Outputs

One synthesized artifact, with a short synthesis note alongside naming the base and why, the cross-judge's verdict, each graft with its source candidate, what was rejected and why, any convergence or dropouts, and the verification result.
