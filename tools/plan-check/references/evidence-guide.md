# Evidence guide: where evidence lives in a plan package

## Diagnosis and grounding

Where it lives: the plan's "Cause"/"Diagnosis" section, if it has one (a
plan with no cause stated at all has nothing for this check to pass on).
The evidence it's judged against is the repro-evidence block's numbered
steps and any step explicitly marked as a control or isolation run. In
live mode: the student's own posted repro comment on the issue (or the
house repro pack when working the house issue), plus any diagnosis a
maintainer already stated in the thread highlights.

What good looks like: the stated cause explains every step the repro
evidence shows, including the control steps — not just the step that
confirms the bug, but the ones that rule alternatives out. A diagnosis
that only restates a thread comment's claim ("as identified in this
thread...") without checking it against a control that contradicts it is
not grounded; the repro evidence outranks an unverified thread claim. A
diagnosis that names the specific function, file, or pipeline stage where
a control shows the behavior already present (or already absent) is well
grounded.

## Scope

Where it lives: the plan's "Scope"/"In scope"/"Not in scope" lines. If the
plan states no scope section, infer it from the files/areas its approach
actually lists. Judged against: the issue's own title and body — the one
behavior reported, not whatever broader problem the plan says that
behavior is secretly a symptom of.

What good looks like: one bounded change addressing the reported behavior,
with anything adjacent-but-not-asked-for named and explicitly deferred
(with a reason) rather than folded in. A plan that opens with language
like "this issue is the visible tip of a structurally unsound X, so this
plan addresses X as a whole" is scope creep dressed as thoroughness,
whatever the technical merit of the broader fix. A files/areas list with
one clear locus (one file, or a small number of directly related files) is
a good sign; a files list spanning unrelated subsystems, plus new CI jobs
or new abstractions, is not.

## Executability

Where it lives: the plan's named files/areas and its approach or ordered
steps (the "Change"/"Approach"/"Files" sections, however the plan labels
them).

What good looks like: every step names a concrete action at a concrete
location — a function, a file, a call site, or (when the exact site isn't
pinned yet) a named area plus a concrete, already-used method for finding
it, such as a debug trace already run and described. That is different
from leaving the choice itself open. Phrases that flag an undone decision
— "whichever is easier," "look into how X works," "maybe also," "not sure
which layer" — mean the plan is not yet executable, however confident its
prose sounds elsewhere; a plan that instead says "exact function to be
pinned after tracing with the debug logs I already have working" has
already done the narrowing and is naming its last concrete step, not
punting on a decision.

## Test plan

Where it lives: the plan's "Test plan" section, judged against the
repro-evidence block's own steps and artifacts (live mode: the student's
posted repro comment, or the house repro pack).

What good looks like: the test plan names a specific, observable
before/after — typically a re-run of the repro evidence's own steps (or
one of its controls), stating what should be seen after the fix that
wasn't seen before (an exit code, a printed value, a timing threshold, a
specific line of output). "Run the full test suite" or "shouldn't crash
anymore" with nothing named to check is not decisive, even when the rest
of the plan is strong — a genuinely good plan has been rejected on this
check alone before.

## Honesty

Where it lives: any "Risk"/"Unknown"/"Open question" language in the plan,
and any claim about completeness, cross-platform behavior, or confidence
that the fix works, read against what the repro evidence and thread
highlights actually settle versus leave open. In live mode, also check
`## Deviations` in `plan.md` after a build: a deviation recorded there,
before the follow-up comment goes out, is what honest-after-the-fact
looks like.

This check does not re-grade ground already covered: whether the stated
cause follows from the repro evidence is the Diagnosis check's job, and
whether a deferral is the right scope call is the Scope check's job. A
cause the Diagnosis check passed as grounded is not re-opened here just
because the exact fix site is still being narrowed down — that is an
Executability question, not a Honesty one.

What good looks like: the plan says plainly what it hasn't verified yet
(a cost not yet measured, a platform not yet tested, a question left for
review) rather than asserting it as settled. A stated, reasoned deferral
is a pass on this check even if a reviewer might disagree with deferring
it. What fails this check is a claim of completeness or cross-condition
certainty with nothing behind it — "this fixes it everywhere," "the cost
is negligible" with no measurement, a platform declared fine that was
never tested — not a diagnosis that other checks have already confirmed
is grounded.

## Comms

Where it lives: the candidate plan comment's full text, read against two
things — the thread highlights' maintainer (OWNER/MEMBER/COLLABORATOR)
comments, and the repo-facts block's stated bug-report template,
contributing asks, and contribution/AI policy. In live mode: the live
issue thread and the repo's `CONTRIBUTING.md`/`AI_POLICY.md`.

What good looks like: when a maintainer has already stated a diagnosis, a
preferred approach, or asked for something specific (testing a patch,
confirming a hypothesis), the comment says so and engages it — even to
knowingly diverge — rather than writing as if the thread were empty. On
policy: read for an explicit disclosure mandate — words like "must be
disclosed," "state the tool used" — and fail only when that mandate exists
and the comment doesn't disclose. A policy that only asks for
understanding/responsibility for changes, or silence, does not require
disclosure to pass.
