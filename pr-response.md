## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to match the project's `verb_to_noun` naming convention (same pattern as `add_to_collection()`). Updated the one call site in `routes/watchlist/watchlist.py` (import and function call).
**How I verified:** Ran `grep -rn "save_to_watchlist" . --include="*.py"` across the repo to confirm no references to the old name remained. Ran the full test suite (`pytest tests/ -v`) to confirm nothing broke.

## Comment 2 — Deduplication
**What I did:** Added an `AlreadyOnWatchlistError` exception and a check in `add_to_watchlist()` that queries for an existing `WatchlistEntry` matching the given `user_id` and `film_id` before creating a new one, raising the exception if one is found. Followed the same pattern as `add_to_collection()`'s handling of `AlreadyInCollectionError`.
**How I verified:** Wrote `test_add_to_watchlist_duplicate_raises` in `tests/test_watchlist.py`, which adds a film to a user's watchlist, attempts to add it again, asserts `AlreadyOnWatchlistError` is raised, and confirms only one entry exists in the database afterward. Test passes.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` and added `test_add_to_watchlist_nonexistent_film_raises`, following the same fixture and assertion structure as `test_add_to_collection_nonexistent_film_raises` in `test_collection.py`.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` — test passes, confirming `FilmNotFoundError` is raised for a nonexistent `film_id`.