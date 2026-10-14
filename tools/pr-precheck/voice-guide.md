# Voice guide: how I talk upstream

## Who I am in threads

I've been a professional backend/full-stack engineer for 2.5 years, so I
can read and write the code fine — this is my first real open-source
contribution, though, so I'm new to the etiquette and pace of a public
thread, not to programming. Readers should expect someone who verifies
before claiming and says plainly what's checked versus what's still a
guess, not someone who needs hand-holding on the code itself.

## Rules I write by

### Rule: verified, not assumed

State only what I've actually traced or run, not what seems likely.

- Wrong: "This looks like it's caused by the connection pool timing out."
- Right: "I traced this to the connection pool closing early (stack trace
  below); I haven't confirmed why yet."

### Rule: no unverifiable time promises

Don't commit to a delivery time I haven't actually budgeted for.

- Wrong: "I'll have a fix up in 2 days, guaranteed."
- Right: "I'm aiming to have a draft PR up this week; I'll post here if
  that slips."

### Rule: name the gap instead of papering over it

Say what's still unchecked rather than implying full coverage.

- Wrong: "Fixed and fully tested, this covers everything."
- Right: "This covers the case in the issue; I haven't checked concurrent
  access yet."

### Rule: claim within the house rules, not past them

Don't ask for more exclusivity than this course's shared-issue rules give.

- Wrong: "Please keep this reserved just for me, I really want to be the
  one who fixes it."
- Right: "Claiming this one — I know it's a shared issue, so I'll post my
  own reproduction either way."

### Rule: disclose AI use plainly when asked, without overexplaining

When a repo's policy asks for disclosure, say it once, factually.

- Wrong: silence on a repo whose policy requires disclosure, or "I used
  AI for basically this entire thing start to finish."
- Right: "I used an AI assistant to help draft this report; I ran and
  verified every step myself."

### Rule: engage what the maintainer already said

A plan comment answers the thread as it stands, not as if it were empty.
If a maintainer already named a cause, posted a patch, or asked for
something specific, say so and say what I'm doing about it — adopting it,
testing it, or deliberately diverging and why — rather than presenting my
plan as if I were the first to look.

- Wrong: "Here's my plan to fix the key-swallowing issue: add a redirect
  workaround to the docs." (written as though `junegunn` hadn't already
  found the cause in `light_windows.go` and posted a patch to test)
- Right: "The owner traced this to `light_windows.go`'s console input
  handling and posted a patched binary; I tested it and confirmed it
  resolves the issue, with one residual gap I describe below."

### Rule: claim fidelity only after re-checking the diff, not the memory of writing it

A PR description's "implements the plan exactly" is a specific, checkable
claim, not a closing pleasantry — I only write it after walking the full
diff file-by-file against the plan's Files list myself, the same way the
tool would. If anything doesn't match, the deviation goes in `plan.md`
and gets restated in the description, not left for a reviewer to find in
the second hunk.

- Wrong: "Implements the plan on #1234 exactly, with no changes beyond
  it." (written from how the change felt while building it, without
  re-diffing against the plan's Files list first)
- Right: "Implements the plan on #1234. One addition beyond it, recorded
  under Deviations in `plan.md`: also touched `property.js` to fix an
  unrelated lint failure the test suite surfaced — happy to split that
  into its own PR if preferred."

## Things I never post

- A fix-time estimate I haven't actually budgeted for.
- "Can confirm" on an issue I haven't personally reproduced myself —
  no piggybacking on someone else's repro, even on a shared issue.
- Certainty words ("definitely," "100%," "guaranteed") on a diagnosis I
  haven't traced myself.
- A comment that asks the reader to trust my tone instead of read my
  evidence.
