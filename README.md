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
