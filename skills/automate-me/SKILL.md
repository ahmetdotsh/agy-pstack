---
name: automate-me
description: Draft or update a personal -mode skill from your recent Antigravity conversation transcripts and working habits. Use for /automate-me, "create/update/refresh my -mode skill", or "capture my working style into a skill".
---

# Automate me

A guided flow for turning the user's working conventions into a skill agents will follow. The output is one `-mode` skill tailored to them (e.g. `jay-mode`, `priya-mode`).

This skill orchestrates three steps: an inline transcript mining pass, Antigravity's built-in **agy-customizations** skill guidelines for authoring `SKILL.md`, and the **unslop** skill for prose discipline. It sequences them. It doesn't replace them.

## Flow

### 0. Check for an existing skill

Look for `.agents/skills/**/*-mode/SKILL.md` in the active workspace and `~/.gemini/config/skills/*-mode/SKILL.md` globally matching the user's handle. If one exists, confirm intent with `ask_question` (unless they already said "update my skill" or similar):

- `(Recommended) Update the existing skill`
- `Start fresh`

Update mode changes the rest of the flow:
- Step 1 mines only history since the skill was last edited (`git log -1 --format=%cI <path>` or file mtime).
- Step 2 asks what's changed or missing, not what to capture from zero.
- Step 4 edits the existing file in place. Preserve sections the user hasn't contradicted. Revise ones with new evidence. Add new sections only for genuinely new rules.

### 1. Mine their history

Locate Antigravity transcripts under `~/.gemini/antigravity/brain/<conversation-id>/.system_generated/logs/transcript.jsonl` (start with compact `transcript.jsonl`; read specific lines from `transcript_full.jsonl` only when `truncated_fields` is present). Filter to conversations that operated on the active workspace path — do not pull private chats from unrelated projects unless the user asks.

Survey recent conversations within that scope (default: last 2–4 weeks, split into 3 time slices) using `run_workflow` `parallel(...)` on `model="flash"` (or parallel `research` subagents). Each slice miner scans its assigned transcripts for the signals below and returns a short structured list of patterns with conversation ID pointers (`conversation://<id>`):

- Response preferences (length, tone, format, "dumb it down" corrections)
- Delegation habits (subagents, `/swarm`, `run_workflow`, specialized workflows, parallelism)
- Verification posture (what "done" means, unit tests vs live repro, reviewers)
- Code and prose discipline (style, principles cited, lint/format tools)
- Process conventions (worktrees, commits, PRs, review/merge tooling)
- Meta preferences (fixing skills mid-task, proposing new ones)

Cross-check across slices before elevating a signal. Patterns seen in 2+ slices are high-confidence. Lone signals are weak and usually get dropped.

### 2. Ask the user directly

Mining misses intent that hasn't come up yet. Use `ask_question` (structured multi-choice, `is_multi_select: true` for category questions) rather than asking the user to type from scratch.

Shape: one or two questions with 4–6 options each. Start broad ("Which areas matter most?"), then follow up on selected areas with specific options. After the structured rounds, one free-form chat question catches anything the options missed.

Don't dump 20 questions.

### 3. Cluster findings

Group the combined signals into sections. Common ones (use only what applies):

- **Response style**: length, tone, format.
- **Autonomy**: how much to do without asking, MCP tool use.
- **Understand first**: which skills to reach for when scoping or investigating a change.
- **Delegation**: subagents, `/swarm`, `run_workflow`, specialized workflows.
- **Prose / code discipline**: principles, lint tools, style guides.
- **Review and verify**: repro posture, verification skills, live-testing tools.
- **Process**: git worktrees, commits, PRs, review/merge tooling.
- **Skills**: skill-authoring habits, fix-the-skill-first, proposing new skills.

The **poteto-mode** skill shows the shape. Read it for granularity. Don't copy its content. The user's rules are not the same as poteto-mode's.

### 4. Draft the skill

Follow Antigravity's **agy-customizations** skill (`skills.md`) rules to author the skill:

- **Path**: preserve an existing mode skill's location. For a new project-scoped mode, default to `.agents/skills/<handle>-mode/SKILL.md` (or `~/.gemini/config/skills/<handle>-mode/SKILL.md` for a global personal skill, or bundle it into a personal plugin via `/plugin`).
- **Handle**: the user's first name or chosen identifier.
- **Frontmatter**: `name: <handle>-mode` (must match folder name) and `description` as a single YAML scalar (quoted or `>-`) that triggers on their name + `/<handle>-mode` + "work in their style", explicitly stating it runs on request rather than on generic keywords like "write code".

### 5. Iterate on prose

Apply the **unslop** skill to every line.

Show the draft to the user and take feedback. Expect multiple iterations. Cut ruthlessly. A mode skill is not a manual.

### 6. Land it

For a project-scoped skill, work in a git worktree off `main`, commit, and open a PR rather than pushing directly to `main`. For a global personal skill under `~/.gemini/config/skills/`, write it directly and link it with a clickable `file://` link.

## Guardrails

- **Don't overfit to one conversation.** A preference stated once and contradicted another time is noise. Require multiple instances before codifying it.
- **Don't be clever.** Restating other skills' contents, inventing metaphors, or writing "poetic" prose for an agent reader is cost without benefit. Keep it operational.
- **Reference, don't inline.** Other skills the user relies on should appear as path references, not pasted excerpts. Same for any principle docs they maintain elsewhere.
- **Keep sections minimal.** Only add a section if the user has a specific, non-default rule there.
- **Name conventions generic.** Use "the user" or "the human" in imperatives, not the author's first name.
- **Don't force symmetry.** If a user has no process rules worth writing down, skip the Process section entirely.

## Evaluation

A `-mode` skill is subjective output. Vibe-check with the user: does it read like them? Did it miss anything? Then ship.

## When not to use

- User wants a task-specific skill (not working conventions): author directly using the **agy-customizations** skill or the Authoring a skill playbook.
- User wants to capture one narrow lesson from the current session: use `/reflect` or Antigravity's native `/learn`.
