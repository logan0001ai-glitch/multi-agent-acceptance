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
  Write a "needs human" marker and stop.
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

### Continuation rounds and renamed rounds

- In a continuation round, write: "Append to the existing change list. Do not overwrite it." At acceptance, compare the entry count with the previous round.
- If a new brief is generated from an old one, search it for the old round name before dispatching. The count must be **0**. An executor that sees the old name will act on the old round, including editing files that were already merged.
- If a new test references a previous round's deliverable, that deliverable must already be merged or be listed in this round's change list. Otherwise it passes only on the machine where the file happens to exist.

### Copy proven steps verbatim

- When switching to a cheaper or lower-quota model, copy the waiting, launch, and retry steps **word for word**. Save on the model, never on the steps.
- Confirm the runtime platform before starting (e.g. an editor restart may silently switch it back to mobile).
- Confirm the app under test is closed before submitting.

## 3. Dispatching

**Read the remaining quota before every dispatch.** If any account the round depends on is below 30%, do not dispatch; report to the user instead. A round that dies mid-way leaves unverified half-changes behind.

**Dispatching and attaching a monitor are one step.** A task without a monitor has not been dispatched. This also applies to re-dispatching.

If you promise "if there's no reply by X, proceed by default", set the timer immediately. Do not rely on memory.

## 4. Monitoring

Use a fixed script, not a model, for polling:

- Check each executor's log every 2 minutes.
- Send a message directly on: started, merged, no output for 20 minutes, disconnected, popup/dialog.
- Also alert on: **no progress for 30 minutes** (output continues but nothing advances), and **the same error 4 times** in one round (looping).
- Derive these thresholds from your own logs, not by feel. Example basis: across 57 implementation rounds, duration median was 42 min, P75 64 min, max 128 min; across 91 logs, the same error appeared at most 4 times (median 1, P95 2), and the one round that hit 4 was genuinely stuck.
- Alert on remaining quota at 30%, not at 0%.
- Wake the coordinator only when acceptance is needed, something is stuck, or something is abnormal.
- Record "already notified" so the same event is not repeated every poll.

## 5. Acceptance

When an executor submits, first check that the evidence belongs to this round: **task brief ID, code identifier (commit/revision), and run time.** If any of the three does not match, the round does not pass. Screenshots taken before the last fix are not evidence.

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

Investigate **read-only**. Dispatch a fix task only after the cause is confirmed.

## 7. Issue logging

When the user sends a screenshot plus a sentence but has not said "go":

1. Look up the current highest issue number to avoid collisions across conversations.
2. Archive the screenshot.
3. Reply with: issue number, likely cause, default fix.
4. Wait for "go", then split the work across executors by file ownership.

The output is a to-do item, not a code change.

## 8. Reporting to the user

- Lead with the verdict: passed / failed / stuck.
- Write "none" for missing data. Never invent or estimate.
- Distinguish "done" from "accepted".
- When reporting numbers, state what they count (e.g. "number of brief files" is not "number of dispatches").
