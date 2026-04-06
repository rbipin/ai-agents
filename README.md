# skills

A repository of agent skill definitions. Each skill lives in its own folder under `skills/` and is described by a single `SKILL.md` file.

## Purpose

This repository is designed to store and maintain guidance for intelligent agents that assist with .NET development, code review, architecture analysis, and skill scaffolding. The focus is on clear, actionable skill metadata rather than application source code.

## Repository structure

- `skills/` — contains agent skill directories
  - `dotnet-code-review/` — expert code review guidance for C# / .NET code
  - `dotnet-developer/` — .NET/C# development best practices and version-aware guidance
  - `dotnet-version-checker/` — checks the official .NET support policy for the latest stable release
  - `technical-deep-dive/` — investigative deep-dive guidance for technical topics
  - `write-skill/` — canonical skill scaffolding template for new skill creation
- `.github/copilot-instructions.md` — workspace-level guidance for any agent or assistant working in this repo

## Skill conventions

Each skill should:

- live in its own directory under `skills/`
- contain a single `SKILL.md`
- include YAML frontmatter with fields such as `name`, `description`, `applyTo`, and optional metadata like `compatibility`, `skills`, or `triggersOn`
- provide a clear purpose, when-to-use guidance, workflow steps, and validation criteria
- stay focused and concise, typically under 500 lines

## How to contribute

1. Review the existing `write-skill/SKILL.md` template.
2. Create a new directory under `skills/` with a lowercase, hyphen-separated name.
3. Add a `SKILL.md` file that follows the repo's frontmatter and workflow conventions.
4. Update existing skills by preserving useful content and adding targeted workflow or metadata changes.

## Notes

- This repository does not currently include build or test automation.
- The main objective is to document and standardize agent skill behavior rather than compile code.
- Keep updates tightly scoped to skill definitions and workspace guidance.
