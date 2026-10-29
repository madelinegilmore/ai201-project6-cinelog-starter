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
**My position:** Keep `public=True` as the default, but treat it as a deliberate product choice rather than an incidental model default.
**Reasoning:** This community film-tracking app should make it easy for users to share what they want to watch and discover films through other users' lists. A public default optimizes for users who add films to participate in that shared discovery, without making them configure visibility on every entry. The API should still allow an entry to be made private when a user wants that control.
**Tradeoff acknowledged:** A private-by-default policy would better protect users who consider their interests personal, and avoids exposing a list before the user makes an explicit choice. The cost is less immediate sharing and discoverability, plus added friction for users who intend to participate publicly. Because this feature is community-oriented, I favor public-by-default here, while recognizing that a privacy-first product should reverse that choice.

## Comment 5 — Sort order
**My position:** Use newest-added-first ordering, matching the reviewer's preference; `get_watchlist()` now orders entries by `date_added` descending.
**Reasoning:** Recently saved films are likely to be the most immediately relevant. This also keeps watchlist behavior consistent with `get_collection()`, which already returns entries newest first.
**Engagement with reviewer's point:** The reviewer noted that users usually want to see what they added recently. I agree that this is a better default than alphabetical order for a watchlist centered on films saved for later. Alphabetical ordering may still help users browse a large list, but it can be added later as an optional sort rather than overriding the recent-first default. Verified with `test_get_watchlist_returns_newest_first()`, which uses titles whose alphabetical order differs from their `date_added` order.

## Comment 6 — Rebase
**What conflicted:** During the earlier rebase onto the UUID refactor, `WatchlistEntry.film_id` conflicted: the feature side declared `Integer`, while main declared `String(36)` to match UUID-based `Film.id`.
**How I resolved it:** Kept main's UUID-compatible `String(36)` foreign key. For this review request, fetched `origin` and ran `git rebase origin/main`; it completed without replay conflicts because `origin/main` was already an ancestor of the feature branch. Then committed the remaining stale integer references separately: the service documents `film_id` as a UUID string and the route documents a UUID request value. Commit: `fix: update WatchlistEntry film_id to UUID after main branch refactor` (`692de28`).
**How I verified no conflict remains:** `Film.id`, `CollectionEntry.film_id`, and `WatchlistEntry.film_id` are UUID-compatible `String(36)` columns; the service and route no longer document integer film IDs. `git log --merges origin/main..HEAD` returned no merge commits. The full `pytest tests/ -v` suite passed after restoring the sort-order work (7 tests).

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
