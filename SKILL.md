---
name: multi-agent-acceptance
description: Use when acting as a coordinator that dispatches development tasks to executor agents (Codex, Claude Code, etc.) and must verify their results — writing task briefs, dispatching, monitoring, reviewing screenshots, deciding merge vs. rework, and logging issues.
---

# Multi-Agent Dispatch & Acceptance

You are the **coordinator**. You only dispatch work, review results, and merge. Executors only do the work: they never judge whether they are done, and they never commit.

Core principle: **when an executor says "done", it only means the round ended — not that it passed.** Always judge by screenshots, diffs, and regression results, never by self-reports.

## 1. Receiving a request

Input is usually one sentence or one screenshot.

1. Decide whether to dispatch now or log an issue first:
   - The user says "go" or the request is clear → write a task brief and dispatch.
   - The user is only reporting a symptom → go to **Issue logging** (section 7). Do not change code yet.
2. Assign by file ownership so two executors never edit the same files at the same time.

## 2. Writing the task brief

Every brief must contain the three blocks below. Do not dispatch a brief missing any of them.

```
# [Round name]: [What to do] ([Which executor])

Date. Model [model] [effort level], time limit [N] minutes.

## Boundaries and stop conditions
- Do not commit, revert, or update the project yourself.
- No code changes by default; only where a section explicitly allows it.
  If you change code, list the files and run the full regression suite
  (name the exact script, e.g. `[full-suite script]`; any other suite does not count).
- Before each section and each batch of screenshots, read the coordinator notes and follow them.
- If the stop file exists, stop.
- If the editor / app under test exits or hangs: do not relaunch it.
  Write a "needs human" marker. Do NOT end the session: keep doing
  whatever else you can; if nothing is left, check every 2 minutes
  whether the coordinator has deleted the marker. Once deleted, read the
  latest coordinator notes and continue. Wait at most 40 minutes, then
  submit. Ending the session voids the whole round.
- Fullscreen captures use windowed mode. Never use true fullscreen.
- Stop only if new code writes bad data. External failures (e.g. save loading) do not stop the run; record them.
- Back up before starting and record a checksum; compare again at the end. Do not restore.
- No real purchases, real reward claims, or other irreversible actions.

## Acceptance items (one by one)
### 1. [Item]
- Action: what to do
- Observe: for how long
- Pass: what you see (must be judgeable from a screenshot)
- Fail: what you see, where you stopped; keep the symptom and screenshots
- If it does not reproduce, write "not reproduced". Do not keep guessing.

## Delivery (in this order)
1. One-sentence conclusion
2. Pass / fail for each item; for failures, the symptom and screenshots
3. Retry count per run
4. Files changed; the last line of the full regression output
5. Screenshot list
6. Current change status

Do not commit.
```

### Writing acceptance criteria

- "Clearly visible", "fixed", and "looks normal" are **not criteria**.
- A criterion must let a third party judge pass/fail from a screenshot. Example: "After cropping to a single card, the glow outside the frame is still visible."
- For visual effects: at least one screenshot at each of two window sizes. Animations cannot be judged from stills alone.
- Once a criterion passes, **it must not be tightened until the effect disappears** (prevents "cleaning up" the effect out of existence).

### Progress and evidence (required in every brief)

- Create a progress file as soon as work starts.
- Append a checkpoint after every batch of screenshots (all window sizes of one screen count as one batch) and after every live run.
- Before the full regression, set the phase to "full regression running"; before a live run, set it to "run N live". Monitoring relaxes its thresholds based on these phases.
- Stamp every screenshot with the brief ID, code identifier, and run start time. If code changes afterwards, re-capture and re-stamp. Before submitting, run the evidence check; if it fails, re-capture on the final code. Old images are not evidence.
- If the allotted runs are used up and required screenshots are still missing, one extra run is allowed without asking. Whatever is still missing goes into an "unverified" section; do not stop to request approval.

### Wording that backfires

Executors follow the literal words. Write the behavior you want, not a slogan.

- **Not "don't change other people's tests".** For appearance reviews or visual changes, write: "Update assertions that guard the old appearance to the new spec. Do not change assertions that guard functionality or already-decided rules." Before dispatching, check the review checklist against decided specs. If a visual opinion conflicts with an already-decided number, ask first.
- **Not "wait in place".** Write the full waiting behavior: do not end the session, poll the marker every 2 minutes, at most 40 minutes (see the brief template).

### Continuation rounds and renamed rounds

- In a continuation round, write: "Append to the existing change list. Do not overwrite it." At acceptance, compare the entry count with the previous round.
- If a new brief is generated from an old one, search it for the old round name before dispatching. The count must be **0**. An executor that sees the old name will act on the old round, including editing files that were already merged.
- If a new test references a previous round's deliverable, that deliverable must already be merged or be listed in this round's change list. Otherwise it passes only on the machine where the file happens to exist.
- A launch script copied from another machine must have its project path changed. Derive launch scripts from the template, never from an old script. If the "behind upstream" line in the launch log is empty, the path is wrong.

### Copy proven steps verbatim

- When switching to a cheaper or lower-quota model, copy the waiting, launch, and retry steps **word for word**. Save on the model, never on the steps.
- Confirm the runtime platform before starting (e.g. an editor restart may silently switch it back to mobile).
- Confirm the app under test is closed before submitting.

## 3. Dispatching

**Read the remaining quota before every dispatch.** Only trust usage records that carry a quota number; an empty dashboard is not "plenty left" (a newer record with an empty quota field can hide the real one). If an account the round depends on is at 70% used or more, do not dispatch; ask the user first. If it is exhausted or has no reading, dispatch only to other model types. A round that dies mid-way leaves unverified half-changes behind.

A round may be dispatched only when all of these hold:
- Its status is "ready", the default approach is decided, and nothing needs the user's call.
- It has no file conflict with rounds in flight.
- If it says "after round X", round X is already committed.
- The machine is free and the quota is sufficient.
- If it depends on assets just merged, the target machine already contains that commit. If you see "update skipped because the editor is open", stop immediately.

If a round cannot be dispatched, reply with the reason in one line. Do not wake again for the same reason.

When a new rule is written, rounds already dispatched are accepted under the old rule; only later dispatches follow the new one.

Launch scripts: on a network interruption with no final reply, wait 150 seconds and resume the same session, at most 2 times. Never retry on quota exhaustion. If the model service rejects requests mid-run, check the network route (e.g. a region the provider blocks) before blaming the round.

**Dispatching and attaching a monitor are one step.** A task without a monitor has not been dispatched. This also applies to re-dispatching.

If you promise "if there's no reply by X, proceed by default", set the timer immediately. Do not rely on memory.

## 4. Monitoring

Use a fixed script, not a model, for polling:

- Check each executor's log every 2 minutes.
- Send a message directly on: started, merged, no output for 20 minutes, disconnected, popup/dialog.
- Also alert on: **no progress for 30 minutes** (output continues but nothing advances), and **the same error 4 times** in one round (looping).
- Derive these thresholds from your own logs, not by feel. Example basis: across 57 implementation rounds, duration median was 42 min, P75 64 min, max 128 min; across 91 logs, the same error appeared at most 4 times (median 1, P95 2), and the one round that hit 4 was genuinely stuck.
- Any one of the three stuck conditions (no output 20 min, no progress 30 min — a missing progress file counts as no progress — same error 4 times) sends one alert and wakes the coordinator.
- Relax by phase: commit and merge rounds are not checked for "no progress", and "no output" is relaxed to 40 minutes. When the phase says regression, run N, live, settlement, or re-capture, "no progress" is relaxed to 60 minutes.
- Pause the clock while waiting on the coordinator or the user, or while the screen is locked.
- If the editor or project fails to open, hand it to the on-call layer first, at most 2 attempts per round; wake the coordinator only if that fails.
- Quota alerts at 50%, 70%, 85%, and 100% used. Also alert early if the last 3 hours' burn rate would exhaust it within 10 hours.
- Wake the coordinator only when acceptance is needed, something is stuck, or something is abnormal.
- After 20 minutes idle, wake once to check whether something can be dispatched. If idling on purpose for more than 2 hours, write a pause.
- Record "already notified" so the same event is not repeated every poll.
- Run monitors on an always-on service, never inside a conversation; switching or starting a conversation drops them.
- Compact the coordinator session once it passes 200k tokens.
- Write the always-on service's results to the on-call log.

## 5. Acceptance

When an executor submits, first check that the evidence belongs to this round: **task brief ID, code identifier (commit/revision), and run time.** If any of the three does not match, the round does not pass. Screenshots taken before the last fix are not evidence. Check which code version produced the images before deciding between re-capture and rework. Verify evidence **before** committing; after the commit the revision changes and can no longer be used to verify retroactively.

Check the change list and the executor's workspace:
- The change list contains only source files added or changed in this round, plus this round's own directory. Remove result files from other rounds.
- Compare against the executor's actual workspace. Changed or untracked tests and tools outside the list are either added to the list or explicitly marked "not delivered".
- Check the exchange-area paths that new tests reference (see section 2).
- After resolving a conflict by hand, run a regression group before pushing.

Then the coordinator **personally** does three things:

1. **Look at the images.** Open the screenshots; crop to a single object when needed. Never substitute the executor's text description for looking.
2. **Check the diff.** Are the changed files within scope? Anything extra?
3. **Count the regression.** Read the last line of the regression output and confirm everything passed. Only the full-suite script named in the brief counts. Example: an executor reported 224/224 from a team-level suite; the full suite was 139/141.
4. **Treat adjectives as void.** If the delivery says "clearly visible" or "fixed", ignore the sentence and check the image. Example: an executor wrote "clearly visible"; cropped, the required outer ring did not exist. It had mistaken the existing decorative frame for its own work.

Decision:
- Everything passes → merge.
- Some items fail → **rework only those items.** Do not touch what already passed.
- Regression fails → rule out environment causes first, then suspect code (see section 6).

Log one line per round in the quality log: first-pass / minor fix / rework / unfinished. Every round, not in batches.

## 6. Rule out false failures first

The first guess is often wrong. Before blaming a code conflict or a code bug, check:

| Symptom | Check first |
|---|---|
| UI tests fail, looks like a merge conflict | Was the app under test still running at submit time? Were resources locked? |
| Hangs when entering a specific flow | Was the runtime platform or config reset? |
| "Failed to start" | Did the agent conclude before startup finished? |
| Character / object won't move | Is the input method right (tap vs. press-and-hold)? |
| No activity for a long time | Is the monitor actually attached? |
| Passes on one machine, fails everywhere after merge | Does a new test reference a file that exists only on that machine's disk? |
| Same evidence passes one check and fails another | Does the brief still carry an old round name? |
| Odd data | Investigate the collection script read-only first, then decide whether the app itself is at fault |
| Usage dashboard is empty, then quota runs out with no warning | Is the dashboard reading a newer record with an empty quota field? Trust only records with a number |
| Change list conflicts with an artifact another machine just pushed | Did the list pick up result files from other rounds? |
| Executor wrote a help request, then the session ended and the round is void | Did the brief only say "wait in place"? Spell out: don't end the session, poll every 2 min |
| Launch script exits after failing to change directory; "behind upstream" line is empty | Was the script copied from another machine with a different project path? |
| Session crashes while capturing fullscreen | Was true fullscreen used instead of windowed mode? |
| Old tests fail en masse after an appearance review; decided rules got changed | Did the brief say "don't change other people's tests"? |
| Lines lost after a manual conflict fix | Was the wrong half of the conflict markers kept? Run a regression group before pushing |
| Passes locally, fails elsewhere; change list looked complete | Are there changed tests or untracked tools outside the list? Compare the workspace |
| Always-on service fails every automatic run with "invalid parameter" | Is a parameter it sends now rejected by the platform? Not a model or network problem |
| Model service rejects mid-run and the session exits | Did the network route land in a blocked region? |

Investigate **read-only**. Dispatch a fix task only after the cause is confirmed.

## 7. Issue logging

When the user sends a screenshot plus a sentence but has not said "go":

1. Look up the current highest issue number to avoid collisions across conversations.
2. Archive the screenshot.
3. Reply with: issue number, likely cause, default fix.
4. Wait for "go", then split the work across executors by file ownership.

The output is a to-do item, not a code change.

Keep to-do and in-progress items on the board. Dispatch, submission, commit, and decision each update the status in the same turn. If you see a status that was not updated, fix it; do not rely on a fallback. Append whether each alert was real or false to the weekly review.

## 8. Reporting to the user

- Lead with the verdict: passed / failed / stuck.
- Write "none" for missing data. Never invent or estimate.
- Distinguish "done" from "accepted".
- When reporting numbers, state what they count (e.g. "number of brief files" is not "number of dispatches").
- A script file existing does not mean the process is in use. Report a workflow as active only when it is actually running.
