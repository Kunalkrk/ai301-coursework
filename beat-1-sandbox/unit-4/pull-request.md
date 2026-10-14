# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/191

**Branch**

fix/36-profile-ingestion-check

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. `agreement: 2/3 scored items` — smoke run, `--limit 3`, first draft of `SKILL.md`/`rubric.md`/`procedure.md`/`evidence-guide.md`. Disagreed on `pkg-02` (gold `accept`, my rubric said `reject`, failed check: `Test evidence` — a scenario proven only by a codified test assertion, not a separate transcript, was being treated as unproven).
2. `agreement: 2/2 scored items` — `--only pkg-02,pkg-04` (`calib-04` as a free canary) after loosening Test evidence to accept a concrete test assertion, not only a transcript, as evidence for a named scenario.
3. `agreement: 17/20 scored items  (bar: 18/20: below the bar)` — first full run with that loosened wording. Disagreed on `pkg-08`, `pkg-13`, `pkg-19` (all gold `accept`, all failed `Test evidence` — this time because a plan's named *control* scenario, not a primary fix-demonstrating one, was held to the same strict evidence standard as the primary scenario).
4. `agreement: 7/7 scored items` — `--only pkg-08,pkg-13,pkg-19,pkg-02,pkg-07,pkg-10,pkg-14` (`calib-04` as canary) after revising Test evidence to distinguish primary scenarios (need concrete evidence) from controls (a plain stated claim suffices, unless contradicted).
5. `agreement: 19/20 scored items  (bar: 18/20: PASS)` — second full confirming run, `--save-run eval-run.txt`. `pkg-04` flipped to a false `accept` — confirmed not model noise by re-running `--only pkg-04` alone with the same result: the loosened wording let a unit test of an internal helper function (`derive_flags_for_probe_output`) count as evidence for the plan's named end-to-end repro, when the plan's test plan specifically asked for that real command to be re-run.
6. `agreement: 5/5 scored items` — `--only pkg-04,pkg-02,pkg-08,pkg-13,pkg-19` after tightening Test evidence once more: a test assertion only counts when it exercises the same observable behavior the plan named, not an internal helper one step removed from it.
7. `agreement: 20/20 scored items  (bar: 18/20: PASS)` — final confirming run, `--save-run eval-run.txt`. This is the run committed as `eval-run.txt`.

**Package analysis**

Package: `pkg-01` (`pandas-dev/pandas#66657`). Gold: `reject` (`standards-wall`). My rubric: `reject`, failed `Standards and comms` — agreed on the first run, no revision needed for this one.

The PR itself is excellent: a real ASAN-verified heap-buffer-overflow repro (`len(df.iloc[40, 0])` going from `99999` to the correct `100000`, confirmed with a clean ASAN run), a diff that matches the plan's named file (`pandas/_libs/src/parser/tokenizer.c`) and approach exactly, and a regression test. Gold still rejects it, because pandas' PR template is a required checklist — `Closes #xxxx`, tests added and passed, pre-commit checks passed, type annotations on new arguments, and an entry in the current `doc/source/whatsnew/vX.X.X.rst` for any bug fix — and the candidate description has no `Closes #66657` line, no checklist at all, and (checkable directly, not just from the description's silence) no `whatsnew` file anywhere in the diff's two touched files (`tokenizer.c` and one test file). My `Standards and comms` check's evidence explicitly says to check the repo-facts block's *required files* against the diff's actual file list, not just read the description's prose, which is exactly what caught this: an otherwise clear-accept PR failing purely because the repo's stated template asks were never addressed.

**Check rationale**

Check, quoted as currently written in `rubric.md`:

> **Test evidence** | The test-evidence section's before/after output and the diff's own test assertions, read against every *primary* scenario the plan's test plan named (each distinct failure mode or repaired behavior the fix addresses), plus whether the repo's own required checks (test suite, lint, typecheck) were run and their outcome shown | Fail if any primary scenario — a scenario demonstrating the fix itself, or one of several distinct failure modes the plan named as something the fix must address — has nothing concrete shown for it anywhere in the package: neither a before/after transcript nor a specific assertion in the diff's own test code pinning the expected post-condition. A scenario backed by a concrete test assertion in the diff (e.g. `assert x == expected_value`) plus a passing test-suite run counts as shown, but only when that assertion exercises the same observable behavior the plan's test plan named — calling the real affected command/function and checking the real described symptom — not when it calls an internal helper the fix extracted and checks that helper's return value one step removed from the actual repro (a unit test on a flag-derivation function is not evidence the real paged output shows colors; it's evidence the helper returns the right list). A plan's named *control* ... is different: a plain stated claim that the control is unchanged passes, as long as nothing else in the package contradicts it and the overall test suite passes ... Otherwise pass. | required

It reads this way because of two separate over-corrections, found in that order. First draft required a before/after transcript for every named scenario, which failed `pkg-02` for a control proven only by a real test assertion — so I added "a test assertion counts as shown evidence." That loosening then failed to distinguish a plan's *primary* scenario (the thing the fix must prove) from its *control* (a side check that something unrelated didn't break), and started demanding transcript-or-assertion rigor even for controls that real PRs just state in prose — failing `pkg-08`/`pkg-13`/`pkg-19`, all genuinely ready. Fixing that let a third gap through: `pkg-04`'s unit test asserted on an internal helper function's return value, not on the actual paged-output-with-colors behavior the issue and plan's test plan named, and my wording didn't yet distinguish "tests the real repro" from "tests a piece the fix was refactored into." The check's current form is the result of closing both gaps without reopening the first one.

**Trade-offs**

The current wording still trusts that a test assertion calling the "real affected command/function" is itself decisive — it doesn't require the assertion to check every facet of the real symptom, only that it's not calling an internal helper in isolation. A PR could still pass this check with an assertion that technically calls the right public function but only checks a shallow property of the result (an exit code, say, without checking the actual rendered output), when the plan's test plan asked for the fuller symptom. I accept that gap because catching it would mean grading the *quality* of an assertion's coverage, which is a different and much fuzzier judgment than the evidence-guide's "the same observable behavior, not an internal helper" line, and I verified the current wording doesn't buy back the packages it's actually built to reject: re-running `pkg-07`, `pkg-10`, `pkg-14`, and `calib-04` (all genuinely `not-tested`, for reasons ranging from "evidence exercises the wrong path" to "one of two named failure modes is completely silent") as canaries alongside every revision, all four still correctly rejected throughout.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
