# Procedure: how this tool grades a PR package

## Read order

1. Apply the frame’s mode and refusal rules. Read rubric and evidence guide before grading.
2. Read the issue and original repro first; record reported behavior and observed failure.
3. Read repo requirements and thread directions; list only applicable asks.
4. Read the plan before the diff; list promised changes, boundaries, tests, and recorded deviations.
5. Read every diff hunk and the commit list, then the test evidence, then title and description. Complete these reads before grading.

## Evidence gathering

1. For plan fidelity, map each hunk to the plan or a deviation. Map every promised change back to delivered work or an explicit omission. Compare description claims to both.
2. For verification, pair each repro or planned test with its command/input, expected result, observed result, and any disclosed limitation. Separately map each required repository check to output or a concrete explanation for absence.
3. For diff quality, locate unrelated hunks, debug lines, dead code, accidental files, or churn. Record file and hunk plus whether the issue requires it.
4. For standards, map each template or policy ask and thread direction to the description’s own text. For live mode include course AI disclosure and gather voice notes separately.
5. Record supporting quotations or specific file/output locations. Do not infer a run from a checkbox or infer policy from another repository.

## Check execution

1. Execute every rubric row in table order against its gathered evidence, independently.
2. Apply the pass condition literally, including honest-shortfall allowances. A disclosed failure is evidence, not a success; verify that claims acknowledge it.
3. If evidence is absent grade fail and identify where it was sought. If present but ambiguous grade unclear and name the conflicting readings. Revisit only the relevant source when gathering was insufficient.
4. Do not run code, fetch extra facts, fix artifacts, or invent new requirements during eval. Live grading inspects supplied outputs rather than silently creating missing proof.

## Verdict assembly

1. Report all grades and one deciding fact per check. Apply the rubric: any required fail or unclear gives reject; otherwise accept.
2. Highlight the first required fail or unclear in table order as the deciding check. For accept identify the weakest evidence among required checks and any preferred concern.
3. Include voice notes only in live mode and report procedure gaps without patching them in the run.
4. Emit the frame’s unchanged JSON schema with all checks in rubric order, preserving unclear grades, followed by nothing.
