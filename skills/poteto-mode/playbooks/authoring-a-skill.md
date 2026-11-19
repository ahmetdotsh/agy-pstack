### Authoring or modifying a skill

**You own the skill's voice.**

1. Follow Antigravity's built-in **agy-customizations** skill (`docs/skills.md`) for authoring `SKILL.md` files (and the **plugin** skill when packaging into a plugin bundle).
2. Validate the skill: YAML frontmatter has `name` matching the folder name and a clear `description`, referenced files exist, and cross-skill links resolve.
3. Add test cases if structural. Skip if subjective.
4. Run **Opening a PR**.

When in doubt, delete. Keep only prose that changes a decision. Tell it to do the thing and skip the reason. Explain only when the rule is confusing without one. Match tone to scope. Point at structural sources (types, READMEs, config) per the **encode-lessons-in-structure** principle skill. Delegate to other skills by path. Don't restate. A workflow you keep hitting that isn't captured → propose a new skill.

**Reply:** summary of the skill, key design decisions, validation notes.
