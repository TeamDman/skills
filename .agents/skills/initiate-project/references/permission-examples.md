# Worked permission evaluations

These examples instantiate the types in [permission-model.md](permission-model.md).
Every request and observation is hypothetical. No sample consent is a grant for
an actual operation. Output is an `Evaluation`; it records no execution.

Each example uses the same complete envelope. Empty collections are explicit.
Source evidence describes the relevant fictional user message or inspection;
real evaluations refer back to actual evidence. Paths containing `<skills-repo>`
are illustrative placeholders, not resolved execution targets.

Contents: interest (1), showing versus saving (2), read versus fetch (3),
checkout and restoration (4–5), project identity (6), clone consent (7–8),
and publication (9).

## 1. Interest is not yet an action request

No concrete operation is proposed until the user's interest is clarified. The question belongs to the interpreted intent, so an empty decision list is meaningful.

```yaml
input:
  id: interest
  intent:
    id: i1
    kind: Interest
    certainty: Inferred
    subject: A shortcut education panel
    delivery:
      kind: Unspecified
    evidence:
    - u1
  actions: []
  grants: []
  state: s1
  facts: {}
  evidence:
  - id: u1
    origin: UserMessage
    summary: It would be neat if we had a shortcut education panel.
output:
  request_id: interest
  decisions: []
  questions:
  - id: q1
    purpose: ClarifyIntent
    subjects:
    - kind: Intent
      intent_id: i1
    prompt: Would you like that to be built, or are we exploring the idea?
  recommendations: []
```

## 2. Showing examples and saving examples are separate actions

The request clearly asks to see examples. Persistence is unresolved. Both candidate actions are explicit, and only the presentation is allowed without that additional decision.

```yaml
input:
  id: show-examples
  intent:
    id: i1
    kind: Show
    certainty: Explicit
    subject: Examples of permission decisions
    delivery:
      kind: Unspecified
    evidence:
    - u1
  actions:
  - id: a-show
    purpose: Explain the permission model
    effect:
      kind: Present
      subject: Example inputs and outputs
  - id: a-save
    purpose: Retain examples for future skill readers
    effect:
      kind: Save
      subject: Reviewed example inputs and outputs
      path: <skills-repo>/.agents/skills/initiate-project/references/permission-examples.md
  grants: []
  state: s1
  facts: {}
  evidence:
  - id: u1
    origin: UserMessage
    summary: Some example inputs and outputs would be neat and I'd like to see some please.
output:
  request_id: show-examples
  decisions:
  - action_id: a-show
    outcome:
      kind: ALLOW
      authority:
      - kind: UserRequest
        evidence_id: u1
      reasons:
      - rule: intent
        explanation: The user explicitly requested examples to see.
        evidence:
        - u1
  - action_id: a-save
    outcome:
      kind: NEEDS_CLARIFICATION
      unresolved:
      - Whether to persist these examples in the proposed artifact
      reasons:
      - rule: delivery
        explanation: Seeing examples does not settle whether they should be saved.
        evidence:
        - u1
  questions:
  - id: q1
    purpose: ChooseDelivery
    subjects:
    - kind: Action
      action_id: a-save
    prompt: Should the examples remain in chat, or should I also save the reviewed examples in the skill's
      permission-examples.md?
  recommendations:
  - action_id: a-show
    advice: Present the requested examples while the optional persistence question remains open.
```

## 3. An old reference clone: one decision per action

Reading and fetching are separate proposals. Each has its own outcome and reasons; neither appears as an unexplained allowance beside a global decision.

```yaml
input:
  id: old-reference
  intent:
    id: i1
    kind: Explore
    certainty: Explicit
    subject: The editor's shortcut API
    delivery:
      kind: Unspecified
    evidence:
    - u1
  actions:
  - id: a-read
    purpose: Research the shortcut API
    effect:
      kind: ReadReference
      repo: G:\Programming\Repos\editor-source
      subject: Shortcut API source
  - id: a-fetch
    purpose: Refresh the source available for that research
    effect:
      kind: FetchReference
      repo: G:\Programming\Repos\editor-source
      remote: origin
  grants: []
  state: s1
  facts:
    a-read:
      within_reference_roots:
        value: true
        evidence:
        - l1
      task_relevant:
        value: true
        evidence:
        - u1
        - l1
  evidence:
  - id: u1
    origin: UserMessage
    summary: Use the local editor-source clone to research the shortcut API.
  - id: l1
    origin: LocalInspection
    summary: The relevant clone is under an authorized reference root; its recorded local update is 18
      months old.
output:
  request_id: old-reference
  decisions:
  - action_id: a-read
    outcome:
      kind: ALLOW
      authority:
      - kind: StandingRule
        rule: reference-read
      reasons:
      - rule: reference-read
        explanation: Relevant inspection is covered by the standing read grant.
        evidence:
        - u1
        - l1
  - action_id: a-fetch
    outcome:
      kind: NEEDS_CONSENT
      missing:
      - Scoped consent to fetch origin in this reference clone
      reasons:
      - rule: reference-mutation
        explanation: Age suggests possible staleness but grants no permission to update local objects
          and refs.
        evidence:
        - l1
  questions:
  - id: q1
    purpose: GrantConsent
    subjects:
    - kind: Action
      action_id: a-fetch
    prompt: May I fetch origin in G:\Programming\Repos\editor-source to refresh its local objects and
      tracking references for this research?
  recommendations:
  - action_id: a-read
    advice: Prefer the existing source when it is sufficient to answer the question.
```

## 4. A temporary revision switch is allowed

All checkout guards are established for this state. Permission to switch does not prove it happened, and any later restoration must be assessed against its own current state.

```yaml
input:
  id: safe-switch
  intent:
    id: i1
    kind: Explore
    certainty: Explicit
    subject: An earlier implementation of the API
    delivery:
      kind: Unspecified
    evidence:
    - u1
  actions:
  - id: a-switch
    purpose: Inspect the earlier API implementation
    effect:
      kind: SwitchRevision
      repo: G:\Programming\Repos\editor-source
      target: def5678
  grants: []
  state: s1
  facts:
    a-switch:
      within_reference_roots:
        value: true
        evidence:
        - l1
      task_relevant:
        value: true
        evidence:
        - u1
      target_local:
        value: true
        evidence:
        - l1
      clean:
        value: true
        evidence:
        - l1
      fully_pushed:
        value: true
        evidence:
        - r1
      no_work_loss:
        value: true
        evidence:
        - l1
      preserve_branch_tips:
        value: true
        evidence:
        - l1
  evidence:
  - id: u1
    origin: UserMessage
    summary: Compare the earlier implementation with what is here.
  - id: l1
    origin: LocalInspection
    summary: 'State s1: the clone is within the authorized roots, starts detached at abc1234, target def5678
      is local, index/worktree/untracked/affected submodules are clean, and a detached switch preserves
      branch tips and ignored files.'
  - id: r1
    origin: RemoteQuery
    summary: Current remote reachability establishes that every local branch and HEAD is published.
output:
  request_id: safe-switch
  decisions:
  - action_id: a-switch
    outcome:
      kind: ALLOW
      authority:
      - kind: StandingRule
        rule: temporary-checkout
      reasons:
      - rule: temporary-checkout
        explanation: Every guard for the standing temporary-checkout permission is established in state
          s1.
        evidence:
        - u1
        - l1
        - r1
  questions: []
  recommendations:
  - action_id: a-switch
    advice: Remember the original detached abc1234 state before switching.
```

## 5. New edits prevent automatic restoration

A false guard is sufficient to reject this operation without asserting that every other guard has been evaluated.

```yaml
input:
  id: unsafe-return
  intent:
    id: i1
    kind: Explore
    certainty: Explicit
    subject: Return after temporary source inspection
    delivery:
      kind: Unspecified
    evidence:
    - u1
  actions:
  - id: a-restore
    purpose: Restore the reference's original stopping point
    effect:
      kind: RestoreRevision
      repo: G:\Programming\Repos\editor-source
      original:
        commit: abc1234
        mode:
          kind: Detached
  grants: []
  state: s2
  facts:
    a-restore:
      clean:
        value: false
        evidence:
        - l2
  evidence:
  - id: u1
    origin: UserMessage
    summary: Use this reference for temporary inspection.
  - id: l2
    origin: LocalInspection
    summary: 'State s2: HEAD is detached at def5678; notes.md has new uncommitted edits. The recorded
      original state is detached at abc1234.'
output:
  request_id: unsafe-return
  decisions:
  - action_id: a-restore
    outcome:
      kind: CONDITION_NOT_MET
      failed:
      - clean
      reasons:
      - rule: restoration
        explanation: The return no longer meets the cleanliness condition.
        evidence:
        - l2
  questions: []
  recommendations:
  - action_id: a-restore
    advice: Defer restoration and preserve the new edits. Any proposed stash, reset, or other handling
      needs its own scope assessment.
```

## 6. A build request still needs a confirmed project identity

The proposed action identifies the exact candidate repository. The output refers to that proposal instead of issuing a general approval to create projects.

```yaml
input:
  id: new-project
  intent:
    id: i1
    kind: Change
    certainty: Explicit
    subject: Build a shortcut display extension
    delivery:
      kind: Unspecified
    evidence:
    - u1
  actions:
  - id: a-create
    purpose: Create the requested extension project
    effect:
      kind: CreateRepository
      owner: example-maintainer
      name: shortcut-demo
      visibility: Public
      license: MPL-2.0
      destination: G:\Programming\Repos\shortcut-demo
  grants: []
  state: s1
  facts: {}
  evidence:
  - id: u1
    origin: UserMessage
    summary: Please build a shortcut display extension.
output:
  request_id: new-project
  decisions:
  - action_id: a-create
    outcome:
      kind: NEEDS_CONSENT
      missing:
      - Confirmation of shortcut-demo as the name
      - Approval of this repository's owner, location, visibility, and license
      reasons:
      - rule: identity
        explanation: The build request does not establish the proposed repository identity.
        evidence:
        - u1
  questions:
  - id: q1
    purpose: GrantConsent
    subjects:
    - kind: Action
      action_id: a-create
    prompt: May I use shortcut-demo and create example-maintainer/shortcut-demo as a public MPL-2.0 project
      destined for G:\Programming\Repos\shortcut-demo?
  recommendations: []
```

## 7. Clone consent includes the actual provenance

The URL and maintainer are fictional. A real evaluation uses the discovered source and evidence, not this example as a trust verdict.

```yaml
input:
  id: third-party-clone
  intent:
    id: i1
    kind: Explore
    certainty: Explicit
    subject: Possible keyboard-display implementations
    delivery:
      kind: Unspecified
    evidence:
    - u1
  actions:
  - id: a-clone
    purpose: Inspect a possible implementation reference
    effect:
      kind: Clone
      url: https://example.org/independent/keyboard-display.git
      destination: G:\Programming\Repos\keyboard-display-reference
      provenance:
        authors: Independent maintainer; identity unverified
        learned_from: A link in a forum discussion
        first_party: false
        trust_assessment: Limited assurance from the available authorship information; code has not been
          reviewed
        risks:
        - Unreviewed files would be brought into the local reference directory
        planned_execution: []
  grants: []
  state: s1
  facts: {}
  evidence:
  - id: u1
    origin: UserMessage
    summary: Research possible implementations for a keyboard display.
  - id: l1
    origin: LocalInspection
    summary: The proposed source was linked in a forum; no informed clone consent is recorded.
output:
  request_id: third-party-clone
  decisions:
  - action_id: a-clone
    outcome:
      kind: NEEDS_CONSENT
      missing:
      - Informed consent for this URL, destination, provenance, and intended use
      reasons:
      - rule: clone-consent
        explanation: Research authorization does not itself authorize this clone.
        evidence:
        - u1
        - l1
  questions:
  - id: q1
    purpose: GrantConsent
    subjects:
    - kind: Action
      action_id: a-clone
    prompt: May I clone https://example.org/independent/keyboard-display.git into G:\Programming\Repos\keyboard-display-reference
      for inspection? It was linked in a forum, has an unverified independent maintainer, and is not first
      party to the editor. The source is unreviewed, so assurance is limited. The proposal brings its
      files locally; it includes no build or install-script execution.
  recommendations: []
```

## 8. Reuse earlier consent for the exact clone

Creation is already complete in this hypothetical state. This evaluates only the clone, reusing the earlier grant without asking again.

```yaml
input:
  id: approved-clone
  intent:
    id: i1
    kind: Change
    certainty: Explicit
    subject: Set up the new extension project
    delivery:
      kind: Unspecified
    evidence:
    - u1
  actions:
  - id: a-clone
    purpose: Clone the newly created extension project
    effect:
      kind: Clone
      url: https://example.org/example-maintainer/shortcut-demo.git
      destination: G:\Programming\Repos\shortcut-demo
      provenance:
        authors: The user's public pseudonymous account
        learned_from: A new project proposed in this conversation
        first_party: true
        trust_assessment: A verified new empty first-party remote; low initial clone risk
        risks:
        - Creates a local checkout at the agreed destination
        planned_execution: []
  grants:
  - id: g1
    effect:
      kind: Clone
      url: https://example.org/example-maintainer/shortcut-demo.git
      destination: G:\Programming\Repos\shortcut-demo
      provenance:
        authors: The user's public pseudonymous account
        learned_from: A new project proposed in this conversation
        first_party: true
        trust_assessment: A verified new empty first-party remote; low initial clone risk
        risks:
        - Creates a local checkout at the agreed destination
        planned_execution: []
    purpose: Clone the newly created extension project
    source: u1
    disclosure:
    - u1
  state: s3
  facts:
    a-clone:
      destination_absent:
        value: true
        evidence:
        - l1
      source_matches_disclosure:
        value: true
        evidence:
        - r1
  evidence:
  - id: u1
    origin: UserMessage
    summary: Confirmed the exact clone after reviewing the proposed source, destination, first-party authorship,
      discovery source, risk assessment, and planned execution.
  - id: l1
    origin: LocalInspection
    summary: 'State s3: the destination is absent.'
  - id: r1
    origin: RemoteQuery
    summary: 'State s3: the agreed new empty remote exists under the expected account and its provenance
      matches the disclosure.'
output:
  request_id: approved-clone
  decisions:
  - action_id: a-clone
    outcome:
      kind: ALLOW
      authority:
      - kind: Consent
        grant_id: g1
      reasons:
      - rule: clone-consent
        explanation: The earlier informed grant covers this exact effect and purpose; current source and
          destination checks match.
        evidence:
        - u1
        - l1
        - r1
  questions: []
  recommendations: []
```

## 9. Publication authority does not satisfy the privacy guard

Evidence identifies the file and category without storing the sensitive value in the example or output.

```yaml
input:
  id: sensitive-push
  intent:
    id: i1
    kind: Change
    certainty: Explicit
    subject: Publish the prepared change
    delivery:
      kind: Unspecified
    evidence:
    - u1
  actions:
  - id: a-publish
    purpose: Push the prepared commit
    effect:
      kind: Publish
      repo: G:\Programming\Repos\shortcut-demo
      destination: https://example.org/example-maintainer/shortcut-demo.git
      artifact: The prepared commit
  grants:
  - id: g1
    effect:
      kind: Publish
      repo: G:\Programming\Repos\shortcut-demo
      destination: https://example.org/example-maintainer/shortcut-demo.git
      artifact: The prepared commit
    purpose: Push the prepared commit
    source: u1
    disclosure: []
  state: s1
  facts:
    a-publish:
      outgoing_review_complete:
        value: true
        evidence:
        - l1
      no_sensitive_content:
        value: false
        evidence:
        - l1
  evidence:
  - id: u1
    origin: UserMessage
    summary: Push the prepared commit to the agreed public repository.
  - id: l1
    origin: LocalInspection
    summary: The outgoing content and commit identity were reviewed. examples/config.json contains a personal
      email address; its value is omitted.
output:
  request_id: sensitive-push
  decisions:
  - action_id: a-publish
    outcome:
      kind: CONDITION_NOT_MET
      failed:
      - no_sensitive_content
      reasons:
      - rule: publication
        explanation: The reviewed outgoing artifact still contains user-identifying information.
        evidence:
        - l1
  questions: []
  recommendations:
  - action_id: a-publish
    advice: Defer publication until the artifact is corrected within the authorized edit scope and reviewed
      again.
```

## Change one observation, reassess the same action

Starting from example 4, keep its action, intent, and other facts unchanged:

| New observation for `a-switch` | Its new outcome |
| --- | --- |
| `fully_pushed` is `false`, evidenced by an unpushed commit on another local branch | `CONDITION_NOT_MET`, with `failed: [fully_pushed]` and the reason/evidence inside that outcome. |
| `fully_pushed` is `unknown`, because tracking refs are stale and remote reachability is unproven | `NEEDS_EVIDENCE`, with `unknown: [fully_pushed]`. A fetch is not implicitly authorized. |

The result still contains exactly one decision for `a-switch`. If read-only
checks or a fetch are proposed next, they become explicit actions in a new
`EvaluationRequest`. Recommendations are not extra decisions.

## Batch questions without merging permission

Example 2's delivery question and example 3's fetch question could be asked
together when both are relevant to the active task. Each retains its own ID,
purpose, and subjects. An answer approving persistence resolves only the
`Save` proposal. It grants nothing to `FetchReference`. Questions that depend
on an earlier answer remain sequential.
