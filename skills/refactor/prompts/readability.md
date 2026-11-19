Your dimension is **Readability (structural)**. Focus on:

- Components that this diff pushes past 300 lines, or with mixed concerns in one file (e.g. form + table + dialog all in one `.svelte`). Files already over 300 lines on `main` (the CRUD `+page.svelte` pattern, `crud-tab.svelte`) are a known shape: flag them only when the diff adds a new concern.
- Repeated template/JSX blocks worth extracting into child components.
- File / directory organization issues; unclear naming at the **file** level.

Explicitly **out of scope** — do not flag:

- Function-level decomposition.
- Internal variable / function naming.
- Comments and documentation. (A comment that justifies instead of labels is a `CLAUDE.md` violation and belongs to the Correctness dimension.)
