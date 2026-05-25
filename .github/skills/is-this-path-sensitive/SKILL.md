---
name: is-this-path-sensitive
description: 'Assess whether a path-like or path-adjacent value is safe to persist in source code, docs, tests, plans, or committed notes. Use when deciding if a path, username, hostname, share name, dataset name, version clue, product code, or similar string should be written down or should require user confirmation first.'
argument-hint: 'Describe the value, where it would be persisted, who would see it, and whether the user already approved publication'
---

# Is This Path Sensitive?

Use this skill before persisting path-like values or nearby strings that may leak more than they appear to.
This is a risk assessment skill, not a binary classifier.

## Outcome

Produce a recommendation that answers:

- what the value reveals
- how public that revelation would be in the chosen artifact
- whether the agent may persist it immediately
- whether the user must be asked first
- whether a redacted or templated form would be safer

## Core Rule

If the value is non-obvious and would be written into source code or committed project artifacts, ask the user before persisting it unless the risk is clearly trivial.

## Risk Dimensions

Evaluate the value across these dimensions:

- personal identity: usernames, family names, initials, home directories
- organization disclosure: company names, internal share names, machine naming schemes, customer names
- product disclosure: codenames, internal product codes, repository names not meant for publication
- environment disclosure: installed tools, SDK roots, dataset names, cache locations, versioned folders
- access disclosure: network shares, drive mappings, storage layout, backup paths
- inference disclosure: strings that indirectly reveal versions, vendors, or internal practices

## Publicity Levels

Assess risk relative to where the value would land:

- private: only the current user or transient session
- family or close collaborators: small trusted audience
- company: visible to coworkers or internal systems
- public: committed source, public docs, screenshots, issue trackers, shared prompts, or published skills

The same string can be acceptable at one level and unacceptable at another.

## Decision Buckets

Use one of these outcomes:

- safe to persist as-is
- persist only in redacted or templated form
- ask the user before persisting
- do not persist; keep it transient

## Heuristics

- A plain project-relative path is usually low risk.
- A user home path is often medium risk because it reveals identity.
- A network share or mapped drive can reveal internal structure even when the path looks ordinary.
- Versioned folders can reveal stack choices and update posture.
- Dataset names, customer names, and product codenames often matter more than the path syntax itself.
- If the value would surprise the user to see committed, ask first.

## Preferred Safer Rewrites

When possible, suggest:

- `%USERPROFILE%\...` instead of a specific home path
- `<repo-root>/...` instead of an absolute checkout path
- `<dataset-root>` instead of a machine-specific dataset location
- `<company-share>` or a redacted placeholder instead of an internal UNC path

## Response Shape

When using this skill, produce:

1. the candidate value
2. what it reveals
3. publicity level for the target artifact
4. recommended action
5. safer rewrite if needed

## Examples Of Ask-First Cases

- absolute home directory paths
- OneDrive or personal cloud paths
- company share names
- machine-specific caches that include usernames
- internal product codes or customer names
- values embedded in source code, tests, screenshots, or public docs

## Examples Of Usually-Safe Cases

- repo-relative paths
- generic placeholder examples
- documented config keys without machine-specific literals
- non-identifying temp paths that are already obviously synthetic