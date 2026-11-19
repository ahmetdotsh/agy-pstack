---
name: interrogate
description: Adversarial review where parallel reviewers try to break a change or design via native /swarm or isolated subagents, keeping only findings proven by a real run. Use for /interrogate, "adversarial review", "multi-model review", "challenge this", "stress test this code", "find blind spots", or "tear this apart".
---

# Interrogate

Adversaries try to break a change or a design. A finding survives only when a run proves it. Report the findings. Do not apply fixes.

## Frame

The scope is what the user names. Otherwise it is the branch diff from its merge base with the base branch, through the working tree. State the intent with the **in-your-own-words** skill, from the user's message, the commits, and the PR description. Ask the user when the intent is unclear.

## Fan out

Read the `interrogate reviewers` line from `~/.gemini/config/rules/pstack-models.md` (default: 3 parallel adversaries on `inherit`, or distinct swarm models if configured via `/setup-pstack`).

Spawn all adversaries in parallel with isolated workspaces (**separate-before-serializing-shared-state**):

- **Preferred (native `/swarm`)**: call `swarm{action:"title", title:"interrogate: <scope>"}` and add each adversary with `swarm{action:"add", name:"adversary-<n>", workspace:"share", ...}` (passing the configured model if not `inherit`, or omitting `model` for `inherit`). If there are uncommitted changes in scope, include a patch file path (or commit on a temporary ref) in the brief so each adversary applies the exact diff in its own workspace before probing.
- **Alternative (`invoke_subagent`)**: spawn parallel `invoke_subagent(TypeName="self", Role="Adversary <n>", Workspace="share", Prompt=...)` workers.

Give each adversary the scope, the intent, and this brief:

> Break this change. Prove each finding with a run against the real code, such as a failing test, a repro script, or command output that shows the wrong behavior. For a design or plan, run a prototype or probe the current tree. Keep repros in your own workspace. Report each finding with a title, the exact command, the output it printed, and the output the intent requires. Leave out anything you did not run. That covers style, naming, structure, and risks that need a future change to matter. No findings is a valid result.

## Verify

Verify each reported finding yourself by rerunning its exact command against the real tree (**prove-it-works**). Keep a finding when the output shows the failure and the failure breaks the intent. Drop the rest. Merge findings that share a root cause.

Adversaries find different failures, so keep every verified finding across all adversaries rather than picking one winner. Clean up any temporary swarm workers once verification finishes.

## Output

- **Intent**: the restatement.
- **Findings**: for each verified finding, the command, its output, how it breaks the intent, and the adversaries that found it. Say "no verified findings" when none survive.
- **Dropped**: for each adversary, the count of dropped findings and why.
