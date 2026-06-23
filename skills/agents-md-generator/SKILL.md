---
name: agents-md-generator
description: Generate or update a project-specific AGENTS.md for non-AI-native repositories. Use when the user asks to create an AGENTS.md, make a repository AI-agent ready, document agent working rules, onboard Codex or other coding agents to an existing project, or scan a project and produce agent instructions.
---

# AGENTS.md Generator

Create a concise `AGENTS.md` that tells coding agents how to work safely in the current repository.

The output is an internal engineering guide, not a consulting report.

## Core Rules

- Scan before asking questions.
- Separate confirmed facts, inferred details, and missing context.
- Ask the project owner targeted questions before generating the final file.
- Do not write guessed commands or conventions as facts.
- Keep the generated `AGENTS.md` short, direct, and repository-specific.
- Preserve any existing `AGENTS.md` unless the user confirms replacement.

## Workflow

### 1. Inspect Existing Agent Instructions

Check whether these files exist:

```powershell
rg --files -g 'AGENTS.md' -g 'CLAUDE.md' -g 'GEMINI.md'
```

If `AGENTS.md` exists, read it before doing anything else. Ask whether to update it, replace it, or create `AGENTS.generated.md`. Do not overwrite it without confirmation.

### 2. Scan Repository Evidence

Use `rg --files` first. Read files that exist from this list:

- `README*`
- `package.json`
- `pnpm-lock.yaml`
- `yarn.lock`
- `package-lock.json`
- `pyproject.toml`
- `requirements*.txt`
- `Pipfile`
- `poetry.lock`
- `pom.xml`
- `build.gradle`
- `settings.gradle`
- `go.mod`
- `Cargo.toml`
- `Makefile`
- `Dockerfile`
- `docker-compose*.yml`
- `.github/workflows/*`
- `.gitlab-ci.yml`
- `Jenkinsfile`
- linter, formatter, and test configuration files

Inspect top-level and second-level directory structure. Skip large generated or dependency folders such as `.git`, `node_modules`, `dist`, `build`, `target`, `.venv`, and `vendor`.

### 3. Present Scan Summary

Before writing `AGENTS.md`, present a compact summary:

```markdown
## Confirmed
- Facts directly found in repository files.

## Inferred
- Likely facts based on file names, dependencies, or structure.

## Missing
- Information that could not be determined safely.

## Questions
- Questions for the project owner.
```

Keep `Inferred` conservative. If a command is not found in project files, list it as missing instead of guessing.

### 4. Ask Owner Questions

Ask 3-8 questions. Prefer fewer questions when repository evidence is strong.

Prioritize:

- Actual install, run, test, lint, format, build, and deploy commands.
- Whether tests and CI are trusted.
- High-risk modules, data flows, or workflows.
- Directories agents should avoid or treat carefully.
- Generated files and migration rules.
- Coding conventions not encoded in tooling.
- What "done" means for typical changes.

Do not ask all questions at once if the answer will be hard to provide. Group related command questions together when that reduces back-and-forth.

### 5. Generate AGENTS.md

Generate this structure:

```markdown
# AGENTS.md

## Project Snapshot
## Tech Stack
## Commands
## Repository Map
## Development Rules
## Testing and Verification
## Agent Workflow
## Risk Areas
## Do Not
## Missing Context
## Codex Notes
```

Write in short imperative prose.

Use this section guidance:

- `Project Snapshot`: State what the project does using confirmed files or owner input.
- `Tech Stack`: List languages, frameworks, runtimes, package managers, and major tools.
- `Commands`: Include only commands found in project files or confirmed by the owner.
- `Repository Map`: Describe important directories and boundaries.
- `Development Rules`: Capture project-specific style, architecture, dependency, and review rules.
- `Testing and Verification`: State exactly what agents must run before claiming completion.
- `Agent Workflow`: Tell agents to read context, make focused edits, preserve conventions, verify, and report changes.
- `Risk Areas`: List modules, files, workflows, or data paths needing extra care.
- `Do Not`: List hard constraints and forbidden actions.
- `Missing Context`: Keep unresolved questions that affect safe agent work.
- `Codex Notes`: Add Codex-specific advice while keeping the rest portable.

Use these default `Codex Notes` unless the project needs stricter guidance:

```markdown
## Codex Notes

- Use `rg` or `rg --files` before slower search commands.
- Read nearby code before editing.
- Use the repository's documented commands for verification.
- Keep changes scoped to the requested task.
- Do not reset, discard, or overwrite user changes unless explicitly asked.
- For large or risky changes, write a short plan before editing.
```

### 6. Self-Review Before Finishing

Check the generated file for:

- Inferred details written as confirmed facts.
- Commands not found in project files or owner answers.
- Generic advice that could apply to any repository.
- Missing testing or verification instructions.
- Missing risk areas.
- Contradictions between repository evidence and owner answers.
- Placeholder text such as `TODO`, `TBD`, or vague filler.

Fix issues before presenting the result.

## Edge Cases

- If the repository is too large, sample top-level structure and the most important config files first.
- If no metadata files exist, generate a minimal file with a prominent `Missing Context` section.
- If the owner cannot answer a question, keep it in `Missing Context`.
- If commands are discovered but may be unsafe or expensive, ask before running them.
- If the user only asks for a template, provide the structure without scanning or writing repository-specific facts.

## Final Response

Summarize:

- The path written.
- Whether an existing `AGENTS.md` was updated or a new file was created.
- Which facts came from owner answers if that matters.
- Any unresolved missing context.
- Verification performed, such as self-review or file inspection.
