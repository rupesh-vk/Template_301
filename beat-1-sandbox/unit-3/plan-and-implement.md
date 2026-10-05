# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

---

## Posted upstream

**GitHub username**

rupesh-vk

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/54#issuecomment-5988460579

Following up on my reproduction above with a plan, before I start on it.

**Cause.** From reading the matching code rather than instrumenting it: `_detect_sections()` builds four regexes per section name, and all four require the name to begin immediately at a line boundary.

    patterns = [
        rf"^{re.escape(section)}\s*$",
        rf"^{re.escape(section)}\s*[:|-]",
        rf"\n{re.escape(section)}\s*$",
        rf"\n{re.escape(section)}\s*[:|-]",
    ]

None of them tolerates horizontal whitespace between the boundary and the name, so on a line reading `    Education:` the next character is a space, nothing matches, and the section never lands in `detected`. That accounts for the contrast in my repro, where the text and the section names were identical and only the indentation differed.

**The same assumption appears a second time.** `_strip_markdown()` strips headings with `re.sub(r"^#+\s+", "", content, flags=re.MULTILINE)`, which needs `#` immediately at the line start for the same reason. I checked what that means in practice: applying the `_detect_sections()` change alone to a scratch copy of HEAD and running the file gives `2 failed, 8 passed` — `test_parse_markdown_resume` and `test_strip_markdown_syntax` still fail, because an indented `    # Education` is never stripped and `_detect_sections()` is then handed a line beginning with `#`. So both patterns are in scope; fixing one without the other leaves the markdown path broken.

**Change.** Insert `[ \t]*` after the leading anchor in the four `_detect_sections()` patterns and in `_strip_markdown()`'s heading pattern. `[ \t]*` rather than `\s*` on purpose — `\s` matches `\n`, which would let a match skip across blank lines and start on a line the anchor never applied to.

The five tests marked `xfail(strict=True)` for this issue come off in the same change, since under `strict=True` a test that starts passing is reported as `XPASS` and fails the suite. I'll add one case asserting indented and unindented text produce the same section list.

Nothing else: no changes to `SECTION_HEADERS`, to the rest of `_strip_markdown()`'s substitutions, or to the `^`/`\n` pattern redundancy, which is real but doesn't serve this bug.

**Verification.** Re-running the two steps from my repro comment at the fix commit. `test_detect_sections` should pass without `--runxfail`; the indented and flush calls should both return `['Skills', 'Experience', 'Education']` in some order, since the function returns `list(set(...))`; and the file should go from `5 passed, 5 xfailed` to all tests passing.

**What I'm still unsure of.** I haven't isolated `test_parse_single_column_resume_text` or `test_parse_resume_no_work_experience` individually, so if either fails for a third reason, removing its marker turns a known failure red. Separately, loosening the anchor widens the match surface: an indented line like `    Skills: Python, Go` nested under Experience will now register as a section, which this function has no way to distinguish from a real heading. That's the cost of the fix rather than a side effect, and I'd rather name it than have it turn up in review.

---

## Your branch

**Branch**

fix/54-section-detection-leading-whitespace

**Evidence**

My Unit 2 reproduction steps, re-run against the built change. Before is at `f89c06f` (what I posted on the issue); after is at commit `d006d21` on the branch above.

**Step 1 — the xfail-marked test.**

Before:

    $ python3 -m pytest tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections --runxfail -v

    tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections FAILED

            sections = parser._detect_sections(text)

            assert isinstance(sections, list)
    >       assert len(sections) > 0
    E       assert 0 > 0
    E        +  where 0 = len([])

    tests/unit/test_resume_parser.py:152: AssertionError
    =========================== short test summary info ============================
    FAILED tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections - assert 0 > 0

After (no `--runxfail` needed, the marker is gone):

    $ python3 -m pytest tests/unit/test_resume_parser.py -v

    collected 10 items

    tests/unit/test_resume_parser.py::TestResumeParser::test_parse_single_column_resume_text PASSED [ 10%]
    tests/unit/test_resume_parser.py::TestResumeParser::test_parse_resume_no_work_experience PASSED [ 20%]
    tests/unit/test_resume_parser.py::TestResumeParser::test_parse_multipage_pdf PASSED       [ 30%]
    tests/unit/test_resume_parser.py::TestResumeParser::test_parse_markdown_resume PASSED     [ 40%]
    tests/unit/test_resume_parser.py::TestResumeParser::test_parse_invalid_content_type PASSED [ 50%]
    tests/unit/test_resume_parser.py::TestResumeParser::test_parse_invalid_list_content PASSED [ 60%]
    tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections PASSED           [ 70%]
    tests/unit/test_resume_parser.py::TestResumeParser::test_strip_markdown_syntax PASSED     [ 80%]
    tests/unit/test_resume_parser.py::TestResumeParser::test_pdf_parsing_error_handling PASSED [ 90%]
    tests/unit/test_resume_parser.py::TestResumeParser::test_parse_preserves_text_content PASSED [100%]

    ================================ 10 passed in 0.29s =================================

The suite went from `5 passed, 5 xfailed` to `10 passed`, with no `XPASS`.

**Step 2 — the indented-versus-flush contrast.** Same script both times:

    python3 - <<'EOF'
    from ingestion.parsers.resume_parser import ResumeParser

    p = ResumeParser()

    indented = """
        Experience:
        Senior Developer at TechCorp

        Education:
        BS Computer Science

        Skills: Python, JavaScript
    """

    flush = "\n".join(line.strip() for line in indented.splitlines())

    print("indented ->", p._detect_sections(indented))
    print("flush    ->", p._detect_sections(flush))
    EOF

Before:

    indented -> []
    flush    -> ['Skills', 'Experience', 'Education']

After:

    indented -> ['Skills', 'Experience', 'Education']
    flush    -> ['Skills', 'Experience', 'Education']

The two inputs now agree, which is the behaviour the issue asked for. Order is not stable because `_detect_sections()` returns `list(set(detected))`.

**Diff shape**, for what it is worth: `2 files changed, 5 insertions(+), 20 deletions(-)` — five one-token regex edits in `ingestion/parsers/resume_parser.py`, and five three-line `xfail` decorators removed from `tests/unit/test_resume_parser.py`.

## Eval iterations

**Run history**

Two runs, in order:

1. Smoke run, `--limit 3`: `agreement: 3/3 scored items`, categories `clear-accept 2/2  wrong-cause 1/1`. Partial, so no bar verdict and no run file. I spent it to confirm the harness was loading all three of my files — the header line `grading 3 package(s) with rubric.md + evidence-guide.md + procedure.md` is what I was checking for, since `procedure.md` is new this week and the skill executes it rather than a workflow in SKILL.md.
2. Full run, 20 scored packages: `agreement: 18/20 scored items  (bar: 18/20: PASS)`, with `categories: clear-accept 5/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4`.

I made no revisions after the full run, so there were no `--only` re-runs and no second full run. The 18/20 above is the agreement line in the committed `eval-run.txt`, and the files fingerprinted in that run's header are the ones uploaded to `tools/plan-check/`.

**Package analysis**

`pkg-05` (source: conda/conda#16502, category `clear-accept`). My rubric returned **reject**; the gold label is **accept**. The run's row reads:

    pkg-05  clear-accept  accept  reject   NO     failed: Unknowns are stated as unknowns

One check, and only that one. The diagnosis cited the repro's own finding, the scope named both an in-scope change and three explicit not-in-scope items, the files were named, and the test plan re-ran the repro's step 4 with a stated post-fix observation (the access log showing a second request).

My rubric read it that way because of one clause in the unknowns check: *"A plan whose approach rests on an assumption that appears nowhere in its risks fails."* The approach says it will "compute the cached response's age from the cache file's stored timestamp". That the cache file stores a timestamp the fix can read is load-bearing — if it does not, the whole approach changes — and the plan never confirms it, never marks it unconfirmed, and never lists it under risks. The risks line reads in full: *"Risk: none identified beyond one extra network request per channel per 24h, which is the constant's existing meaning."* That names a cost, not an unknown, so under my pass condition the plan asserted certainty it had not earned.

The gold label is the better call, and I can say why my clause missed it. Two maintainers had already settled the direction on the thread — travishathaway naming the constant, danyeaw confirming it needed wiring into `get_notice_response_from_cache()` — so the plan's confidence is borrowed from the thread rather than invented, and the unstated assumption is one a maintainer reading the thread would not question. My clause has no notion of an assumption whose answer is already established in the room; it treats every unconfirmed fact the same way regardless of how settled it is.

`pkg-14`, also `clear-accept`, failed this check too, alongside three others. That `pkg-05` failed on this one alone makes it the cleaner case: it isolates the clause rather than mixing it with other disagreements.

**Check rationale**

Quoted from the `Unknowns are stated as unknowns` row of the `rubric.md` uploaded to `tools/plan-check/`, exactly as it reads now:

> | Unknowns are stated as unknowns | The plan's risks and unknowns, read against the confidence of its other sections. | Anything the author has not confirmed is marked as unconfirmed where it is used, and the plan names at least one thing that could turn out otherwise, or states plainly that it found none and why. A plan whose approach rests on an assumption that appears nowhere in its risks fails. A risks section that lists only generic hazards unconnected to this change fails. | required |

It reads that way because of what I rejected. My first draft asked whether the plan "has a risks section" and whether that section "is substantive" — and I threw it out, because that grades the write-up's shape and its tone, and two graders applying "substantive" disagree with each other while agreeing about the plan. Shape-shaped checks are the thing the rubric template warns about twice.

So the condition is written as a relation between two parts of the same document: the confidence of the approach, and what the risks section admits. That is checkable by a second grader, because it reduces to a mechanical question — take each thing the approach relies on, and look for it in the risks.

The second sentence exists because of the opposite failure. A plan can carry a risks section that satisfies any presence test while stating nothing: generic hazards like "this could introduce a regression" read identically under every plan for every issue. Naming that shape explicitly is what lets the check catch false confidence that is fluent rather than absent, which is the form it usually takes.

**Trade-offs**

The unknowns check gives up plans whose unstated assumption is one the thread has already settled, and I can name the two it cost: `pkg-05` and `pkg-14`, both `clear-accept`, both gold `accept`, both `failed: Unknowns are stated as unknowns`. In `pkg-05` the unstated assumption — that the cache file carries a readable timestamp — sits under an approach that two maintainers had already endorsed in the thread, and my clause has no way to see that endorsement.

I did not change the check after the run, and the reason is what the loosening would cost elsewhere. The obvious revision is to excuse an assumption the thread has already settled. But `thread-convention` is a 2-package category and it matched 2/2 precisely because my rubric reads the thread as something a plan must answer to; a clause that lets the thread *excuse* an unstated assumption pushes in the opposite
