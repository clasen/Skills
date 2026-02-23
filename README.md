# Skills

A personal collection of agent skills — reusable instruction sets that teach AI coding assistants how to handle specific tasks and workflows.

## What's here

| Skill | Description |
|-------|-------------|
| [skill-builder](./skill-builder/) | Interactive guide for creating new agent skills from scratch |

## skill-builder

`skill-builder` walks you through creating high-quality agent skills based on the open Agent Skills standard. It helps you package any repeatable workflow, process, or domain knowledge into a portable `SKILL.md` file that an AI agent can load and apply consistently across sessions.

**Trigger it when you want to:**
- Create a new skill or write a `SKILL.md`
- Convert an existing workflow into a reusable skill
- Standardize a process so the agent follows it automatically

## How skills work

A skill is a folder containing a `SKILL.md` file (with YAML frontmatter and Markdown instructions) and optionally scripts, references, and assets. The agent reads the skill file to gain specialized knowledge and then applies it during the session.

## Agent behavior

The [`AGENT.md`](./AGENT.md) file defines how the AI agent should behave across all sessions in this workspace. It establishes workflow orchestration rules, task management conventions, and core principles the agent follows automatically.

**Benefits:**
- **Consistency** — the agent plans before acting, tracks progress, and verifies work without being asked
- **Self-improvement** — the agent captures lessons after corrections to avoid repeating mistakes
- **Higher quality output** — elegance and root-cause thinking are baked in by default, not optional
- **Less hand-holding** — bugs get fixed autonomously; tasks get managed through structured todos
