# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/36

**Verdict output**

```
## issue-select: 3 candidates from codepath/pathreview-ai301-fa26-s1

Accepted, ranked by fit:

1. #36 — POST /reviews endpoint has no test for a profile with no ingested documents (api, tests, tier-1). Best fit: writing this test means tracing through the review service and ingestion-status logic you don't already know — closest match to "reading someone else's codebase cold" from your fit profile, more than a single-line patch would give you.
2. #61 — Health check DB probe fails under SQLAlchemy 2.x (api, tier-1). Strong fit on your production Python/REST background (you've hit this exact "wrap raw SQL in text()" class of bug before), but it's a narrower, more contained fix than #36.
3. #37 — API docs missing request body schema for two endpoints (docs, api, tier-1). Lowest fit: pure documentation, no code path to trace or tests to reason about — safest first PR, but least aligned with wanting the code-reading/PR-review experience.

Rejected: none — all three passed every required check.

[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/36",
    "checks": [
      {"name": "Repo is alive", "grade": "pass", "evidence": "last push 2026-09-16, not archived"},
      {"name": "Scope fits a newcomer", "grade": "pass", "evidence": "single named file (tests/unit/test_review_routes.py), one concrete test case, 2-3hr estimate given"},
      {"name": "Nobody is already on it", "grade": "pass", "evidence": "no assignee, 0 comments, no linked PR"},
      {"name": "Contribution policy allows AI-assisted work", "grade": "pass", "evidence": "no stated AI policy anywhere in the repo"},
      {"name": "Maintainer engagement is fast", "grade": "fail", "evidence": "0/5 sampled issues have a maintainer reply"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61",
    "checks": [
      {"name": "Repo is alive", "grade": "pass", "evidence": "last push 2026-09-16, not archived"},
      {"name": "Scope fits a newcomer", "grade": "pass", "evidence": "single-file fix, exact repro error quoted, good-first-issue label"},
      {"name": "Nobody is already on it", "grade": "pass", "evidence": "no assignee, 0 comments, no linked PR"},
      {"name": "Contribution policy allows AI-assisted work", "grade": "pass", "evidence": "no stated AI policy anywhere in the repo"},
      {"name": "Maintainer engagement is fast", "grade": "fail", "evidence": "0/5 sampled issues have a maintainer reply"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/37",
    "checks": [
      {"name": "Repo is alive", "grade": "pass", "evidence": "last push 2026-09-16, not archived"},
      {"name": "Scope fits a newcomer", "grade": "pass", "evidence": "two named endpoints, exact files listed, 2-3hr estimate given"},
      {"name": "Nobody is already on it", "grade": "pass", "evidence": "no assignee, 0 comments, no linked PR"},
      {"name": "Contribution policy allows AI-assisted work", "grade": "pass", "evidence": "no stated AI policy anywhere in the repo"},
      {"name": "Maintainer engagement is fast", "grade": "fail", "evidence": "0/5 sampled issues have a maintainer reply"}
    ],
    "verdict": "accept"
  }
]
```

**The verdict must record `accept` for this issue.** Choose an issue your own skill
accepts. If your skill rejects every candidate you try, that is a signal about your
rubric rather than about the issues: revise it and re-run — retries are unlimited and a
partial re-run costs about $0.20 — or run the skill on different candidates. Output
recording `reject` for the issue you chose earns no credit for this field.

---

## Eval iterations

Quote source text directly in each field below. Paraphrase does not satisfy them.

**Run history**

1. `agreement: 3/3 scored items` — smoke run, `--limit 3`, first draft of the rubric.
2. `agreement: 18/20 scored items  (bar: 18/20: PASS)` — first full run, same draft. Disagreed on `issue-09` (gold `accept`, my rubric said `reject`, failed check: `Scope fits a newcomer`) and `issue-20` (gold `reject`, my rubric said `accept`).
3. `agreement: 2/2 scored items` — `--only issue-09,issue-20` after tightening the "Scope fits a newcomer" check (required 2+ closed PRs instead of 1+ before calling it a pattern of abandoned attempts, and added a clause for unscoped feature requests with an undecided core detail).
4. `agreement: 19/20 scored items  (bar: 18/20: PASS)` — full confirming run, `--save-run eval-run.txt`. `issue-01` flipped to a false `reject` this run (grading is model-based and not fully deterministic run-to-run) on the new feature-request clause.
5. `agreement: 2/2 scored items` — `--only issue-01,issue-20` after narrowing that clause to require an explicit hedge like "TBD" or "possibly X, if needed" in the issue's own text, rather than just the absence of a maintainer comment.
6. `agreement: 20/20 scored items  (bar: 18/20: PASS)` — final confirming run, `--save-run eval-run.txt`. This is the run committed as `eval-run.txt`.

**Issue analysis**

Issue: `issue-09`. Gold label, quoted from `gold-labels.json`: `"verdict": "accept"`, `"note": "old but valid bounded feature; the 2022 claim is stale and the maintainer invited takers"`. My rubric's final decision on it: `accept`, matching gold.

Reasoning: `issue-09` (conda's "conda config clear option") is a 2018 feature request with exactly one closed linked PR (`conda/conda#11627`) and a 2022 claim comment ("I'd like to take a swing at this... Does it need to be assigned to me?") that a stale-bot later flagged with no further activity. My first rubric draft failed this issue on "Scope fits a newcomer" because that check's original wording flagged any multi-year-old issue with "one or more closed, unmerged PRs in its history" as a sign of real difficulty — one stale PR from a since-abandoned attempt tripped that wording even though gold treats it as fine. I changed the rubric text to require "two or more closed, unmerged PRs... one closed PR alone is not a pattern," which stopped penalizing `issue-09`'s single old attempt while still failing `issue-15` (two closed PRs, `zulip/zulip#20840` and `#23123`, plus years of claim/unclaim churn), which gold rejects on scope grounds.

**Check rationale**

Check, quoted as currently written in `rubric.md`:

> **Scope fits a newcomer** | The issue title and body, and the comment thread (eval bundle text, or the live issue page and thread) | Fail if any of: the issue is a self-described tracking/umbrella issue (title or body says "megaissue"/"tracking", or the body is mainly a list of many other issue numbers to pick from rather than one task); the thread shows the design is still being actively debated with no maintainer decision settling it; a maintainer states the fix needs changes to core internals with no concrete plan given; the issue has been open for multiple years and has two or more closed, unmerged PRs in its history (repeated abandoned attempts — one closed PR alone is not a pattern); it is a pure usage/support question with no code change implied; or it is a feature/enhancement request whose own text explicitly flags a core piece of the ask as undecided (words like "TBD", "not identified yet", or a hedge like "possibly X, if needed" about what the change even is) and no maintainer comment resolves it. The absence of a maintainer comment or a "good first issue" label is not by itself a fail, and neither is a large task, as long as every file/section/change it asks for is spelled out explicitly (no undecided piece). A terse body, a bug report without repro steps, or an acceptance-criteria checklist does not by itself fail this check. Otherwise pass. | required

Reasoning: this check carries three of the four "first-issue killer" families from the lecture (scope, plus age/difficulty signals that overlap with claim history) in one place, because they all show up in the same evidence (the issue body and thread) rather than the repo-facts block. The "two or more closed PRs" threshold and the explicit-hedge-word requirement for feature requests both exist specifically to stop the check from failing an issue on a single weak, incidental signal — they were added after eval disagreements showed the looser wording (any closed PR; any unendorsed feature request) over-rejected clean issues like `issue-09` and, transiently, `issue-01`.

**Trade-offs**

The clause requiring an explicit hedge word ("TBD", "not identified yet", "possibly X, if needed") is what let `issue-20` (excalidraw's "Add company logo shape to the toolbar," rejected by gold as "one-line feature wish with no spec and a product decision hiding inside") fail correctly, because its body contains "Logo asset TBD" and "possibly app wiring in `excalidraw-app` if needed." But that same looser wording, before I added the explicit-hedge requirement, also mis-flagged `issue-01` (conda's fully-itemized docs task) as unscoped in one run — the grading model read its line "Consider a global `troubleshooting.rst` entry. Lower priority, but worth naming." as an admittedly-unclear surface, even though every other piece of that task names an exact file and section. I re-ran `--only issue-01,issue-20` after tightening the clause to require the explicit hedge words and confirmed both graded correctly. What this trades off going forward: the check still asks the grading model to judge natural-language hedging rather than matching a fixed keyword list, so a differently-worded but equally unscoped feature request could still land on either side of the line between runs. I accepted that risk rather than hard-coding a keyword list, because a keyword list would miss real cases like `issue-20`'s "possibly X, if needed" phrasing that don't use the word "TBD."

---

## Selection rationale

Graded on whether all three are answered, in your own words. Not on how good the
reasoning is, and not on length — a short honest answer to each earns the full marks.
This is also the basis for the claim comment you write in Unit 2.

**Selection rationale**

1. Fit to interests and time available: I've spent 2.5 years writing Python/REST backends professionally, so a FastAPI test-writing task is squarely in what I already do at work, and the issue's own 2-3 hour estimate fits what I can commit to as a first open-source PR without over-committing. It's also the one candidate of the three that isn't just a mechanical patch or a docs edit, which matches wanting to actually get better at the contribution loop rather than picking the easiest possible ticket.

2. What the verdict identified correctly, and what I weighed beyond it: the rubric correctly confirmed the mechanical facts — repo active as of last week, nobody assigned or commenting, no AI-contribution ban, and a bounded single-file ask with an effort estimate already given by the maintainer. What it couldn't weigh is depth of learning: #36, #61, and #37 all passed the identical required checks, so the rubric alone gives no reason to prefer one over the other. I picked #36 over the equally-accepted #61 (a narrower one-line SQLAlchemy fix) and #37 (a pure docs edit) because writing a new test forces me to read the review service and how ingested-document state is modeled, which the rubric's checks don't and structurally can't measure.

3. Anticipated difficulty in claiming it: technically, low-to-moderate — the fix path is clear (add a test in `tests/unit/test_review_routes.py`), but there's no existing example in the thread of how "no ingested documents" is represented for a profile, so I'll need to read the ingestion and review-service code myself rather than copy a pattern from a comment. Socially, close to zero: no assignee or comments yet, though the repo-wide sample I checked (5 other issues, all comments from students, none from a maintainer) tells me not to expect maintainer feedback before I open the PR, and the Path Review house rule means a classmate could still claim it in parallel at no cost to me.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
