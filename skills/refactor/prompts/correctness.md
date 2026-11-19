Your dimension is **Correctness**. Focus on:

- Logic bugs in changed code: off-by-one, wrong conditionals, missing null checks at boundaries, broken control flow, mishandled edge cases.
- Violations of `CLAUDE.md` (global or project) or any feedback memory rule. Scan MEMORY.md's one-line entries and read whichever linked memory files look relevant to the files in scope — each typically encodes a recurring trap the user has been bitten by before.
- Concurrency / async correctness: stale `$derived` after `await`, unawaited promises, race conditions, missing Convex OCC retries.

Explicitly **out of scope** — do not flag:

- Anything `tsc` would catch (type mismatches, missing properties, implicit any). The user runs typecheck separately.
