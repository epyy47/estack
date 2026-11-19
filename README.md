# agent-setup

Shared Claude Code configuration: engineering standards prompt, custom skills, and CI workflow templates.

## Structure

```
agent-setup/
├── .gitattributes              # Line ending rules and binary asset declarations
├── system-prompt.md            # Install as user-scope CLAUDE.md
├── commit-message-format.md    # Conventional Commits rules
├── github-workflows/           # Drop-in GitHub Actions templates
│   ├── pr-checks.yml           # Lint, check, test & build on PRs to main
│   ├── deploy.yml              # Vercel deployment (npm)
│   └── deploy-pnpm.yml         # Vercel deployment (pnpm)
├── output-styles/              # Claude Code output styles
│   └── ste100.md               # ASD-STE100 Simplified Technical English responses
└── skills/                     # Claude Code custom skills
```

## Skills

| Skill                      | What it does                                                                                                          |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `commit-local`             | Conventional Commits message from the staged diff, then a local commit after approval. No staging, push, or fetch.    |
| `create-pr`                | Opens a PR via `gh`, generating title and body from the branch diff and commits.                                      |
| `refactor`                 | Severity-ranked pre-commit review across correctness, idiomatic/perf, and readability. Reports only, applies nothing. |
| `grilling`                 | Interviews you about a plan or design until every branch of the decision tree is resolved.                            |
| `batch-grilling`           | Same interview, batched — asks every currently-answerable question per round instead of one at a time.                |
| `write-a-skill`            | Scaffolds a new skill with proper structure and progressive disclosure.                                               |
| `convex`                   | Router — points underspecified Convex requests at the right skill below.                                              |
| `convex-migration-helper`  | Schema and data migrations via widen-migrate-narrow and `@convex-dev/migrations`.                                     |
| `convex-performance-audit` | Diagnoses slow reads, subscription cost, OCC conflicts, and function limits.                                          |

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
