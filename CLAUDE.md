# {{PROJECT_NAME}} — Claude Code instructions

<!--
Boilerplate CLAUDE.md. Copy to `.claude/CLAUDE.md` (or the repo root) of a new project,
replace every {{PLACEHOLDER}}, and delete sections that do not apply.
Keep this file short: it loads into every session. Put detail in `.claude/rules/*.md`
and link it from the index below.
-->

{{One-paragraph description: what the project is, who uses it, what "production" means here.}}

## Session start

Run `date -u` and show the result. Cross-check the clock against an external source before any status work:

```bash
date -u; curl -sI https://api.github.com/ | grep -i '^date:'
```

A local clock that is days behind makes cached tool output (e.g. `gh`) look fresh while being stale, with no error anywhere.

## Index

| Document | Description |
|----------|-------------|
| [rules/MEMORY.md](./rules/MEMORY.md) | Index of project rules; one line per rule |
| [rules/working-agreements.md](./rules/working-agreements.md) | {{Optional: the long form of the rules below}} |
| {{ARCHITECTURE.md}} | {{Structure, data flow}} |
| {{DEVELOPMENT.md}} | {{Local setup, workflow}} |
| {{DEPLOYMENT.md}} | {{How changes reach production}} |

---

## Critical rules (quick reference)

### 1. Gated actions — draft, show, wait

These are outward-facing: someone else sees the result and cannot easily un-see it. Never do them as a side effect of another task. Each needs an explicit instruction ("commit", "push", "post it", "send it").

| Action | Rule |
|---|---|
| Commit / push / open a PR | Never automatically. Make the change, describe it, wait. |
| Comment on an issue or PR | Draft it in chat first. |
| Send a chat message (Slack, Teams, email) | Draft it in chat first. Even when told "send X a message", show the wording before sending; most messaging tools have no edit or delete. |
| Bulk-edit a shared board, tracker or config | Confirm scope first. Single-card updates during agreed work are fine. |
| Destructive operations (drop, delete, force-push, restore over data) | Confirm first, and look at the target before acting. |

Reading anything is always fine.

### 2. Never fabricate an identifier — and link every one you cite

- Issue/PR numbers, comment ids, commit SHAs, URLs, file paths, line numbers, config ids: retrieve each with a tool before citing it. If you have not seen it in tool output this session, fetch it.
- Cite every reference as a clickable markdown link, every occurrence, including inside tables and in the "Next" section:
  - Issue/PR: `[repo#123](https://github.com/{{ORG}}/repo/issues/123)`
  - Commit: `[abc1234](<remote>/commit/abc1234)`; look the remote up with `git remote get-url origin`, never infer it from the directory name.
  - Local file: a path relative to the opened workspace root, e.g. `[src/app.ts:42](src/app.ts#L42)`.
- Link every place the user may need to go, not only identifiers: the console, dashboard or form behind each step they must do by hand (cloud/DNS/registrar panels, API-token pages, removal or support forms), inline on that step and again in "Next". Prefer the deepest stable URL (the API-tokens page, not the dashboard home). Check each with `curl -sIL` before citing it; a page behind login or bot protection answers 403 or redirects, so label it "behind login, not verified past it". Never invent a deep link — link the closest verified page and give the menu path from there. A hand-off with only a menu path costs the user a round trip to ask for the URL.
- Ranges (`#10–#15`) and bare SHAs cannot be clicked; expand them.
- When posting into GitHub, a bare `#123` resolves against the repo you post in. Qualify every cross-repo reference, every token: `other-repo#12 / other-repo#13`, never `other-repo#12 / #13`.

### 3. Label your evidence

Mark non-trivial claims **verified** (read from a tool this session), **assumption**, or **estimate**. When two documents disagree, name both rather than silently picking one. Never state a plan as a status ("specified", not "built").

### 4. Verify the artefact, not the machine

- Verify against what ships (the built image, the merged commit, the migration in the repo), not against local state you prepared by hand. Anything applied by hand is fiction until the deployed artefact carries it.
- Assert on content, not on exit codes or status lines. A 200 on `/`, an exit code of 0, or "it returned rows" are all compatible with a broken system. Use `set -o pipefail`; read the script's own exit code, not a trailing `tail`'s.
- Prove a check discriminates: run it once with input that must fail.
- A board/ticket status is a claim. Resolve the linked PRs and merge commits before repeating it.
- For anything a decision rests on, bypass caches (`gh api --cache 0 …`).

### 5. Close substantive responses with a numbered "Next"

- Name **one** recommended action as item 1; order by consequence-if-delayed, not effort.
- Separate *actionable now* from *blocked on a person* (name the person).
- Number every suggested item, option or open question, in one continuous sequence per response, so the user can reply "do 2 and 3, skip 1".
- "Nothing to do, waiting on X" is a valid answer. Do not invent work.

### 6. Feed insight back into rules, in the same session

When a session teaches something that would have made the work faster or safer — especially anything that failed **silently** — update the rule or skill that should have said it, now, not as a promised follow-up. Record the cost ("caused a 3-day outage"). End each change with a one-line **Rules:** proposal naming the file to update, or "none needed".

Store persistent knowledge in committed `.claude/rules/<topic>.md` (plus a line in `rules/MEMORY.md`), not in per-machine memory.

### 7. File an issue before fixing a problem

Once investigation shows something is actually broken, file the issue first (symptom, evidence, proposed fix), then do the work and reference it. A ticket written after the fix loses the symptom and the wrong turns, which are the reusable part. Not needed for pure investigation, doc/rule edits, or changes the user directs turn by turn.

---

## Git

- **Branching:** every issue starts on a new branch cut from a freshly fetched base: `git fetch -q origin {{BASE_BRANCH}} && git switch -c fix/<issue>-<slug> origin/{{BASE_BRANCH}}`. Never branch from `HEAD` or another feature branch.
- **PR target:** {{BASE_BRANCH}}. Never infer the base from the repo's default branch; check the repo's contributing docs.
- **Issue keywords:** {{Choose one: "Use `Relates to #N`; issues stay open until validated in production" OR "Use `Fixes #N`"}}.
- **Commit identity:** {{Name <email>}}. If the environment injects `GIT_AUTHOR_*`/`GIT_COMMITTER_*` variables, they override `git config`; set all four inline on every commit, rebase, amend and cherry-pick, and check with `git log -1 --format='%an <%ae> | %cn <%ce>'`.
- Stage explicit paths, never `git add -A`. After a commit hook (lint-staged, formatters) runs, confirm the committed bytes are what you verified: `git show --stat HEAD`.

## Development

- Run every script as `cd <absolute path> && <command>`, so it never depends on the current directory.
- {{Build/test/lint commands, and which ones NOT to run (e.g. "never run the production build during development")}}
- {{Type-safety rules, e.g. "no `as any`; regenerate types instead"}}
- {{UI rules, e.g. "no hardcoded user-facing text; use translation keys"}}
- {{Verification rule, e.g. "browser-verify UI changes before committing"}}

## Secrets

- Secrets live in {{SECRET_STORE}}. Never commit them; name where they are kept instead.
- Never put a secret in a command's argv: approved commands are stored verbatim in the permission allowlist, and argv is visible in `ps`. Read from env or a 600-mode file.
- Write secrets with `printf '%s'`, never `echo` (a trailing newline breaks HTTP headers and reads like a bad token).
- A leaked secret is fixed by rotation, not deletion. Prove the old value is dead.
- Before granting someone access to a repo or committing files pulled off a host, scan for credential values.

## Writing for GitHub and chat

- Do not hard-wrap prose; one line per paragraph, bullet or table cell.
- Diagrams: use ```mermaid, not ASCII art. Style with `stroke`, not `fill`; keep `#123` out of node labels.
- Chat messages: direct and factual, no filler ("please review X", not "could I get a review when you're happy"). Reply in the thread of the message you are answering. Address people by their chat display name, resolved by lookup, not inferred from their GitHub handle.

## Quick start

```bash
{{clone / install / run commands}}
```

| Thing | Value |
|---|---|
| Local app URL | {{http://localhost:PORT}} |
| Database | {{connection command}} |
| Issue tracker / board | {{URL}} |
