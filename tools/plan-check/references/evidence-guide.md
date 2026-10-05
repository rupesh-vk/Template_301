# Evidence guide: where evidence lives in a plan package

## Diagnosis and grounding

**Where it lives.** The plan's cause is stated in its diagnosis section — the sentence naming what in the code produces the wrong behaviour — and is sometimes restated in the approach, in which case both are read and any difference between them is itself a finding. The behaviour that cause must explain lives in the repro-evidence block: its recorded observations, the output or values it shows, the failing assertion it names, and any line saying what it did *not* establish. In live mode: the plan's diagnosis in `plan.md`, read against my own posted repro comment on the issue thread, and against the issue body for the behaviour originally reported.

**What good looks like.** The stated cause cites a specific observation the repro evidence actually records, and no observation in that evidence contradicts it. A diagnosis follows from the evidence when you can point at the line of recorded output that makes it the explanation rather than one of several. It ignores the evidence when the evidence contains a contrast the cause does not account for — a case that worked, a value that came back right — and it contradicts the evidence when the cause would predict an observation the evidence shows did not happen. A cause the evidence cannot settle is acceptable *when labelled*: a plan that says which step would confirm it is grounded, while one that asserts the same thing flatly is not, and the only difference between them is a clause.

## Scope

**Where it lives.** The in-scope statement and the not-in-scope line of the plan, plus the list of files or areas it names as the ones it will touch. In live mode: the scope section of `plan.md` and its file list, read against the branch I will actually push.

**What good looks like.** The boundary can be read off the plan without inference — a reader can say what will change and what will not, and does not have to deduce the second from silence. Every item inside the boundary serves the stated cause, and anything that does not is either removed or justified in a sentence showing it is required to fix this one. One bounded change looks like a short file list in which each entry traces to the diagnosis; a drive-by rewrite looks like a fix plus a tidy-up — a rename, a reformat, a dependency bump, a second defect noticed on the way — carried along because the author was already in the file. Breadth alone is not the test: a change touching six files that all follow from one cause is bounded, and a change touching one file that also reformats it is not.

## Executability

**Where it lives.** The plan's approach, its named files or areas, and whatever it says about the order of work. In live mode: the approach section of `plan.md`, read with the fork's clone open.

**What good looks like.** A reader with the repo open can identify the first concrete change and the file it belongs in, and begin, without asking the author a question. The behaviour to change is described in terms of what the code should do differently, not only in terms of the outcome wanted — "stop anchoring the header match to the start of the line" is executable, "make indented resumes parse" is a restatement of the bug. The test is whether any step still contains a decision the plan left unmade; a plan that names two possible approaches without choosing is not executable, however well it understands the problem.

## Test plan

**Where it lives.** The plan's test plan section, read alongside the repro evidence's own steps and artifacts, which it is supposed to re-run. In live mode: the test plan in `plan.md` next to the commands and output in my posted repro comment.

**What good looks like.** It re-runs the behaviour the repro pinned down, by the same route the repro took, and names what will be observed afterwards in terms someone else could check: a value, a printed output, a named test and the result expected from it. A decisive test plan names the thing that will be different; a vague one asserts a state — that the fix "works", "is verified", "passes the tests" — which is a conclusion rather than an observation. Where the plan tests a different path than the repro used, it has to say why that path is equivalent, because a test exercising a path the bug was never shown on proves nothing about the bug.

## Honesty

**Where it lives.** The plan's risks and unknowns section, read against the confidence of everything above it; and, after a build, the `## Deviations` section where what actually changed is recorded. The assumptions to check against the risks are found in the approach — the things it relies on being true without having tested them.

**What good looks like.** Every assumption load-bearing in the approach appears in the risks, marked as unconfirmed, and the plan names at least one thing that could turn out otherwise — or states plainly that it found none and says what it checked. False confidence is the failure shape here, and it reads as fluency: a plan whose approach quietly depends on a function behaving a certain way, with a risks section listing only generic hazards ("the change could introduce a regression"), has stated no unknowns at all. Generic risks are the tell, because they would appear unchanged under any plan for any issue. An honest deviation, recorded after the build, says what changed from the plan and why, and is worth more than a plan that happened to be right.

## Comms

**Where it lives.** The plan comment, read in two directions. Against the thread: the thread highlights in an eval package, or the live issue thread, for any direct question asked of the contributor and any preference a maintainer has stated about approach. Against the repo: the repo-facts block, or in live mode `CONTRIBUTING.md`, the `README`, `.github/` templates, and any `AGENTS.md` or AI-use policy, for stated conventions, required template fields, and AI-assistance disclosure requirements.

**What good looks like.** Thread-aware means the comment answers what was actually asked — if a maintainer raised a concern or asked a question, the comment addresses it rather than posting past it, and if a maintainer stated a preference about approach, the comment either follows it or says why not. Boilerplate, by contrast, is a comment that would read identically under any issue: it summarises a plan without engaging anything specific to this thread. On the repo side, every stated requirement that applies to the comment is satisfied in the comment's own text — most sharply, where the policy requires disclosing AI assistance, an AI-assisted plan whose comment carries no disclosure fails however good the plan is. A repo silent on a requirement cannot fail on it.
