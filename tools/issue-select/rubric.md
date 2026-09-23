# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Repo is alive | Repo-facts block: `archived:` flag, `last push to any branch` date, and the dates in `last 5 default-branch commits` (live mode: the repo front page and commit history) | Fail if `archived: yes`. Otherwise fail if `last push to any branch` (or the newest of the last 5 default-branch commits, whichever is more recent) is more than 180 days before the capture date (eval mode) or today (live mode). Otherwise pass. | required |
| Scope fits a newcomer | The issue title and body, and the comment thread (eval bundle text, or the live issue page and thread) | Fail if any of: the issue is a self-described tracking/umbrella issue (title or body says "megaissue"/"tracking", or the body is mainly a list of many other issue numbers to pick from rather than one task); the thread shows the design is still being actively debated with no maintainer decision settling it; a maintainer states the fix needs changes to core internals with no concrete plan given; the issue has been open for multiple years and has two or more closed, unmerged PRs in its history (repeated abandoned attempts — one closed PR alone is not a pattern); it is a pure usage/support question with no code change implied; or it is a feature/enhancement request whose own text explicitly flags a core piece of the ask as undecided (words like "TBD", "not identified yet", or a hedge like "possibly X, if needed" about what the change even is) and no maintainer comment resolves it. The absence of a maintainer comment or a "good first issue" label is not by itself a fail, and neither is a large task, as long as every file/section/change it asks for is spelled out explicitly (no undecided piece). A terse body, a bug report without repro steps, or an acceptance-criteria checklist does not by itself fail this check. Otherwise pass. | required |
| Nobody is already on it | Repo-facts `this issue: assignees:` and `linked PRs:` lines, plus claim comments in the thread ("I'll take this", "can I work on this", "working on this", `@...bot claim`) | Fail if assignees is non-empty, or any linked PR has state `open`, or the thread has a claim comment posted within 90 days of the capture date (eval) / today (live) that no maintainer has since marked available again. A claim comment older than 90 days with no follow-up activity, or a linked PR that is only `closed` or `merged`, does not fail this check. Otherwise pass. | required |
| Contribution policy allows AI-assisted work | Repo-facts `contribution policy` line (and any named AI policy file) | Fail only if the policy states an outright ban on AI-generated code or documentation. Disclosure requirements, "you must understand and test every change," human-review requirements, and similar conditions pass. No `CONTRIBUTING.md` or no stated policy passes. | required |
| Maintainer engagement is fast | Repo-facts `maintainer first-response sample` line, and any maintainer (owner/member/collaborator) reply in this issue's own thread | Pass if this issue's thread already has a maintainer reply, or at least 2 of the 5 sampled issues show a first maintainer response within 14 days. | preferred |

## Verdict rule

Accept only if all four `required` checks grade `pass`. If any required check
grades `fail` or `unclear`, the verdict is `reject` — an issue you cannot
verify is not a first issue you should take. `preferred` checks never change
the verdict; use them only to rank issues the required checks already
accepted (higher engagement ranks higher), and mention the deciding
`preferred` check in the summary for the top-ranked candidate.
