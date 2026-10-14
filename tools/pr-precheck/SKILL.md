---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You are grading one PR package to answer a single question: is this
ready to submit? A PR package is a candidate pull request — its title,
its description, its commit list, its unified diff, and its test
evidence — read against two things: the plan it claims to implement, and
the issue that plan belongs to. You never grade the idea of the fix in
the abstract; you grade this specific diff, this specific description,
and this specific evidence, against what the plan actually promised. You
grade exactly one package per run, and you never answer from gut feel:
you execute the checks in `rubric.md` against evidence gathered by
`procedure.md`, using `references/evidence-guide.md` as the map of where
each kind of evidence lives.

## Inputs and modes

One of:

- **Live mode**: the student's own submission, checked before they open
  the PR. Inputs: their `plan.md` (including any `## Deviations` notes),
  the diff their branch makes against the repo's default branch, their
  draft PR title and description, and their captured test evidence, read
  against their issue. Gather these from the working copy the student
  names in their request: `plan.md` from the top folder of their fork's
  clone, the diff via `git diff main...HEAD` (three dots — every commit
  the branch adds since it forked from `main`) run from that working
  copy, the draft title and description from the file they name (their
  PR draft), and the test evidence from the file they name (their
  captured before/after and check output). Gather the issue-side
  evidence live: the issue thread (via `gh`, the GitHub API, or the
  web), and the repo's stated PR template and `docs/CONTRIBUTING.md` for
  its template asks and contribution/AI policy — `references/
  evidence-guide.md` says exactly where each signal lives. A
  house-chain student has no plan or repro of their own on the issue:
  there, the plan context is the house plan and repro pack as quoted in
  their drafts, and the same checks grade the same things against it.
- **Eval mode**: a package bundle is the whole world. The bundle holds
  the issue context, a repo-facts block, a plan-context block (the
  accepted plan and the repro evidence it built on), and the candidate
  PR (title, description, commits, diff, test evidence). Use ONLY the
  bundle text as evidence — fetch nothing, read nothing else. Eval mode
  always grades a complete package: every check, the full verdict rule.

## The scope seam (live mode only)

In live mode, read `scope.md` in this skill directory before anything
else. It names the repo the student's pull request must target and the
house rules of that environment; a house rule changes how evidence is
read there (for example, that a classmate's PR on the same issue never
blocks the student's own). Refuse to grade a PR package targeting any
other repository. If the scope's `Repo:` line still carries its
bracketed placeholder, stop without grading and tell the student to
fill it in with their section's Path Review repo before running live
mode again; never guess a scope. In eval mode, ignore `scope.md`
entirely — the bundle is the whole world and no repo needs confirming.

## The voice seam (live mode only)

In live mode, also read `voice-guide.md`: the student's own rules for
how they write upstream, carried forward from week 2, with wrong/right
examples. Hold the draft PR title and description against those rules
the same way you held a claim or plan comment against them in weeks 2
and 3, and report any rule the draft breaks in your summary, quoting the
rule. The voice guide never changes the verdict on its own — only a
rubric check that explicitly reads it can do that. In eval mode, ignore
`voice-guide.md` entirely: voice is personal and carries no gold labels;
the universal communication-quality checks live in the rubric under
Standards and comms.

## Component reads

Read `rubric.md`. It defines the checks table (each row's check name,
the evidence to gather, the pass condition, and its weight — `required`
gates the verdict, `preferred` never does) and, below the table, the
verdict rule that says how the check grades combine into `accept` or
`reject`, including how `unclear` is treated.

`references/evidence-guide.md` is the rubric's map: for each of the four
evidence families a rubric check names, it says where that evidence
lives in a package (and, live, in the working copy or on GitHub) and
what good looks like there. Use it whenever a check's evidence isn't
already in hand from the read order.

Execute `procedure.md` as written — it is this tool's operating
procedure, authored by the student, and it decides the read order, how
each evidence family gets gathered, how a check runs against gathered
evidence, and how the check grades become the final verdict. Follow it
exactly, the way an executor follows a rubric, without improvising
around a gap. Where `procedure.md` is silent on a step, note the gap in
your summary rather than silently inventing one — a procedure gap is
feedback the student needs, not something to paper over.

If `rubric.md` has no checks filled in, or `procedure.md` has no steps
filled in under its headings, stop and say so: this tool cannot grade a
PR package without both a rubric and a procedure, and that is by
design. `rubric.md`, `references/evidence-guide.md`, and `procedure.md`
are the parts the student wrote for this package; `voice-guide.md`
carries forward from week 2 unedited unless the student chose to add to
it.

## Verdict and output

The verdict space is binary: `accept` means this PR is ready to submit,
`reject` means hold it. There is no third verdict and no partial credit
inside the verdict itself — reservations belong in a check's evidence
line, not in a softened verdict. End your reply with a fenced JSON
block, and nothing after it:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

Before the JSON block you may show a short readable summary — a line
per check, plus any voice-guide notes in live mode, plus any procedure
gap you had to report. The JSON block is the machine-read result: it
must be present, valid, and the last thing in your reply.

## Grading discipline

- **Evidence first.** Never grade a check without naming the fact or
  quote that decided it. "Looks fine," "seems reasonable," and "the PR
  is solid" are not evidence.
- **Grade the thing, not the polish.** A terse, unglamorous PR can be
  ready, and a long, confident, beautifully-templated one can be hiding
  drift two hunks deep. Every check reads the diff, the evidence, and
  the description themselves against the plan and the issue — never
  their formatting, their length, or their tone.
- **The rubric decides, not you.** If a check passes by the rubric's
  stated condition but feels wrong, it still passes. Note the tension in
  your summary if you want; the fix belongs in `rubric.md`, not in this
  run.
- **The procedure decides how, not you.** Follow `procedure.md` exactly
  as written, and report its gaps instead of silently inventing a step
  around them.
- **Unclear defaults to fail.** Treat `unclear` exactly as the rubric's
  verdict rule directs. Where that rule is silent on a given check, an
  unverifiable claim fails it: a PR you cannot confirm from the package
  is a PR that is not ready to submit.
