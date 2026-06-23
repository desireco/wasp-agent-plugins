This project uses Wasp, a batteries-included framework for building full-stack web apps with React, Node.js, and Prisma.

## Development Guidelines

### Start a Wasp Development Session with Full Debugging Visibility

Run the plugin's `start-dev-server` skill with the recommended options to give Claude full debugging visibility:

- Start the Wasp development server as a background task to give Claude direct access to server logs and build errors.
- Select the Chrome DevTools MCP server to give Claude visibility into browser console logs, UI functionality, network requests, and runtime errors.

### Documentation

Always fetch and verify your knowledge against the current Wasp documentation before taking on tasks, answering questions, or doing any development work in a Wasp project as your Wasp knowledge may be outdated:

1. Run `wasp version` to get the current Wasp CLI version.
2. Find and fetch the correct version of the Wasp documentation maps from the [LLMs.txt index](https://wasp.sh/llms.txt). The map contains raw markdown file GitHub URLs of all documentation sections.
3. Fetch the guides relevant to the current task or query from those raw.githubusercontent.com URLs directly - do NOT use HTML page URLs.

### Database Schema and Migrations

Always run database migrations with the `--name` flag:

```bash
wasp db migrate-dev --name <descriptive-name>
```

Changes to `schema.prisma` are not applied until database migrations are run.

**Track pending migrations:** The dev server warns about this, but users may miss it if Wasp is running as a background task. Continue coding freely but inform users of pending migrations before testing/viewing the app and offer to run migrations when the user wants to.

#### Migrations in non-interactive (agent) shells

`wasp db migrate-dev` runs Prisma's `migrate dev`, which **aborts in non-interactive environments** (no TTY) whenever it would normally prompt — e.g. adding a unique constraint that may cause data loss. There is no `--yes` flag.

```
Error: Prisma Migrate has detected that the environment is non-interactive, which is not supported.
```

Do **not** work around this by running raw `npx prisma` commands against `.wasp/out`. Wasp wraps Prisma with DB/auth setup, and the Wasp CLI docs warn that calling Prisma directly can desync that setup. If you hit the non-interactive abort in an agent/background shell, ask the user to run `wasp db migrate-dev --name <descriptive-name>` in their own terminal — it needs a TTY to prompt safely.

### Seeding the Database

Wasp ships a first-class seeding mechanism for database-wide initial data (dev fixtures, prod reference data) via `db.seeds`. Fetch the current docs for the declaration syntax. The agent-specific notes:

- `wasp db seed` runs the only defined seed, or prompts you to choose if multiple are defined. `wasp db seed <seed-name>` runs a specific one.
- Wasp does **not** track seeds as "already run" — `wasp db seed` executes them every time, so make seed functions idempotent (e.g. `upsert`).
- Use seeds for **database-wide** initial data, not **per-user** defaults. For data every user (including future signups) must have, run an idempotent Action from the client on load instead.

## Project Reference

### Config File Format

How you configure a Wasp app depends on the Wasp version. Detect the format before reading docs or editing the config:

- **`main.wasp`** → **Wasp DSL** (Wasp `< 0.24`): a custom config language (`app Name { ... }`).
- **`main.wasp.ts`** → TypeScript config, but in one of two flavors:
  - **TS Config** (Wasp `< 0.24`): imports `wasp-config`, uses `new App(...)` plus method calls like `app.page(...)`.
  - **Wasp Spec** (Wasp `>= 0.24`): imports `@wasp.sh/spec`, uses a single `app({ ..., spec: [...] })` call.

  The filename alone can't tell TS Config from Wasp Spec. Disambiguate by the import (`wasp-config` vs `@wasp.sh/spec`) or by running `wasp version`.

Always read the config docs for the **detected** format before editing (see [Documentation](#documentation)): the Wasp DSL and TS Config live under the **legacy** guides, the Wasp Spec under the **Wasp Spec** general docs.

### Structure

```
.
├── .wasp/                    # Wasp output (auto-generated, do not edit)
├── public/                   # Static assets
├── src/                      # Feature code: server `operations.ts` and client `pages.tsx` files
├── main.wasp or main.wasp.ts # Wasp config file: routes, pages, auth, operations, jobs, etc.
├── schema.prisma             # Database schema (Prisma)
```

### Recommended Code Organization

Unless user specifies otherwise, use a vertical, per-feature code organization (not per-type):

```
src/
├── tasks/
│   ├── tasks.wasp.ts      # Wasp Spec only (0.24+): per-feature config split
│   ├── TasksPage.tsx      # Page component
│   ├── TaskList.tsx       # Component
│   └── operations.ts      # Queries & actions
├── auth/
│   ├── auth.wasp.ts       # Wasp Spec only (0.24+)
│   ├── LoginPage.tsx
│   └── google.ts
```

Splitting config across per-feature `*.wasp.ts` files is a Wasp Spec feature (0.24+). With the Wasp DSL or TS Config, all config lives in the single `main.wasp` / `main.wasp.ts` file.

### Starter Templates

Highly recommend that the user chose one of the following templates when scaffolding a new Wasp app:

```bash
wasp new my-basic-app -t basic # creates a basic starter app with core Wasp features like auth, operations, pages, etc.
wasp new my-saas-app -t saas # creates a full-featured SaaS starter app with auth, payments, demo app, AWS S3, and more (OpenSaaS.sh)
```

See the **Starter Templates** section in the Wasp documentation for more templates.

### Customization

**Do NOT configure Vite, Express, React Query, etc. the usual way.** Wasp has its own mechanisms for customizing these tools. See the **Project Setup & Customization** section in the Wasp docs.

### Advanced Features

Wasp provides **advanced features**:

- custom HTTP API endpoints
- background (cron) jobs
- type-safe links
- websockets
- middleware
- email sending

See the **Advanced Features** section in the Wasp docs for more details.

### Wasp Conventions

#### Imports

**In TypeScript `src/` files** (same across all Wasp versions):

- ✅ `import type { User } from 'wasp/entities'`
- ✅ `import type { GetTasks } from 'wasp/server/operations'`
- ✅ `import { getTasks, createTask, useQuery } from 'wasp/client/operations'`
- ✅ `import { SubscriptionStatus } from '@prisma/client'` (for Prisma enums)
- ✅ Local code: relative paths `import { X } from './X'`

**In the config file**, the import syntax depends on the [config format](#config-file-format):

**Wasp DSL (`main.wasp`, `< 0.24`)** — import inside a declaration using the `@src` alias:

- ✅ `fn: import { getTasks } from "@src/tasks/operations"`
- ❌ Never relative paths

**TS Config (`main.wasp.ts` with `wasp-config`, `< 0.24`)** — use `{ import, from }` (or `{ importDefault, from }`) objects with the `@src` alias:

- ✅ `fn: { import: "getTasks", from: "@src/tasks/operations" }`
- ✅ `component: { importDefault: "MainPage", from: "@src/MainPage" }`

**Wasp Spec (`main.wasp.ts` with `@wasp.sh/spec`, `0.24+`)** — package imports are normal; your own code is imported with a **relative** path plus a `with { type: "ref" }` suffix:

- ✅ `import { app, page, route } from "@wasp.sh/spec";`
- ✅ `import App from "./src/App" with { type: "ref" };`
- ✅ `import { getTasks } from "./src/tasks/operations" with { type: "ref" };`

See the config docs for your version (linked from [Config File Format](#config-file-format)) for more details.

#### Operations

Wasp operations are Queries (read) and Actions (write), declared in the config and implemented in `src/`. Fetch the docs for the full API. The agent-specific workflow:

- **Declare every Entity an operation touches** in its `entities:` array — Wasp uses this both to inject `context.entities` and to auto-invalidate Query caches (see below).
- **Avoid manual `invalidateQueries` when entity-based invalidation already covers it** (see [Cache invalidation](#cache-invalidation-dont-write-it-yourself)).

##### Adding a new operation (the type-bootstrap loop)

Generated operation types (`GetTasks`, `CreateTask`, `MarkTaskAsDone`, …) live in `wasp/server/operations` but **only exist after you declare the operation in the config AND Wasp recompiles**. Expect this sequence every time:

1. Write the operation in `src/X/operations.ts`, annotating it with `satisfies GetFoo<Args, Output>` (or `const getFoo: GetFoo<…> = …`) — importing the type from `wasp/server/operations`. **You will see "no exported member" / `Cannot find name 'GetFoo'` errors here. This is expected, not a bug.**
2. Declare it in the config (`main.wasp.ts` for Wasp Spec): `query(getFoo, { entities: ["Foo"], auth: true })` or `action(createFoo, { entities: ["Foo"], auth: true })`.
3. Let Wasp recompile: if `wasp start` is running it watches and regenerates automatically; otherwise run `wasp compile` (not `wasp build`, which builds for production and clears `.wasp/out`).
4. The errors vanish. The `satisfies`/annotation now type-checks.

Do not try to "fix" step 1's errors before step 3 — they cannot be resolved until the type is generated.

##### Calling operations from the client

```ts
import { useQuery, getTasks, createTask } from "wasp/client/operations";

// Query
const { data, isLoading } = useQuery(getTasks, { lensId }, { enabled: !!lensId });

// Action — call directly with async/await (the default)
await createTask({ description: "…" });
```

⚠️ Call actions directly with `async/await` by default. Use Wasp's `useAction` hook only when you need **optimistic updates** — it is the only native manual cache-invalidation mechanism Wasp exposes (see below).

##### Cache invalidation (don't write it yourself)

Wasp **auto-invalidates** Query caches by shared Entity. If an Action and a Query both declare `entities: ["Task"]`, the Query refetches automatically after the Action runs. Per the docs: _"Wasp invalidates a Query's cache whenever an Action that uses the same Entity is executed… Wasp keeps the Queries 'fresh' without requiring you to think about cache invalidation."_

**Do not add manual `invalidateQueries` calls for queries that share an entity with the action.**

Manual cache control is only needed when:
- A Query spans entities whose dependency Wasp can't infer — declare **all** of them in the Query's `entities:` array so auto-invalidation covers it.
- You want **optimistic** updates (use the `useAction` hook's `optimisticUpdates` config).
- You need something beyond entity-based invalidation — fall back to React Query directly.

When you do need manual control, import the client from React Query, **not** from Wasp (Wasp does not re-export it):

```ts
import { useQueryClient } from "@tanstack/react-query"; // ✅ NOT wasp/client/operations
import { getTasks } from "wasp/client/operations";

const qc = useQueryClient();
qc.invalidateQueries({ queryKey: getTasks.queryCacheKey }); // Wasp exposes the cache key on the query
```

##### Client import cheat-sheet

| Import | From |
| --- | --- |
| `useQuery`, `useAction`, operation functions (`getTasks`, `createTask`, …) | `wasp/client/operations` |
| `useQueryClient` (manual cache control) | `@tanstack/react-query` |
| Server operation types (`GetTasks`, `CreateTask`) | `wasp/server/operations` (type-only) |
| Entity types (`Task`, `User`) | `wasp/entities` (type-only) |
| `sanitizeAndSerializeProviderData` (build `providerData` for an AuthIdentity, e.g. seeding a verified test user) | `wasp/server/auth` |

##### Creating a verified user for E2E / integration tests

For tests that need a pre-existing authenticated user, seed one directly with Wasp's auth helpers rather than driving the signup flow. Use `sanitizeAndSerializeProviderData` from `wasp/server/auth` — it hashes the password and returns the `providerData` JSON string the email provider expects (don't hand-roll it). The email provider requires `emailVerificationSentAt` and `passwordResetSentAt`:

```ts
import { sanitizeAndSerializeProviderData } from "wasp/server/auth";

const providerData = await sanitizeAndSerializeProviderData<"email">({
  hashedPassword: "TestPass123!",
  isEmailVerified: true,
  emailVerificationSentAt: null,
  passwordResetSentAt: null,
});

// Nested create: User → Auth → AuthIdentity (schema generated in .wasp/out/db/schema.prisma)
await prisma.user.create({
  data: {
    auth: {
      create: {
        identities: {
          create: {
            providerName: "email",
            providerUserId: "test@example.com",
            providerData,
          },
        },
      },
    },
  },
});
```

For pure client unit tests, prefer `mockServer` / `mockQuery` from `wasp/client/test` instead — see the Testing section of the Wasp docs.

## Troubleshooting

### Debugging

Always ground your knowledge against the [Wasp documentation](#documentation).

If you don't have full debugging visibility as described in the [Start a Wasp Development Session with Full Debugging Visibility](#start-a-wasp-development-session-with-full-debugging-visibility) section, do the following:

1. Insist that the user run the `start-dev-server` skill as described in the [Start a Wasp Development Session with Full Debugging Visibility](#start-a-wasp-development-session-with-full-debugging-visibility) section.
2. If the user refuses, ask them to share the output of the `wasp start` command and the browser console logs.

### Common Mistakes

| Symptom                                                      | Fix                                                                                                       |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| `context.entities.X undefined`                               | Add entity to `entities: [...]` in the Wasp config file                                                   |
| Schema changes not applying                                  | Run `wasp db migrate-dev --name <descriptive-name>`                                                       |
| Can't login after email signup with `Dummy` email provider   | Check the server logs for the verification link or set SKIP_EMAIL_VERIFICATION_IN_DEV=true in .env.server |
| Types stale/IDE errors after changes                         | Restart TS server `Cmd+Shift+P`                                                                           |
| Wasp not recognizing changes                                 | **WAIT PATIENTLY** as Wasp recompiles the project. Re-run `wasp start` if necessary.                      |
| Persistent weirdness after waiting patiently and restarting. | Run `wasp clean` && `wasp start`                                                                          |
| `Cannot find name 'GetTasks'` (or any `GetX`/`CreateX`) in a new operations file | The generated type doesn't exist until you declare the operation in the config **and** let Wasp recompile (`wasp start` watches, or `wasp compile`). See [Operations](#adding-a-new-operation-the-type-bootstrap-loop). |
| `Prisma Migrate … non-interactive environment`               | `migrate dev` aborts in agent/background shells. Don't run raw `npx prisma` — ask the user to run `wasp db migrate-dev --name …` in their terminal (see [Migrations in non-interactive shells](#migrations-in-non-interactive-agent-shells)). |
| `useQueryClient` is not exported from `wasp/client/operations` | Import it from `@tanstack/react-query` directly.                                                          |
| Mutations don't refresh the UI                               | First check that every overlapping Entity is declared in the Query's `entities:` — Wasp auto-invalidates by Entity. Only add manual `invalidateQueries` for cross-entity/optimistic cases. |
