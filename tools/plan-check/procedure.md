# Procedure: how this skill grades a plan package

## Read order

1. Read the issue context first (title, body, labels). Write down the one
   reported behavior in one sentence: what happens, under what trigger.
2. Read the thread highlights next. Note any comment from an OWNER,
   MEMBER, or COLLABORATOR that states a diagnosis, names a preferred
   approach, posts a patch, or asks the reporter/a contributor to do
   something specific. This is the "explicit maintainer direction" the
   Comms check reads against.
3. Read the repro-evidence block next, before reading the candidate plan.
   For each numbered step, write down what was run and what was observed.
   Flag any step the block itself calls a control, or any step whose
   result rules something in or out (a step that reproduces with a
   mechanism absent, or that pins down which stage of a pipeline a value
   changes at). These flagged steps are what the Diagnosis and Test-plan
   checks are graded against, so finding them before reading the plan
   stops the plan's own framing from deciding what counts as evidence.
4. Only then read the candidate plan in full, in whatever order it's
   written (diagnosis, scope, approach/files, test plan, risks), followed
   by the candidate plan comment.
5. Live mode only: the repro evidence is the student's own posted repro
   comment on the issue, or the house repro pack when working a house
   issue. If neither the plan nor the comment quotes any repro evidence
   at all, do not invent it or fetch it yourself — note the absence, since
   it is itself what the Diagnosis and Test-plan checks grade.

## Evidence gathering

For each check, pull exactly this before grading it:

- **Diagnosis and grounding**: the plan's stated cause (step 4's notes),
  plus every flagged control/isolation step from step 3.
- **Scope**: the plan's in-scope/not-in-scope statement, or, if absent,
  the full files/areas list from its approach section; plus the
  one-sentence reported behavior from step 1.
- **Executability**: the plan's named files/areas and its approach or
  ordered steps, verbatim.
- **Test plan decisiveness**: the plan's test plan section, plus the
  repro evidence's steps/artifacts from step 3 it should map onto.
- **Honesty**: any risk/unknown/deferral language anywhere in the plan,
  plus any claim used by another check that was not fully backed there
  (e.g., a diagnosis detail the repro evidence doesn't actually pin down,
  stated instead as settled).
- **Comms**: the plan comment's full text, the flagged maintainer
  comments from step 2, and the repo-facts block's contribution/AI-policy
  line.

## Check execution

1. Grade in this order: Diagnosis and grounding, Scope, Executability,
   Test plan decisiveness, Honesty, Comms. Earlier checks inform later
   ones (a wrong-cause diagnosis makes a decisive-sounding test plan
   pointless to grade generously, since it would be testing the wrong
   thing), but every check still gets its own grade and evidence line.
2. Grade each check `pass`, `fail`, or `unclear` using only the evidence
   pulled for it in the previous stage. If the evidence a check needs is
   genuinely absent from the package (no repro evidence quoted at all, no
   test plan section present), grade that check `unclear` — do not infer
   or fill the gap from general knowledge of the project.
3. A check is graded once per package; do not re-read the whole package
   per check once the Read order and Evidence gathering stages are done.
4. Quote the exact phrase or step that decided each grade as that check's
   evidence line — the same standard for a `pass` as for a `fail`.

## Verdict assembly

1. Apply the verdict rule in `rubric.md`: `accept` only if all six
   required checks graded `pass`.
2. Any check graded `fail` or `unclear` makes the verdict `reject`.
3. When the verdict is `reject`, the output's summary names every failing
   or unclear check, each with its evidence line, not just the first one
   found — a plan can fail more than one check at once.
4. The same six grades, fed through the same rule, must produce the same
   verdict every time; if a grading pass would produce a different
   verdict than an earlier pass on an unchanged package, re-grade rather
   than report the discrepancy.
