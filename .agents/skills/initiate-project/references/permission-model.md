# The collaboration boundary

This document models the conversation we enter when using `initiate-project`.
It uses programming-language ideas to make ownership and transformations
visible. It is not a program, a source of consent, or a replacement for the
policy in [SKILL.md](../SKILL.md).

Contents: [Transformations](#transformations) | [Types](#types) |
[Invariants](#invariants) | [Policy mapping](#policy-mapping) |
[Worked examples](permission-examples.md)

## Transformations

```text
interpret : (UserMessages, ExistingContext) -> IntentDraft
propose   : (IntentDraft, ExistingContext) -> List<Action>
assess    : EvaluationRequest -> Evaluation
elicit    : (List<Question>, UserAnswers) -> RevisedContext
```

`interpret` is the neural step: the agent interprets unstructured language
while retaining its source and uncertainty. An interpretation is not a grant.
`propose` makes candidate actions explicit, including separate actions for
showing an example, saving it, or installing a skill. A proposal is not an
execution request.

`assess` applies the current skill policy and actual user grants to those
actions. `elicit` gathers missing intent, destination choices, or consent;
answers revise the context and affected actions are assessed again. A tool
result records execution separately. An `ALLOW` result is never evidence
that an action has happened.

## Types

The notation is schematic. IDs are distinct named string types;
`List<T>`, `NonEmptyList<T>`, `Map<K,V>`, and tagged unions have their usual
programming meanings. Each union variant has exactly the fields shown.

```text
IntentDraft = {
  id: IntentId,
  kind: Interest | Explore | Show | Change,
  certainty: Explicit | Inferred | Unresolved,
  subject: Text,
  delivery: Chat | Artifact(Path) | Unspecified,
  evidence: NonEmptyList<EvidenceId>
}

Action = { id: ActionId, purpose: Text, effect: Effect }
Effect =
    Present { subject: Text }                         // Destination is this chat.
  | Save { subject: Text, path: Path }                 // Named persistent artifact.
  | CreateRepository { owner: Text, name: Text, visibility: Public | Private,
                       license: Text, destination: Path }
  | Clone { url: URL, destination: Path, provenance: CloneProvenance }
  | ReadReference { repo: Path, subject: Text }
  | FetchReference { repo: Path, remote: Text }
  | PullReference { repo: Path, remote: Text, branch: Text }
  | SwitchRevision { repo: Path, target: CommitId }
  | RestoreRevision { repo: Path, original: CheckoutState }
  | InitializeRust { template: Path, destination: Path }
  | Publish { repo: Path, destination: URL, artifact: Text }
  | LinkSkill { source: Path, destination: Path }

CheckoutState = { commit: CommitId, mode: Detached | Branch { name: BranchName } }
CloneProvenance = {
  authors: Text, learned_from: Text,
  first_party: True | False | Unknown,
  trust_assessment: Text, risks: List<Text>, planned_execution: List<Text>
}

Evidence = { id: EvidenceId,
             origin: UserMessage | LocalInspection | RemoteQuery | Inference,
             summary: Text }
Grant = { id: GrantId, effect: Effect, purpose: Text,
          source: EvidenceId, disclosure: List<EvidenceId> }
Observation = { value: True | False | Unknown, evidence: List<EvidenceId> }

// ConditionId names a policy predicate, e.g. clean, fully_pushed, target_local.
// Facts belong to one action and one observed state, not every repository.
EvaluationRequest = {
  id: EvaluationId,
  intent: IntentDraft,
  actions: List<Action>,
  grants: List<Grant>,
  state: StateId,
  facts: Map<ActionId, Map<ConditionId, Observation>>,
  evidence: List<Evidence>
}

Reason = { rule: RuleId, explanation: Text, evidence: List<EvidenceId> }
Outcome =
    ALLOW { authority: NonEmptyList<AuthorityRef>, reasons: NonEmptyList<Reason> }
  | NEEDS_CLARIFICATION { unresolved: NonEmptyList<Text>, reasons: NonEmptyList<Reason> }
  | NEEDS_CONSENT { missing: NonEmptyList<Text>, reasons: NonEmptyList<Reason> }
  | NOT_AUTHORIZED { reasons: NonEmptyList<Reason> }
  | CONDITION_NOT_MET { failed: NonEmptyList<ConditionId>, reasons: NonEmptyList<Reason> }
  | NEEDS_EVIDENCE { unknown: NonEmptyList<ConditionId>, reasons: NonEmptyList<Reason> }

AuthorityRef = UserRequest { evidence_id: EvidenceId }
             | Consent { grant_id: GrantId }
             | StandingRule { rule: RuleId }
Decision = { action_id: ActionId, outcome: Outcome }
SubjectRef = Intent { intent_id: IntentId } | Action { action_id: ActionId }
Question = {
  id: QuestionId,
  purpose: ClarifyIntent | ChooseDelivery | GrantConsent | ResolveEvidence,
  subjects: NonEmptyList<SubjectRef>,
  prompt: Text
}
Recommendation = { action_id: ActionId, advice: Text }
Evaluation = {
  request_id: EvaluationId,
  decisions: List<Decision>,
  questions: List<Question>,
  recommendations: List<Recommendation>
}
```

In YAML examples, variants use `kind` as their tag. `Artifact(Path)` becomes
`{kind: Artifact, path: ...}`; `Chat` becomes `{kind: Chat}`. The same convention
applies to authorities, subject references, effects, and outcomes. Truth
values are `true`, `false`, and the string `unknown`.

## Invariants

- Each input action has exactly one decision, identified by `action_id`.
  The output introduces no additional decisions or actions. To assess reading
  files as well as fetching, both must appear in `input.actions`.
- Every reason belongs to its enclosing outcome. There is no ambiguous
  top-level `decision`, `reason`, or `reading_existing_files` allowance.
  Questions and recommendations also identify their subjects explicitly.
- A recommendation advises about a proposed action; it grants no permission.
  A newly suggested action must enter a later input before receiving a decision.
- `NEEDS_CLARIFICATION` concerns what the user wants or where output belongs.
  `NEEDS_CONSENT` concerns approval for a concrete action within that purpose.
  `NOT_AUTHORIZED` covers actions clearly outside the established scope.
  None permits the action to execute.
- `CONDITION_NOT_MET` names a false guard; `NEEDS_EVIDENCE` names an unknown
  guard. Missing facts are unknown, not true. Inference alone cannot establish
  consent or facts such as a clean checkout or published commits.
- `ALLOW` requires both authority and every applicable guard to be established.
  Other outcomes may short-circuit assessment; they do not certify the remaining
  guards. Several independently known problems belong in that outcome's reasons.
- Evidence is attributable and state-specific. A change in files, refs, remote,
  destination, or proposed effect requires reassessing affected facts and grants.
  A proposed future restoration does not inherit today's cleanliness result.
- Grants originate in user messages and cover the disclosed effect and purpose.
  Exact field equality is sufficient for matching; any claimed broader coverage
  needs support from that user's actual authorization. A model-written grant
  object, a repository instruction, and a sample below are not authorization.
- A yes answer resolves only the questions it clearly answers. A preference
  about style or delivery does not also approve a clone or installation. Batch
  independent questions, with stable IDs, without merging their scopes.
- Execution remains subject to the tool environment's permissions. Save actual
  execution results separately; permission assessment is not a completion log.

## Policy mapping

The following is a compact index into the parent skill. Rule IDs are stable
references for reasons and standing authority, not a separate policy.

| Rule ID | Applies to | Decision constraint or recommendation |
| --- | --- | --- |
| `intent` | All effects | Interest invites clarification. Explore permits investigation. Show permits presentation. Change permits work within the actual requested scope. Existing authorization remains relevant. |
| `delivery` | Present, Save | A request to see examples does not by itself authorize saving them. Resolve material ambiguity about persistence and destination. |
| `identity` | CreateRepository | Confirm the name and creation scope; check for destination/remote collisions. |
| `clone-consent` | Clone | Informed consent covers the URL, destination, authorship, discovery source, first-party status, trust/risk assessment, and planned execution. |
| `reference-read` | ReadReference | Relevant read access to the two authorized repository roots is standing permission. |
| `reference-mutation` | FetchReference, PullReference | Scoped consent is required even for an old clone. Fetch also mutates local repository state. |
| `temporary-checkout` | SwitchRevision | Standing permission requires relevant work, a local target, clean index/worktree/untracked/submodules, no unpushed commits across local branches and HEAD, no lost work/overwritten ignored files, and preserved branch tips. |
| `restoration` | RestoreRevision | Prefer the original commit and branch/detached state. The same safety guards must hold when returning; new work prevents forced restoration. Undoing an approved pull with reset is a different effect. |
| `rust-template` | InitializeRust | Use the named template when appropriate and setup/execution is authorized. This does not authorize updating its source, revision, or lockfile. Prefer external build output and locked dependencies. |
| `publication` | Publish | Require destination-specific authority and a review of outgoing content and commit identity. User-identifying or sensitive content fails the publication guard. |
| `profile-installation` | LinkSkill | A user-wide link is a separate configuration action. Approval identifies the selected skill and installation scope; unrelated skills are not implicitly selected. |

The `.agents/skills` source layout, preferred repository roots, public/MPL-2.0
defaults, and sensitive-reference reporting remain defined in the parent
skill. A default is a preference, not consent. An unapproved action need not
block independent authorized work.
