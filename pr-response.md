# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in the service and updated the route import and call site to match the project's `verb_to_noun` convention.
**How I verified:** Searched the entire project for `save_to_watchlist()`. The search found its definition in `services/watchlist_service.py` and its import/call in `routes/watchlist/watchlist.py`; both were updated. A follow-up search found no remaining code references to the old name.

## Comment 2 — Deduplication
**What I did:** Followed `add_to_collection()` in `services/collection_service.py`: its duplicate check searches for an existing entry matching both `user_id` and `film_id`, then raises `AlreadyInCollectionError` when found. `add_to_watchlist()` now checks for the same user/film pair and raises `AlreadyOnWatchlistError` for a duplicate; the route returns HTTP 409 Conflict.
**How I verified:** Added `test_add_to_watchlist_duplicate_raises()`, which adds the same film twice, asserts the second call raises `AlreadyOnWatchlistError`, and checks that only one row remains. `pytest tests/test_watchlist.py tests/test_collection.py` passed (5 tests); the full `pytest tests/ -v` suite also passed (6 tests).

## Comment 3 — Missing test
**What I did:** Added `test_add_to_watchlist_nonexistent_film_raises()` using the same in-memory app/sample-user fixtures, fake UUID, and `pytest.raises(FilmNotFoundError)` assertion as `test_add_to_collection_nonexistent_film_raises()`.
**How I verified:** Used `test_add_to_collection_nonexistent_film_raises()` in `tests/test_collection.py` as the model: same isolated in-memory app and sample-user fixtures, fake UUID, and `pytest.raises(FilmNotFoundError)` assertion. Ran `pytest tests/test_watchlist.py -v`; both watchlist tests passed, including the nonexistent-film case.

## Comment 4 — Default visibility
**My position:** Keep the existing `public=True` default and make it an intentional discoverability choice for a community film-tracking app.
**Reasoning:** Public watchlists let users share and discover what other members want to watch without extra setup. The current review request asks for the rationale to be documented; it does not ask to change the default.
**Tradeoff acknowledged:** A public default can expose a user's interests to others. Users who expect privacy may be surprised, so visibility should remain explicit in the API and a private-by-default policy would be preferable if privacy becomes the product priority.

## Comment 5 — Sort order
**My position:** Use newest-added-first ordering, matching the reviewer's preference.
**Reasoning:** Recently saved films are likely to be the most immediately relevant, and the collection service already uses descending `date_added` ordering.
**Engagement with reviewer's point:** This addresses the reviewer's point that users usually want to see recent additions first. Alphabetical ordering can aid browsing, but should not override the requested default; it could be offered later as an optional sort. The service still needs its ordering implementation updated to match this decision.

## Comment 6 — Rebase
**What conflicted:** WatchlistEntry.film_id was Integer on the feature branch but String(36) on main after the film-ID UUID refactor.
**How I resolved it:** Kept main's String(36) UUID type for the watchlist foreign key.
**How I verified no conflict remains:** Rebase completed successfully and the working tree has no unmerged paths. `Film.id`, `CollectionEntry.film_id`, and `WatchlistEntry.film_id` use UUID-compatible `String(36)` columns; the full test suite passed after the rebase and feature changes.

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
