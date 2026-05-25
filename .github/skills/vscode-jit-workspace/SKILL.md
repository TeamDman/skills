---
name: vscode-jit-workspace
description: 'Create a minimal VS Code .code-workspace for the user''s current task by selecting only the relevant repos or subfolders, using local dependency declarations and local repo discovery to pull in nearby projects that matter. Use when the user wants a task-scoped workspace to reduce VS Code, Copilot, and search/indexing load in a large multi-repo environment.'
argument-hint: 'Describe the task, the main repo, any likely dependency or sibling repo relationships, and whether the output should be private machine-local or safe to publish'
---

# VS Code JIT Workspace

Use this skill to create a just-in-time `.code-workspace` file that includes only the folders relevant to the task at hand.

This skill is for users who keep many repos on disk and want smaller workspaces so VS Code search, Copilot, file watching, and indexing stay responsive.

## Default loadout

Unless there is a strong reason not to, the default starting loadout for problem-solving work is:

- `TeamDman/Skills`
- `TeamDman/Agent-Scratchpad`
- the active project repo
- only the extra reference repos or directories that materially help with the current task

Treat that as the baseline shape.

Do not start from the giant everything-workspace if the goal is focused implementation, debugging, or profiling.

## Outcome

Produce a task-scoped `.code-workspace` that:

- includes the main repo or narrowest useful subfolder
- includes only sibling repos that materially help with the current task
- excludes unrelated repos even if they are often open together
- uses low-noise search and watcher excludes

## Core procedure

1. Identify the concrete task and the main repo being edited.
2. Inspect local relationship signals before deciding what to include:
   - workspace members
   - `Cargo.toml` path dependencies
   - git dependencies and renamed packages
   - README notes about forks or local development workflows
   - notes in local repo guides or scratchpads
3. Use local repo discovery when the relevant sibling repo is not obvious.
4. Include only the folders that actively help with the task.
5. Write a descriptively named `.code-workspace` file.

## Local repo discovery

On this machine, prefer `locate-git-projects-on-my-computer.exe` when you need to discover whether a repo or package is already available locally.

Useful cases:

- the user mentions a crate or repo name but not its path
- a dependency seems important and you want to know whether there is a local checkout
- you want to shrink a workspace to only repos relevant to the task

The tool emits project entries with fields such as:

- `path_on_disk`
- `names`
- `outlinks`
- `authors`
- `last_activity_on`

Use the tool to discover relevant repos, but do not blindly persist or publish the discovered absolute paths.

If you need to persist a path-like value outside a transient local workspace file, review it with `is-this-path-sensitive` first.

## Selection heuristics

Prefer the narrowest useful folder, but widen scope when the task demands it.

- Use a crate or subfolder instead of the whole repo when the work is isolated.
- Keep the whole repo when root config, workspace features, or cross-crate navigation are likely relevant.
- Add sibling repos when a local fork, path dependency, git dependency, or active debugging reference makes them part of the current problem.
- For performance or observability work, look for profiling, tracing, logging, or instrumentation dependencies and include the relevant local repo if you have it checked out.
- Do not add a repo just because it is vaguely related.

## Teamy-Studio guidance

When the task is inside Teamy-Studio:

- keep `Skills` in the workspace when cross-repo conventions or machine-workflow guidance are likely relevant
- start from the most relevant crate if the work is isolated
- include the repo root instead of a single crate when workspace features or app-level integration matter
- add `Agent-Scratchpad` when repo guides, plans, or prior investigations are likely useful
- add `tracy` for profiling-oriented work when the task concerns performance instrumentation or Tracy integration
- add `tracey` only when the task is about spec coverage, tracing requirements, or rules, not runtime profiling

For timeline viewer performance work, a good default workspace usually includes:

- `Skills`
- `Teamy-Studio`
- `Agent-Scratchpad`
- `tracy`

## Output shape

Use a small `folders` list plus light excludes to reduce noise.

Good default settings:

```json
{
  "files.exclude": {
    "**/target": true
  },
  "search.exclude": {
    "**/target": true,
    "**/.git": true,
    "**/node_modules": true
  },
  "files.watcherExclude": {
    "**/target/**": true,
    "**/.git/objects/**": true
  }
}
```

## Naming

Name workspace files after the task, not just the repo.

Examples:

- `teamy-studio-timeline-perf.code-workspace`
- `teamy-studio-figue-help-debug.code-workspace`
- `youtube-channel-sync-investigation.code-workspace`

## Publishing guardrail

Shared or public skills may mention tool names, repo names, and generic example commands.

Shared or public skills should not embed:

- real home-directory paths
- mapped drive paths
- customer, family, or employer names hidden inside paths
- concrete discovery output from local scans

Prefer placeholders such as:

- `<repo-root>`
- `<scratchpad-root>`
- `<skills-root>`
- `<local-checkout-of-dependency>`
- `<workspace-output-path>`
