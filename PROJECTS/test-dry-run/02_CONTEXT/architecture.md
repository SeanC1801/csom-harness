# Architecture — Scored Options

| Component | Choice | Compatibility | Efficiency | Difficulty | Risk/Bottlenecks | Overall | Why |
|---|---|---|---|---|---|---|---|
| Notes integration | A: Python + Canvas API + AppleScript | 5 | 4 | 3 | 3 | 4 | You said "native notes" means Apple Notes on your Mac, and AppleScript is Apple's own sanctioned way to script Notes, so it fits exactly. Difficulty is real though: you haven't touched the Canvas API yet, and AppleScript's Notes support has known quirks (needs the app open, occasional permission prompts). |
| Notes integration | B: CLI triggers an Apple Shortcut | 4 | 3 | 2 | 4 | 2 | Also native to Apple's ecosystem, and Shortcuts' drag-and-drop builder is easier to start than AppleScript syntax. But it splits logic across two separate tools, cutting against your own non-goal of not over-engineering. Looping over every course uniformly is also awkward in Shortcuts compared to real code. |
| Notes integration | C: Direct write to Notes' SQLite database | 1 | 3 | 5 | 5 | 1 | Unsupported and undocumented, Apple could break it on any OS update, and a bad write risks corrupting a database used daily. Also the hardest to build correctly, directly working against "simply creating a working product." Included mainly so the comparison has a clear worst case. |

## Decision

**Approved: Option A** (Python + Canvas API + AppleScript). Options B and C were rejected, kept above as a record of what was considered and why they were passed over.

## Contracts — skipped

This is a small-scale, solo project (one person building all systems themselves, no parallel team members to misalign with), so the contract-establishment step was skipped. That discipline exists to prevent integration mismatches between separate builders working in parallel against a deadline; it doesn't apply here.

## Technical Depth

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

Every course is processed through the same three calls (business rule: no per-course special-casing).

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

Interfaces with the To-Do Formatter's output directly, no intermediate file.

### CLI Trigger

A single entry point (e.g. `python3 sync_todo.py`) that calls Canvas Data Fetcher → To-Do Formatter → Notes Writer in sequence, and prints a short success/failure summary.
