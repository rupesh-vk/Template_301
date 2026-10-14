# Plan — issue #54: resume section detection fails on text with leading whitespace

## Diagnosis

`_detect_sections()` in `ingestion/parsers/resume_parser.py` builds four regexes per known section name and searches the lowercased text with `re.MULTILINE`:

    patterns = [
        rf"^{re.escape(section)}\s*$",
        rf"^{re.escape(section)}\s*[:|-]",
        rf"\n{re.escape(section)}\s*$",
        rf"\n{re.escape(section)}\s*[:|-]",
    ]

Every one of the four requires the section name to begin immediately at a line boundary — directly after `^` (which under `MULTILINE` matches at each line start) or directly after `\n`. None of them tolerates horizontal whitespace between that boundary and the name. When a line reads `    Education:`, the character following the line start is a space, so no pattern matches and the section is never appended to `detected`.

This is grounded in the reproduction I posted on the issue thread:

> indented -> []
> flush    -> ['Skills', 'Experience', 'Education']

The two runs used the same text and the same section names; the only difference was leading whitespace on each line. The empty list on the first run is `detected` never being appended to, which is what the failing assertion in `test_detect_sections` reports:

> E       assert 0 > 0
> E        +  where 0 = len([])

I have confirmed the mechanism by reading the four patterns above against that contrast. I have not instrumented the regex engine to observe the non-match directly.

## Scope

**In scope — one bounded change:** allow horizontal whitespace between the line boundary and the anchored token in the patterns that assume no indentation — the four in `_detect_sections()`, and the heading pattern in `_strip_markdown()` — and remove the `xfail` markers that exist only to record this bug.

`_strip_markdown()` is in scope because it is the same cause, not a second one. Its heading pattern reads `re.sub(r"^#+\s+", "", content, flags=re.MULTILINE)`, which requires `#` immediately at the line start on exactly the same assumption the four detection patterns make. On an indented markdown resume the `#` is never stripped, so `_detect_sections()` is handed `    # education` and fails to match it even once its own anchors are loosened. Fixing one without the other leaves the markdown path broken.

The marker removal is not a second change. The five markers are `@pytest.mark.xfail(strict=True, reason="issue #54: resume section detection fails on leading whitespace")`. Under `strict=True`, a test that starts passing is reported as a failure (`XPASS`), so the suite goes red on the fix unless the markers come off. They are part of this fix, not a tidy-up alongside it.

**Not in scope:**
- The contents of `SECTION_HEADERS`. No section names are added or removed.
-- `_parse_pdf()`, `_parse_markdown()`, and the `ParseResult` metadata shape.
- The rest of `_strip_markdown()`. Only its heading pattern is touched; the link, bold, italic, code-block, inline-code, HTML, and whitespace-collapse substitutions are left exactly as they are.
- The redundancy between the `^...` and `\n...` pattern pairs. Under `re.MULTILINE` the `^` forms already cover what the `\n` forms match, so two of the four are dead weight — but removing them is a refactor that does not serve this cause, and I am leaving them.
- Trailing-whitespace handling, which `\s*` already covers.

## Files

- `ingestion/parsers/resume_parser.py` — the four patterns in `_detect_sections()`.
- `tests/unit/test_resume_parser.py` — removal of the five `xfail` markers for issue #54, and one new case.

## Approach

1. In `_detect_sections()`, insert `[ \t]*` after the leading anchor in each of the four patterns, so they read `rf"^[ \t]*{re.escape(section)}\s*$"`, `rf"^[ \t]*{re.escape(section)}\s*[:|-]"`, and the `\n` forms likewise.

   `[ \t]*` rather than `\s*` is deliberate: `\s` matches `\n`, so `\s*` after the anchor would let the match skip across blank lines and begin on a line the anchor never applied to, which would detect sections at positions that are not line starts at all.
2. In `_strip_markdown()`, change the heading substitution from `r"^#+\s+"` to `r"^[ \t]*#+\s+"`, for the same reason and with the same `[ \t]` rather than `\s` choice. This also drops the leading indentation on heading lines, which is the behaviour the unindented path already produces.

3. Remove the five `@pytest.mark.xfail(...)` decorators in `tests/unit/test_resume_parser.py` that cite issue #54, along with the now-unused import if `pytest.mark.xfail` is not used elsewhere in the file.

4. Add one test asserting that the same text indented and unindented produces the same section list, so a future re-anchoring of these patterns fails loudly rather than silently.

## Test plan

Re-running the two steps from my posted reproduction, at the fix commit rather than `f89c06f`.

**Step 1.** The test that currently fails on its length assertion:

    python3 -m pytest tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections -v

Before: `XFAIL` normally, and `FAILED ... assert 0 > 0 / where 0 = len([])` under `--runxfail`.
After: `PASSED`, with no `--runxfail` needed, because the marker is gone.

**Step 2.** The indented-versus-flush contrast, run exactly as posted:

    print("indented ->", p._detect_sections(indented))
    print("flush    ->", p._detect_sections(flush))

Before:

    indented -> []
    flush    -> ['Skills', 'Experience', 'Education']

After: both lines print the same three sections, in any order — `['Skills', 'Experience', 'Education']` — since `_detect_sections` returns `list(set(detected))` and the order is not stable.

**Step 3.** The whole file, to confirm no `XPASS` and no regression:

    python3 -m pytest tests/unit/test_resume_parser.py -v

Before: `5 passed, 5 xfailed`.
After: `11 passed` — the five previously-xfailed tests, the five that already passed, and the new case.

## Risks and unknowns

**Two of the other four xfailed tests had a second cause, now in scope.** I originally planned on the assumption that all five markers recorded the same fault, having run only `test_detect_sections` with `--runxfail`. Applying the `_detect_sections()` change alone to a scratch copy of `HEAD` and running the file gives `2 failed, 8 passed`: `test_parse_markdown_resume` and `test_strip_markdown_syntax` still fail, because `_strip_markdown()`'s heading pattern is anchored the same way. That is what moved it into scope above. The remaining two markers — `test_parse_single_column_resume_text` and `test_parse_resume_no_work_experience` — I have still not isolated individually; step 3 of the test plan is what surfaces them if they differ.

**Loosening the anchor admits false positives.** Any indented line whose first word is a section name now registers — a bullet reading `    Skills: Python, Go` nested under Experience would add `Skills`, where before it did not. That is the intended behaviour for this issue and is indistinguishable from it by this function, which has no notion of document structure. I am accepting a wider match surface, not avoiding it, and I note it because it is the cost of the fix rather than a side effect of it.

**`detected_sections` is consumer-facing metadata.** It is written into `ParseResult.metadata` by both `_parse_pdf()` and `_parse_markdown()`. Callers that previously received `[]` for indented resumes will now receive populated lists. I have not read those callers and do not know whether any of them branches on an empty list.

**Unconfirmed:** that `[ \t]*` is sufficient for real PDF extraction. `pypdf`'s `extract_text()` may emit non-breaking spaces or other Unicode whitespace that `[ \t]` does not cover. `test_parse_multipage_pdf` passes today, so the PDF path is exercised, but not against an indented fixture.

## Deviations

Two deviations, one before the build and one during it.

**`_strip_markdown()` moved into scope before I started.** My first draft scoped the change to `_detect_sections()` only, on the assumption that all five `xfail` markers recorded one fault. Running the `_detect_sections()` change alone against a scratch copy gave `2 failed, 8 passed`, because `_strip_markdown()`'s heading pattern `r"^#+\s+"` is anchored on the same assumption. I widened the scope and the posted plan comment reflects that, so the plan as posted is the plan as built.

**The regression-test omission from Unit 3 was closed during Unit 4.** The original build at `d006d21` omitted approach step 4 and had `10 passed` rather than the predicted `11 passed`. During PR preparation I added `test_detect_sections_matches_unindented_text`, parametrized for four spaces and one tab. Both indented inputs and the unindented control must produce exactly `Experience`, `Education`, and `Skills`, compared as sets because return order is unspecified. This implements the posted plan's comparison and adds coverage for both characters accepted by `[ \t]*`. The parser file now has `12 passed`, rather than the original prediction of `11`, because the one test function contributes two cases. No additional production behavior changed.

Everything else went as planned. The diagnosis held; the production change remains the five regex edits, with five marker removals and the planned comparison test in the test file. Both changed files are the ones named. Planning documents and captured evidence remain outside the fix commits.