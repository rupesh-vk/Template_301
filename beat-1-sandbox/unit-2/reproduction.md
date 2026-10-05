# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`.

---

## Your identity upstream

**GitHub username**

rupesh-vk

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/54#issuecomment-5987374021

I'd like to work on this one. I'm new to this project, so to set expectations up front: I'm claiming the investigation, not a fix.

What I'm going to do next is set the project up from the repo's own docs, then run the snippet in the issue body against `_detect_sections()` in `resume_parser.py` and compare the indented input to the same text unindented — since per the report that whitespace is the only difference between the failing and passing cases. I'll also run the three tests the issue names in `tests/unit/test_resume_parser.py`.

I'll report back with what I find either way, including if I can't reproduce it, with the environment I ran in and the output I got.

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/54#issuecomment-5987632306

Reproduced on the current `main`. Details below so this can be re-run.

**Environment**

- macOS 26.5.2 (arm64)
- Python 3.11.4
- `codepath/pathreview-ai301-fa26-s1` at commit `f89c06f`
- pytest 8.4.1

**Steps**

1. Clone the repo and check out `f89c06f`.

2. The five tests covering this path are already marked `xfail` against this issue, so run one with `--runxfail` to see the actual assertion rather than a skipped mark:

        python3 -m pytest tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections --runxfail -v

3. To isolate the whitespace as the variable, run the same text through `_detect_sections()` twice — once as written, once with each line stripped:

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

**Observed**

Step 2 fails on the length assertion, with the detected-section list empty:

        tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections FAILED

                sections = parser._detect_sections(text)

                assert isinstance(sections, list)
        >       assert len(sections) > 0
        E       assert 0 > 0
        E        +  where 0 = len([])

        tests/unit/test_resume_parser.py:152: AssertionError
        =========================== short test summary info ============================
        FAILED tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections - assert 0 > 0

Step 3 shows the same input succeeding once the indentation is removed, and nothing else changed:

        indented -> []
        flush    -> ['Skills', 'Experience', 'Education']

**What this does and does not establish**

It establishes that `_detect_sections()` returns an empty list for text whose section headings carry leading whitespace, and returns `['Skills', 'Experience', 'Education']` for the same text with that whitespace stripped. That matches the behaviour described in the issue, and leading whitespace is the only difference between the two runs above.

It does not establish the cause. My current guess is the `^` anchor in the section-header matching inside `_detect_sections()`, since that would explain why a heading stops matching once it is no longer at the start of the line — but I have not confirmed that by reading the matching code, and I am not claiming it as a finding.

Four further tests in the same file (`test_parse_single_column_resume_text`, `test_parse_resume_no_work_experience`, `test_parse_markdown_resume`, `test_strip_markdown_syntax`) carry the same `xfail` marker for this issue; I ran only `test_detect_sections` with `--runxfail`, so I have not verified the other four fail for this same reason.

## Eval iterations

**Run history**

Two runs, in order:

1. Smoke run, `--limit 3`: `agreement: 3/3 scored items`. Partial, so it printed no bar verdict and wrote no run file. I spent it only to confirm the harness could load my rubric and evidence guide from the installed copy at `~/.claude/skills/repro-check/` before committing a full run's credit.
2. Full run, 20 scored packages: `agreement: 18/20 scored items  (bar: 18/20: PASS)`, with `categories: clear-accept 6/8  disclosure 1/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4`.

I made no revisions after the full run, so there were no `--only` re-runs and no second full run. The 18/20 above is the agreement line in the committed `eval-run.txt`, and the rubric and evidence guide fingerprinted in that run's header are the files uploaded to `tools/repro-check/`.

**Package analysis**

`pkg-05` (source: conda/conda#16543). My rubric returned **reject**; the gold label is **accept**. The run's row reads:

        pkg-05  accept  reject   NO     failed: Steps re-runnable by a stranger

Only that one check failed. The environment record was complete (conda 26.7.0, Python 3.12.7, macOS 15.5 osx-arm64, libmamba solver), the artifact showed the issue's own behaviour rather than an adjacent one, the words claimed no more than the artifact showed, and conda's `CONTRIBUTING.md` "Generative AI" section states no disclosure requirement the comments fail to meet.

My rubric read it that way because of one clause in the steps check: *"every input the run consumes is supplied or linked"*, with the row's closing sentence making a step *"that consumes a fixture the package never supplies"* an explicit fail. The report's first step is "wrote a minimal `env.yml` containing a valid `dependencies:` list plus a `category:` section". That file is consumed by every command that follows, and its contents are described but never shown. Under my clause, described is not supplied, so the check failed and the required-fail rule carried the verdict to reject.

The gold label is the better call here, and I can say why my clause missed it. The description is unambiguous enough that a reader could reconstruct a sufficient `env.yml` from it, and more importantly the file's exact contents are immaterial to this bug: the issue is which stream `EnvironmentSectionNotValid` is written to, not what the yaml declares. The decisive artifact is self-contained — piping stdout into `python3 -m json.tool` returns `Expecting value: line 1 column 1 (char 1)` — and that output does not depend on the dependency list at all. My clause is written as though every unshown input were load-bearing, and in this package none of it was.

`pkg-12` failed on the same check, which tells me this is one systematic over-strictness rather than two separate misreadings.

**Check rationale**

Quoted from the `Steps re-runnable by a stranger` row of the `rubric.md` uploaded to `tools/repro-check/`, exactly as it reads now:

> | Steps re-runnable by a stranger | The reproduction steps in the repro report, read as an ordered sequence from a stated starting state to the trigger, together with every input those steps consume (files, commands, data, config) and any setup the report says it relied on. | A reader who has the recorded environment and no other context could execute the steps in order and reach the trigger: the starting state is stated, every input the run consumes is supplied or linked, and no step depends on a fact the package never states. A step that says to set the project up without naming the doc or command that does it, or that consumes a fixture the package never supplies, fails. | required |

It reads that way because of what I rejected in writing it. My first instinct was a structural condition — steps are numbered, start from a clone, name a command — and I rejected that, because a structure-shaped check grades the write-up's shape rather than its outcome, and two graders applying it disagree with each other about formatting while agreeing about the package. So the pass condition is phrased as a question about a reader: could someone with the recorded environment and no other context execute this sequence and reach the trigger. That is an outcome a second grader can test for by trying to answer it.

The evidence column is written to make that question answerable rather than intuitive. Naming the inputs explicitly — files, commands, data, config — is what stops the check from collapsing into a general impression of thoroughness, and it is what caught the `unfollowable-comms` packages, where steps genuinely depend on facts the package never states. That category matched 3/3.

**Trade-offs**

The steps check gives up packages whose unshown input is reconstructable or immaterial, and I can name the two it cost: `pkg-05` and `pkg-12`, both `failed: Steps re-runnable by a stranger`, both gold `accept`. The clause "every input the run consumes is supplied or linked" draws no distinction between an input a reader cannot guess and one whose contents do not bear on the bug, and in `pkg-05` the `env.yml` was the second kind.

I did not change the check after the run, and the reason is a judgement about what the loosening would cost. The clause that lost those two packages is the same clause that earned `unfollowable-comms 3/3` and contributes to `no-evidence 4/4`: in those packages the unshown input is exactly what makes the reproduction unfollowable. A revision adding something like "unless the input's contents are immaterial to the behaviour under test" moves the judgement of materiality from the reader to the grader, and a grader who may decide an input does not matter is a grader who can be talked out of the check in the cases where it works. The eval README is explicit that a loosened check can flip packages that agreed before, and the packages at risk here sit in the two categories this check is carrying.

So the 2 points of disagreement are the price of 7 matches elsewhere, paid knowingly. The run as committed is `18/20` with every category matched including `disclosure 1/1`, and the uploaded rubric is the one that produced it — no untested wording is being presented as though it had earned that score.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
