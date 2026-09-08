# Canvas To-Do Sync — Technical Proposal

Prepared for: Sean Caling
Document type: Technical Proposal

## Executive Summary

An automated to-do-list that pulls assignment, quiz, and activity data from Canvas (via the official Canvas LMS API) and places it into Apple Notes on request. The user triggers a sync through a CLI command; the tool reads every course the same way, checking each course's assignments, quizzes, and activities tabs, and writes the results into Notes.

Success is defined narrowly: running the CLI command reliably produces an up-to-date to-do list in Apple Notes, with no manual copying.

## Objectives & Scope

**Objectives:**
- Running the CLI command triggers a full check of Canvas and updates the to-do list in Apple Notes automatically, with no manual copying.
- Every course is checked the same way, for assignments, quizzes, and activities tabs.

**In scope:** an automated process that lists what's needed for school by pulling data from Canvas, and a defined output format for the to-do list.

**Non-goal:** no features beyond the to-do list itself. Explicitly avoid over-engineering; the goal is a simple, working product.

## Systems Overview

| # | System | Responsibility | Primary Tech |
|---|---|---|---|
| 1 | Canvas Data Fetcher | Authenticate and pull assignments, quizzes, and activities per course | Python + Canvas LMS API |
| 2 | To-Do Formatter | Turn raw Canvas data into a consistent to-do list format | Python |
| 3 | Notes Writer | Write/update the formatted list into Apple Notes | AppleScript (`osascript`) |
| 4 | CLI Trigger | Single command that runs the fetch → format → write pipeline on request | Python (CLI entry point) |

## Architecture

**Approach: Python + Canvas API + AppleScript.** Chosen because "native notes" means Apple Notes on the user's Mac, and AppleScript is Apple's own sanctioned way to script Notes, so it fits exactly. The main tradeoff accepted: the user hasn't touched the Canvas API yet, and AppleScript's Notes support has known quirks (needs the app open, occasional permission prompts).

### Canvas Data Fetcher

Authenticates with a Canvas API token (sent as a Bearer token) and calls Canvas's REST endpoints per course:

```python
import requests

BASE_URL = "https://<your-school>.instructure.com/api/v1"
HEADERS = {"Authorization": f"Bearer {CANVAS_API_TOKEN}"}

def get_courses():
    return requests.get(f"{BASE_URL}/courses", headers=HEADERS).json()

def get_assignments(course_id):
    return requests.get(f"{BASE_URL}/courses/{course_id}/assignments", headers=HEADERS).json()

def get_quizzes(course_id):
    return requests.get(f"{BASE_URL}/courses/{course_id}/quizzes", headers=HEADERS).json()
```

Every course is processed through the same calls, no per-course special-casing.

### To-Do Formatter

Takes the raw assignment/quiz/activity dicts from each course and normalizes them into one flat list of `{course, title, due_at, type}` items, sorted by due date, before handing off to the Notes Writer.

### Notes Writer

Uses `osascript` to update a dedicated note in Apple Notes:

```bash
osascript -e 'tell application "Notes"
  set theNote to note "To-Do List" of folder "Notes"
  set body of theNote to "<formatted to-do text>"
end tell'
```

### CLI Trigger

A single entry point (e.g. `python3 sync_todo.py`) that calls Canvas Data Fetcher → To-Do Formatter → Notes Writer in sequence, and prints a short success/failure summary.

## Risks & Open Questions

- **AppleScript/Notes.app flakiness.** AppleScript's Notes support has known quirks, it may require Notes.app to be open, and can trigger permission prompts. This matters because the CLI trigger is supposed to "just work" on request; an unexpected permission dialog breaks that.
- **No Canvas API token yet.** The user has not generated Canvas API access. This matters because nothing in the pipeline can be built or tested end-to-end until this exists, it's the first real blocker, not a future risk.

## Recommendation & Next Steps

1. Get Canvas API access (generate an API token) — the current blocker; nothing else can be tested without it.
2. Pull data via the Canvas API — list courses, then read assignments/quizzes/activities uniformly for each.
3. Format the to-do list output.
4. Build the AppleScript (`osascript`) piece that writes/updates Notes.app.
5. Wire steps 2-4 behind a single CLI command.
6. Test against one real course first, then expand to all courses.
