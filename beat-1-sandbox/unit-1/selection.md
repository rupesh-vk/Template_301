# Unit 1 — Issue Selection

GitHub username: `rupesh-vk`

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/54

Resume section detection fails on text with leading whitespace.

**Verdict output**

The following is the installed skill's standalone live-mode output for the selected issue, pasted verbatim. Two prose details need correction: it says four required checks, but the rubric and JSON contain five; its fit note mentions reproducing tonight, while tonight's task is selection and reproduction belongs to Unit 2. Neither changes the per-check grades or accept verdict. Before selecting it, the same rubric graded three candidates: #54 and #68 were accepted, and #60 was rejected because an implementation PR was open. The multi-candidate run ranked #54 first.

All evidence gathered. Now applying the rubric.

---

**Check-by-check summary:**

**1. Human maintenance — PASS**
Most recent commit by `Aburke225` (human, not a bot) on 2026-09-16 — 7 days ago, well within 90 days. No owner/member comment in the issue thread, but recent human-authored default-branch commits satisfy the OR condition.

**2. Repository continuity — PASS**
`isArchived: false`; last push 2026-09-16 (7 days ago), comfortably within the 90-day threshold. No release required when push recency passes.

**3. Bounded contribution — PASS**
One specific bug in one method (`_detect_sections()` in `resume_parser.py`): section-header regex anchored to `^` fails on PDF text with leading whitespace. Labels: `bug`, `good first issue`, `tier-1`. No umbrella sub-items, no design dispute, no abandoned PRs.

**4. Available work — PASS**
Issue is OPEN, zero assignees, zero comments (no claims at all), no cross-referenced PRs in timeline. Classroom house rule (classmate claims don't block) is moot — there are none.

**5. Compatible contribution policy — PASS**
`docs/CONTRIBUTING.md` is silent on AI-assisted contributions; no AI policy file found. Silence passes per rubric.

**6. Easy verification (preferred) — PASS**
Body contains a runnable Python snippet with `observed: []` / `expected: Education, Skills`, names the exact method (`_detect_sections()`), file (`resume_parser.py`), and three specific test functions in `tests/unit/test_resume_parser.py`.

---

**Verdict:** All four required checks pass → **accept**.

**Fit note:** Excellent match for the student profile. Small Python bug in a single well-named method, three explicitly named failing tests, a concrete input-output pair, and `tier-1` difficulty — exactly the bounded issue the student described needing to reproduce tonight before Unit 2.

```json
{
  "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/54",
  "checks": [
    {
      "name": "Human maintenance",
      "grade": "pass",
      "evidence": "Aburke225 (human) committed to default branch on 2026-09-16, 7 days before today — within 90-day threshold."
    },
    {
      "name": "Repository continuity",
      "grade": "pass",
      "evidence": "Not archived; last push 2026-09-16 (7 days ago), within 90-day push threshold."
    },
    {
      "name": "Bounded contribution",
      "grade": "pass",
      "evidence": "Single bug: _detect_sections() regex fails on indented PDF text; labels bug + good-first-issue + tier-1; no sub-items, no design dispute, no abandoned PRs."
    },
    {
      "name": "Available work",
      "grade": "pass",
      "evidence": "Issue OPEN, assignees=[], comments=[], no cross-referenced PRs in timeline."
    },
    {
      "name": "Compatible contribution policy",
      "grade": "pass",
      "evidence": "docs/CONTRIBUTING.md is silent on AI-assisted contributions; no AI policy or AGENTS.md file found."
    },
    {
      "name": "Easy verification",
      "grade": "pass",
      "evidence": "Body supplies runnable Python snippet (observed: [], expected: Education, Skills), names _detect_sections() in resume_parser.py, and three specific xfail test functions."
    }
  ],
  "verdict": "accept"
}
```

---

## Eval iterations

**Run history**

Before evaluation, I expanded the three practice checks into a rubric covering human maintenance, repository continuity, bounded scope, availability, and contribution policy. I made easy verification preferred rather than required so missing reproduction details would not automatically reject a bounded issue.

- Setup attempts: an empty-template preflight stopped before grading, and the first attempted full run could not complete because the Claude login had expired. Neither produced a complete scored result or a valid saved run.
- Completed run 1, after refreshing authentication: `agreement: 18/20 scored items  (bar: 18/20: PASS)`

The complete run matched every category: `claimed 4/4`, `clear-accept 6/8`, `dead-repo 3/3`, `policy 1/1`, and `scope 4/4`. I read both disagreements, `issue-09` and `issue-19`. I kept the rubric unchanged after that run, so this first completed run is also the final confirming run saved in `eval-run.txt`. There were no scored partial reruns. The separate live-mode candidate checks do not have gold-label agreement scores.

**Issue analysis**

For `issue-09` (conda/conda#7617), my rubric returned **reject**, while the gold label was **accept**. The evaluation row reads:

```text
issue-09  accept  reject   NO     failed: Available work
```

The snapshot has no assignee and lists only a closed linked PR. However, MesaJonathan wrote on January 20, 2022: “I'd like to take a swing at this as my first open-source contribution. Does it need to be assigned to me?” The grader treated that as an unwithdrawn work claim, which fails my availability rule even though the claim is more than four years old at the snapshot date. The closed PR alone was not the rejection reason.

My rule has no expiration for a claim, so it can mistake abandoned work for current ownership. The gold acceptance is consistent with treating the old claim and closed PR as inactive, although the label itself does not explain the staff's reasoning. The other required checks passed, so this disagreement isolates the availability check.

For the other disagreement, `issue-19`, my rubric returned **reject** and the gold label was **accept**. The grader interpreted the proposed matcher, threading, and multiprocessing changes as several workstreams. Its required `Bounded contribution` failure caused rejection; the failed preferred `Easy verification` check did not affect the verdict. This shows that the rubric can be too conservative when one bug report includes several possible approaches.

**Check rationale**

The complete `Available work` row below is quoted exactly from the uploaded `tools/issue-select/rubric.md`:

> | Available work | Issue state, assignees, linked PR states, and the entire comment thread including informal PR references and work claims. | Issue is open, with no assignee, no open implementation PR, and no unwithdrawn explicit work claim or reported implementation PR in the thread. Empty sidebar fields do not override a claim in comments. A closed unmerged PR alone is not an active claim. Apply the live scope's classroom house rule to classmates' claim comments; eval snapshots have no classroom exception. | required |

The comment-thread requirement matters because formal assignment and PR fields can miss an existing contributor. In the bat calibration bundle, the empty sidebar fields did not reveal the contributor's claim and PR reference in the comments. I kept availability required because a small, well-described issue can still duplicate someone else's work. For live Path Review issues, the course's house rule means classmates' claim comments do not block my selection; that exception does not apply to the public-repository eval snapshots.

**Trade-offs**

The phrase “no unwithdrawn explicit work claim” is deliberately conservative, but it has no freshness threshold. The actual cost is visible in `issue-09`: the saved run says `failed: Available work`, even though the claim is from 2022 and the linked PR is closed. I accept that this version can reject an issue whose earlier contributor has moved on. Adding an expiration threshold could recover such issues, but would also require a decision about how old a claim must be before treating it as inactive.

I did not change the check after the completed run. The final uploaded rubric matches the fingerprint in that saved run, so no untested wording change is being presented as if it had earned 18/20. I also recognize the scope false rejection in `issue-19`; passing the course's target does not mean every verdict is correct.

---

## Selection rationale

**Selection rationale**

1. **Fit and available time:** I wanted a small Python bug with existing tests. Issue #54 gives a specific input condition—leading whitespace before résumé headings—and names the parser function and related tests. That makes it a manageable starting point to carry into Unit 2 while finishing this selection assignment tonight.
2. **What the verdict caught, and what I weighed:** The verdict correctly identified recent human maintenance, a bounded issue, compatible contribution rules, and an available issue with a concrete example. I also weighed my preference for a focused Python task and the value of comparing indented and unindented text. A passing verdict does not guarantee how long my environment setup or debugging will take; I have not reproduced the issue yet.
3. **Anticipated claiming difficulty:** At the live check, the issue had no assignee, comments, or linked implementation PR found by the skill. I therefore expect little claim coordination, but I will recheck its status before posting in Unit 2. The course's house rule allows classmates to share an issue, so another student's claim comment alone would not prevent mine. I have selected the issue, not claimed it, and will write and check my own claim before posting.

---

Preparation note: AI assistance helped draft the rubric and organize this write-up. The eval transcript was generated by the supplied course harness, and the live verdict above was generated by the installed skill. No claim or reproduction comment has been posted as part of Assignment 1.
