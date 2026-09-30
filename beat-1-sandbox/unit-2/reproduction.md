# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy
kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**GitHub username**

Kunalkrk

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/36#issuecomment-5902981372

````
Hi, I would like to take this one as my first contribution to Path Review.

Plan: set up the repo locally, trace how a profile's ingestion status is checked in the review routes, and write a test in `tests/unit/test_review_routes.py` covering a profile with no ingested documents against `POST /reviews`, verifying it returns an appropriate error instead of crashing. 
I'll post my reproduction (the current behavior, plus the passing test) here once I have it.
````

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/36#issuecomment-5903153178

````
Environment: Python 3.11 (venv), pathreview installed via `pip install -e ".[dev]"`
from a fresh fork/clone, Windows 11 (Git Bash per the repo's own `docs/SETUP.md`).
No Docker/Postgres/Redis running: `tests/unit` is marked in `pyproject.toml` as
"fast, no external dependencies," and this stayed true — the whole suite,
including the new test, ran against a mocked DB session with no backing
services.

Steps:

```
$ git clone https://github.com/Kunalkrk/pathreview-ai301.git
$ cd pathreview-ai301
$ python -m venv .venv
$ .venv/Scripts/pip install -e ".[dev]" -q
$ .venv/Scripts/pytest tests/unit/test_review_routes.py -v -m unit
```

I added `tests/unit/test_review_routes.py`, using an HTTP-level `TestClient`
against `api.main.app` (overriding `get_current_user` and `get_db`, the same
way `tests/unit/test_review_service.py` mocks the DB layer, since no route-level
test existed yet in this file). Before writing an assertion, I ran the actual
call twice: once with an arbitrary `profile_id` and once more as a control,
to see whether `POST /reviews` behaves any differently for a profile with no
ingested documents versus any other profile. This isn't pushed anywhere yet,
so here's the file in full to make this reproducible without it:

```python
"""Tests for the /reviews HTTP routes (api/routes/reviews.py)."""

from datetime import datetime
from unittest.mock import AsyncMock, Mock, patch
from uuid import uuid4

import pytest
from fastapi.testclient import TestClient

from api.main import app
from api.middleware.auth import get_current_user
from core.database import get_db
from core.models.user import User


@pytest.mark.unit
class TestReviewRoutes:
    """Test suite for POST /reviews."""

    @pytest.fixture
    def fake_user(self):
        return User(id=str(uuid4()), email="test@example.com", hashed_password="x")

    @pytest.fixture
    def client(self, fake_user):
        async def _override_get_current_user():
            return fake_user

        async def _override_get_db():
            session = AsyncMock()
            yield session

        app.dependency_overrides[get_current_user] = _override_get_current_user
        app.dependency_overrides[get_db] = _override_get_db
        try:
            yield TestClient(app, raise_server_exceptions=False)
        finally:
            app.dependency_overrides.clear()

    @pytest.fixture
    def mock_review_class(self):
        """Patch review_service.Review so db.refresh() doesn't need a real DB
        round trip to populate id/created_at/updated_at."""
        with patch("core.services.review_service.Review") as MockReview:
            review = Mock()
            review.id = uuid4()
            review.profile_id = uuid4()
            review.status = "pending"
            review.sections = None
            review.overall_score = None
            review.error_message = None
            review.created_at = datetime.utcnow()
            review.updated_at = datetime.utcnow()
            MockReview.return_value = review
            yield MockReview

    @pytest.mark.xfail(
        strict=True,
        reason="issue #36: POST /reviews does not check whether the profile "
        "has any ingested documents before accepting the review request; it "
        "returns 200/pending for an empty profile exactly as it would for one "
        "with content, so there is no error for this test to confirm yet",
    )
    def test_create_review_for_profile_with_no_ingested_documents_returns_error(
        self, client, mock_review_class
    ):
        """POST /reviews for a profile with no ingested documents should
        return a client error, not silently accept the review."""
        profile_id = str(uuid4())

        resp = client.post("/reviews", json={"profile_id": profile_id})

        assert resp.status_code >= 400, (
            f"expected an error for a profile with no ingested documents, "
            f"got {resp.status_code}: {resp.text}"
        )

    def test_create_review_currently_returns_200_regardless_of_ingestion_state(
        self, client, mock_review_class
    ):
        """Documents today's actual behavior: the endpoint accepts the
        request and returns pending status without inspecting ingestion
        state at all. This passes today; it should stop passing once #36
        adds the validation, at which point it (not the xfail above) is the
        one to update."""
        profile_id = str(uuid4())

        resp = client.post("/reviews", json={"profile_id": profile_id})

        assert resp.status_code == 200
        assert resp.json()["status"] == "pending"
```

Observed (verbatim, from a `TestClient` call against a profile with no
ingested documents, and a second arbitrary profile as a control):

```
=== Case: profile with no ingested documents ===
status: 200
body: {"id":"7150e1b1-3221-4e04-95c8-4d0559e4ff75","profile_id":"b0b97b31-e59c-478c-955a-13d3bf3a2608","status":"pending","sections":null,"overall_score":null,"error_message":null,"created_at":"2026-09-30T02:48:19.442605","updated_at":"2026-09-30T02:48:19.442605"}

=== Case: a second, arbitrary profile_id (control) ===
status: 200
body: {"id":"1c7e113d-0ec2-4cb6-955a-b571b75a0343","profile_id":"02a4c857-6c93-4163-9e57-378e75395db9","status":"pending","sections":null,"overall_score":null,"error_message":null,"created_at":"2026-09-30T02:48:19.479608","updated_at":"2026-09-30T02:48:19.479608"}
```

Actual: the endpoint returns 200 and creates a `pending` review no matter
what the profile looks like. Reading `api/routes/reviews.py` and
`core/services/review_service.py` explains why: `create_review` only ever
does `Review(profile_id=..., status="pending", ...)`, `db.add`, `db.commit`,
`db.refresh` — it never queries `IngestedSource` or checks the profile's
`github_username`/`portfolio_url`/`resume_text` fields. That check only
happens later, inside `process_review`, which runs in a background task
*after* the response is already sent, and even there the ingestion step
(`_run_ingestion_pipeline`) just returns an empty list rather than raising,
and the downstream agent/RAG steps are placeholder stubs that emit the same
two canned sections regardless of input.

Expected (per the issue): the endpoint returns an appropriate error for a
profile with no ingested documents, instead of crashing.

What I can confirm: it does not crash, but it also does not error — it
succeeds identically to a profile with content, so there is currently no
"appropriate error" behavior for a test to verify. That is a validation gap,
not a crash, and it is a gap I can show is missing rather than only assert
is missing.

The two tests in `tests/unit/test_review_routes.py` capture this precisely:
`test_create_review_currently_returns_200_regardless_of_ingestion_state`
passes today, documenting the actual behavior. Its companion,
`test_create_review_for_profile_with_no_ingested_documents_returns_error`,
is marked `xfail(strict=True)` (the same pattern the codebase already uses
for its other seeded issues, e.g. issue #65 in `test_review_service.py`) and
currently fails as expected — it asserts the desired error behavior, which
doesn't exist yet. Full run:

```
tests/unit/test_review_routes.py::TestReviewRoutes::test_create_review_for_profile_with_no_ingested_documents_returns_error XFAIL
tests/unit/test_review_routes.py::TestReviewRoutes::test_create_review_currently_returns_200_regardless_of_ingestion_state PASSED
1 passed, 1 xfailed
```

Full suite still green after adding these: `376 passed, 54 xfailed` (up from
375/53 on main), so nothing existing broke.

What I have not done yet: exercised this against a real Postgres-backed
`IngestedSource` table (only unit-level, mocked DB) or decided the actual
fix (a check in `create_review`/the route, probably a 422 if the profile has
no `IngestedSource` rows and none of the three source fields set). That's
next, once the claim comment's plan gets going.
````

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. `agreement: 2/3 scored items` — smoke run, `--limit 3`, first draft of the rubric. Disagreed on `pkg-03` (gold `accept`, my rubric said `reject`, failed check: `Claims match evidence`).
2. `agreement: 1/1 scored items` — `--only pkg-03,calib-02` (calib-02 is calibration, not scored) after loosening "Claims match evidence" to focus on the report's core claim rather than every supporting remark.
3. `agreement: 20/20 scored items  (bar: 18/20: PASS)` — first full run with the loosened check.
4. `agreement: 19/20 scored items  (bar: 18/20: PASS)` — confirming run, `--save-run eval-run.txt`. `pkg-01` flipped to a false `reject` on this run (model-grading noise, not a rubric gap — confirmed by re-running `--only pkg-01` alone immediately after and getting the correct `accept`).
5. `agreement: 20/20 scored items  (bar: 18/20: PASS)` — final confirming run, `--save-run eval-run.txt`. This is the run committed as `eval-run.txt`.

**Package analysis**

Package: `pkg-03` (`BurntSushi/ripgrep#2779`). Gold: `accept` (`clear-accept`). My rubric's first draft: `reject`, failed on `Claims match evidence`.

The report reproduces the issue's exact multiline `--replace` line-numbering bug on the current release, with the exact 12-line input file and exact command shown, output matching the issue's own "actual behavior" block exactly. It then adds one extra sentence: "Dropping `-r '$1'`... reports 1, 4, 7, 10 correctly, which matches the owner's note that `--replace` is required to trigger it" — stated in prose, with no separate command/output shown for that specific negative control. My first-draft check required *every* claim in the report to have its own shown artifact, so it failed this sentence as an unbacked assertion, even though the report's core claim (the bug reproduces on 15.2.0) was fully backed by a real, shown transcript. Gold treats the whole package as a clean accept — the negative control is corroborating color for a fact the maintainer (`BurntSushi`) already stated in the thread, not a fresh diagnosis the report is asking a reader to take on faith. I re-read the check against that gap and revised it to grade the report's *core* claim against its evidence, rather than treating every sentence as an independent claim needing its own transcript.

**Check rationale**

Check, quoted as currently written in `rubric.md`:

> **Claims match evidence** | The report's core claim — it reproduces the issue, it doesn't, or a stated cause/diagnosis — read against what the shown artifact actually demonstrates | Fail if the core claim has no artifact behind it anywhere in the report (an assertion, a diagnosis, a "guaranteed" or "definitely" with nothing shown), or the shown artifact doesn't support the specific claim made. An honest, evidenced cannot-reproduce — a real attempt shown, a plain statement that it didn't trigger, and a stated guess at what differed — is a pass; it does not need to have succeeded at reproducing the bug. A brief corroborating remark that doesn't carry the core claim on its own (e.g. noting that removing a flag the issue's own thread already named as required stops the failure) doesn't need its own separate transcript, as long as it's consistent with the main shown artifact and doesn't introduce a new, unverified diagnosis. Otherwise pass. | required

It reads this way because the first draft ("every claim needs an artifact") over-rejected `pkg-03`: real bug reports routinely include a quick, low-stakes corroborating remark ("and removing X fixes it") without re-running and re-pasting a whole second transcript for it, especially when that remark just confirms something a maintainer already said in the thread. Requiring a transcript for every such remark punished exactly the kind of terse-but-complete reporting the worksheet's `calib-01` package (lazygit) was supposed to anchor as fine. The revision keeps the strict standard for the claim that actually decides the verdict — no diagnosis or reproduction claim gets a pass on vibes — while not demanding proof-of-everything for incidental color.

**Trade-offs**

The loosened wording is what let `pkg-03` pass correctly, and it also means the check will tolerate a report whose one corroborating remark turns out to be inaccurate, as long as that remark isn't the thing carrying the verdict and doesn't contradict the main shown artifact — that's a real case this check will now miss (an inaccurate side-remark slipping through as harmless color). I accepted that gap deliberately: I re-ran `--only pkg-03,calib-02` as a canary before the confirming full run specifically so a real no-evidence package (`calib-02`, all assertion, no environment, no steps, no artifact at all) still failed the way it should under the loosened wording — it did — which is the check I have that the loosening didn't quietly let real no-evidence packages start passing too.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
