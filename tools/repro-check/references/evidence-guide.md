# Evidence guide: where proof lives in a reproduction package

## Environment

Where it lives: the repro report's own environment line(s) — usually near
the top, before the steps. In an eval bundle this is inside the "Candidate
repro report" section, not the repo-facts block (repo-facts describes the
issue's target repo, not what the reporter actually ran). In live mode,
this is the draft repro comment itself.

What good looks like: names the tool/library version and the OS at minimum;
also names whatever the specific bug turns out to hinge on (a driver, a
build profile, a shell, a locale) — read the issue first to know what that
is for this bug, since it's different every time. If the version differs
from the one the issue names, the report says so and says why it still
counts (tested current release, notes the delta) rather than leaving the
reader to notice a mismatch on their own.

## Steps

Where it lives: the repro report's numbered or shown steps, plus any config
file, input, or flags they depend on. In an eval bundle, this is the command
blocks and any inline config inside "Candidate repro report."

What good looks like: a stranger with the stated environment could run the
exact same steps and land in the exact same place — every flag, every input
value, every config file's relevant contents given, not referenced from
somewhere the reader can't reach. A step like "set up the project" or "use
our internal config" with nothing shown is not followable, even if the
reporter's own run was genuine. If the trigger needs a specific input shape
(a file count, a name length, a particular flag combination), the steps
show that shape, not a description of it.

## Behavior shown

Where it lives: the output excerpt, stack trace, log lines, or screenshot
inside "Candidate repro report," read side-by-side with the issue's own
description of the failure (its title, body, and any exact error text or
trigger conditions quoted there).

What good looks like: the artifact shows the same specific failure the
issue describes — same error type, same trigger conditions, same symptom —
not a different failure that happens to occur along the way (a syntax error
standing in for a panic, a different exception thrown by a modified input,
an old version's since-fixed bug standing in for the current one). Check
what actually produced the artifact (the exact command, the exact input)
against what the issue's own reproduction needed, not just whether the
final printed text sounds similar. A deviation (different version, shifted
input) is fine only when the report both says so and the shown behavior is
still the issue's specific failure, not an adjacent one.

## Honesty

Where it lives: the report's core claim — it reproduces the issue, it
doesn't, or a stated cause/diagnosis — read against whichever artifact in
"Behavior shown" is supposed to back that specific claim.

What good looks like: the core claim doesn't outrun its artifact. A core
claim with a shown artifact behind it that actually demonstrates it is a
pass. A core claim stated with confidence, urgency, or repetition
("guaranteed," "definitely," "verified," "every single time") but no
artifact shown anywhere is a fail regardless of how certain it sounds. An
honest cannot-reproduce is a pass, not a fail, when it shows a real attempt
(real commands run, real output shown) and states plainly what happened and
what the reporter thinks differed from the triggering conditions — it does
not need to have succeeded at triggering the bug to count as honest,
evidenced work. A short corroborating remark that isn't itself the core
claim (a quick "and removing the flag the thread already named stops it")
doesn't need its own separately shown transcript when it's consistent with
the main shown artifact and doesn't introduce a new diagnosis of its own —
only the claim the verdict actually rides on needs its proof shown.

## Comms

Where it lives: the claim comment's own wording, read against the repo's
stated bug-report template and contribution/AI policy in the repo-facts
block (live mode: the repo's `CONTRIBUTING.md`/`AI_POLICY.md` and any issue
template).

What good looks like: the claim comment states a specific, honest intent
(what the poster found, what they'll do next) rather than boilerplate
(generic praise, "please assign me," presumptuous claims to have the issue
"reserved") or an unverifiable promise (a guaranteed fix time). On policy:
read the repo's stated AI policy for an explicit disclosure mandate — words
like "must be disclosed," "state the tool used" — and fail only if that
mandate exists and neither comment discloses. A policy that only asks
contributors to understand or take responsibility for their changes, or a
repo with no stated AI policy at all, does not require disclosure to pass.
