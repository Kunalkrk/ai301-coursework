# Procedure: how this tool grades a PR package

## Read order

1. Read the issue first (title, body). Write down the one reported
   behavior in one sentence.
2. Read the thread highlights. Flag any comment from an OWNER, MEMBER,
   or COLLABORATOR that states a diagnosis, a preferred approach, or
   asks for something specific — this is the maintainer direction the
   Standards and comms check reads against.
3. Read the plan-context block in full before reading anything about
   the candidate PR. From it, write down four things as a checklist you
   will hold the PR against: the stated cause, the in-scope/not-in-scope
   lines verbatim, the exact Files list, and the test plan as discrete
   numbered scenarios (not a paragraph). Read the plan before the PR,
   never after: the PR's own description is an interested party's
   account of its fidelity, and reading the plan first stops that
   account from setting the frame the diff gets measured against.
4. Only then read the candidate PR, in this order: title, description,
   commit list, the full diff (every hunk, not a skim), test evidence
   last. Reading the diff in full before the test evidence stops a
   confident description or a clean-looking test section from deciding
   what you look for in the diff.
5. Live mode: the plan-context block is the student's `plan.md`
   (deviation notes included); the diff is `git diff main...HEAD` run
   from the top of the branch's working copy; the PR title/description
   come from the student's named draft file; the test evidence comes
   from the student's named evidence file. A house-chain student's plan
   context is the house plan and repro pack as quoted in their drafts;
   if the drafts quote none at all, that absence is what Plan fidelity
   and Test evidence grade, not a reason to fetch it yourself.

## Evidence gathering

For each check, pull exactly this before grading it:

- **Plan fidelity**: a complete list of every file the diff touches
  (from step 4), checked one by one against the plan's Files list and
  in-scope/not-in-scope lines (from step 3) and any deviation note; the
  description's fidelity claims, quoted verbatim.
- **Test evidence**: the plan's test plan as numbered scenarios (from
  step 3); for each one, whether the test-evidence section shows a
  before/after for that specific scenario; the stated outcome of any
  repo-required check (test suite, lint, typecheck) in the test
  evidence.
- **Diff quality**: every hunk of the diff (from step 4), scanned
  specifically for commented-out lines, dead/unused code, debug prints,
  and pure-reformatting hunks; the commit message list as a secondary
  signal only.
- **Standards and comms**: the repo-facts block's PR-template sections
  and required files, the stated contribution/AI policy, and the
  flagged maintainer comments from step 2 — each read against the
  title/description from step 4.

## Check execution

1. Grade in this order: Plan fidelity, Test evidence, Diff quality,
   Standards and comms. Plan fidelity goes first because every other
   check is measured against what the plan actually asked for, not what
   the PR's own framing claims it did — grading Test evidence before
   confirming fidelity risks crediting decisive evidence for the wrong
   change.
2. Grade each check `pass`, `fail`, or `unclear` using only the evidence
   pulled for it in the stage above. If a check's evidence is genuinely
   absent from the package (no test-evidence section at all, no stated
   PR template in the repo-facts block to compare against), grade that
   check `unclear` rather than inferring from general knowledge of the
   project.
3. A check is graded once per package; do not re-read the whole package
   per check once Read order and Evidence gathering are complete.
4. Quote the exact file, hunk, scenario name, or template line that
   decided each grade as that check's evidence — the same standard for
   a `pass` as for a `fail`.

## Verdict assembly

1. Apply the verdict rule in `rubric.md`: `accept` only if all four
   required checks graded `pass`.
2. Any check graded `fail` or `unclear` makes the verdict `reject`.
3. When more than one check fails, the output's summary names every
   failing or unclear check with its own evidence line, not just one —
   but when picking which single check's evidence to foreground as "the
   deciding check" in the readable summary, use the rubric's own row
   order (Plan fidelity, then Test evidence, then Diff quality, then
   Standards and comms) and name the first one that failed.
4. The same four grades, fed through the same rule, must produce the
   same verdict every time; if a second pass over an unchanged package
   would produce a different verdict, re-grade rather than report the
   discrepancy.
