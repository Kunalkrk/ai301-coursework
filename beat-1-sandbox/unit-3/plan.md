# Plan: issue #36 — `POST /reviews` has no test for a profile with no ingested documents

## Diagnosis

From my week-2 repro comment on the issue:

> Actual: the endpoint returns 200 and creates a `pending` review no matter
> what the profile looks like. Reading `api/routes/reviews.py` and
> `core/services/review_service.py` explains why: `create_review` only ever
> does `Review(profile_id=..., status="pending", ...)`, `db.add`, `db.commit`,
> `db.refresh` — it never queries `IngestedSource` or checks the profile's
> `github_username`/`portfolio_url`/`resume_text` fields.

Confirmed with two verbatim `TestClient` calls (same repro comment): a
profile with no ingested documents and an arbitrary control profile both
return `200 {"status": "pending", ...}`. Neither `create_review_endpoint`
(`api/routes/reviews.py`) nor `create_review`
(`core/services/review_service.py`) ever fetches the `Profile` row or
checks whether it has any content — the only place ingestion state is
ever touched is `process_review`'s background `_run_ingestion_pipeline`,
which runs *after* the response is already sent and never raises even on
an empty profile (it just returns `[]`). So the gap is exactly where I
found it in week 2: no synchronous validation exists for "does this
profile have anything to review," anywhere before the 200 goes out.

## Scope

**In scope:** add a synchronous check in `create_review_endpoint` that
rejects a request for a profile with no ingested content, before a
`Review` row is created.

**What "no ingested content" means here, stated as an assumption:** none
of `github_username`, `portfolio_url`, or `resume_text` set on the
`Profile`. I'm checking the profile's source fields, not the
`IngestedSource` table, because `IngestedSource` rows are only ever
created inside `process_review` — they're empty for *every* profile
before its first review finishes processing, including profiles that are
perfectly fine to review. Checking `IngestedSource` would reject the
first legitimate request for any profile. The issue's own wording ("a
profile exists but has no associated ingested content") is ambiguous
between these two readings; I'm going with the one that doesn't produce
false rejections on valid first requests.

**Not in scope:**
- `process_review`'s background pipeline itself (the placeholder
  `_run_agent_orchestration`/`_run_rag_retrieval_generation` stubs that
  generate the same canned sections regardless of input). That's a
  separate, bigger problem than this issue asks about.
- A missing-ownership check on `create_review` — `create_review_endpoint`
  currently never verifies the `profile_id` belongs to `current_user`
  before creating a review for it. My fix happens to close this
  incidentally (see Deviations/risk below), but I'm not setting out to
  fix it as its own thing; it's a pre-existing gap the issue doesn't
  mention, and treating it as this PR's job would be scope creep.
- Any change to how `IngestedSource` rows themselves get created.

## Files

- `api/routes/reviews.py` — `create_review_endpoint`: fetch the profile
  via `profile_service.get_profile` before calling `create_review`;
  raise `HTTPException(404)` if not found/not owned (reusing the same
  pattern already used in `get_review_endpoint`), and
  `HTTPException(422)` if the profile has none of the three content
  fields set (the same status code `create_profile_endpoint` already
  uses for an unprocessable request — e.g. a bad resume file type).
- `tests/unit/test_review_routes.py` — the file I added in week 2.
  Remove the `xfail` marker from
  `test_create_review_for_profile_with_no_ingested_documents_returns_error`
  (it should genuinely pass once the check exists), and rewrite
  `test_create_review_currently_returns_200_regardless_of_ingestion_state`
  into a control test for a profile that *does* have content (since 200
  will no longer be the actual behavior for an empty profile).

## Approach

1. In `api/routes/reviews.py`, import `get_profile` from
   `core.services.profile_service` (already used the same way in
   `api/routes/profiles.py`).
2. In `create_review_endpoint`, before the existing `create_review(...)`
   call: `profile = await get_profile(db, data.profile_id, current_user.id)`.
3. If `profile` is `None`, raise `HTTPException(404, "Profile not found")`.
4. If `not (profile.github_username or profile.portfolio_url or profile.resume_text)`,
   raise `HTTPException(422, "Profile has no content to review — add a GitHub username, portfolio URL, or resume first")`.
5. Otherwise, proceed exactly as today (call `create_review`, schedule
   `process_review`, return the `ReviewResponse`).
6. Update `tests/unit/test_review_routes.py` per the Files section above,
   adding a `mock_profile` fixture (patterned on the existing
   `mock_review_class` fixture) covering both the empty-profile and
   has-content cases, with `get_profile` overridden/patched the same way
   `get_current_user` and `get_db` already are.

## Test plan

Re-running my week-2 repro steps against the change:

- **Before (week 2, quoted above):** `POST /reviews` with a profile that
  has none of the three content fields set → `200`, review created with
  `status: "pending"`.
- **After (expected):** the same request → `422`, no `Review` row
  created, with a message naming the missing content.
- **Control (expected unchanged):** the same request for a profile with
  at least one of the three fields set → still `200`, `status: "pending"`,
  same as today.

In the test file: remove the `xfail` from
`test_create_review_for_profile_with_no_ingested_documents_returns_error`
and confirm it passes on its own (currently it's expected to fail, by
design, until this fix lands); rewrite the second test to assert `200`
for a profile constructed with `github_username` set, so the control case
stays covered. Run `pytest tests/unit -v -m unit` and confirm the full
suite is still green with no `xfail` left unresolved for this issue.

## Risks and unknowns

- **Status code choice (422 vs. 400):** I'm matching the existing
  precedent in `create_profile_endpoint` (422 for an unprocessable
  resume file type), but I haven't checked whether the frontend has any
  specific handling tied to 422 vs. 400 for this flow — flagging for
  review rather than assuming it doesn't matter.
- **Incidental ownership fix:** because `get_profile` already filters by
  `user_id`, a `profile_id` that exists but belongs to a different user
  will now 404 instead of silently creating a review for someone else's
  profile. I believe this is strictly a safety improvement, but I
  haven't verified there's no code or test elsewhere relying on the old
  (no ownership check) behavior — I'll check for that before opening
  the PR, not assuming it's fine.
- **Edge case not handled:** a profile whose content fields were later
  cleared (e.g. `resume_text` unset after a successful review already
  ran and populated `IngestedSource`) would now be rejected by my check
  even though `IngestedSource` rows exist for it. I think this is rare
  enough and arguably still reasonable (nothing *current* to review) to
  leave alone, but I haven't confirmed anything in the codebase exercises
  that path.

## Deviations

Everything in Approach and Test plan landed as planned: the check in
`create_review_endpoint` (404 for missing/not-owned profile, 422 for no
content), the `xfail` removed from
`test_create_review_for_profile_with_no_ingested_documents_returns_error`
(it now passes for real, asserting `422` specifically rather than just
`>= 400`), and the sibling test rewritten into
`test_create_review_for_profile_with_content_returns_200` as the
has-content control.

One addition beyond the plan: a third test,
`test_create_review_for_nonexistent_profile_returns_404`. The plan
named the 404 branch as part of the approach but didn't put it in the
test plan — once I'd written the fixture to mock a missing profile for
the 422 test's sibling case, covering the 404 branch the same way cost
nothing extra, so I added it rather than leave a new code path (and a new
possible regression surface) untested. Full suite stays green:
`378 passed, 53 xfailed` (up from 376/54 before this branch), lint
(`ruff check`) and typecheck (`mypy`) both clean on the two changed
files.

The ownership-check side effect flagged as a risk in the plan held as
expected: nothing else in the test suite or the route layer assumed the
old no-ownership-check behavior, so no additional changes were needed to
account for it.

No other deviations. The posted plan comment remains accurate; no
follow-up correction comment needed on the issue.
