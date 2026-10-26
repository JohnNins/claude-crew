# Project skill template for the crew

The developer, ui-expert, and tester each look for a project-specific skill named
`crew-<project>-<role>` in `.claude/skills/` and invoke it before starting work.
That skill holds the parts of the project the generic agent definition can't know:
stack, folder layout, naming, test rules, build commands, PII rules.

(The planner doesn't use a project skill — it works from docs, not a stack.)

## Folder layout

```
<project>/.claude/skills/
├── crew-<project>-developer/SKILL.md
├── crew-<project>-ui-expert/SKILL.md
└── crew-<project>-tester/SKILL.md
```

## SKILL.md shape

```markdown
---
name: crew-<project>-<role>
description: Project-specific context the <role> needs. Load at the start of a <role> task.
---

# Stack
- Languages, frameworks, versions.

# Folder layout
- Where <role>'s code lives, what to leave alone.

# Conventions
- Naming, patterns, "we don't do X here".

# Commands
- Install, run, build, test — exact commands.

# Gotchas
- The things new contributors always get wrong.
```

Keep each skill to one screen. The agent definition carries the generic rules; this file
carries the project-only facts.

## Auto-load

Each engineer-style agent's prompt already says:

> If a skill named `crew-<project>-<role>` is in the available-skills list, invoke it first
> with the Skill tool.

So you don't call the skill by hand — the agent does it on launch.
