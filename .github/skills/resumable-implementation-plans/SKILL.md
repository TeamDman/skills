---
name: resumable-implementation-plans
description: Create or maintain a living Markdown implementation plan for a feature, integration, refactor, migration, or multi-version change. Use when implementation must survive context restarts or compaction without losing user intent and needs explicit scope, design decisions, task-level progress, validation evidence, risk management, or cross-target acceptance.
---

# Resumable Implementation Plans

Create a plan that is an executable work contract, not an idea list or detached
work log. A new agent must be able to resume safely from the file alone.

## Plan against evidence

Inspect before writing or revising a plan:

- Read repository and directory instructions, the existing plan, related docs,
  source entry points, tests, build commands, and release/propagation workflow.
- Identify the actual modules, files, data boundaries, dependencies, and test
  harnesses involved. Cite paths and commands that will help the next agent.
- Separate:
  - **verified foundation** — facts established by code, tests, or prior work;
  - **confirmed decisions** — choices already made by the user or project;
  - **assumptions** — provisional beliefs that need validation; and
  - **open decisions** — choices that materially change design, scope, or
    acceptance.
- Never claim a behavior, API, dependency version, test result, or commit
  exists without evidence from the repository or an authoritative source.

Ask only the questions that change architecture, public behavior, irreversible
data choices, support policy, or the testable definition of success. Otherwise
make a stated, reversible working assumption.

## Preserve user intent before shaping the plan

Treat the user's instructions as a requirements source, not as conversational
background to summarize loosely. Before deciding the plan structure:

- Extract every distinct requested outcome, constraint, preference, concern,
  example that carries design meaning, explicit non-goal, path, tool, target,
  ordering statement, and correction from the available user messages.
- Preserve small qualifiers such as “later,” “may,” “primarily,” “without,” and
  “rather than.” They often distinguish phases, confidence, or ownership.
- Give each active requirement a stable ID when the request is multi-part,
  multi-turn, likely to be compacted, or otherwise nuanced.
- Keep an authoritative guidance/requirements ledger in the plan. Do not remove
  or silently generalize an entry. If the user changes direction, mark the old
  entry as superseded by the new stable ID.
- Map every active requirement to scope, a decision gate, one or more work items,
  validation, or an explicit reasoned non-goal. Omission is not an acceptable
  disposition.
- Distinguish the user's confirmed direction from an agent recommendation even
  when they currently agree.
- When the full conversation is available, inspect the original user messages;
  do not rely on memory or an intermediate summary. When compaction has already
  removed original messages, use the plan's existing ledger and available
  durable notes, state that limitation in the audit evidence, and do not claim
  to have reread unavailable text.

A suitable ledger and traceability shape is:

```markdown
## Authoritative user guidance ledger

| ID | Active guidance | Required plan consequence | Superseded by |
| --- | --- | --- | --- |
| U1 | <faithful requirement, retaining important qualifiers> | <where it changes scope/design/acceptance> | — |

## Guidance traceability

| Guidance | Plan coverage | Evidence when complete |
| --- | --- | --- |
| U1 | 1.2, G3, overall criterion 4 | Pending |
```

The plan must remain useful without the conversation. A fresh agent should be
able to reconstruct not just what to implement, but why alternatives were kept,
deferred, or rejected.

## Triple-check intent before declaring the plan ready

The plan is only ready once we have literally triple checked that no intent from
the user has been omitted without explicit direction from the user.

Perform three separate passes; repeating one generic review three times does not
satisfy this requirement:

1. **Extraction pass — instruction to ledger.** Reread the available original
   user instructions from beginning to end. Atomize compound requests and verify
   that every outcome, constraint, path, tool, example with design force,
   sequencing statement, concern, correction, and non-goal has an active ledger
   entry or an explicit supersession.
2. **Traceability pass — ledger to executable plan.** For every active ledger
   entry, locate concrete coverage in scope, gates, tasks, validation, and/or
   completion criteria. Then check the inverse: every material plan choice must
   be supported by verified evidence, user guidance, or a clearly labeled
   reversible assumption.
3. **Adversarial omission pass — plan back to instructions.** Reread the user
   instructions after the plan appears complete and actively search for nuance
   that was weakened, merged, reordered, converted into an assumption, or lost.
   Pay special attention to later clarifications, contrasts between tools or
   targets, confidence qualifiers, lifecycle/ownership boundaries, and examples
   that imply required behavior.

Record durable evidence near the top of the plan rather than merely stating
“reviewed” in chat:

```markdown
**Intent audit:** Passed YYYY-MM-DD against <message/source range>

## Intent audit evidence

- **Pass 1 — extraction:** <what sources were reread; ledger IDs added/fixed>
- **Pass 2 — traceability:** <how every active ID was mapped; gaps fixed>
- **Pass 3 — adversarial omission:** <qualifiers/corrections checked; gaps fixed>
- **Known source limitation:** None, or <unavailable compacted material and the
  durable ledger/note used instead>.
```

If any pass finds an unrepresented material intent, repair the ledger and plan,
then rerun all three passes because the repair can change other coverage. If a
source limitation prevents a defensible audit, mark the intent audit pending and
ask only for the material missing direction that cannot be recovered safely.

## Use the living-plan protocol

Give every plan a short metadata block and this update rule near the top:

```markdown
**Plan status:** Active
**Primary implementation root:** `<path or branch>`
**Last updated:** YYYY-MM-DD
**Intent audit:** Pending

## How to update this plan

- `[ ]` Not started
- `[~]` In progress
- `[x]` Complete
- `[!]` Blocked

Put a work item's status in its heading. Update its heading and its completion
notes together. A phase is complete only when every work item in it is
`[x]`. Record decisions, commit IDs, validation results, and follow-ups below
the task they affect; do not append a detached chronological work log.
```

Use `[!]` only with the exact blocker, the last evidence gathered, and the
condition that would unblock it. Do not use it merely for unfinished work.

Keep at most one current implementation focus unless parallel work is
intentional and the plan names the independent owners or tracks.

## Shape the plan around the change

Use only the sections that add decision-making value, but normally include:

1. **Authoritative user guidance ledger and traceability** — stable intent that
   survives compaction, plus where each requirement is implemented and proven.
2. **Intent audit evidence** — the three distinct readiness passes and any known
   source limitation.
3. **Purpose** — the observable outcome and its primary user/system value.
4. **Scope** — explicit in-scope and out-of-scope boundaries.
5. **Established foundation** — prerequisites already completed and deliberately
   not reopened.
6. **Confirmed constraints** — non-negotiable invariants, ownership/lifecycle
   rules, compatibility policy, security/privacy constraints, and repository
   workflow constraints.
7. **Design questions that must be closed before implementation** — a gate table
   for material unknowns.
8. **Source and implementation references** — relevant paths, commands, issue
   context, and authoritative external sources.
9. **Execution order** — a compact dependency flow where phase ordering is not
   obvious.
10. **Phases and work items** — the implementation contract.
11. **Overall completion criteria** — release-level proof, not a restatement of
   the task list.
12. **Risk register** — risks that need a mitigation or a validation gate.

Do not add a generic “current status” section that duplicates task statuses.
Use the task headings and completion notes as the authoritative current state.
Add a brief “next slice” only when the immediate next action cannot be inferred
from the first `[ ]` or `[~]` work item.

### Define scope precisely

Write scope in terms of observable behavior and ownership boundaries. State
what will not be changed so adjacent work does not expand incidentally.

For example, distinguish:

- production integration from test-fixture infrastructure;
- a stable public contract from internal storage representation;
- a feature implementation from dependency/toolchain redesign;
- a supported target from a target intentionally outside the support matrix.

When a prior plan established a prerequisite, record it as foundation and
reference it. Do not re-plan or silently alter that prior decision.

### Turn uncertainty into a decision gate

Before implementation, make material design choices visible in a table:

```markdown
| Question | Required decision | Acceptance consequence |
| --- | --- | --- |
| Entry point | Which objects expose the feature? | Topology tests cover every allowed and rejected entry point. |
| Public contract | Exact names, inputs, outputs, errors, and lifecycle | Call-level tests assert every declared behavior. |
| Compatibility | Which targets are supported and how adapters work? | A target matrix records supported, excluded, and adapter-tested cases. |
```

A gate is not a brainstorming list. Each question must name the decision needed
and the test or documentation consequence. Close the gate in completion notes
before downstream implementation begins.

## Write task-level work contracts

Organize phases by dependencies and meaningful slices, not generic departments.
Prefer a progression such as foundation → contract/reconnaissance → narrow
adapter/core behavior → adjacent surfaces → end-to-end acceptance →
documentation/propagation/release.

Give each work item this form:

````markdown
### [ ] 2.1 <verb-led, bounded outcome>

**Work:**

- Change the named modules or behavior.
- Preserve the named invariant or lifecycle.
- Add the smallest meaningful proof.

**Validation:**

```pwsh
<exact current command>
````

**Completion criteria:** <observable condition that makes this item true>
```

For an active or completed item, place **Completion notes** immediately below
the heading. Include only durable evidence:

- the actual design decision made;
- affected paths or public surface;
- commit IDs when useful for provenance;
- commands run and their relevant outcome;
- intentional exceptions, follow-ups, or support limits.

Do not use vague completions such as “implemented” or “tests pass.” Name what
was proven and by which command/test.

Keep work items small enough to complete and validate independently. Split
work that combines a public contract decision, state-model change, integration,
and release work. A phase may contain several related tasks, but it should not
hide unrelated changes behind one completion checkbox.

## Put validation beside the behavior

Every non-trivial task needs a validation strategy before implementation.
Choose the smallest proof that exercises the claimed layer:

- unit or parser test for pure logic;
- integration test for a boundary adapter;
- real runtime/GameTest/browser/CLI invocation for discovery and wiring;
- migration or fixture test for persisted data;
- documentation/example check for public contracts.

A direct adapter test does not prove runtime registration. A smoke test does
not prove feature behavior. State which layer each test proves.

Use exact commands from the current repository. Include filters, branches,
working directories, prerequisites, and environment assumptions when they
matter. Treat successful commands as evidence only when they ran against the
current source tree, not a stale installed binary.

For multi-target work, include an acceptance matrix rather than vague
“test all versions” wording:

```markdown
| Target/dialect | Support status | Required validation | Evidence |
| --- | --- | --- | --- |
| Baseline target | supported | compile + focused + full suite | pending |
| Adapter target | supported via adapter | compile + adapter test | pending |
| No upstream dependency | explicitly unsupported | ordinary compile; source excluded by toolchain policy | pending |
```

Record unavailable dependencies, unsupported targets, and intentionally skipped
tests explicitly. Do not delete shared feature sources merely to make one
target compile when a source-set/toolchain exclusion preserves propagation
history more safely.

## Plan compatibility and propagation deliberately

When code must span versions, loaders, platforms, or deployment targets:

- Name the primary implementation target and implement the common intent there
  first.
- Separate genuine target-specific adapters from ordinary divergence.
- State the propagation method, conflict-resolution rule, and audit command.
- Preserve newer-target behavior during propagation; do not overwrite it with
  the baseline implementation.
- Put source-set exclusions, dependency availability, and target support policy
  in the toolchain/configuration plan so shared source remains merge-friendly.
- Add focused validation for each adapter or exclusion boundary.

Do not make a later target the silent source of truth. Any exception must be
documented with its support consequence.

## Include documentation, release, and risks when relevant

Add a final phase for user-facing documentation, changelog/release notes,
propagation, and cross-target acceptance when the change reaches users.

Write overall completion criteria as a checklist of release truths, for
example: every task has evidence; public contract and docs agree; data
integrity is preserved; targeted and full validation pass; supported targets
are proven or accurately scoped out.

Use a risk register for risks that can invalidate the plan:

- state corruption or lifecycle bypass;
- public API versus internal implementation coupling;
- migration/compatibility drift;
- test false confidence;
- cache/network/authentication or external-service limits;
- propagation and adapter drift.

For each risk, name the guardrail, test, or phase that controls it.

## Repository-specific extensions

Treat a repository’s required specification/coverage system as a first-class
plan surface only when that repository uses one. Inspect its instructions and
include the exact spec, traceability, coverage, or quality commands alongside
the affected phases. Do not impose Tracey, Tracy, or another tool on unrelated
repositories.

## Review before handoff

Before ending a planning or implementation session, verify:

- The three-pass intent audit was performed against the best available source,
  is recorded in the plan, and remains honest about compacted/unavailable text.
- Every active guidance/requirement ID has a concrete disposition and every
  material plan choice is evidence-backed, user-directed, or labeled assumption.
- No user intent was omitted, weakened, or superseded without explicit user
  direction.
- The plan distinguishes facts, decisions, assumptions, and open gates.
- Scope and non-goals prevent accidental expansion.
- Every phase has an order justified by dependencies.
- Every work item has work, validation, and observable completion criteria.
- Completed items contain local evidence, not a detached summary.
- Public behavior, data boundaries, and compatibility policy are explicit.
- The support/acceptance matrix covers every advertised target.
- Risks have concrete mitigations.
- A fresh agent can identify the next safe action from the plan alone.

If any of the first three checks fails, the plan is still draft regardless of
how complete its task list appears.
