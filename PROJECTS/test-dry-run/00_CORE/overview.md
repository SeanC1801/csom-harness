# Executive Summary

An automated to-do-list that pulls assignment, quiz, and activity data from Canvas (via the official Canvas LMS API) and places it into Apple Notes on request. The user triggers a sync through a CLI command; the tool reads every course the same way, checking each course's assignments, quizzes, and activities tabs, and writes the results into Notes.

Success is defined narrowly: running the CLI command reliably produces an up-to-date to-do list in Apple Notes, with no manual copying. The explicit non-goal is adding any feature beyond the to-do list itself, and avoiding over-engineering in favor of a simple, working product.

## Recommendation & Next Steps

Approved approach: Python script using the Canvas API, writing into Apple Notes via AppleScript (`osascript`), triggered by a CLI command.

Build order:

1. Get Canvas API access (generate an API token) — this is the current blocker, see `03_TRACKING/blocked.md`.
2. Pull data via the Canvas API — list courses, then read assignments/quizzes/activities uniformly for each.
3. Format the to-do list output.
4. Build the AppleScript (`osascript`) piece that writes/updates Notes.app.
5. Wire steps 2-4 behind a single CLI command.
6. Test against one real course first, then expand to all courses.
