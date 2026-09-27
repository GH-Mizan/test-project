# Project guidelines

This file is the source of truth for this **Node.js + TypeScript** app. Follow it when writing, reviewing, or changing code.

## Stack

- Runtime: Node.js (current LTS)
- Language: TypeScript, `strict` enabled
- Package manager: npm (lockfile committed)
- Source: `src/` — compiled output: `dist/` (do not edit `dist/` by hand)
- Module style: ESM (`"type": "module"` in `package.json`) unless the repo already uses CommonJS

## Layout

```
src/
  index.ts          # process entry
  ...
tests/              # or colocated `*.test.ts` next to source
dist/               # build output, gitignored
```

- Put new modules next to related code. Do not invent extra top-level folders without a reason.
- Public API of a file: named exports. Default export only for a single obvious module (e.g. the app factory).

## TypeScript

- Keep `strict` on. Do not weaken `tsconfig` to make a change compile.
- Prefer explicit types on exported functions and public APIs. Infer locally when it stays obvious.
- Do not use `any`. Use `unknown` and narrow. Use `satisfies` when it keeps object literals typed.
- Import Node built-ins with the `node:` prefix (`node:fs`, `node:path`, `node:http`).
- Prefer `async`/`await` over mixed promise chains. Never ignore a returned promise.
- Enable and respect path aliases only if they already exist in `tsconfig`; otherwise use relative imports.

```ts
// ❌ BAD
async function load(id: any) {
  fetch(`/items/${id}`);
}

// ✅ GOOD
async function load(id: string): Promise<Item> {
  const res = await fetch(`/items/${id}`);
  if (!res.ok) {
    throw new Error(`Failed to load item ${id}: ${res.status}`);
  }
  return (await res.json()) as Item;
}
```

## Node.js

- Handle errors at the boundary (HTTP handler, CLI, queue consumer). Log with context, then fail or return a typed error — do not swallow.
- Validate all external input (env, argv, HTTP body, query) before use.
- Read config from environment variables. Provide a small typed config module; do not scatter `process.env` reads.
- Never commit `.env`, credentials, or keys. Document required vars in `.env.example` only.
- Use `node:` APIs and the standard library first. Add a dependency only when it is clearly better than writing a small helper.
- Pin major versions in `package.json`. Do not add both a library and a competing duplicate (e.g. two HTTP frameworks).

## Scripts

Keep these npm scripts (names may already exist — match them):

- `build` — `tsc`
- `start` — run compiled `dist/`
- `dev` — TypeScript watch or tsx/ts-node for local runs
- `test` — project test runner
- `lint` / `typecheck` — if configured

Run `npm run typecheck` (or `tsc --noEmit`) after non-trivial TypeScript edits.

## Code style

- Change only what the task requires. No drive-by refactors.
- Match existing naming, formatting, and file layout.
- Prefer clear names over comments. Comment only when the why is not obvious.
- Do not leave `console.log` debug noise, unused imports, or commented-out code. Use a small logger if the app already has one.
- Format with the project formatter (Prettier/ESLint) if present; do not fight it.

## Testing

- Use the repo’s test runner (Node `node:test`, Vitest, or Jest — whichever is already set up).
- Cover behavior at boundaries: parsers, config, handlers, error paths.
- Tests should fail if the bug returns. Avoid asserting only implementation internals.
- Run the tests that cover your change before calling the work done.

## HTTP / APIs (when present)

- Keep routing, validation, and business logic in separate layers.
- Return consistent JSON error shapes. Use correct status codes.
- Verify with real requests (`curl`, a test client, or integration tests), not only a typecheck.
- Check empty, invalid, and error inputs, not only the happy path.

## Git and pull requests

- Do not commit unless asked. Do not push, force-push, or amend published history unless asked.
- Never update git config or skip hooks.
- Commit messages: 1–2 sentences on **why**. Use `add`, `update`, or `fix` accurately.
- Do not commit `node_modules`, secrets, or local env files.
- Pull requests: short summary of what changed and why, plus a test plan (`npm test`, `npm run build`, any manual `curl` steps).

## Security

- Treat all untrusted input as untrusted until validated.
- Do not log secrets, tokens, or full request bodies that may contain credentials.
- Do not add exploit, malware, or attack-procedure code.

## Agent / AI assistance

- Read this file before making project-wide decisions.
- Infer conventions from existing code first; do not override them with generic templates.
- After substantive edits, check diagnostics on touched files and fix issues you introduced.
- Prefer `npm` scripts over ad-hoc `node`/`tsc` flags so local and CI stay aligned.
