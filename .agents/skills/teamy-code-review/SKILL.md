---
name: teamy-code-review
description: Review Teamy's code changes, pull requests, or staged and unstaged diffs against maintained TeamyStyle guidance and relevant Cloud Terrastodon design patterns. Use during code review, including architectural and maintainability review; keep findings grounded in the requested change.
---

# Teamy Code Review

## Load the design references

Read `G:/Programming/Repos/teamy-docs/src/teamystyle.md` and the linked chapters relevant to the change. This is the maintained style guide, not a checklist to copy into this skill. Preserve its distinctions between established requirements, recommendations and proposals.

Use `D:/Repos/Azure/Cloud-Terrastodon` as design inspiration. Read its `AGENTS.md` and inspect relevant current source examples for CLI/request/operation separation, semantic domain types, file organization, output, cancellation or resource ownership. Cite the specific example used. Historical code is not automatically compliant with current conventions. Borrow applicable patterns, not Azure-specific dependencies or an entire architecture.

On another machine, locate these references within known repository roots or use the published [TeamyStyle](https://teamdman.github.io/teamy-docs/teamystyle.html) and [Cloud Terrastodon source](https://github.com/AAFC-Cloud/Cloud-Terrastodon). Do not create a checkout just to run this review without authorization. State unavailable references and identify which version/source informed the review.

Read the target repository's own instructions and accepted design decisions. If they conflict with these references, expose the conflict rather than silently rewriting project policy or treating a reference project's local convention as universal.

## Review the actual change

Establish the requested comparison: commit range, pull request, staged diff or unstaged diff. Keep HEAD, index and worktree changes distinct; include relevant untracked files only when they belong to the requested review. Preserve existing work.

Check behavior and invariants first, then assess the design against TeamyStyle. Look for meaningful type ownership, enums instead of interchangeable strings for closed states, validated boundaries, discoverable type files, separation of CLI syntax from domain operations, and evidence for claimed output, cancellation and recovery behavior. Follow the maintained guide's qualifications; do not invent requirements or force unrelated patterns onto the project.

Use Cloud Terrastodon examples to make a concrete alternative understandable. Explain why the pattern fits this domain. Do not recommend broad rewrites merely to resemble the reference repository.

## Report actionable findings

Lead with confirmed findings ordered by impact. Give the affected file/lines, the reachable failure or specific design mismatch, supporting evidence or guidance section, and a focused correction. Separate correctness defects, documented style violations, recommendations and open questions. State what was inspected or tested and what remains unverified; no findings does not establish safety.

Keep the requested human-review boundary: reviewing does not authorize editing, staging, committing, merging or publishing. Offer reviewable changes when useful, but do not apply them unless requested.
