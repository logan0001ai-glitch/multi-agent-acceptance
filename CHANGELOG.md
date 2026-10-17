# Changelog

Every rule here comes from a real round that went wrong. Dates are US Eastern.

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
