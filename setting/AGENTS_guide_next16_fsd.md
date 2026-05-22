# AGENTS.md

This file defines the operating rules for AI coding agents working in this repository.

Keep this file under 500 lines. Do not duplicate README, docs, or framework documentation. Add only agent-specific rules that affect implementation decisions.

## Operational Commands

Use pnpm only.

Do not use npm, yarn, bun, or generated lockfiles from other package managers.

Use these commands from the repository root:

- Install dependencies: `pnpm install`
- Development server: `pnpm dev`
- Production build: `pnpm build`
- Type check: `pnpm typecheck`
- Lint: `pnpm lint`
- Unit tests: `pnpm test:unit`
- E2E tests: `pnpm test:e2e`

Production build must be executed as `pnpm build`.

Do not run `next build` directly unless the user explicitly asks.

Do not run `next build --webpack`.

Do not add Webpack configuration. Next.js 16 uses Turbopack for development and production builds by default.

If a script is missing, inspect `package.json` first and add or adjust the script intentionally. Do not invent commands.

## Project Context

This is a Next.js 16 App Router frontend project using Cache Components and Feature-Sliced Design.

Tech stack:

- Next.js 16
- React
- TypeScript
- Tailwind CSS
- pnpm
- Turbopack
- Vitest
- Playwright
- Feature-Sliced Design

## Source of Truth

Before changing Next.js routing, rendering, caching, metadata, server actions, route handlers, proxy, or build behavior, read the installed Next.js docs first:

- `node_modules/next/dist/docs/`

Before changing React and Next.js performance-sensitive code, use the `vercel-react-best-practices` skill.

Do not copy the skill contents into this file.

## Golden Rules

### Immutable Rules

- Use `pnpm` only.
- Use `pnpm build` as the only production build command.
- Keep Turbopack as the bundler.
- Keep `cacheComponents: true`.
- Do not use Webpack escape hatches.
- Do not use `middleware.ts`; use `proxy.ts` for Next.js 16 request interception.
- Do not expose secrets to Client Components.
- Do not commit local secret files.
- Do not use `any`.
- Do not use type assertions to bypass type errors.
- Do not silence TypeScript with `@ts-ignore` or `@ts-expect-error` unless the reason is documented and no safer option exists.

### Do

- Prefer Server Components by default.
- Add `"use client"` only when browser APIs, state, effects, event handlers, or client-only libraries are required.
- Keep route files thin.
- Use Suspense boundaries for request-time dynamic data.
- Push mutation behavior into `features`.
- Keep `entities` read-only.
- Keep public APIs explicit with `index.ts`.
- Prefer type narrowing, schema validation, type guards, and `satisfies` over assertions.
- Verify each meaningful step before moving to the next.

### Don't

- Do not move code to the client to avoid server/data-flow decisions.
- Do not await unrelated data sequentially.
- Do not put create, update, delete, submit, login, logout, or other user actions in `entities`.
- Do not import another slice's internal files.
- Do not use wildcard public API exports.
- Do not add dependencies for small utilities.
- Do not hardcode tokens, API keys, cookies, or secret values.
- Do not print secret values in logs, chat, commits, or test output.

## Next.js 16 Rules

### Cache Components

This project uses `cacheComponents: true`.

When Cache Components are enabled, do not use these route segment configs:

- `export const dynamic = "force-dynamic"`
- `export const dynamic = "force-static"`
- `export const revalidate = ...`
- `export const fetchCache = ...`
- `export const dynamicParams = ...`

Use these mechanisms instead:

- `use cache`
- `use cache: private`
- `use cache: remote`
- `cacheLife`
- `cacheTag`
- `updateTag`
- `revalidateTag`
- `<Suspense>`
- `connection()` when dynamic rendering is required and request-time APIs are not already used

GET Route Handlers follow the Cache Components rendering model. Apply the same caching caution to them.

### Cache Approval Protocol

Before adding, removing, or changing caching behavior, ask the user unless the task already provides an explicit cache policy.

Caching behavior includes:

- adding or removing `use cache`
- adding or removing `use cache: private`
- adding or removing `use cache: remote`
- changing `cacheLife`
- changing `cacheTag`
- changing `updateTag` or `revalidateTag`
- changing `generateStaticParams`
- changing whether a component is streamed or cached

Ask for:

- whether the data may be cached
- whether the data is public, private, or per-user
- desired freshness or TTL
- invalidation trigger
- whether stale data is acceptable

Never cache authorization decisions, permissions, sessions, secrets, or user-specific private data unless the user explicitly confirms the policy.

If data must be fresh for every request, do not use `use cache`. Wrap the component in `<Suspense>` and stream it at request time.

Read `cookies()`, `headers()`, `params`, and `searchParams` outside cached scopes. Pass primitive values into cached functions or components only when the cache policy allows it.

Do not pass unresolved runtime Promises into a cached function or component.

### Await and Waterfall Rules

Do not fetch all page data at the top of `page.tsx` just to pass props down.

Prefer this structure:

- `page.tsx` composes the route shell.
- independent route sections are separate Server Components.
- each async section awaits its own data.
- request-time sections are wrapped in `<Suspense>`.
- cached sections declare their own cache policy.

Do not create sequential waterfalls like:

- await A
- await B
- await C

when A, B, and C are independent.

For independent data in the same scope:

- start all promises first
- then await with `Promise.all`

For independent UI sections:

- colocate the async read in the section component
- wrap each section with Suspense when it may block request-time rendering

Only await at the page level when the result is required to decide the route, metadata, access control, or top-level composition.

If a parent Server Component awaits before rendering children, it can still block child rendering. Keep blocking awaits as low as possible.

### Server and Client Boundaries

Server Components are the default.

Client Components must be small and isolated.

Client Components must not import:

- database clients
- server-only modules
- private environment accessors
- secret-bearing code
- server actions except through supported form/action wiring

Use `"use client"` only at the smallest necessary boundary.

## Feature-Sliced Design Rules

### FSD Hierarchy

Use the FSD hierarchy:

- Layer
- Slice
- Segment

Layers define responsibility and dependency direction.

Slices divide a layer by business domain or product meaning.

Segments divide a slice by technical purpose.

This project uses Next.js `src/app` as the App Router and route composition boundary. Do not create a separate FSD `pages` layer unless the user explicitly asks.

Primary layers:

- `src/app`
- `src/widgets`
- `src/features`
- `src/entities`
- `src/shared`

Dependency direction:

- `app` may import from `widgets`, `features`, `entities`, `shared`
- `widgets` may import from `features`, `entities`, `shared`
- `features` may import from `entities`, `shared`
- `entities` may import from `shared`
- `shared` must not import from business layers

Forbidden imports:

- feature to feature
- entity to feature
- entity to widget
- shared to entity
- shared to feature
- shared to widget
- direct imports into another slice's internals

### Layer Responsibilities

`src/app`:

- Next.js routes
- layouts
- route-level composition
- providers
- metadata
- loading and error boundaries
- server actions only when route-local and not reusable
- proxy and app-level configuration

`src/widgets`:

- large reusable page blocks
- composition of multiple features and entities
- route section UI that is larger than a single feature

`src/features`:

- user actions
- create, update, delete
- submit flows
- authentication actions
- mutations
- feature forms
- feature-specific validation and state
- mutation API calls

`src/entities`:

- business entity representation
- read-only entity data access
- entity schemas and types
- entity display components
- pure entity transformations

`src/shared`:

- UI kit
- base utilities
- API primitives
- config
- routing constants
- framework-agnostic helpers
- no business-specific logic

### Entity Read-Only Rule

`entities` may contain read operations only.

Allowed in `entities`:

- `getUser`
- `readPost`
- `listProducts`
- entity mappers
- entity schemas
- entity display UI
- pure read transformations

Forbidden in `entities`:

- create
- update
- delete
- submit
- login
- logout
- follow
- like
- purchase
- upload
- send
- any mutation or command

Mutation logic belongs in `features`.

### Segment Definitions

Use these segment names inside slices:

- `ui`: components, visual presentation, local display formatters
- `model`: schemas, types, state, business rules, pure transformations
- `api`: backend interactions, request functions, DTO mappers
- `lib`: slice-local helpers used by this slice
- `config`: feature flags and slice-level configuration

Avoid vague segment names:

- `components`
- `hooks`
- `utils`
- `helpers`
- `types`

Use `ui`, `model`, `api`, `lib`, or `config` unless there is a strong project-specific reason.

### Public API and Encapsulation

Every slice must expose only its allowed external surface through `index.ts`.

External imports must use the slice public API.

Allowed:

- `import { LoginForm } from "@/features/auth"`
- `import { UserAvatar } from "@/entities/user"`
- `import { Button } from "@/shared/ui/button"`

Forbidden:

- `import { LoginForm } from "@/features/auth/ui/LoginForm"`
- `import { userSchema } from "@/entities/user/model/user-schema"`
- `import { formatDate } from "@/features/post/lib/format-date"`

Inside the same slice, use relative imports with explicit paths.

Do not import from the slice's own `index.ts` inside that slice.

Do not use wildcard exports:

- no `export * from "./ui/LoginForm"`
- no `export * from "./model/auth-model"`

Use named re-exports only:

- `export { LoginForm } from "./ui/LoginForm"`
- `export { loginSchema } from "./model/login-schema"`

Expose only what other layers need.

Do not expose internal helpers, private schemas, internal hooks, or temporary implementation details.

For `shared/ui` and `shared/lib`, prefer per-module public APIs to avoid oversized barrels:

- `@/shared/ui/button`
- `@/shared/lib/date`

## Testing Strategy

Use Vitest for unit tests.

Unit tests are only for pure functions in:

- `entities`
- `features`

Good unit test targets:

- schema parsing
- pure mappers
- pure validators
- pure formatting
- pure state transitions
- deterministic business rules

Do not unit test network communication.

Do not mock HTTP just to test request flows.

Move communication and runtime behavior tests to Playwright E2E.

E2E tests cover:

- browser flows
- route behavior
- forms
- server actions
- route handlers
- authentication flows
- cache invalidation behavior
- API communication
- user-visible integration behavior

Use `*.test.ts` for Vitest unit tests.

Use Playwright specs for E2E tests.

Before finishing pure function changes, run:

- `pnpm test:unit`
- `pnpm typecheck`

Before finishing route, cache, server action, or communication changes, run:

- `pnpm test:e2e`
- `pnpm build`

## Task Decomposition and Verification

Split work into small, verifiable steps.

Before implementation:

- identify the target layer
- identify whether the change is read or mutation
- identify whether the change affects caching
- identify the smallest verification command

After each step:

- verify the step before continuing
- prefer the smallest relevant check
- do not wait until the end to discover type, lint, or build failures

Use this verification order:

- pure logic change: `pnpm test:unit`
- type-level change: `pnpm typecheck`
- lint-sensitive change: `pnpm lint`
- route/render/cache change: `pnpm build`
- browser or communication change: `pnpm test:e2e`

When verification is unclear, ask the user how they want the change verified.

Pre-validate before committing or handing off to avoid slow lint-staged failures.

## Secrets and Env Safety

Treat environment variable values as sensitive unless they are known test-mode flags.

Never print or paste secret values in:

- chat responses
- commits
- shared logs
- test output
- screenshots
- documentation

Sensitive values include:

- tokens
- API keys
- cookies
- session IDs
- database URLs
- private URLs
- OAuth secrets
- webhook secrets

Mirror CI environment variable names and modes exactly.

Do not inline literal secret values in commands.

If a required secret is missing locally, stop and ask the user. Do not invent placeholder credentials for execution.

Never commit local secret files.

When documenting env setup, use placeholder-only examples.

When sharing command output, summarize and redact sensitive-looking values.

Client-exposed environment variables must use the approved public prefix only and must not contain secrets.

## Naming Conventions

React component files:

- PascalCase
- `.tsx`
- examples: `LoginForm.tsx`, `HeroBanner.tsx`

Logic, schema, hook, and utility files:

- kebab-case
- `.ts`
- examples: `login-schema.ts`, `use-auth.ts`, `format-korean-date.ts`

Folders:

- kebab-case
- examples: `hero-carousel`, `bulletin-list`

Variables and functions:

- camelCase
- examples: `formatKoreanDate`, `requireAuth`

Constants:

- UPPER_SNAKE_CASE
- examples: `ADMIN_ROUTES`, `BASE_URL`

Next.js reserved files keep official lowercase names:

- `page.tsx`
- `layout.tsx`
- `loading.tsx`
- `error.tsx`
- `not-found.tsx`
- `route.ts`
- `proxy.ts`

## TypeScript Rules

Use strict TypeScript.

Do not use `any`.

Do not use `as any`.

Avoid type assertions.

Prefer:

- inferred types
- explicit domain types
- `unknown` with narrowing
- user-defined type guards
- schema validation
- `satisfies`
- discriminated unions

A type assertion is allowed only when:

- the boundary is trusted
- runtime validation has already happened
- no safer TypeScript construct is practical
- the reason is clear in the code

Do not hide errors with:

- `@ts-ignore`
- unsafe casts
- fake non-null assertions
- broad object types
- placeholder interfaces

## Anti-Patterns

Do not do these:

- top-level `page.tsx` awaiting all independent data before rendering
- sequential awaits for independent data
- route segment configs removed by Cache Components
- adding `use cache` without user-approved cache policy
- caching personalized data accidentally
- using `dynamic = "force-dynamic"` as an escape hatch
- moving server logic into Client Components
- adding `"use client"` to large component trees
- putting mutations in `entities`
- importing slice internals from outside the slice
- wildcard public API exports
- feature-to-feature imports
- broad barrel files in `shared/ui` or `shared/lib`
- using `any`
- using type assertions to bypass errors
- hardcoding secrets
- logging environment values
- adding Webpack configuration
- using `next build --webpack`

## Standards and References

Do not duplicate README, docs, CONTRIBUTING, or framework documentation in this file.

Prefer references over copied content.

When project documentation and code conflict, report the mismatch instead of silently choosing one.

When adding a new convention, update the source document or propose a new one.

## Maintenance Policy

If these rules conflict with the current codebase, do not silently ignore the conflict.

Report the mismatch and propose one of:

- update the code to follow this file
- update this file to match the intended architecture
- add a temporary exception with a reason

When rules and code diverge, prefer making the divergence explicit.