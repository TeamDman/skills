---
name: machine-workflow
description: 'Use Teamy-specific local workflow tools and conventions to discover repos, shrink workspace scope, and preserve useful context without overloading editors or leaking machine-specific details. Use when deciding how to find relevant local projects, what to keep open in VS Code, and which tool is appropriate for Codex versus GitHub Copilot or other editor workflows.'
argument-hint: 'Describe the task, whether you are trying to find local repos or shrink editor scope, and whether the advice is for transient local work or something that may be committed'
---

# Machine Workflow

Use this skill for Teamy-specific local development workflow guidance.

This skill encodes cross-repo practices and machine-specific preferences at a high level without publishing unnecessary machine detail.

## Outcome

Produce a recommendation or implementation step that:

- uses the right local discovery tool for the task
- keeps VS Code workspace scope small by default
- distinguishes Codex-relevant guidance from editor-specific Copilot guidance
- avoids persisting sensitive machine detail unless truly necessary

## Default working set

Unless the task strongly suggests otherwise, start from:

- `TeamDman/Skills`
- `TeamDman/Agent-Scratchpad`
- the active project repo
- only the extra reference repos or directories that directly help with the current problem

This is the normal baseline for focused work.

## Tool guide

### `locate-git-projects-on-my-computer.exe`

Use when:

- you need to know whether a dependency or repo already has a local checkout
- the user names a crate or repo but not its path
- you want to discover relevant sibling repos before building a small workspace

Strengths:

- repo and package discovery across the machine
- combines git and package metadata
- useful for narrowing to likely local references

Limitations:

- discovery output may contain sensitive machine-specific absolute paths
- good for local decisions, not for direct publication into shared artifacts

### `teamy-mft`

Use when:

- you need broad indexed path discovery fast
- you are searching for filesystem markers such as `Cargo.toml`, `.git`, logs, outputs, or caches
- you need machine-wide lookup that is broader than one repo

Strengths:

- fast indexed filesystem discovery
- useful when you know a pattern but not the repo or path

Limitations:

- returns filesystem-level evidence, not higher-level project judgment
- results may need filtering and interpretation

### `code`

Use when:

- you want to open a repo or workspace in VS Code
- you want to add a folder to the current VS Code window with `code -a <dir>`
- you want to create or use a small `.code-workspace` instead of a giant multi-repo session

Strengths:

- supports additive workspace shaping with `-a`
- fits iterative "open only what we need" workflows

Limitations:

- VS Code and GitHub Copilot behaviors are editor-specific
- advice about what should be open for Copilot responsiveness is not always directly relevant to Codex itself

## Applicability boundary

Distinguish between:

- local editor ergonomics
- Codex task execution
- shared published knowledge

Examples:

- "Use `code -a` to expand the current VS Code session" is editor-specific and useful when the user works primarily in VS Code.
- "Create a small `.code-workspace` for the current task" is useful for both VS Code and the broader workflow.
- "Open these five repos because Copilot search gets slow with fifty repos" is often relevant to VS Code, but not automatically a rule for Codex outside that editor context.

## Least-privilege persistence

It is usually safe to persist:

- tool names
- command shapes
- workflow heuristics
- generic placeholders

It is usually not safe to persist without review:

- raw discovery output
- concrete absolute paths from local scans
- private datasets, backups, or personal files revealed during discovery

If a path-like value might be recorded, run `is-this-path-sensitive` first.

## Preferred workflow

1. Start with `Skills`, `Agent-Scratchpad`, and the active project.
2. Use `locate-git-projects-on-my-computer.exe` or `teamy-mft` to discover likely relevant references.
3. Add only the few repos or directories that directly help.
4. Prefer a dedicated `.code-workspace` when the task will last more than a quick edit.
5. Use `code -a` for local interactive expansion when the user wants to keep the current VS Code window.
