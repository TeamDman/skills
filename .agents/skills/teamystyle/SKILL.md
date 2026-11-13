---
name: teamystyle
description: Apply TeamyStyle when designing, implementing, refactoring, or reviewing Teamy's software, especially Rust domain types, validated boundaries, and associated functions or methods. Read the maintained Teamy Docs sections relevant to the task; also use when explicitly asked for TeamyStyle.
---

# TeamyStyle

Use the maintained documentation to make concrete design decisions. This skill locates and applies the style; it is not a frozen copy of the style manual.

## Find the authoritative guidance

- On Teamy's desktop, start at `G:/Programming/Repos/teamy-docs/src/teamystyle.md`. Read `G:/Programming/Repos/teamy-docs/AGENTS.md` for repository guidance and `src/SUMMARY.md` for navigation. The source checkout is authoritative for current local work; generated `book/` output may lag.
- On another machine, use a user-provided checkout or look for a sibling `teamy-docs` directory within known workspace/repository roots. Confirm `src/teamystyle.md` and the repository identity (`https://github.com/TeamDman/teamy-docs`) rather than scanning the whole filesystem.
- If no local checkout is available, read [TeamyStyle](https://teamdman.github.io/teamy-docs/teamystyle.html) and its relevant linked chapters. The canonical [source repository](https://github.com/TeamDman/teamy-docs) contains `src/teamystyle.md`; follow its current default branch. Do not clone, fetch, or modify a reference checkout without authorization covering that action.
- If sources are unavailable, state that limitation instead of inventing policy. When local and published text differ, identify the source used and retain its distinctions between established requirements, recommendations, proposals, and qualified implementation evidence.

## Read only the sections the task needs

Start with TeamyStyle's introduction and relevant sections, then follow the detailed chapter that owns the contract:

- Domain/API design: **Separate arguments, requests and operations**, **Put one type in one discoverable file**, and **Keep one meaning across front ends**. Consult `src/plan-records.md` for typed versus executable records and effect-boundary validation.
- New CLI foundations and enforcement: `src/rust-cli-template.md`, `src/code-standards.md`, and `src/teamystyle-template-audit.md`.
- CLI interfaces and results: `src/good-cli.md`, `src/cli-parsing.md`, `src/output-shape.md`, and `src/logging.md`.
- Stateful effects and recovery: `src/cancellation.md`, `src/moving-files.md`, and `src/repository-hygiene.md`.
- Other concerns: use `src/problem-index.md` and `src/SUMMARY.md` to select the relevant chapter instead of loading the entire book.

## Apply the guidance to the requested change

Model meaningful identities, policies, and states with domain types/newtypes when they prevent confusion or own an invariant. Validate at construction or the appropriate boundary; a wrapper alone does not establish validity, current object identity, or authorization. Avoid needless wrappers without domain meaning.

Prefer associated constructors, conversions, and methods when behavior naturally belongs to a type and makes its invariants and ownership discoverable. This reflects the user's preference and the docs' `AddActionArgs::into_request` and colocated inherent-implementation examples; it is not a universal prohibition on free functions. The same chapter deliberately illustrates a standalone `add_action` operation. Choose the owner that expresses the domain and dependency direction clearly.

Preserve CLI-to-request-to-operation-to-result boundaries and apply the documented type-file conventions with their stated qualifications. Adopt related CLI, output, logging, cancellation, and dependency conventions only where the task calls for them. Cite the relevant source section when explaining a consequential design choice; distinguish a documented requirement from your implementation judgment.

Respect the user's scope, target repository instructions, existing dirty work, and approval boundaries. Applying this skill does not authorize broad rewrites, changes to style policy, publication, or unrelated repository operations. Validate the behavior changed with evidence appropriate to its contract.
