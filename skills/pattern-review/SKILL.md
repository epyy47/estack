---
name: pattern-review
description: Compare a teammate's PR branch against origin/main and list every pattern with no precedent on main, or that differs from main's way, each with a verdict (Align / Adopt / Keep local). Optionally post the findings as a PR comment. Use when the user runs /pattern-review [PR number], asks to review a branch for new patterns, or asks whether a PR follows repo conventions. Never edits files.
---

# /pattern-review — New-Pattern Review of a PR Branch

Document-only review skill. It spawns two read-only subagents in parallel, one per code area, to compare the PR branch with `origin/main`, then merges their findings into one inline report with a verdict per pattern. It does not hunt for bugs; `/refactor` covers those. **Never edits files.**

Workflow position:

```
teammate: /grilling → implementation → /refactor → /create-pr
reviewer: /pattern-review [PR] → (post to PR / plan the Align items)
```

## Step 1 — Scope

1. If the user passed a PR number as the argument, run `gh pr checkout <n>`. If the checkout fails (dirty tree, no such PR, no `gh`), print the error and stop.
2. If the current branch is `main`, stop and say a PR branch must be checked out.
3. Run `git fetch origin main`.
4. Scope = `git diff --name-only origin/main...HEAD`. Committed changes only. Uncommitted work is out of scope because the branch belongs to a teammate. State this to the user.
5. Print the file list and the total count.
6. Run `gh pr view --json number,url --jq '"\(.number) \(.url)"'` to find the PR. If none, note "no PR found" and continue; Step 4 will be unavailable.
7. If the scope exceeds 30 files, print a one-line warning and continue. There is no confirmation gate: the PR defines the scope.

## Step 2 — Launch subagents in parallel

1. Split the scoped files into two groups:
   - **Convex**: paths under `src/convex/`.
   - **UI**: paths under `src/routes/` or `src/lib/`.
   - Any other path (docs, config, tests outside those trees) goes to the group whose files it sits closest to. If unclear, put it in UI.
2. Dispatch one `general-purpose` subagent per non-empty group, in **a single message with multiple Agent calls** so they run concurrently.
3. Build each subagent's prompt as `_shared.md` + the group file, concatenated verbatim. `Read` `/Users/sunray-my/.claude/skills/pattern-review/prompts/_shared.md` **once**, and each group file:

   | Group | Prompt file |
   |---|---|
   | Convex | `/Users/sunray-my/.claude/skills/pattern-review/prompts/convex.md` |
   | UI | `/Users/sunray-my/.claude/skills/pattern-review/prompts/ui.md` |

4. Substitute these placeholders in the shared contents:
   - `<FILE LIST>` → the group's file paths, one per line.
   - `<MEMORY PATH>` → the absolute path to this project's auto-memory `MEMORY.md`, as given in the Memory section of your system prompt.
   - `<PROJECT ROOT>` → the absolute path of the git repo root (`git rev-parse --show-toplevel`).
5. Pass the concatenated text as the `prompt` argument to the Agent call. Do not add boilerplate at dispatch time. If you want to add something, fix the prompt files instead.

## Step 3 — Merge and render

After the subagents return:

1. **Dedupe**: when two subagents describe the same pattern (same files, same shape), merge into one entry.
2. **Assign codes**: stable codes `P1`, `P2`, … in render order. The user refers to them later ("post without P2").
3. **Assign a verdict** to each New or Deviation entry, using the rules below. Each verdict needs a one-line reason. Each entry must cite the nearest `origin/main` precedent as `path:line`, or state "no precedent on main".
4. **Print inline only.** Do not write any file in this step.

If a subagent errors or returns nothing, render the rest and add one line: `_<group> review unavailable — subagent failed_`.

### Verdict rules

| Verdict | When |
|---|---|
| **Align with main** | `main` has a precedent for the same problem, and the branch differs with no reason tied to the new requirement. |
| **Adopt as precedent** | No precedent on `main`, the problem will recur elsewhere, and the shape fits the spirit of CLAUDE.md. The entry includes a one-line CLAUDE.md rule to add. |
| **Keep local** | No precedent, and the problem is specific to this feature. Or the divergence avoids a worse copy (for example, one generic single-field form body instead of two identical bodies). |

### Output template

```
# /pattern-review — <branch> vs origin/main (PR #<n> | no PR found)

<One paragraph: what the branch adds, in plain words. Commit count, file count, +/- lines.>

## New Patterns (no precedent on main)
**P1** <short pattern name> — `path/a.ts`, `path/b.svelte`
  - What: <the shape, one or two sentences>
  - Main: no precedent on main. Nearest: `path:line` — <how it relates>
  - Verdict: Adopt as precedent — <reason>. CLAUDE.md line: "<proposed rule>"
**P2** …
  - Verdict: Keep local — <reason>

## Deviations (main solves this another way)
**P3** <short pattern name> — `path/c.ts`
  - Branch: <what the branch does>
  - Main: `path:line` — <what main does>
  - Verdict: Align with main — <reason>

## Shared Component Changes
- `src/lib/components/x.svelte` — <change>. Callers affected: <list>. Visible effect: <one line or "none">.

## Checked, Aligned with Main
- <one-liner per rule or shape that matched a precedent>

## Noticed in Passing
- <at most a few items; clear bugs met by chance; "None" if empty>

## Recommended Pick
- Align with main: P3, P5
- Adopt as precedent: P1 (add the CLAUDE.md line)
- Keep local: P2, P4
Reply `post` to comment this on PR #<n>, `post without P2, P5` to drop items, or `plan` to fix the Align items.
```

## Step 4 — Post to the PR (hard gate)

1. Run this step only on an explicit `post` or `post without …` reply from the user. Never post on your own.
2. If Step 1 found no PR, say so and stop.
3. Build the body: the rendered text from Step 3 in the same words, minus any dropped codes, minus the `## Recommended Pick` section and the reply footer. Keep the remaining codes unchanged. Do not renumber. Drop a section header if its section becomes empty.
4. Write the body to `/tmp/pattern-review-<n>.md` with a quoted heredoc, then run `gh pr comment <n> --body-file /tmp/pattern-review-<n>.md`.
5. Print the comment URL that `gh` returns.
6. Add no AI attribution footer.

## `plan` reply

The skill ends here. The Align items go through the user's normal plan-first flow: show a concrete plan, wait for a go-ahead, then implement via subagents.
