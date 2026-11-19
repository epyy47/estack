---
name: refactor
description: Severity-ranked pre-commit review of the current branch across correctness, idiomatic/perf, and readability. Use when the user runs /refactor, asks to refactor before commit, or wants a pre-commit review. Does NOT apply fixes.
---

# /refactor — Pre-commit Refactor Review

Document-only review skill. Spawns three read-only subagents in parallel to analyze the in-scope diff, then merges findings into an inline refactor plan. **Never edits files.**

Workflow position:

```
/grilling → implementation → /refactor → (draft a fix plan / apply fixes)
```

## Step 1 — Confirm scope (hard gate)

1. Compute the default scope as the union of `git diff --name-only main` (committed and uncommitted changes vs `main`) and `git ls-files --others --exclude-standard` (untracked files). `main...HEAD` is wrong here: it drops uncommitted work from a pre-commit review.
2. Print the file list and total count to the user.
3. If the user passed natural-language scope as an argument (e.g. "the position CRUD feature"), resolve it directly with Glob/Grep and show the resolved list for confirmation.
4. Ask the user to **confirm, narrow, or override** the scope.
5. If the final scope exceeds 30 files, warn and require explicit confirmation before proceeding.

**Hard gate:** do not start Step 2 until the user has confirmed a concrete file list. If the user does not respond with a clear scope, stop and wait.

## Step 2 — Launch 3 subagents in parallel

After scope is locked, dispatch all three subagents in **a single Agent tool message containing three Agent calls** (so they run concurrently). Use `subagent_type: general-purpose` for each.

Build each subagent's prompt as `_shared.md` + the dimension file, concatenated verbatim:

1. `Read` `~/.claude/skills/refactor/prompts/_shared.md` **once** (read-only constraints, severity rubric, output schema), and each dimension file:

   | Dimension                            | Prompt file                                        |
   | ------------------------------------ | -------------------------------------------------- |
   | Correctness                          | `~/.claude/skills/refactor/prompts/correctness.md` |
   | Performance / idiomatic / anti-fluff | `~/.claude/skills/refactor/prompts/idiomatic.md`   |
   | Readability (structural)             | `~/.claude/skills/refactor/prompts/readability.md` |

2. Substitute these placeholders in the shared contents:
   - `<FILE LIST>` → the confirmed scope from Step 1, one path per line.
   - `<MEMORY PATH>` → the absolute path to this project's auto-memory `MEMORY.md`, as given in the Memory section of your system prompt.
3. Pass shared + dimension contents as the `prompt` argument to the Agent call. Do not edit or paraphrase anything else. If you find yourself wanting to add boilerplate at dispatch time, fix the prompt files instead.

## Step 3 — Merge and render

After all three subagents return:

1. **Dedupe**: when two subagents flag the same `(file, line)` with semantically equivalent findings, merge into one entry. Keep the highest severity and a combined dimension tag (e.g. `Correctness + Idiomatic`).
2. **Build TL;DR**: list every Must-fix as a one-liner with `file:line`. Include counts of Should-fix and Nice-to-have.
3. **Render by dimension** in this order: Correctness → Idiomatic & anti-fluff → Readability. Within each, sort by severity (Must-fix → Should-fix → Nice-to-have), then by file.
4. **Recommend a pick**: take every Must-fix and Should-fix by default. From Nice-to-have, pick only the items worth the change for this repo, each with a one-line reason, and list the skipped codes on one line. Zero Nice-to-have picks is a valid answer; say so. With no Must-fix or Should-fix, the pick is the Nice-to-have choice alone.
5. **Print inline only.** Do not write any file.

If a subagent errors or returns nothing, render the remaining sections normally with a one-line note: `_<dimension> review unavailable — subagent failed_`.

### Output template

Each finding gets a **short code** (`C1`, `C2` … for Correctness; `I1`, `I2` … for Idiomatic; `R1`, `R2` … for Readability). Codes are stable within the response — the user can say "apply C1, skip I2" to act on specific items.

```
# /refactor — review for <scope summary>

## TL;DR
- N Must-fix • M Should-fix • K Nice-to-have
- <numbered list of Must-fix one-liners: C1 `file:line` — …>

## Correctness
### Must-fix
**C1** `path/file.svelte:42` — <finding>
  - Why: <reason>
  - Suggested: <fix sketch>
**C2** `path/file.svelte:88` — <finding>
  …
### Should-fix
**C3** …
### Nice-to-have
**C4** …

## Idiomatic & anti-fluff
### Must-fix
**I1** …
…

## Readability
### Must-fix
**R1** …
…

## Recommended Pick
- Take: C1, C3, I1, R2 (every Must-fix and Should-fix)
- Nice-to-have: I4 — <one-line reason>; R5 — <one-line reason>
- Skip: C4, I6, R7
Reply `go` to plan this pick, or adjust it (e.g. "drop I4, add R7").
```
