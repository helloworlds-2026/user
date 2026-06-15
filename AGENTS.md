# AGENTS.md

## Cursor Cloud specific instructions

### Ecosystem overview
This `user` repo is the Vue 3 + Vite **customer-facing storefront** for the Dujiao-Next digital-goods shop. It is one of three sibling repos:

| Repo | Role | Dev command | Port |
|------|------|-------------|------|
| `dujiao-next` | Go backend REST API + async worker | `go run ./cmd/server` | 8080 |
| `admin` | Vue 3 + Vite admin console | `npm run dev` | 5174 |
| `user` (this repo) | Vue 3 + Vite storefront | `npm run dev` | 5173 |

### Running this app
- Standard dev/build commands are in `README.md` (`npm install`, `npm run dev`). Dev server runs on port **5173**.
- The Vite dev server proxies `/api`, `/uploads`, `/sitemap.xml`, and `/robots.txt` to `http://localhost:8080`, so the **backend (`dujiao-next`) and Redis must be running** for products/data to load. See the `dujiao-next` repo's `AGENTS.md` for backend + Redis startup.

### Non-obvious gotcha: `npm run build` currently fails on pre-existing type errors
- `npm run build` runs `vue-tsc -b` first, which currently fails with pre-existing type errors in committed source (e.g. `currentYear` not defined in `src/components/Footer.vue`). These are not environment issues.
- The **Vite dev server (`npm run dev`) does not type-check**, so development works normally despite the failing production type-check.
