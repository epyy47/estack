---
name: pr-comments
description: Fetch the unresolved review comments on a PR (any reviewer, bot or human), validate each one against the code, and propose a fix for the legit ones. Use when the user runs /pr-comments [free text], or asks to get/validate PR comments, e.g. "get the 2 latest codex comments and validate them" or "validate this <discussion URL>". Never edits files before an explicit go.
---

# /pr-comments — Validate PR Review Comments

Read-only until the user says `go`.

## 1. Fetch

Default: every unresolved review thread on the current branch's PR. The free-text argument narrows it:

| Text | Effect |
|---|---|
| `#N` or a PR URL | that PR instead of the current branch's |
| a `#discussion_r<ID>` URL | that one comment only |
| a word like `codex`, `greptile`, a login | keep threads whose first comment author login contains the word (case-insensitive) |
| `P1`, `P2`, `P3` | keep bodies with that badge (Codex: `![P1 Badge]`, Greptile: `<img alt="P1"`) |
| `latest`, `2 latest` | sort by createdAt, keep the last N |
| `all` | include resolved threads too |

```sh
PR=${PR:-$(gh pr view --json number -q .number)}
gh api graphql -f owner="$(gh repo view --json owner -q .owner.login)" -f name="$(gh repo view --json name -q .name)" -F pr="$PR" -f query='
query($owner:String!,$name:String!,$pr:Int!){ repository(owner:$owner,name:$name){ pullRequest(number:$pr){ headRefName
  reviewThreads(first:100){ nodes{ id isResolved isOutdated path line originalLine
    comments(first:10){ nodes{ databaseId createdAt author{login} url body } } } } } } }' \
--jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved==false)
  | "=== thread=\(.id) outdated=\(.isOutdated) \(.path):\(.line // .originalLine)\n"
    + (.comments.nodes | map("[\(.databaseId)] \(.author.login) \(.createdAt)\n\(.body)") | join("\n---\n"))'
```

Single comment: `gh api repos/{owner}/{repo}/pulls/comments/<ID> --jq '{user:.user.login,path,line,original_line,body,diff_hunk}'`.

Print an inventory line per selected thread: `#  databaseId  author  severity  path:line  topic`. If fewer match than the user asked for, say so. Do not guess.

Note: the user's own replies also sit in these threads. Read the whole thread, not only the first comment, so a point the user already answered is not re-validated.

## 2. Validate

For each thread, check the claim against the real code and cite `path:line`:

1. Read the cited code on the PR branch.
2. Is the issue new here, or already on `origin/main`? (`git show origin/main:<path>`)
3. Find the sibling sites with the same shape. A fix covers all of them (Pattern over YAGNI).
4. Find how `main` already solves this class of problem. Propose that shape. Say "no precedent" if none.
5. Take any behaviour the user declared intended as given. Offer reply options, not a fix.
6. Merge two threads that raise the same issue into one entry.

## 3. Report, then stop

```
# /pr-comments — PR #<n> <title>

| # | Topic | Author | Sev | Verdict |
|---|---|---|---|---|

## 1. <topic> — `path:line` — Legit
- Claim: <one sentence>
- Evidence: <path:line bullets>
- Severity: <agree / overstated because …>
- Siblings: <list / none>
- Precedent: `path:line` / none
- Fix: <file → exact change>, <test → failing case first>

## 2. <topic> — `path:line` — Not legit
- Why: <path:line evidence>. Reply draft: "<one line>"

## 3. <topic> — Intended
- Options: A <recommended> / B

Say `go` (or `go without 2`) to implement, or `reply` to draft thread replies.
```

Verdicts: **Legit**, **Not legit**, **Intended**. Qualify a Legit with "lower severity" or "already on main" when true.

## After `go`

- Implement through subagents, one per group of files, TDD where the test tiers apply. Run the tests and lint on the changed files only.
- Do not stage, commit or push. Hand off to `/commit-local`.
- `reply`: draft every reply in chat, post only after approval, one per comment:
  `gh api -X POST repos/{owner}/{repo}/pulls/<N>/comments/<ID>/replies -F body=@/tmp/reply-<ID>.md --jq .html_url`
  A reply that cites a SHA needs the push first. Check `git status -sb` and say if the branch is ahead.
- `resolve`: only when asked, per thread id from step 1:
  `gh api graphql -f query='mutation{resolveReviewThread(input:{threadId:"<THREAD_ID>"}){thread{isResolved}}}'`
- No AI attribution anywhere.
