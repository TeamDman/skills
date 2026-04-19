---
name: resumable-implementation-plans
description: 'Create resumable implementation plans for features, refactors, migrations, UI work, or reverse-engineering work. Use when a plan file must capture the overall goal, SMART sub-goals, phase-by-phase done criteria, current progress, remaining work, and Tracey or Tracy specification updates and coverage work alongside implementation.'
argument-hint: 'Describe the change, current state, and whether Tracey spec and coverage planning must be included'
---

# Resumable Implementation Plans

Use this skill when drafting or updating a plan file that needs to do more than list ideas.
The plan should guide real implementation, break work into independently valuable slices, and serve as the restart point for future conversations or compacted context.

## Outcome

Produce a plan that:

- states the overarching goal and constraints clearly
- breaks work into discrete phases with actionable tasks
- gives each phase a concrete definition of done
- treats Tracey specification and coverage work as first-class tasks when relevant
- records what has already been completed, what is currently in focus, and what remains
- lets a new agent resume the work from the plan file alone

## When To Use

Use this skill for:

- feature plans
- UI plans
- refactor plans
- migration plans
- reverse-engineering roadmaps
- plans that must stay useful across multiple conversations
- repositories that track behavior with Tracey or Tracy specifications and coverage queries

Do not use this skill for a one-shot checklist with no ongoing status, or for generic brainstorming that is not yet ready to become an implementation plan.

## Core Principles

1. Plan against the real codebase, not an imagined architecture.
2. Break work into phases that each move the system forward in a measurable way.
3. Keep requirements, assumptions, and open questions separate.
4. When Tracey is in play, update the spec plan and the implementation plan together.
5. The plan file is part roadmap, part handoff artifact, and part current-state snapshot.

## Procedure

### 1. Build context from the real environment

Before writing the plan:

- read the user request and any rough notes or existing design docs
- read repository instructions that affect structure or conventions
- inspect the relevant code structure, commands, modules, tests, and docs
- inspect existing spec documents and the local Tracey config when the repo uses Tracey
- if updating an existing plan, read the plan file first and determine what has changed since it was written

Do not start with a generic outline before you know the actual integration points.

### 2. Extract the real planning inputs

From the gathered context, identify:

- the overarching goal
- the observable product or workflow requirements
- repository or architecture constraints
- environment assumptions that should be explicit
- backend gaps that block higher-level work
- already-completed work that should be reflected in the plan
- remaining work that still needs sequencing

If important uncertainty remains, ask a small number of targeted questions before finalizing the plan.
Ask only the questions that materially change the structure of the plan.

### 3. Decide the Tracey specification strategy early

If the repository uses Tracey, do not leave spec work as an afterthought.

Use this branching rule:

- if the change introduces a new behavior area, new user-facing subsystem, or distinct interaction model, create a dedicated Tracey spec for it by default
- if the change is a narrow extension of an existing tracked surface, extend the existing spec instead

Only reuse an existing spec when the fit is obviously narrow, clean, and easier to maintain than a dedicated spec.

When current implementation exists but is not mapped:

- add or refine the spec if the behavior is intentional and should be preserved
- tighten or remove the implementation if the behavior is accidental, obsolete, or too vague to promise

### 4. Establish the Tracey baseline and workflow

When Tracey is relevant, include the concrete command workflow in the plan.

Use these commands as the standard baseline loop:

```powershell
tracey query status
tracey query uncovered
tracey query unmapped
tracey query unmapped --path <path>
tracey query validate --deny warnings
```

Use this follow-up after implementation coverage is under control:

```powershell
tracey query untested
```

The plan should explicitly state:

- the current baseline, when known
- how new requirements will be added to Tracey
- how touched implementation will receive references as it lands
- whether repo-wide coverage debt needs its own cleanup phase to reach full implementation coverage

### 5. Choose phase boundaries that produce real progress

Each phase should be independently valuable and should bring the project closer to the goal when completed on its own.

Good phase boundaries usually separate:

- scaffolding and command registration
- core state or data-model work
- user interaction layers
- backend support required by higher layers
- progress, observability, or instrumentation work
- hardening, tests, docs, and coverage cleanup

Prefer vertical slices over giant categories like "Implement UI" or "Build backend".

### 6. Write the plan structure explicitly

Use a structure close to this:

1. Goal
2. Current Status
3. Constraints And Assumptions
4. Product Requirements
5. Architectural Direction
6. Tracey Specification Strategy
7. Phased Task Breakdown
8. Recommended Implementation Order
9. Open Decisions
10. First Concrete Slice

The Current Status section is always required, even on day one.

Include at least:

- done so far
- current focus
- remaining work
- immediate next step

For a brand-new plan, seed Current Status with the context work already completed, such as reading the repo, choosing the spec strategy, or recording the initial Tracey baseline.

### 7. Write SMART sub-goals inside each phase

For each phase, include:

- objective
- tasks
- definition of done

Keep tasks concrete enough that an implementer can act on them without having to reinterpret the plan.

A good phase:

- has a clear purpose
- has tasks tied to known files, modules, or workflows when possible
- has completion criteria that can be checked
- does not hide unrelated work inside one bucket

### 8. Make the plan resumable on purpose

The plan is not finished when the phase list exists.
It also needs to be usable after context compaction or in a fresh conversation.

To make the plan resumable:

- record what has already been completed
- keep remaining work current instead of letting it drift from reality
- note important decisions that have already been made
- leave open questions in a dedicated section
- include the next recommended slice so a new agent can restart quickly

After each meaningful work session, update the plan before stopping.
When updating an in-progress plan, revise the Current Status section first.
Do not rely on the chat history to communicate progress.

### 9. Review the plan before stopping

Before considering the plan complete, verify that:

- the overall goal is clear
- the requirements are observable and not just aspirational
- assumptions are explicit
- open questions are separated from committed requirements
- the phases are ordered and independently meaningful
- each phase has a definition of done
- Tracey spec work is included when relevant
- completed work and remaining work are both represented
- a new agent could resume from the file alone

## Branching Rules

Use these decision rules while planning:

- New behavior area: create a dedicated Tracey spec by default.
- Small extension of an existing tracked area: extend the existing spec.
- Existing implementation is unmapped but intentional: add or refine the spec and map it.
- Existing implementation is unmapped and should not be promised: tighten or remove it.
- Significant backend dependency for a user-facing feature: add an explicit backend phase.
- Existing repo-wide Tracey debt blocks meaningful progress reporting: add a coverage-cleanup phase instead of pretending the new work is isolated.
- Brand-new plan: seed Current Status with context gathered so far.
- After each meaningful work session: update done so far, remaining work, and next step before stopping.
- In-progress plan: update Current Status before adding new phases.

## Recommended Section Shape

Use this outline as the default shape for the written plan:

```markdown
# <Plan Title>

## Goal

## Current Status
- Done so far:
- Current focus:
- Remaining work:
- Next step:

## Constraints And Assumptions

## Product Requirements

## Architectural Direction

## Tracey Specification Strategy

## Phased Task Breakdown

## Recommended Implementation Order

## Open Decisions

## First Concrete Slice
```

Adjust the headings when needed, but keep the same functional content.

## Anti-Patterns

Avoid these failures:

- writing a plan before reading the relevant code and docs
- producing generic phases that are not tied to real integration points
- listing tasks without definitions of done
- hiding Tracey work in a final cleanup note instead of planning it alongside implementation
- assuming future agents will reconstruct progress from chat history
- mixing unresolved questions into the committed requirements
- forcing a new behavior area into the wrong spec instead of creating a dedicated one
- leaving the plan static after work has already started

## Completion Checklist

Before you finish, confirm all of these:

- The plan matches the repository's structure and conventions.
- The plan contains a real Current Status section.
- The phases are actionable and independently meaningful.
- The plan includes spec and coverage work when Tracey is relevant.
- The plan distinguishes done work from remaining work.
- The next recommended step is explicit.
- The file can serve as a restart point in a new conversation.