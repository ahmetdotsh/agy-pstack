# Set up pstack

In this page you install the plugin in Antigravity, configure your per-role model preferences, and run your first task. Setup is one command plus a short conversation.

## Install the plugin

Install `pstack` globally into Antigravity's plugins folder:

```bash
git clone https://github.com/ahmetdotsh/agy-pstack.git ~/.gemini/config/plugins/pstack
```

Or per workspace by placing it in `.agents/plugins/pstack/`. Manage or toggle the plugin at any time with Antigravity's native **`/plugin`** command (changes take effect live without restarting).

## Pick your models

Run:

```text
/setup-pstack
```

[`/setup-pstack`](../../skills/setup-pstack/SKILL.md) inspects the live models available to your Antigravity account via `swarm{action:"models", all:true}`, asks for a budget (`balanced`, `max`, or `lean`), shows each role, and writes `~/.gemini/config/rules/pstack-models.md`.

- **`balanced` (default):** code, prose, and judgment delegates use `inherit` (your active conversation model); routine bulk sweeps use `flash`; `/swarm` workers and panels omit `model` (running on your model) unless you explicitly pick distinct panel models.
- **`max`:** `pro` tier for code and judgment delegates, `flash` for bulk sweeps, and multi-model `/swarm` panels when distinct models are available.
- **`lean`:** `flash` for delegates, `flash_lite` for sweeps, and smaller `inherit` panels.

You only override what you care about. Delete any role line in `~/.gemini/config/rules/pstack-models.md` to fall back to the skill's default.

## Accept the verification offers, or don't

At the end of setup, `/setup-pstack` checks whether your project has a scripted way to prove app behavior (`.agents/skills/verify-*/SKILL.md` or an existing harness). If not, it offers once to generate one with [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `.agents/skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does (using Antigravity's built-in `/browser` subagent, `cua-driver`, Playwright, PTY, or HTTP). It proves the skill end to end once before handing it over.

Then setup offers once to generate a formal verification skill with [`/create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md), unless the project already has one. Say yes when your code has concurrency, retries, state machines, or parsers where bugs are costly and tests rarely hit the edge case.

After setup, start a new conversation so `~/.gemini/config/rules/pstack-models.md` is loaded.

## Keep the cost in check

pstack spends extra tokens on subagents, `/swarm` workers, and adversarial reviews. To spend fewer:

- Rerun `/setup-pstack` and pick `lean`.
- Keep roles on `inherit` with a faster active model.
- Shorten panel lists (`arena runners`, `interrogate reviewers`).
- Save `/poteto-mode` for work that needs rigor. A small, obvious edit doesn't.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the task checklist (`task.md` artifact or inline checklist). Its first items are the matched playbook's steps — the Feature playbook for this prompt.

Next: [Route work through `/poteto-mode`](./02-poteto-mode.md).
