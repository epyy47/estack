# estack

Shared Claude Code configuration: engineering standards prompt, custom skills, and CI workflow templates.

## Structure

```
estack/
├── .gitattributes              # Line ending rules and binary asset declarations
├── system-prompt.md            # Install as user-scope CLAUDE.md
├── commit-message-format.md    # Conventional Commits rules
├── github-workflows/           # Drop-in GitHub Actions templates
│   ├── PULL_REQUEST_TEMPLATE.md # PR body template (Why / What changed / Verify / Notes)
│   ├── pr-checks.yml           # Lint, check, test & build on PRs to main
│   ├── deploy.yml              # Vercel deployment (npm)
│   └── deploy-pnpm.yml         # Vercel deployment (pnpm)
├── output-styles/              # Claude Code output styles
│   └── ste100.md               # ASD-STE100 Simplified Technical English responses
└── skills/                     # Claude Code custom skills
    ├── grilling/
    ├── pattern-review/
    ├── pr-comments/
    └── refactor/
```

## Skills

| Skill            | What it does                                                                                                                                                                           |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `refactor`       | Severity-ranked pre-commit review across correctness, idiomatic/perf, and readability. Reports only, applies nothing.                                                                  |
| `pattern-review` | Compares a PR branch with `origin/main` and lists each new or different pattern with a verdict (Align / Adopt / Keep local). Can post the findings as a PR comment. Never edits files. |
| `pr-comments`    | Gets the unresolved review comments on a PR, checks each one against the code, and proposes fixes for the valid ones. Edits nothing before an explicit go.                             |
| `grilling`       | Interviews you about a plan or design until every branch of the decision tree is resolved.                                                                                             |

Copy to `~/.claude/skills/` for every project, or a repo's `.claude/skills/` for just that one:

```bash
cp -r skills/refactor ~/.claude/skills/
```

Then invoke by name — `/refactor`.

## Output Styles

Copy to `~/.claude/output-styles/`:

```bash
cp output-styles/ste100.md ~/.claude/output-styles/
```

Then select with `/output-style ste100`. `ste100` makes responses follow ASD-STE100 Simplified Technical English writing rules.

## GitHub Workflows

Copy the templates you need into a project's `.github/workflows/`:

```bash
cp github-workflows/pr-checks.yml /path/to/project/.github/workflows/
```

`deploy.yml` and `deploy-pnpm.yml` are the same Vercel pipeline for npm and pnpm respectively — take one, not both. Both expect `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`, and `VERCEL_TOKEN` as repository secrets.

`PULL_REQUEST_TEMPLATE.md` goes in `.github/`, not `.github/workflows/`:

```bash
cp github-workflows/PULL_REQUEST_TEMPLATE.md /path/to/project/.github/
```
