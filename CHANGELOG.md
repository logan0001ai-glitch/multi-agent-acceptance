# Changelog

Every rule here comes from a real round that went wrong. Dates are US Eastern.

## 2026-10-19

### Briefs
- **"Needs human" no longer means "stop".** Executors ended the session after writing the marker, voiding the round. They now keep working, poll every 2 minutes, wait at most 40 minutes, then submit.
- **Windowed mode for fullscreen captures.** True fullscreen crashed sessions.
- **Progress file and stamped evidence are required in every brief.** Phases ("full regression running", "run N live") let monitoring relax its thresholds; stale screenshots must be re-captured on the final code.
- **Write the behavior, not a slogan.** "Don't change other people's tests" and "wait in place" were read literally and backfired; the skill now gives the wording to use instead.

### Dispatch and monitoring
- **Quota: trust only records with a number; stop at 70% used.** An empty dashboard hid an exhausted account. Alerts now fire at 50/70/85/100% used, plus a burn-rate warning.
- **Dispatch checklist.** A round goes out only when it is ready, has no file conflicts, its dependencies are committed, and the target machine has the merged assets.
- **Phase-aware stuck detection.** Missing progress file counts as no progress; thresholds relax for commit/merge and regression/live phases; the clock pauses while waiting on people.
- **Monitors run on an always-on service, not inside a conversation.** Launch scripts are derived from the template, never copied from another machine.

### Acceptance
- **Verify evidence before committing.** After the commit the revision changes and can no longer be used to check the screenshots retroactively.
- **Compare the change list against the executor's workspace.** Result files from other rounds are removed; untracked tests and tools are listed or marked "not delivered". Run a regression group after a manual conflict fix.
- Ten new rows in the symptom table.

## 2026-10-17

### Acceptance
- **"Full regression" must name one script.** An executor reported 224/224 from a team-level suite. The full suite, run at submit, was 139/141. The brief now names the full-suite script, and acceptance reads only that line.
- **Adjectives in a delivery are void.** An executor wrote that the effect was "clearly visible". Cropped, the required outer ring did not exist; it had counted the existing decorative frame as its own work.
- **Evidence must match brief ID, code revision and run time.** Screenshots taken before the final fix, or under another round's name, were submitted as proof. Any mismatch means the round does not pass.

### Briefs
- **Search new briefs for the old round name; the count must be 0.** A brief generated from an older one kept the old name. One executor followed it, edited 4 already-merged files and overwrote 3 images.
- **Continuation rounds append to the change list.** A continuation round rewrote the list as its own 6 files and dropped the previous 35 entries. Acceptance now compares entry counts.
- **Referenced deliverables must be merged or listed.** A new test passed 133/133 on one machine only because the referenced file existed on that disk. After merge, every other machine failed.

### Dispatch and monitoring
- **Read quota before every dispatch; below 30%, stop and report.** A shared account ran out mid-day and two rounds died mid-way; one left 19 changed files unverified. The same thing happened on a second account that night. Neither was warned at 30%.
- **Alert on 30 minutes without progress and on the same error 4 times.** Thresholds come from logs: 57 rounds (median 42 min, P75 64, max 128) and 91 logs (same-error max 4, P95 2).

## 2026-10-15
- Initial release.
