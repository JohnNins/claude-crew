# claude-crew

A tiny Claude Code crew that acts as a four-person software team: planner, developer,
ui-expert, tester. Each one owns a single lane and will stop and hand off when a task
crosses into another agent's work.

Models are matched to the job so you don't pay Opus prices for everything.

| Agent      | Role                                                            | Model  |
| ---------- | --------------------------------------------------------------- | ------ |
| planner    | Scopes work, answers questions, writes docs — everything non-code | sonnet |
| developer  | Backend, data, logic, imports, build scripts — non-UI code      | sonnet |
| ui-expert  | Design + UI code; keeps the look modern and non-generic         | opus   |
| tester     | Writes tests, finds bugs, writes repro reports                  | sonnet |

**Lane rule**: every agent lists what it *can* and *cannot* do. On a cross-lane task they
stop and name the right agent.

**Why Opus only on ui-expert**: design taste is where the capability gap shows up most, and
a bad UI design costs more to redo than a bad backend diff. Everyone else runs on Sonnet.

## Install

Copy or symlink `agents/` into one of:

- `~/.claude/agents/` — available in every project on this machine.
- `<project>/.claude/agents/` — only in that project.

```powershell
# user-level (Windows PowerShell)
Copy-Item -Recurse agents\* $HOME\.claude\agents\
```

```bash
# per-project (bash)
cp -r agents/*.md <project>/.claude/agents/
```

## Project-specific skills

Agents are intentionally project-agnostic. To give them a project's stack, folder layout,
and conventions, add **skills** to that project at `<project>/.claude/skills/`:

```
<project>/.claude/skills/
├── crew-<project>-developer/SKILL.md
├── crew-<project>-ui-expert/SKILL.md
└── crew-<project>-tester/SKILL.md
```

Each engineer-style agent auto-invokes its matching skill on launch. `planner` doesn't need
one — it works from docs, not from a stack.

See `docs/skill-template.md` for the pattern.

## How to use

Direct dispatch:

```
Agent({
  subagent_type: "developer",
  description: "Add CSV import endpoint",
  prompt: "Add a POST /students/import endpoint that accepts a CSV upload.
           Follow the crew-<project>-developer skill if it's listed."
})
```

Plan first, then dispatch:

```
Agent({
  subagent_type: "planner",
  description: "Scope CSV import",
  prompt: "Scope CSV student import for the Students page. Produce a task list naming
           which agent owns each task (developer, ui-expert, tester)."
})
```

## Honest note on token cost

A multi-agent crew does not automatically save tokens — each spawned agent has to be
re-briefed (input tokens) and carries its own system prompt (more input tokens). You save
tokens when:

- The main thread would otherwise fill up with research / tool output (fork a lookup).
- A tier downshift is a real win (planner → simple Q&A on Sonnet instead of Opus).

The crew's real value is **scope discipline**: each agent refuses to drift. If your pain
is "work keeps growing beyond what I asked for", this helps. If your pain is "I'm burning
tokens on small features", a single Sonnet session going straight at it is cheaper.
