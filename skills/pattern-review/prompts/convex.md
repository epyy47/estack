Your group is **Convex** (`src/convex/**`). Inventory these shapes in every in-scope file and check each against `origin/main`:

- **Schema**: table and field naming, index names (the `by_<fk>` rule vs compound indexes like `by_template_seq`), optional vs nullable fields, enum unions and where their literals live.
- **Validators**: `v.optional(v.union(<type>, v.null()))` for clearable fields, `clearNulls` usage before `patch`/`insert`, where arg validators live.
- **Auth checks**: `requireAdmin` vs `requireFeature`, one check per function, consistency within a module.
- **Helpers**: where helper functions live (`*Helpers.ts`), server-side trim and blank checks, uniqueness checks (case-sensitive or not), what a helper returns.
- **Query shape**: join and label fields returned (nickname, company code), `.unique()` vs `.first()`, pagination and batch-delete limits, `Promise.all` cascades.
- **Ordering**: `seq` fields and reorder mutations.
- **Migrations and seeds**: file location under `migration/`, idempotency, header comment shape.
- **Tests**: `convex-test` files beside the module, `// @vitest-environment edge-runtime`.
- **Error handling**: `ConvexError` messages, `convexErrorMessage` on the client side.

Explicitly **out of scope**. Do not flag: performance of a query, N+1, and bugs. `/refactor` covers them.
