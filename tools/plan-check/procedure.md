# Procedure: how this skill grades a plan package

## Read order

1. Read the **issue** first — title, body, and the behaviour it claims. Note the behaviour in one sentence; this is what every later check is measured against.
2. Read the **repro evidence** second, before the plan. Note three things: what environment it ran in, what it actually observed (the values, the output, the failing assertion), and what it explicitly says it did *not* establish. Read this before the plan so the plan's claims are judged against the evidence rather than the evidence being read through the plan's framing — reading them the other way round makes a confident wrong diagnosis look supported.
3. Read the **repo facts** third: contribution policy, stated conventions, any AI-use disclosure requirement, and any template the comment must follow. Note each requirement that applies to a comment, as a list.
4. Read the **thread highlights** fourth. Note any direct question asked of the contributor, and any preference a maintainer has stated about approach.
5. Read the **plan** fifth, straight through, without grading. Note its stated cause, its scope statement, its file list, its approach, its test plan, and its risks.
6. Read the **plan comment** last, as a maintainer would see it — on its own, with nothing else open.

Do not begin grading until all six are read. A check graded mid-read gets regraded once the rest is in, which is how two runs on the same package disagree.

## Evidence gathering

For each check, pull from the location named, and record the quoted text rather than an impression of it.

1. **Diagnosis follows from the evidence** — pull the plan's stated cause (the sentence naming what produces the wrong behaviour), and pull every observation from the repro evidence. Record the cause, and record which observation the plan cites for it. If it cites none, record that.
2. **Change is bounded** — pull the scope statement (what will change, what will not) and the file list. Record each item in the change, and for each, whether the plan ties it to the stated cause.
3. **Fix targets the cause, not the symptom** — pull the approach. Record where in the code the change acts, and compare that to where the repro evidence locates the wrong behaviour. Record both locations.
4. **A stranger could start executing it** — pull the approach and file list again, this time reading for the first action. Record what the first concrete change would be and which file it is in, or record that it cannot be determined.
5. **Test plan proves something observable** — pull the test plan and the repro evidence's steps side by side. Record what the test plan re-runs, and record the stated post-fix observation verbatim.
6. **Unknowns are stated as unknowns** — pull the risks section, and pull any assumption load-bearing in the approach. Record each assumption and whether it appears in the risks.
7. **Comment fits the thread and the repo's conventions** — pull the comment, the requirement list from the repo facts, and any direct question from the thread. Record each requirement and whether the comment's own text meets it.

Record only what the package contains. Do not supply a fact from general knowledge of the project; if the package does not state it, it is absent.

## Check execution

1. Grade the checks in the table order above: diagnosis, scope, cause-not-symptom, executable, test plan, unknowns, conventions. The order matters because the first three establish what the plan claims to be doing, and the later checks are graded against that claim.
2. Grade each check only against the evidence recorded in the gathering stage. Do not re-read the whole package to settle a check; if the recorded evidence is insufficient, return to the one location that check names and record what is there.
3. Apply the pass condition as written. Where it names a specific failing shape ("a change that suppresses the symptom", "a test plan that asserts the fix works without naming what is observed"), look for that shape before forming a general impression.
4. When the evidence for a check is genuinely absent from the package — not ambiguous, but not there — grade it **fail**, not unclear, and record what was looked for and where. Absence is a finding about the plan.
5. Grade **unclear** only when the evidence is present but admits two readings that lead to different grades. Record both readings.
6. A check never borrows a grade from another check. A plan with an excellent diagnosis and no test plan fails the test plan check.

## Verdict assembly

1. Collect the grade for every required check.
2. Convert each `unclear` on a required check to `fail`, per the rubric's verdict rule, and keep a record that it was converted.
3. If every required check is now `pass`, the verdict is **ready** (accept). Otherwise the verdict is **hold** (reject).
4. Preferred checks are reported but never entered into step 3.
5. Identify the deciding check: on a hold, the first required check in table order that failed; on a ready, the required check whose recorded evidence was thinnest.
6. Quote the deciding check's recorded evidence in the output — the plan's own words, not a paraphrase — so the author can see what was read.
7. Emit the per-check summary followed by the JSON block with the verdict.

The same recorded grades must produce the same verdict every time. Nothing in this stage reconsiders a check.