# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

Kunalkrk

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/36#issuecomment-6029818828

````
Plan based on my reproduction above: `create_review_endpoint`
(`api/routes/reviews.py`) and `create_review`
(`core/services/review_service.py`) never fetch the `Profile` or check
its content fields, so `POST /reviews` returns `200`/`pending` for a
profile with nothing ingested exactly as it would for one with content —
no crash, but no validation either.

Fix: fetch the profile via the existing `profile_service.get_profile`
before creating the review, and reject with `422` if none of
`github_username`, `portfolio_url`, or `resume_text` are set (matching
the `422` precedent already used in `create_profile_endpoint` for an
unprocessable resume file). I'm checking the profile's own fields rather
than the `IngestedSource` table, since `IngestedSource` is empty for
every profile before its first review finishes processing — checking it
would reject the first legitimate request too.

One side effect worth flagging: `get_profile` already filters by owner,
so this incidentally closes a separate, pre-existing gap where
`create_review` never checked that the profile belongs to the requesting
user. I'm not setting out to fix that as its own thing — it's just what
reusing the existing helper gets me for free — and I'll double check
nothing else depends on the old behavior before the PR.

Test plan: re-run my week-2 repro (quoted in full in `plan.md`) against
the change — an empty profile should now get `422` instead of `200`, and
a profile with content should still get `200` as today. The `xfail` I
added to `tests/unit/test_review_routes.py` in week 2 should start
passing for real once this lands, and I'll rewrite its sibling test into
a has-content control.

Full plan, including the scope boundary and the things I'm deliberately
not touching (the background pipeline's placeholder logic, and not
treating the ownership-check side effect as its own fix), is in
`plan.md` on my branch. Will post here once the PR is up.
````

---

## Your branch

**Branch**

fix/36-profile-ingestion-check

**Evidence**

Before (quoted verbatim from my week-2 repro comment, profile with no ingested documents):

````
$ git clone https://github.com/Kunalkrk/pathreview-ai301.git
$ cd pathreview-ai301
$ python -m venv .venv
$ .venv/Scripts/pip install -e ".[dev]" -q
$ .venv/Scripts/pytest tests/unit/test_review_routes.py -v -m unit

=== Case: profile with no ingested documents ===
status: 200
body: {"id":"7150e1b1-3221-4e04-95c8-4d0559e4ff75","profile_id":"b0b97b31-e59c-478c-955a-13d3bf3a2608","status":"pending","sections":null,"overall_score":null,"error_message":null,"created_at":"2026-09-30T02:48:19.442605","updated_at":"2026-09-30T02:48:19.442605"}
````

After (re-ran the same scenario against the built change, on `fix/36-profile-ingestion-check`):

````
$ .venv/Scripts/python _verify_fix.py

=== AFTER FIX: profile with no ingested documents ===
status: 422
body: {"detail":"Profile has no content to review — add a GitHub username, portfolio URL, or resume first"}

=== AFTER FIX: profile WITH content (control) ===
status: 200
body: {"id":"f7edb7e8-5ba1-475c-b313-36505a74e9f7","profile_id":"d65801af-e58d-4feb-8934-1fe679f44893","status":"pending","sections":null,"overall_score":null,"error_message":null,"created_at":"2026-10-07T02:50:18.492125","updated_at":"2026-10-07T02:50:18.492125"}
````

Full suite after the change, confirming nothing else broke:

````
$ .venv/Scripts/pytest tests/unit -m unit
378 passed, 53 xfailed
````

(376 passed/54 xfailed before this branch; the `xfail` on
`test_create_review_for_profile_with_no_ingested_documents_returns_error`
is now a real pass, and two new tests — the has-content control and a
profile-not-found 404 case — are both green. `ruff check` and `mypy` on
the two changed files are clean.)

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. `agreement: 3/3 scored items` — smoke run, `--limit 3`, first draft of the rubric/procedure/evidence guide.
2. `agreement: 19/20 scored items  (bar: 18/20: PASS)` — first full run. Disagreed on `pkg-14` (gold `accept`, my rubric said `reject`, failed checks: `Executability`, `Honesty`).
3. `agreement: 4/4 scored items` — `--only pkg-14,pkg-17,pkg-18,pkg-16` after revising Executability (to distinguish a named area with an already-demonstrated localization method from an undecided choice) and Honesty (to stop re-grading a cause the Diagnosis check already passed as grounded). `pkg-17`/`pkg-18` (unbuildable) and `pkg-16` (wrong-cause) were canaries to confirm the loosening didn't let real rejects through.
4. `agreement: 20/20 scored items  (bar: 18/20: PASS)` — final confirming run, `--save-run eval-run.txt`. This is the run committed as `eval-run.txt`.

**Package analysis**

Package: `pkg-14` (`zellij-org/zellij#5174`). Gold: `accept` (`clear-accept`). My rubric's first draft: `reject`, failed on `Executability` and `Honesty`.

The plan diagnoses an OSC-color-leak-on-reattach bug, grounded in a real regression-window control (0.44.1 clean, 0.44.2+ leaking) and names the area precisely — the reattach handshake in `zellij-server`'s client connection handling and `zellij-client`'s terminal query issuance — but says "exact functions to be pinned in the PR after tracing the query issuance with debug logs, which I have working." My first-draft Executability check read "exact functions to be pinned... after tracing" as an undecided choice left open, the same shape as the actually-unbuildable packages ("not sure which layer," "whichever is easier"), and failed it. My first-draft Honesty check separately flagged "predates the reattach-path change in 0.44.2" as an unflagged certainty — but that claim is exactly what the 0.44.1-vs-0.44.2 control in the repro evidence supports, and my Diagnosis check had already (correctly) passed it as grounded. Two different checks were both, in different ways, punishing a plan that had actually done real investigative work (a debug trace already run, a version-bisected regression) because it hadn't also pre-committed to a single function name. I revised Executability to treat a named area plus an already-demonstrated localization method as executable, and revised Honesty to stop re-opening ground the Diagnosis check already settled.

**Check rationale**

Check, quoted as currently written in `rubric.md`:

> **Executability** | The plan's named files/areas and its approach or ordered steps | Fail if a stranger could not start without first deciding something the plan left open: no file or area named, an approach described as "look into X," "upstream or vendored, whichever is easier," or "maybe also check," or a choice between stated options or layers left unmade. A named area (even spanning a couple of related modules) whose exact function is still being pinned down does not fail this on its own when the plan names a concrete, already-in-hand method for finding it (a debug trace already run and described, not "I'll investigate") — that is normal for a plan written before the PR, not an open choice. Otherwise pass. | required

It reads this way because the first draft conflated two different things under one phrase, "exact location not yet named": a plan that genuinely doesn't know which of several approaches or layers to take (the real unbuildable packages: `calib-02`'s "poke around," `pkg-17`'s "gocui? tcell? not sure," `pkg-18`'s "whichever is easier") versus a plan that has already localized the bug to a specific area using a concrete method and is naming its one remaining concrete step. Only the first is actually unexecutable; the second is just honest about not having opened the PR yet. The revision keeps failing every plan in the first group while no longer failing `pkg-14`, which belongs in the second.

**Trade-offs**

The loosened wording trusts a plan's stated claim that it has "a debug trace already working" without requiring that trace to be pasted into the plan itself — unlike my Unit 2 "Claims match evidence" check, this rubric doesn't demand the method's output be shown, only that the method be named concretely. That's a real case this check will now miss: a plan that falsely claims to have already localized a bug with a method it hasn't actually run would pass here, where a stricter version might catch it by demanding the trace be shown. I accepted that gap because requiring every in-progress investigative claim to be fully evidenced inside the plan itself would reject exactly the honest, still-narrowing-down plans (like `pkg-14`) this check exists to accept — and I verified the loosening doesn't buy back the real unbuildable packages for free: `pkg-17` and `pkg-18`, re-run as canaries alongside `pkg-14`, both still correctly reject, because neither names a method at all, concrete or not.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
