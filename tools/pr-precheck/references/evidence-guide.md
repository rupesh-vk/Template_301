# Evidence guide: where evidence lives in a PR package

## Plan fidelity (harness category: silent-drift)

Eval: plan-context scope, approach, boundaries, and deviation notes beside candidate PR Diff and Description. Live: plan.md including Deviations beside git diff main...HEAD and pr_draft.md. Map both directions: every hunk has a planned purpose or explained deviation, and every promised outcome is delivered or explicitly left out. Good evidence explains changes and limitations without claiming more than the diff implements.

## Test evidence (harness category: not-tested)

Eval: original repro in Plan context, its test plan, Candidate PR Test evidence, Description, and required checks in Repo facts. Live: plan.md and original repro/house pack beside test_evidence.md and Testing in pr_draft.md; use docs/CONTRIBUTING.md and the PR template for required checks. Good evidence names commands or inputs and observable results on the reported path, including before/after or regression assertions. Each required repo check has an outcome or a concrete disclosed reason it could not run. A failure reported accurately remains evidence; a checkbox or “tests pass” alone proves no behavior.

## Diff quality (harness category: unreviewable)

Eval: Candidate PR Diff and Commits. Live: full committed branch diff and commit list. Read every hunk, not only the file list. Good changes serve the reported issue with necessary supporting edits; debug leftovers, dead code, commented-out blocks, unrelated cleanup, generated debris, and gratuitous formatting churn obscure review even when acknowledged.

## Standards and comms (harness category: standards-wall)

Eval: Repo facts template asks and contribution/AI policy plus Thread highlights beside Candidate PR Title and Description. Live: .github/PULL_REQUEST_TEMPLATE.md, docs/CONTRIBUTING.md, issue thread, and course AI disclosure requirement beside pr_draft.md; use voice-guide.md only for live voice notes. Good text supplies meaningful answers to every applicable section, issue linkage, requested disclosures, and maintainer directions. Justified N/A is acceptable. No template or policy in the bundle means no invented requirement. Whether claims match the diff belongs to plan fidelity.
