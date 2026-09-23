---
name: persist-path-like-references
description: 'Persist and summarize path-like references mentioned by the user. Use when a response includes Windows paths, Unix-style paths, home-relative paths, UNC paths, mount-like paths, or other filesystem-like strings that should be recorded in the project for later use.'
argument-hint: 'Describe the paths, where they came from, which project artifact should record them, and whether sensitivity review has already happened'
---

# Persist Path-Like References

Use this skill when a conversation surfaces filesystem-like references that should survive beyond the current chat.
This skill is for turning those references into project memory, notes, config inputs, plans, or docs in a deliberate way.

## Outcome

Produce a project-local artifact that:

- captures the path-like references mentioned by the user
- summarizes what each path is for
- records enough context for future agents to use the reference correctly
- avoids silently publishing risky values without review

## What Counts As Path-Like

Treat these as path-like references:

- Windows drive paths such as `C:\Users\name\project`
- UNC paths such as `\\server\share\folder`
- Unix-style paths such as `/home/name/project`
- home-relative paths such as `~/repo`
- mount-like paths such as `g:\Programming\Repos\...`
- tool output that clearly points to files, folders, working directories, caches, datasets, or install roots

## Procedure

1. Extract the path-like references exactly as the user provided them.
2. Determine the role of each reference: source tree, dataset, cache, install root, output folder, scratch area, or example-only path.
3. Summarize the paths in a project-local artifact that matches the repo workflow.
4. Record why the path matters, not just the literal string.
5. If the path could reveal private or organization-specific information, stop and invoke the `is-this-path-sensitive` skill before persisting it.

## Preferred Persistence Targets

Choose the lightest project-local artifact that will still be discoverable later:

- active implementation plan
- repo notes
- troubleshooting doc
- setup doc
- machine-specific non-source config sample when the repo already uses one

Do not scatter the same path across multiple files without a clear reason.

## Required Summary Shape

When persisting a path-like reference, record:

- the literal path or a redacted form when appropriate
- what it points to
- why it matters to the current task or project
- whether it is expected to vary by machine
- whether user confirmation was required before writing it down

## Guardrails

- Do not assume all paths are safe to commit just because the user typed them.
- Do not copy user-specific paths into source code defaults unless the user explicitly wants machine-specific behavior.
- Prefer documenting variables, placeholders, or categories when the literal path is not required.
- If a path is only an example, label it as an example instead of treating it as canonical configuration.

## Good Uses

- a dataset root that future experiments must reuse
- a cache directory that explains a failing build or tool lookup
- a project-local note about where a binary, SDK, or external checkout lives
- implementation plans that need to remember where the user pointed the agent

## Bad Uses

- blindly committing a home directory path into source
- writing company-internal share paths into public docs without review
- persisting a path when a variable name or placeholder would be enough

## Relationship To Sensitivity Review

This skill assumes persistence is desired, but not automatically safe.
If there is any doubt about usernames, hostnames, customer names, product codes, internal share names, version clues, or project naming leakage, run the `is-this-path-sensitive` skill first and follow its result.