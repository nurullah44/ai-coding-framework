# ai-coding-framework

A copyable, stack-agnostic template for building software with coding agents (Claude Code, Codex, Pi) without losing architecture or code quality as volume grows.

**Status: draft.** It changes only when something learned in real use proves worth keeping.

## Use it

Click **Use this template**, or copy these files into your repo. Then fill in the `TODO`s in `AGENTS.md`.

Claude Code loads `AGENTS.md` by itself only when there is no `CLAUDE.md`. If you add one (`/init` creates it), make its first line `@AGENTS.md`, or the rules stop loading.

## Idea

Agents scale only as far as you trust their output. This template gives a project:

- **Verification:** agents prove their work on the running app with one CLI (`.agents/skills/verify/`) and a feature map, instead of claiming it works.
- **A trust ladder:** every correction is fixed at the strongest layer that can hold it: codebase structure, then static analysis, then guidance, and human review last.
- **A task loop:** issues move through plan → build → independent review → merge, with evidence at each step.
- **A small always-loaded context:** only `AGENTS.md` and skill descriptions load every session; everything else loads per role.

## Layout

```
AGENTS.md                 always loaded; commands, hard rules, pointers
docs/agents/              process, trust ladder, invariants, roles, templates, garden
.agents/skills/verify/    verification contract, stub CLI, feature map
.claude/skills            symlink to .agents/skills
```

## Sources

- Lauren Tan: [talk](https://youtu.be/Z-jNqqIYGm4), [interview with Matt Pocock](https://www.youtube.com/live/MN9dGgmLyso), [pstack](https://github.com/cursor/plugins/tree/main/pstack), [verification-skill-example](https://github.com/poteto/verification-skill-example)
- Matt Pocock: [skills](https://github.com/mattpocock/skills)
- Alexey Grigorev: [AI-native development: specifications](https://aishippingblog.com/p/ai-native-development-specifications)
- [Bend](https://bend-lang.com/): laws checked by proof, the idea behind `docs/agents/invariants.md`
