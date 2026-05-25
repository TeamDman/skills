---
name: publishing-guidelines
description: 'Review machine-specific workflow notes, skills, docs, examples, and helper artifacts before they are committed or shared. Use when deciding how to preserve useful local workflow knowledge without leaking sensitive paths, usernames, internal structure, or other unnecessary machine-specific details.'
argument-hint: 'Describe what artifact may be persisted, how public it will be, and which machine-specific values or examples it currently contains'
---

# Publishing Guidelines

Use this skill when turning local workflow knowledge into committed skills, docs, notes, templates, or examples.

The goal is to preserve useful preferences and workflows without leaking unnecessary machine-specific detail.

## Outcome

Produce a recommendation or edited artifact that:

- keeps the workflow insight
- removes or abstracts unnecessary sensitive detail
- follows least privilege for what is revealed
- asks the user before persisting risky values

## Core rule

Persist the workflow, not the accidental exhaust.

Good to preserve:

- tool names
- repo names that are already intended to be shareable
- command shapes
- decision heuristics
- generic field names and output schemas
- placeholders showing where user-specific values belong

Usually do not persist without review:

- absolute home-directory paths
- drive mappings and storage layout
- usernames, family names, or machine names
- company or customer names hidden inside paths
- raw output from local discovery tools
- examples that expose backup locations, cloud-sync paths, datasets, or private documents

## Procedure

1. Decide how public the artifact will be:
   - private local note
   - small trusted repo
   - company-visible
   - public or potentially public
2. Extract the machine-specific values it currently contains.
3. Keep only the values that are necessary for the workflow to make sense.
4. Replace the rest with placeholders, categories, or safer examples.
5. If any path-like or path-adjacent value may still reveal more than intended, run `is-this-path-sensitive`.

## Preferred transforms

Prefer:

- `%USERPROFILE%\\...` instead of a concrete home path
- `<repo-root>` instead of an absolute checkout path
- `<dataset-root>` instead of a private dataset location
- `<local-tool-on-path>` instead of a hard-coded install path
- `locate-git-projects-on-my-computer.exe` as a tool name instead of pasting its real discovery output

## Principle of least privilege

If a shared skill only needs to teach that a tool exists and what category of problem it solves, do that.

Do not also reveal:

- where the tool is installed
- which exact repos it found on one machine
- which personal files happen to live nearby

## Examples

Good:

- "Use `locate-git-projects-on-my-computer.exe` to discover whether a dependency has a local checkout."
- "Persist a workspace example with placeholder paths and describe what each folder represents."
- "State that a scratchpad repo exists for notes, without listing private note locations."

Ask first or redact:

- concrete absolute paths in examples
- discovery output listing all local repos
- paths containing personal cloud folders, backups, or family data
- screenshots or snippets that expose private directory names

## Relationship to existing skills

- Use `is-this-path-sensitive` when deciding whether a path-like value is safe to persist.
- Use `persist-path-like-references` only after sensitivity review if a path genuinely needs to be recorded.

## Response shape

When using this skill, produce:

1. what the artifact is trying to preserve
2. which values are safe to keep as-is
3. which values should be templated, redacted, or removed
4. whether user confirmation is required before persisting
