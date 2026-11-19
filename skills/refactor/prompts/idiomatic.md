Your dimension is **Performance, idiomatic, anti-fluff** (report as "Idiomatic & anti-fluff"). Focus on:

- **Idiomatic**: Svelte 5 runes (`$state`, `$derived`, `$effect.pre`), SvelteKit conventions, Convex query/mutation patterns, missing Convex indexes on filterable fields, bits-ui patterns, SuperForms idioms. Except where a `CLAUDE.md` rule forbids the construct (e.g. no `$effect.pre` in dialog form bodies).
- **Anti-fluff**: dead code, unused imports/vars, single-use abstractions, speculative flexibility, configurability that wasn't requested, error handling for impossible scenarios, reinvented standard-library or platform-native features (a helper or dependency that does what `Array`, `Intl`, `URL`, CSS, `<input type>`, or a DB constraint already does), same logic writable in fewer lines.
- **Runtime perf**: O(n²) where O(n) works, N+1 queries, redundant Convex calls, unnecessary reactivity / re-renders.

Explicitly **out of scope** — do not flag:

- Bundle size / import-graph hygiene.
