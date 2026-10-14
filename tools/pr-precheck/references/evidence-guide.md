# Evidence guide: where evidence lives in a PR package

## Plan fidelity (harness category: silent-drift)

Where it lives: the plan-context block's Plan section — its stated
cause, its in-scope/not-in-scope lines, and its named Files — read
against the candidate PR's full diff and its description's own fidelity
claims. In live mode: your `plan.md`, including any `## Deviations`
section (a deviation recorded there before the PR goes out is honest;
one that exists only in the diff is not), read against `git diff
main...HEAD` and the description in your `pr_draft.md`.

What good looks like: list every file the diff touches before judging
anything — a diff with three changed files should have three checked,
not one. Each one either appears in the plan's Files list, is a natural
and minimal companion to a listed file (a test file for a listed source
file), or is named in a deviation note. A new CLI flag, config option,
or other feature the plan's not-in-scope line named explicitly is drift
even when it sits in the same function or file as the real fix — scope
is about what changed, not about which file it changed in. A
description that says "exactly as planned" or "no changes beyond it" is
itself a claim this check grades: verify it against the enumerated
diff, don't take it as settling the question. A description that
under-claims relative to a disclosed, reasoned deviation is not drift;
read the deviation note and the description together.

## Test evidence (harness category: not-tested)

Where it lives: the candidate PR's test-evidence section, read against
every scenario the plan's test plan named and the repro evidence it
built on. In live mode: your `test_evidence.md`, read against your
`plan.md`'s test plan and your week-2 repro steps.

What good looks like: split the plan's test plan into two kinds of
named scenario before checking anything. A *primary* scenario
demonstrates the fix — one of the distinct failure modes or repaired
behaviors the plan said this change must address (a plan naming two
failure modes needs both shown, not just the one that happens to pass
easily). A *control* confirms something unrelated or pre-existing stays
unchanged (a different flag, an already-working case, a sibling
code path the fix shouldn't touch). Every primary scenario needs
something concrete: either a before/after transcript, or a specific
assertion in the diff's own test code that pins the exact expected
post-condition (`assert console.export_text(clear=False) == ""` is
shown evidence for that scenario on its own, once the test suite's
passing run confirms it holds) — one of these forms, not both, but not
bare prose with nothing to point to either. A test assertion only
counts when it exercises the same observable behavior the plan's test
plan named — the real affected command or public function, checked
against the real described symptom — not when it calls an internal
helper the fix introduced or extracted and checks only that helper's
return value: a unit test asserting a flag-derivation function returns
`--RAW-CONTROL-CHARS` is not evidence that paged output actually shows
colors under pwsh, when the plan's test plan specifically asked for that
end-to-end repro to be re-run. When the plan names a real-world repro
command, look for that command (or an equivalent that reaches the same
code path end to end), not a unit test of a piece it was refactored
into. A control is held to a
lighter standard: a plain stated claim that it's unchanged (`"the
no-ampersand control unchanged"`) is good enough on its own, as long as
nothing elsewhere in the package contradicts it and the overall test
suite passes — real PRs don't re-prove every regression guard with its
own transcript, and a rubric that demands one will reject clean, honest
work. Whichever kind of scenario, the shown output must exercise the
actual repaired code path (the exact repro command or input the
issue/plan named), not an adjacent path the fix never touched — that
distinction applies to primary scenarios, since a control is supposed to
exercise an unchanged path by definition. A repo's own required checks
(a test suite, a lint pass, a typecheck) are run and their outcome is
visible, even when the outcome is a known, disclosed failure — an
honestly reported failing check is evidence; silence about whether a
required check ran at all is not.

## Diff quality (harness category: unreviewable)

Where it lives: the unified diff itself, hunk by hunk, and the commit
message list. In live mode: `git diff main...HEAD` and `git log
main..HEAD --oneline` on your branch.

What good looks like: every hunk is either the fix itself or a directly
related test/doc/changelog addition — nothing a reviewer has to mentally
filter out. Debris has specific tells, not a vibe: a commented-out line
(even one), a function defined but never called, a `// TODO` or `// DBG`
left in, a block that reproduces a prior attempt in a comment "for
reference," or a hunk whose only effect is reformatting/re-indenting
code that didn't need to change. Commit messages like "wip," "fix,"
"fmt + cleanup" don't fail this check by themselves, but they're exactly
the pattern that correlates with debris still being present — read the
diff itself to confirm either way, don't grade the commit log alone.

## Standards and comms (harness category: standards-wall)

Where it lives: the repo-facts block's stated PR-template sections,
required files, and contribution/AI policy, read against the candidate
PR's title and description; the thread highlights' maintainer comments,
read against whether the description engages any explicit direction
there. In live mode: the repo's actual PR template file and
`docs/CONTRIBUTING.md`, read against your `pr_draft.md`.

What good looks like: every section the template asks for carries real,
specific content, not a restated heading or an unchecked box pretending
to be checked. When the template requires a specific file as part of
the fix (a changelog entry, a `whatsnew` entry for a bug fix) that file
is actually present in the diff — a description that says "changelog
added" with no such file in the enumerated diff is the same failure as
an empty section. On policy: read for an explicit disclosure mandate —
"must be disclosed," "state the tool used" — and fail only when that
mandate exists and the description doesn't disclose; a policy that only
asks for responsibility/understanding of changes, or a repo with no
stated policy, does not require disclosure to pass. (Whether the
description's own claims match the diff's actual contents is Plan
fidelity's question, not this one — this check is about the repo's
stated asks, not internal consistency.)
