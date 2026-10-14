# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/196

**Branch**

`fix/54-section-detection-leading-whitespace`

## Eval iterations

**Run history**

The initial restricted-environment attempt could not invoke Claude successfully and was interrupted. It produced no completed agreement score and no submission transcript. After confirming the CLI could run with access to its local setup, I completed one full run of all 20 scored packages, with no rubric revisions or partial reruns afterward:

```text
categories: clear-accept 6/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3
agreement: 19/20 scored items  (bar: 18/20: PASS)
```

This is the final score in the unchanged harness-generated `eval-run.txt` committed alongside this write-up. All four graded skill files match that transcript’s fingerprints. Live draft checks are separate from the evaluation score. The Claude CLI live attempt returned the account spending-limit message and produced no verdict. Codex subsequently applied the same installed rubric/procedure to the committed draft and returned accept; this is recorded as a Codex review, not a Claude run.

**Package analysis**

I selected **pkg-03**, a `silent-drift` package. My rubric returned **reject** and the gold label is **reject**. The recorded row is:

```text
pkg-03  silent-drift  reject  reject   yes
```

The accepted plan explicitly excludes “resolving symlinks to make repo-root styling apply through them, or any config surface change.” The candidate description claims “Implements the posted plan exactly” and “No other behavior changes.” However, the diff also rewrites the truncation helper and adds a `resolve_symlinks` config field and default, accompanied by separate refactor and feature commits. Those changes are outside the accepted boundary and have no disclosed deviation. The literal condition in my plan-fidelity check rejects silent additions; the description-fidelity check also compares those exact claims with the diff. Passing before/after evidence for the requested fallback does not justify unrelated config or refactor work.

**Check rationale**

Current row quoted exactly from `tools/pr-precheck/rubric.md`:

> | Diff follows the plan or disclosed deviations | Plan scope, boundaries, approach, and deviations beside all diff hunks and description scope statements. | Each change serves the issue and is accounted for by the plan or an explicit explanation of what changed and why. Planned work omitted or narrowed is disclosed rather than silently claimed complete. Honest, explained partial delivery passes; silent additions or omissions fail. | required |

I chose this comparison between plan, deviations, and actual hunks instead of a check that merely asks whether the PR says it follows the plan. A claim of fidelity is not proof. The pass condition deliberately permits explained partial delivery because an honest shortfall can be reviewable; it holds silent omissions and additions so a maintainer does not have to discover the mismatch independently. I did not revise this check after the completed full run.

**Trade-offs**

Treating required unclear grades as rejection favors verifiable evidence over optimistic interpretation. The completed run caught all four failure families, but falsely rejected **pkg-08**, whose gold label is accept. The harness named “Description matches the delivered change” and “Repository checks are accounted for” as its failed checks. The package actually supplies a specific reproduction, tracked-file control, passing integration test, `go test ./...`, and `go generate ./...` with no cheatsheet diff. That false rejection shows room for stricter-than-intended execution of my wording. I retained the evaluated files because the full run reaches 19/20 with every category matched; I accept this known precision cost rather than claiming perfect agreement. No loose revision or canary rerun occurred.

## Supporting Unit 4 artifacts

- `plan.md`: the original local Unit 3 plan used as the basis for this PR, with Deviations updated during Unit 4 to record completion of the missing comparison test. The original production scope is unchanged.
- `test_evidence.md`: complete before/after output and check results, including the expected empty integration suite.
- `pr_draft.md`: title and description prepared for opening the PR.
- `live-precheck.txt`: Codex’s per-check review using the installed rubric/procedure, ending with the fixed accept/reject JSON.
- `live-precheck-claude-attempt.txt`: the exact spending-limit message from the unsuccessful required Claude live attempt.

The existing Unit 3 submission files remain unchanged. These supporting copies let a reviewer inspect the exact plan and evidence used for this Unit 4 PR.
