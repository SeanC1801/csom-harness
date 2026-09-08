# Objectives

- Running the CLI command triggers a full check of Canvas and updates the to-do list in Apple Notes automatically, with no manual copying.
- Every course is checked the same way, for assignments, quizzes, and activities tabs.

# Scope

**In scope:** an automated process that lists what's needed for school by pulling data from Canvas, and a defined output format for the to-do list.

**Non-goal:** no features beyond the to-do list itself. Explicitly avoid over-engineering; the goal is a simple, working product.

# Systems Overview

| # | System | Responsibility | Primary Tech |
|---|---|---|---|
| 1 | Canvas Data Fetcher | Authenticate and pull assignments, quizzes, and activities per course | Python + Canvas LMS API |
| 2 | To-Do Formatter | Turn raw Canvas data into a consistent to-do list format | Python |
| 3 | Notes Writer | Write/update the formatted list into Apple Notes | AppleScript (`osascript`) |
| 4 | CLI Trigger | Single command that runs the fetch → format → write pipeline on request | Python (CLI entry point) |
