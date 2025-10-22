## Repo summary

Small Cloudflare Workers TypeScript project. Entry point: `src/index.ts` (exports an `ExportedHandler<Env>`). Tests use Cloudflare's Vitest worker pool (`@cloudflare/vitest-pool-workers`) and live/integration style tests from `test/index.spec.ts`.

## What you need to know to be productive

- This is a Cloudflare Worker built with Wrangler. Primary developer scripts are in `package.json`:
  - `pnpm run dev` / `pnpm run start`: run `wrangler dev` locally (hot-reload worker runtime)
  - `pnpm run deploy`: `wrangler deploy` to publish
  - `pnpm run test`: run `vitest` (integration tests use `SELF.fetch` and the Cloudflare test pool)
  - `pnpm run cf-typegen`: regenerate `worker-configuration.d.ts` from Wrangler types

- Configuration lives in `wrangler.jsonc`. Add bindings (KV, Durable Objects, secrets, etc.) there and re-run `pnpm run cf-typegen` to update TypeScript `Env` types.

- Runtime types are already generated in `worker-configuration.d.ts`. Use those types for accurate `Env` and Request/Response shapes.

## Code patterns and conventions

- Handler shape: default export is an object with a `fetch` method, typed as `ExportedHandler<Env>` (see `src/index.ts`). Keep side-effects off the top-level module where possible; prefer `ctx.waitUntil()` for background work.

- Tests:
  - Unit-style tests call the handler directly: create a `Request` instance and `createExecutionContext()` from `cloudflare:test` and call `worker.fetch(request, env, ctx)`.
  - Integration-style tests use `SELF.fetch('https://example.com')` to exercise the worker as a running service (the test pool uses the `wrangler.jsonc` config, see `vitest.config.mts`).

- Type helpers: tests and code rely on the generated typings in `worker-configuration.d.ts`. If you add bindings, run `pnpm run cf-typegen` so editors and tests remain typed.

## Testing and debugging tips

- To run tests locally in the cloudflare worker environment: `pnpm run test`. The vitest config uses the Cloudflare worker pool and points to `./wrangler.jsonc` so the runtime matches your local wrangler config.

- To debug interactively: `pnpm run dev` then visit http://localhost:8787/ (default Wrangler dev host). Use console logs or the Worker Devtools in the browser.

## Files to consult when making changes

- `src/index.ts` — main handler
- `wrangler.jsonc` — runtime configuration and bindings
- `worker-configuration.d.ts` — generated runtime TypeScript types (run `pnpm run cf-typegen` to refresh)
- `test/index.spec.ts` and `vitest.config.mts` — test patterns and Cloudflare test pool setup
- `package.json` — developer scripts

## Useful examples to copy from

- Test pattern (unit-style): see `test/index.spec.ts`, the pattern of creating `IncomingRequest`, `createExecutionContext()` and `waitOnExecutionContext(ctx)` before asserting response text.

## Constraints / gotchas

- Secrets and environment values should be set via `wrangler` secrets or `vars` in `wrangler.jsonc`. Do not hardcode secrets into source.
- When you change bindings in `wrangler.jsonc` remember to regenerate types (`pnpm run cf-typegen`) or tests may break due to missing `Env` properties.
- The project uses `@biomejs/biome` for formatting (`pnpm run lint` runs `biome format --write`) — follow existing formatting.

## When you edit this file

If you add or remove build/test workflows, update the scripts in `package.json` and this document. If you add bindings, update `wrangler.jsonc` and regenerate `worker-configuration.d.ts`.

---
If anything here is unclear or you want more detail (deployment environment, expected bindings, CI commands), tell me what I should add and I will iterate.
