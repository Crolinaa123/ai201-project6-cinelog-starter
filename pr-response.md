## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to match the project's `verb_to_noun` naming convention (same pattern as `add_to_collection()`). Updated the one call site in `routes/watchlist/watchlist.py` (import and function call).
**How I verified:** Ran `grep -rn "save_to_watchlist" . --include="*.py"` across the repo to confirm no references to the old name remained. Ran the full test suite (`pytest tests/ -v`) to confirm nothing broke.

## Comment 2 — Deduplication
**What I did:** Added an `AlreadyOnWatchlistError` exception and a check in `add_to_watchlist()` that queries for an existing `WatchlistEntry` matching the given `user_id` and `film_id` before creating a new one, raising the exception if one is found. Followed the same pattern as `add_to_collection()`'s handling of `AlreadyInCollectionError`.
**How I verified:** Wrote `test_add_to_watchlist_duplicate_raises` in `tests/test_watchlist.py`, which adds a film to a user's watchlist, attempts to add it again, asserts `AlreadyOnWatchlistError` is raised, and confirms only one entry exists in the database afterward. Test passes.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` and added `test_add_to_watchlist_nonexistent_film_raises`, following the same fixture and assertion structure as `test_add_to_collection_nonexistent_film_raises` in `test_collection.py`.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` — test passes, confirming `FilmNotFoundError` is raised for a nonexistent `film_id`.

## Comment 4 — Default visibility
**My position:** I think the new entries should be set to public
**Reasoning:** This is so that people are able to browse what others want to watch, follow friends, and get recommendations based on what is trending and on the rise. 
**Tradeoff acknowledged:** However, there is a concern for privacy leak. Every film you add becomes visible to strangers before you have made a choice willing to share your watchlist with others.

## Comment 5 — Sort order
**My position:** Date-added (recent first)
**Reasoning:** This order optimizes for "what did I just add, what am I likely to watch soon". This works as a working-list mental model, and it is useful if people add films with the intention of watching them shortly after.
**Engagement with reviewer's point:** I agree with the reviewers point because people want to keep up with they added recently because sometimes that list may become overwhelming with multiple movies added, thus it is useful to have an order that shows what you added recently to keep up with your current interests. 

## Comment 6 — Rebase
**What conflicted:** I fetched the updated main, ran git rebase origin/main. Moroever, the Film.id and CollectionEntry.film_id had been migrated from integer to UUID on main. Note the interesting part, there's no textual merge conflict was flagged by git, because the commit that added the watchlist code never actually touched models.py itself, since WatchlistEntry already existed in the shared ancestor commit and main's UUID refactor commit removed it entirely. So the "conflict" was silent/structural rather than a normal conflict-marker situation.

**How I resolved it:** After the rebase completed without prompting for conflict resolution, checked models.py and found WatchlistEntry was missing entirely. So, I manually re-added the class with film_id as db.String(36) to match the new UUID-based Film.id, keeping the rest of the class (id, user_id, date_added, public, to_dict()) consistent with its pre-rebase version.

**How I verified no conflict remains:** To verify, I ran pytest tests/ -v and confirmed all 6 tests passed. Ran git log --oneline --graph to confirm the branch history has no merge commits introduced by the rebase itself. I also checked services/watchlist_service.


## PR description
Adds a watchlist feature allowing users to save films they want to watch. Includes a WatchlistEntry model, add_to_watchlist/get_watchlist service functions, and REST endpoints (GET /watchlist/<user_id>, POST /watchlist/<user_id>/add).

Design decisions:
- Watchlist entries default to public visibility in order to access what others are following and to get recommendations. 
- Results are sorted by recent-first in order to keep up with current interests and trends.

Manual testing:
1. Run `python app.py`
2. POST to /watchlist/<user_id>/add with body {"film_id": "<uuid>"} — confirm a 201 response with the new entry
3. GET /watchlist/<user_id> — confirm the film appears in the list
4. POST the same film_id again — confirm it's rejected (duplicate check)
5. POST with a nonexistent film_id — confirm a not-found error

## AI Usage
Used AI for codebase orientation in order to understand what add_to_collection() and the existing test patterns did before writing the equivalent watchlist code myself. I also used it to catch a couple of mechanical issues during implementation (a broken line from a manual edit, a missing exception class) and to help script the interactive rebase/commit-message rewrite in Milestone 4. 