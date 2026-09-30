# Rubric: is this reproduction package ready to post?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Environment recorded | The repro report's environment line(s), read against the issue's target version/OS/config and against whatever the bug's own behavior turns out to depend on (build profile, driver, platform) | Fail if the report gives no environment at all (no tool/library version, no OS), or omits a detail the issue itself shows the bug depends on (e.g. a Windows-specific issue naming a driver, a bug that only fires in one build profile). A version different from the issue's is fine only if the report says so explicitly (tested on the current release, notes the delta) rather than silently substituting an older or unverified version. Otherwise pass. | required |
| Steps followable | The repro report's steps and any config/input it relies on | Fail if a stranger with the stated environment could not run the same steps and land in the same place: a step that's missing a flag/input the trigger needs, a step that says "set up the project" with no commands, or a repro that depends on private/unshared resources (an internal repo, an unshared config) nobody else can access. Otherwise pass. | required |
| Behavior matches the issue | The artifact shown (output, trace, screenshot) compared line-by-line against the issue's own description of the failure — same trigger, same error/behavior, not a different one that merely looks similar | Fail if the shown artifact is a different failure than the one the issue describes (a graceful validation error standing in for a panic, a compile error standing in for a runtime crash, an old version's known bug standing in for the current one) even when the report narrates it as confirming the issue. A real, acknowledged version/config deviation that still reproduces the same specific behavior is fine. Otherwise pass. | required |
| Claims match evidence | The report's core claim — it reproduces the issue, it doesn't, or a stated cause/diagnosis — read against what the shown artifact actually demonstrates | Fail if the core claim has no artifact behind it anywhere in the report (an assertion, a diagnosis, a "guaranteed" or "definitely" with nothing shown), or the shown artifact doesn't support the specific claim made. An honest, evidenced cannot-reproduce — a real attempt shown, a plain statement that it didn't trigger, and a stated guess at what differed — is a pass; it does not need to have succeeded at reproducing the bug. A brief corroborating remark that doesn't carry the core claim on its own (e.g. noting that removing a flag the issue's own thread already named as required stops the failure) doesn't need its own separate transcript, as long as it's consistent with the main shown artifact and doesn't introduce a new, unverified diagnosis. Otherwise pass. | required |
| Comms respects the room | The claim comment's own wording, and the repo's stated bug-report template and contribution/AI policy from the repo-facts block | Fail if the claim comment is boilerplate or over-promises (generic flattery, an unverifiable guarantee like a fixed delivery time, presumptuously asking to "reserve" the issue) rather than stating a specific, honest intent; or if the repo's stated policy contains an explicit AI-disclosure mandate (it says AI use must be disclosed / the tool and extent stated) and neither the claim comment nor the repro report discloses it. A policy that only asks contributors to understand/take responsibility for changes, without an explicit disclosure mandate, does not fail on silence — and neither does a repo with no stated AI policy at all. Otherwise pass. | required |

## Verdict rule

Accept (ready to post) only if all five `required` checks grade `pass`. Any
required check graded `fail` or `unclear` makes the verdict `reject` (hold) —
proof you cannot verify, or a claim you cannot back, is not ready to post.
There are no `preferred` checks this week; every family the lecture named
gates the verdict, because a package that's weak on any one of them is not
ready regardless of how strong the rest is.

On a claim-only draft (no repro report yet), "Behavior matches the issue" and
"Claims match evidence" report `unclear`/`not yet applicable` per `SKILL.md`
and are excluded from the verdict; the verdict then answers only whether the
claim comment itself is ready, using "Comms respects the room" (and
"Environment recorded"/"Steps followable" if the draft already states them).
