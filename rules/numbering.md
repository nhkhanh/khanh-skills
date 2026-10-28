# Numbering: a number is an address

**Rule:** every item a reader might act on gets a number from one sequence that runs for the whole thread. A number never changes meaning, is never reused, and is written as a bold number in a bullet, never as a markdown ordered list.

**Why:** the user answers by reference ("do 2 and 3, skip 1"). If a number restarts, repeats or shifts, that reply becomes ambiguous and the wrong action gets done. Markdown causes this silently: an ordered list keeps only its first number and renumbers the rest, so `17. 18. 19. 30.` renders as 17, 18, 19, 20.

Every skill and response follows this file. A skill that produces steps, options, findings or a "Next" section links here instead of restating it.

## What to number

- **1.** Number every item a reader might act on: Next-step actions, plan steps, phases, options, open questions, findings, and rows of a status or verdict table (as a number column).

## One sequence per thread

- **2.** The first response starts at 1. Every later response continues from the highest number used anywhere earlier in the thread. Never restart at 1 in a follow-up.
- **3.** An item keeps its number for its whole life. Carried forward, it keeps its number. Closed, it is reported as "7 — done", and 7 is never reused for something else.
- **4.** Within a response, the body and the Next section share one sequence: if the body ends at 6, Next starts at 7. A Next action about an item already in the body reuses that item's number instead of getting a new one: `- **42.** Ask Ed to rebase (item 42 above)`. One action, one number.
- **5.** The Next section names one recommended action as its first item. List order is priority order, by consequence if delayed, not effort; new numbers are handed out in that order. Split "Actionable now" from "Blocked on <person>". The sequence continues across both groups, so a response never contains a second "1.".
- **6.** When relaying a subagent or tool report, renumber its items into the thread's sequence. Never quote the report's own numbering.

## How to write a numbered item

- **7.** Never use a markdown ordered list. Write each item as a bullet with a bold number, `- **30.** Close the PR …`, even when the numbers happen to be consecutive. A number column in a table is also safe, because table cells are not renumbered.

Wrong — the source says 30, the reader sees 20:

```markdown
17. Rebase the branch
18. Rerun the tests
19. Ask for review
30. Close the PR
```

renders as:

```text
17. Rebase the branch
18. Rerun the tests
19. Ask for review
20. Close the PR
```

Right — every number survives rendering:

```markdown
- **17.** Rebase the branch
- **18.** Rerun the tests
- **19.** Ask for review
- **30.** Close the PR
```

## Check before sending

- **8.** The first new number is above the highest number used earlier in the thread.
- **9.** No number appears twice in the response for two different items. A Next action reusing a body item's number (item 4) is the same item, not a duplicate.
- **10.** No number now means something different from what it meant earlier in the thread.
- **11.** No line outside a code fence starts with `<digits>. `. For a file:

```bash
awk '/^```/{f=!f; next} !f && /^[[:space:]]*[0-9]+\. /{print FILENAME ":" FNR ": " $0}' <file>.md
```
