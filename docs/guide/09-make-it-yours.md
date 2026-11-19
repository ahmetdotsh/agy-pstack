# Make it yours

`poteto-mode` is one person's style. The playbooks, routing, and role configs underneath it work just as well with yours. This page covers generating a personal mode, capturing lessons from a session, fixing the repo so agents stop repeating mistakes, authoring a focused skill, and testing a skill change before you trust it.

Start smaller than you think. You don't need many skills on day one, or even this whole plugin. Prompt plainly, watch where agents fail, and add a skill or a check when the same failure shows up twice.

## Generate your own mode with `/automate-me`

```text
/automate-me
```

You don't describe your style, because [`/automate-me`](../../skills/automate-me/SKILL.md) reads it out of your Antigravity conversation history. It mines your recent transcripts for repeated preferences in replies, delegation, verification, code, prose, and process, then asks via `ask_question` which patterns are really you. It drafts `.agents/skills/<your-name>-mode/SKILL.md` (or `~/.gemini/config/skills/<your-name>-mode/SKILL.md`) following Antigravity's `agy-customizations` standard, runs the draft through [`/unslop`](../../skills/unslop/SKILL.md), and opens a PR from a worktree when project-scoped.

Run it again whenever your habits drift:

```text
/automate-me update my mode skill with everything since its last edit
```

Update mode mines only the history since the skill last changed. It keeps rules you haven't contradicted, revises the ones with new evidence, and adds sections only for new patterns.

## Capture a session's lessons with `/reflect` (and native `/learn`)

Right after a task that taught you something, run:

```text
/reflect that took way too long. capture what we learned so the next run doesn't repeat it.
```

[`/reflect`](../../skills/reflect/SKILL.md) reads the conversation's `transcript.jsonl`, keeps only lessons that would change a future agent's work, and sorts proposals into `Accepted`, `Rejected`, and `Backlog`. It waits for your approval before editing any skill. Approve a proposal only if it would change a future decision. One weird session is an anecdote, not a rule.

For general behavioral preferences that belong outside any single skill, pair `/reflect` with Antigravity's native **`/learn`** command.

## Fix the environment with `/correct`

When you correct agents for the same mistake again and again, the fix belongs in the repo, not in your next prompt. Rank the options by how well they hold:

1. Make the mistake impossible with architecture or a better data structure.
2. Block it with types, or with a lint or CI check whose error names the fix.
3. Catch it with a test.
4. Write it down as a doc or agent rule (`AGENTS.md` / `GEMINI.md`). Nothing fails when an agent skips a rule, so this comes last.

Human review isn't on the list. A reviewer who must catch the same mistake on every PR is the problem this fixes. [`/correct`](../../skills/correct/SKILL.md) does the work:

```text
/correct agents keep calling the database client directly instead of going through the repository layer
```

It reads recent commits, reverts, review comments, and comments that explain workarounds, then groups the mistakes into classes. A class counts once it has happened twice. It fixes the most frequent classes one commit each, at the highest level that works, and proves each new check fails on a real past mistake.

Run it with no argument and it finds the classes from history on its own. `/reflect` improves skills from one session; `/correct` changes the repo so a mistake class can't come back.

## Author a focused skill

When you already know the workflow you want to capture:

```text
/poteto-mode write a skill for verifying database migrations in this repo
```

Writing a skill matches the [Authoring or modifying a skill playbook](../../skills/poteto-mode/playbooks/authoring-a-skill.md), which follows Antigravity's built-in `agy-customizations` standard (and `/plugin` when bundling), validates frontmatter and links, and ships the result through Opening a PR.

Two special cases have their own generators:
- App verification: [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) and [`/maintain-verification-skill`](../../skills/maintain-verification-skill/SKILL.md).
- Formal verification: [`/create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md) and [`/maintain-formal-verification-skill`](../../skills/maintain-formal-verification-skill/SKILL.md).

## Write docs to a standard with `/technical-writing`

For docs, RFCs, READMEs, PR descriptions, and commit messages:

```text
/technical-writing review the readme changes
```

[`/technical-writing`](../../skills/technical-writing/SKILL.md) applies a layered standard (Diátaxis + Google developer style + STE + Global English). Its goal is prose that a tired engineer understands on the first read.

## Test a skill change blind

A skill edit affects every future session, so test it like the experiment it is:

```text
/poteto-mode run the eval playbook on this skill change. same task for both variants, candidates stay blind.
```

The [Eval playbook](../../skills/poteto-mode/playbooks/eval.md) is built around one failure mode: the observer effect. Candidate agents get an organic-looking task in sanitized directories (`Workspace: "branch"`), never seeing the words "eval" or "candidate", and one blinded judge scores all outputs under neutral labels while chain-following is verified directly from Antigravity's `transcript.jsonl`.

**Pitfall:** don't edit a skill mid-task because it's misbehaving. Fix it in its own PR and keep the task moving. A skill edit that ships tangled into feature work is invisible to review and impossible to evaluate.

Next: [Recipes and pitfalls](./10-recipes-and-pitfalls.md).
