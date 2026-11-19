## Evidence-First Reasoning

- Retrieve evidence (code, docs, logs, git history) before concluding, and cite it.
- Say "no", "I don't have that information", or disagree when evidence warrants. Conclusions come from evidence, not agreement.

## TDD When Planned

When a plan specifies TDD, strict red-green-refactor:

- Failing test BEFORE any implementation → minimum code to pass → refactor only after green.
- Don't skip steps for efficiency — the cycle is the required approach.
- No plan mentioning TDD → normal implementation flow.

## Do Not Overrun Project Scripts

- Run check, lint and test scripts only when necessary or right before handover the work back to user.
- When running lint scripts, default run against the changed files for speed instead of the entire codebase (leave this to the user or request from the user if necessary)

## Code Comments Label, Not Justify

- Comments are terse labels only (e.g. `// --- Party enums ---`). No asides explaining design tradeoffs, rationale, or citing CLAUDE.md rules, that reasoning goes in the chat response or PR body.

## Plan Before Implement

- Show the full concrete fix plan (files, exact changes, rationale) as a plain chat message and wait for an explicit go-ahead before any edit, even after the recommendation itself was approved. Plan mode is disabled in settings; never try `EnterPlanMode`.

## Delegate Implementation to Subagents

- After plan approval, implement via subagents, not inline in the main thread; the main context holds the plan, decisions, and review, not file contents and diffs. Use parallel Explore agents by default on anything multi-file.
- Split the plan into phases that don't share files, one agent per phase (parallel when independent, sequential when a later phase consumes an earlier one's output). Give each agent exact file paths, patterns to copy, and verification commands; they start with no context. Relay the outcome, not the diffs.

## Label the Recommended Option

- In `AskUserQuestion`, put the recommended choice first AND end its label with `(Recommended)`. Keep the justification in the description; burying the recommendation there alone is not enough.

## Pattern over YAGNI for Siblings

- When a fix applies to several sibling call sites, apply it to every one, including a site still under its limit today (e.g. 300 rows against a ~585-row Convex transaction ceiling). Say what the ceiling is, then include it; do not argue YAGNI for it.
- Coverage across every sibling is the rule, not inlining. Extract or reuse a shared helper freely (e.g. `src/lib/bulk-delete.ts`).
