Your group is **UI** (`src/routes/**`, `src/lib/**`). Inventory these shapes in every in-scope file and check each against `origin/main`:

- **Route layout**: page root structure, `+page.svelte` vs child components, `checklist.md` per route folder, where `useQuery` fires (page level vs component).
- **Dialog shells**: `{#if open}{#key initial?._id ?? 'create'}<XFormBody/>{/key}{/if}`, config-object variants, `DialogFormFooter`, scroll-trap classes.
- **Form bodies**: `superForm(defaults(seed, zod4(schema)))`, `untrack` snapshot, `*-field.svelte` wrappers vs hand-rolled `Form.Field`, `SelectField` for enums.
- **Data tables**: `DataTableToolbar`, `DataTableScrollArea`, `debouncedSearch`, `ListScopeSwitcher` layer, column `meta.cellClass`.
- **Shared components under `src/lib/components`**: new props, new snippets, changed sizes or widths, and every caller on main.
- **Composition**: snippets vs child components, where types live (`<script module>` exports vs a `.ts` file like `detail-helpers.ts`), two-way imports.
- **Tabs**: mount all vs `{#if tab === x}` per tab.
- **Labels and text**: Title Case vs sentence case, `font-mono` sites, `data-icon`, `#each` keys, `aria`/`sr-only` text shape.
- **Mutations from the client**: `$derived` snapshot before `await`, toast wording, `convexErrorMessage`.

Explicitly **out of scope**. Do not flag: visual polish, readability, bugs. `/refactor` covers them.
