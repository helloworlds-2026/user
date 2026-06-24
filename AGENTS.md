# AGENTS.md

## Cursor Cloud specific instructions

This repo (`user`) is the Vue 3 + Vite + TypeScript customer-facing storefront
for the Dujiao-Next ecosystem. It is a pure frontend; it talks to the Go backend
in the sibling `dujiao-next` repo.

Dependencies are refreshed automatically by the startup update script (pnpm); do
not reinstall them here. Standard commands are in `README.md` / `package.json`.

### Running
`pnpm dev` serves on `http://localhost:5173` (`strictPort: true`). The dev server
proxies `/api`, `/uploads`, `/sitemap.xml`, and `/robots.txt` to the backend at
`http://localhost:8080`, so the `dujiao-next` backend (and its Redis) must be
running for products/orders to load.

### Build / typecheck
`pnpm build` runs `vue-tsc -b` (typecheck) then `vite build`.

### Tests
Tests under `tests/` use the built-in `node:test` runner but are written in
TypeScript. Run a test file **directly** so type-stripping applies:

```bash
node --experimental-strip-types tests/paymentResumePolicy.test.ts
```

Do not use `node --test tests/` — the `--experimental-strip-types` flag is not
propagated to the spawned `.ts` test files, so it fails with `MODULE_NOT_FOUND`.
