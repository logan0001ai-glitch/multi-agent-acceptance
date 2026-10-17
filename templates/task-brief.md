# [Round name]: [What to do] ([Which executor])

Date. Model [model] [effort level], time limit [N] minutes.

## Boundaries and stop conditions

- Do not commit, revert, or update the project yourself.
- No code changes by default; only where a section explicitly allows it. If you change code, list the files and run the full regression suite: `[full-suite script]`. No other suite counts.
- Continuation round: append to the existing change list. Do not overwrite it.
- Before each section and each batch of screenshots, read the coordinator notes and follow them.
- If the stop file exists, stop.
- If the editor / app under test exits or hangs: do not relaunch it. Write a "needs human" marker and stop.
- Stop only if new code writes bad data. External failures do not stop the run; record them.
- Back up before starting and record a checksum; compare again at the end. Do not restore.
- No real purchases, real reward claims, or other irreversible actions.

## 1. [First item]

- Action:
- Observe for:
- Pass:
- Fail:
- If it does not reproduce, write "not reproduced". Do not keep guessing.

## 2. [Next item]

- Action:
- Observe for:
- Pass:
- Fail:

For visual items: one screenshot at each of two window sizes. Animations cannot be judged from stills. After cropping to a single object, the effect must still be visible; once it passes, do not tighten it further.

## Delivery (in this order)

1. One-sentence conclusion
2. Pass / fail for each item; for failures, the symptom and screenshots
3. Retry count per run
4. Files changed; the last line of the full regression output (from the script named above)
5. Screenshot list
6. Current change status
7. Brief ID, code revision and run time the screenshots were taken against

Do not describe results with adjectives ("clearly visible", "fixed"). The coordinator ignores them and checks the images.

Do not commit.
