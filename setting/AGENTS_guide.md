# AGENTS.md

This file defines operating rules for AI coding agents working in this repository.
Keep this file under 500 lines. Do not duplicate README, docs, CONTRIBUTING, or framework documentation. Add only agent-specific rules that affect implementation decisions.

## Project Variables

Fill these values when starting a project.

- Next.js version: `[NEXTJS_VERSION]`
- Architecture: `[ARCHITECTURE_NAME]`
- Architecture rules source: `[ARCHITECTURE_DOC_PATH_OR_SHORT_DESCRIPTION]`
- Package manager: `pnpm`
- Production build command: `pnpm build`
- Development command: `pnpm dev`
- Type check command: `pnpm typecheck`
- Lint command: `pnpm lint`
- Unit test command: `pnpm test:unit`
- E2E test command: `pnpm test:e2e`
- Bundler policy: `[Turbopack / Webpack / project-specific]`
- Caching mode: `[cacheComponents=true / previous model / custom / unset]`

If `Next.js version`, `Architecture`, or `Caching mode` is unset, inspect the codebase first and ask the user before making version-specific or architecture-specific decisions.

## Operational Commands

Use `pnpm` only. Do not use npm, yarn, bun, or lockfiles from other package managers.

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
Do not invent commands. If a script is missing, inspect `package.json` first and update it intentionally.

## Source of Truth

- Commands: `package.json` scripts.
- Next.js behavior: `node_modules/next/dist/docs/` when available.
- If installed docs are unavailable: official docs matching `[NEXTJS_VERSION]`.
- React and Next.js performance-sensitive work: `vercel-react-best-practices` skill when available.
- Agent behavior, verification policy, naming conventions, security rules, TypeScript safety rules, and filled architecture boundaries: this file.

Do not copy skill contents into this file.

## Project Context

This is a Next.js frontend project.
Business goal: `[WRITE_1_OR_2_SENTENCES_ONLY]`

Tech stack:

- Next.js `[NEXTJS_VERSION]`
- React
- TypeScript
- Tailwind CSS
- pnpm
- `[BUNDLER]`
- Vitest
- Playwright
- `[ARCHITECTURE_NAME]`

Do not add a directory tree here. Agents can inspect the codebase directly.

## Version-Specific Next.js Slot

Fill this section after choosing `[NEXTJS_VERSION]`.

Required project decisions:

- Routing model: `[App Router / Pages Router / hybrid]`
- Request interception file: `[proxy.ts / middleware.ts / none / version-specific]`
- Caching model: `[cacheComponents=true / previous model / custom]`
- Allowed route segment configs: `[LIST_OR_LINK]`
- Disallowed route segment configs: `[LIST_OR_LINK]`
- Data fetching rules: `[SHORT_POLICY]`
- Server Actions policy: `[SHORT_POLICY]`
- Route Handlers policy: `[SHORT_POLICY]`
- Metadata policy: `[SHORT_POLICY]`
- Build and bundler policy: `[SHORT_POLICY]`
- Deprecated APIs for this version: `[LIST_OR_LINK]`

Before changing routing, rendering, caching, metadata, server actions, route handlers, request interception, or build behavior:

- read the installed docs for the selected Next.js version
- verify whether the rule applies to `[NEXTJS_VERSION]`
- do not apply rules from another Next.js version without confirmation

Do not use a Next.js escape hatch unless it is valid for `[NEXTJS_VERSION]` and documented in this project.
If version-specific behavior is unclear, ask the user before changing it.

## Caching Policy Slot

Caching mode: `[CACHE_MODE]`

Fill these fields per project:

- Public or private data:
- Per-user or shared:
- Freshness requirement:
- TTL or lifetime:
- Invalidation trigger:
- Stale data allowed:
- Runtime APIs involved:
- Affected routes or components:

Before adding, removing, or changing caching behavior, ask the user unless the task already provides an explicit cache policy.

Caching behavior includes cache directives, cache lifetime, cache tags, invalidation, static or dynamic rendering decisions, route segment config changes, streaming or Suspense boundary changes, and generated params behavior.

Never cache these without explicit user confirmation:

- authorization decisions
- permissions
- sessions
- secrets
- tokens
- cookies
- private user data
- personalized responses

If data must be fresh for every request, do not add caching as an optimization without user approval.

## Await and Waterfall Rules

Do not fetch all page data at the top of `page.tsx` just to pass props down.

Prefer this structure:

- route file composes the route shell
- independent route sections are separate Server Components
- each async section owns its own read when possible
- blocking awaits stay as low as possible
- request-time sections use Suspense when appropriate
- cached sections declare their cache policy near the read

Do not create sequential waterfalls when requests are independent.
For independent data in the same scope, start all promises first and await them with `Promise.all`.

Only await at the route level when the result is required for access control, redirects, not-found decisions, metadata, top-level composition, or route-level errors.
If a parent Server Component awaits before rendering children, it can block child rendering. Keep blocking awaits as close to the consuming component as possible.

## Server and Client Boundaries

Server Components are the default.
Add `"use client"` only when the component requires browser APIs, React state, effects, event handlers, client-only libraries, or DOM access.
Client Components must be small and isolated.

Client Components must not import:

- database clients
- server-only modules
- secret accessors
- private environment variables
- backend SDKs intended for server use
- server-only utilities

Do not move code to the client to avoid server/data-flow decisions.

## Architecture Slot

Architecture: `[ARCHITECTURE_NAME]`
Architecture source: `[ARCHITECTURE_DOC_PATH_OR_SHORT_DESCRIPTION]`

Define these before implementing architecture-sensitive changes:

- top-level units:
- dependency direction:
- public API rule:
- allowed cross-module imports:
- forbidden cross-module imports:
- read ownership:
- mutation ownership:
- UI ownership:
- shared utility ownership:
- naming for modules:
- testing boundary:

If `[ARCHITECTURE_NAME]` is unset, do not invent boundaries. Inspect existing code and ask the user.

Architecture Golden Rules:

- `[RULE_1]`
- `[RULE_2]`
- `[RULE_3]`

Do not put generic frontend advice here. Do not list every folder. Do not duplicate documentation.

### Public API and Encapsulation

Each module, slice, or feature boundary must expose only its intended external surface.
Use `index.ts` only when it acts as a deliberate public API.
External imports must use public APIs.
Do not import another module's internal files unless the architecture explicitly allows it.
Do not use wildcard exports.

Avoid:

- `export * from "./internal-module"`
- broad barrels that expose implementation details
- importing from another module's `ui`, `model`, `api`, or `lib` internals

Prefer named re-exports:

- `export { SomeComponent } from "./ui/SomeComponent"`
- `export { someSchema } from "./model/some-schema"`

Expose only what other modules need. Do not expose internal helpers, temporary types, private schemas, or implementation-only hooks.

## Optional FSD Fill-In

Use this section only when `[ARCHITECTURE_NAME]` is Feature-Sliced Design. If the project does not use FSD, delete or ignore this section.

FSD hierarchy:

- Layer
- Slice
- Segment

Layers define responsibility and dependency direction.
Slices divide a layer by business domain or product meaning.
Segments divide a slice by technical purpose.

Common layers:

- `app`
- `widgets`
- `features`
- `entities`
- `shared`

Common segment names:

- `ui`
- `model`
- `api`
- `lib`
- `config`

Avoid vague segment names: `components`, `hooks`, `utils`, `helpers`, `types`.

Fill project-specific FSD policy:

- `app` responsibility:
- `widgets` responsibility:
- `features` responsibility:
- `entities` responsibility:
- `shared` responsibility:
- entity mutation policy:
- feature import policy:
- public API policy:

## Testing Strategy

Use Vitest for unit tests.
Unit tests are for pure functions only.

Good unit test targets:

- schema parsing
- pure validators
- pure mappers
- pure formatters
- pure state transitions
- deterministic business rules

Do not unit test network communication.
Do not mock HTTP just to test request flows unless the user explicitly asks.
Move communication and runtime behavior tests to Playwright E2E.

E2E tests cover browser flows, route behavior, forms, server actions, route handlers, authentication flows, cache invalidation behavior, API communication, and user-visible integration behavior.
Use `*.test.ts` for Vitest unit tests. Use Playwright specs for E2E tests.

Before finishing pure function changes, run:

- `pnpm test:unit`
- `pnpm typecheck`

Before finishing route, cache, server action, or communication changes, run:

- `pnpm test:e2e`
- `pnpm build`

If a required command is missing, inspect `package.json` and update scripts intentionally.

## Task Decomposition and Verification

Split work into small, verifiable steps.

Before implementation, identify target layer or module, read or mutation behavior, caching impact, server or client boundary, and the smallest verification command.
After each step, verify before continuing. Prefer the smallest relevant check. Do not wait until the end to discover type, lint, or build failures.

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

Never print or paste secret values in chat responses, commits, shared logs, test output, screenshots, or documentation.
Sensitive values include tokens, API keys, cookies, session IDs, database URLs, private URLs, OAuth secrets, and webhook secrets.

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

Next.js reserved files keep official names: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, `route.ts`, `proxy.ts`, and `middleware.ts` only if valid for `[NEXTJS_VERSION]`.

## TypeScript Rules

Use strict TypeScript.
Do not use `any` or `as any`.
Avoid type assertions.

Prefer inferred types, explicit domain types, `unknown` with narrowing, user-defined type guards, schema validation, `satisfies`, and discriminated unions.

A type assertion is allowed only when the boundary is trusted, runtime validation has already happened, no safer TypeScript construct is practical, and the reason is clear in the code.

Do not hide errors with `@ts-ignore`, unsafe casts, fake non-null assertions, broad object types, or placeholder interfaces.
Use `@ts-expect-error` only when the error is intentional, temporary, and documented.

## Dependency Rules

Before adding a dependency:

- check whether the project already has an equivalent
- check whether the dependency is needed on the client
- avoid large client-side dependencies
- prefer small, maintained packages
- explain why the dependency is needed

Do not add dependencies for small utilities that can be implemented safely in a few lines.
Do not change the package manager. Do not create additional lockfiles.

## Anti-Patterns

Do not do these:

- top-level `page.tsx` awaiting all independent data before rendering
- sequential awaits for independent data
- applying Next.js rules from the wrong version
- changing caching behavior without a cache policy
- caching personalized data accidentally
- moving server logic into Client Components
- adding `"use client"` to large component trees
- importing across architecture boundaries without approval
- importing module internals from outside the module
- wildcard public API exports
- broad barrel files that expose internals
- using `any`
- using type assertions to bypass errors
- hardcoding secrets
- logging environment values
- adding dependencies without checking existing options
- changing package managers
- using build or bundler escape hatches without user approval

## Standards and References

Do not duplicate README, docs, CONTRIBUTING, or framework documentation in this file.
Prefer references over copied content.
When project documentation and code conflict, report the mismatch instead of silently choosing one.
When adding a new convention, update the source document or propose a new one.
Keep this file focused on agent behavior and implementation constraints.

## Context Map

Use this section only when two or more nested `AGENTS.md` files exist. Do not add a context map for a small project.

Format:

- **[TRIGGER_OR_WORK_AREA](./relative/path/AGENTS.md)** — One-line reason to read this nested file.

Entries:

- **[ADD_WHEN_NEEDED](./path/AGENTS.md)** — Add only for high-context zones.

## Nested AGENTS.md Policy

Create a nested `AGENTS.md` only when a folder has a distinct context.

Valid reasons:

- separate package or dependency boundary
- different framework or runtime
- high-density business domain
- separate test strategy
- separate build or deployment behavior
- architecture rules that do not apply globally

Nested files must not repeat root rules. Nested files must include only local rules. Nested files must stay under 500 lines.

## Maintenance Policy

If these rules conflict with the current codebase, do not silently ignore the conflict.

Report the mismatch and propose one of:

- update the code to follow this file
- update this file to match the intended architecture
- add a temporary exception with a reason

When rules and code diverge, prefer making the divergence explicit.
