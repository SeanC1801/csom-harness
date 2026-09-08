# Risks & Open Questions

- **AppleScript/Notes.app flakiness.** AppleScript's Notes support has known quirks, it may require Notes.app to be open, and can trigger permission prompts. This matters because the CLI trigger is supposed to "just work" on request; an unexpected permission dialog breaks that.
- **No Canvas API token yet.** The user has not generated Canvas API access. This matters because nothing in the pipeline can be built or tested end-to-end until this exists, it's the first real blocker, not a future risk.
