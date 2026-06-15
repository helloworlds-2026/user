# AGENTS.md

## Cursor Cloud specific instructions

### What this repo is
This is a **frontend-only** Vue 3 + TypeScript + Vite SPA ("Dujiao-Next User Web"), the customer-facing storefront for the Dujiao-Next digital-goods platform. There is **no backend code in this repository**.

### Services / commands
Standard scripts live in `package.json`:
- `npm run dev` — Vite dev server. Binds `0.0.0.0:5173` with `strictPort: true` (fixed port, see `vite.config.ts`). No type-checking in dev.
- `npm run build` — runs `vue-tsc -b` (full type-check) then `vite build`. The type-check is much stricter than `dev`: a type error anywhere (e.g. an undefined template variable) fails the whole build even though `npm run dev` still serves fine.
- `npm run preview` — preview a production build.
- No `lint` or `test` scripts are defined in this repo.

### Backend dependency (important, non-obvious)
The SPA is **fully gated on the backend**. On startup the Vue Router guard awaits `GET /api/v1/public/config`; until that succeeds, `App.vue` renders only a loading spinner with **no graceful degradation**. So without a backend the app appears to "hang" on a spinner — this is expected, not a frontend bug.

`vite.config.ts` proxies `/api`, `/uploads`, `/sitemap.xml`, and `/robots.txt` to `http://localhost:8080`. To run the product end-to-end you must have the external Dujiao-Next backend API running on port **8080** (it owns the database; not part of this repo), or point `VITE_API_BASE_URL` at a reachable backend.

API responses use the envelope `{ status_code: 0, msg, data }` and the client reads `response.data.data` (see `src/api/client.ts`). When there is no real backend available, a lightweight stand-in that serves the `/api/v1/public/*` endpoints (config, products, product detail, categories, banners, posts, member-levels) in that envelope is enough to render and exercise the storefront. Note the app loads `config` once and gates routing on it, so the backend must be up **before** the first page load (otherwise refresh after starting it).
