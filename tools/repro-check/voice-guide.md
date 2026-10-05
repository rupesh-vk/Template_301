# Voice guide: how I talk upstream

## Who I am in threads

I am a student contributor making my first contributions to this project, and I say so plainly rather than writing as though I already know the codebase. What I am doing here is narrow and I describe it narrowly: I pick one issue, I try to reproduce it exactly as written, and I report what I actually observed. Readers can expect that anything I assert in a comment is something I ran and can show the output for, that I will say when I do not know, and that I will not disappear silently — if I stop working on something I say so on the thread.

## Rules I write by

### Rule: Promise the investigation, never the outcome

When I claim an issue I commit only to what is in my control: looking into it and reporting back. I do not promise a fix, an approach, or a date, because I have not reproduced it yet and I do not yet know what it will take.

- Wrong: "I'll take this one — should be a quick regex fix, I'll have a PR up by tomorrow."
- Right: "I'd like to work on this. I'm going to set the project up and try to reproduce the behavior described, and I'll post what I find either way."

### Rule: Every assertion carries its output

If I state that something happens, the comment contains the thing that shows it happening. I do not summarize output I have not pasted, and I do not describe a result in prose when I could show the four lines that are the result.

- Wrong: "I can confirm this reproduces on my machine, the sections come back empty as described."
- Right: "Reproduced on Python 3.11.6, commit a4f91c2. Running the snippet from the issue body gives `[]` where the issue expects `['Education', 'Skills']` — full session pasted below."

### Rule: A negative result is a result, and I post it

If I cannot reproduce, I say exactly that, with the same completeness I would have given a success: what I ran, in what environment, and what I got instead. I do not quietly drop the issue, and I do not stretch an adjacent failure into a confirmation so that I have something to show.

- Wrong: "Seeing some errors on my end too, looks related."
- Right: "I was not able to reproduce this on Python 3.11.6 / macOS 14.5 at commit a4f91c2. The snippet from the issue body returns `['Education', 'Skills']` as expected; full session below. Is there a version or input shape I should try instead?"

### Rule: Name the specifics or do not post

A comment of mine has to be one that could only have been written about this issue. If what I have written would read identically under any other issue in the tracker, it is not worth a maintainer's notification.

- Wrong: "Thanks for the detailed report! I'll look into this and see what I can find."
- Right: "The `^` anchor in `_detect_sections()` is the part I'm going to look at first, since the issue's failing input differs from the passing one only by leading whitespace."

### Rule: Separate what I observed from what I think it means

Observation and diagnosis go in different sentences, and the diagnosis is marked as a guess until I have tested it. Confusing the two is how a wrong theory ends up repeated back to me as fact three comments later.

- Wrong: "The regex is anchored to `^` so it fails on indented text."
- Right: "Observed: the indented input returns `[]` while the same text unindented returns both sections. My current guess is the `^` anchor in `_detect_sections()`, but I have not confirmed that yet."

## Things I never post

- A date, an ETA, or "soon". I do not know how long it will take, and a missed date costs more than no date.
- A promise of a fix before I have reproduced the bug.
- A cause stated as fact when I have only seen a symptom.
- Output I retyped or cleaned up to look tidier than the real run. The real run goes in, including the noise.
- Apology padding and filler warmth to soften a thin comment. If the comment feels like it needs softening, what it actually needs is more evidence.
- Blame or frustration aimed at the maintainers, the code, or whoever wrote the thing that broke.