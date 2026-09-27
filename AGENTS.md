# Base44 Dev Environment

## Project

Enatega Multivendor food-delivery platform — a monorepo of frontends. The
backend (GraphQL API) is **proprietary and NOT included** in this repository.

The preview runs the customer web storefront (`enatega-multivendor-web`), a
Next.js 16 (App Router, webpack) + React 19 app on host port 3000.

## Run

```bash
docker compose -f docker-compose.base44.yml up -d
```

- Service `web` uses `node:20.16-bookworm-slim`, bind-mounts
  `./enatega-multivendor-web` at `/app`, and runs
  `npm install && npx next dev --webpack -H 0.0.0.0 -p 3000`.
- `node_modules` and `.next` are anonymous volumes (kept out of the host mount).
- `HUSKY=0` skips the `prepare`/husky hook (no `.git` inside the mounted subdir).
- `WATCHPACK_POLLING=true` enables file-watch polling for the bind mount.

## Backend / API

The app talks to a GraphQL API via `NEXT_PUBLIC_SERVER_URL` (HTTP) and
`NEXT_PUBLIC_WS_SERVER_URL` (WebSocket). These are **public** client config
(`NEXT_PUBLIC_` prefix), not secrets. Without a real backend the UI shell still
renders (the `ConfigurationProvider` falls back to defaults on query errors),
but there is no catalog/order data. Provide a real API URL to get live data.

## Next.js config tweaks (dev-only, production unchanged)

`enatega-multivendor-web/next.config.mjs` was adjusted so the app is visible in
the Base44 preview iframe — all changes are gated on `NODE_ENV !== "production"`:

- `allowedDevOrigins` includes `3000-<BASE44_PUBLIC_HOST_SUFFIX>` so the preview
  origin can load dev assets / HMR (Next.js blocks unknown dev origins).
- `frame-ancestors` is `*` in dev (strict `'none'` in prod) and `X-Frame-Options:
  DENY` is omitted in dev, so the preview iframe can embed the page.

## Verify

- `curl -s -o /dev/null -w '%{http_code}' http://localhost:3000/` → `200`
- Title is `Enatega Multivendor`.
- First request compiles (~20s); subsequent requests are fast.
- `docker compose -f docker-compose.base44.yml ps` shows `web` as `healthy`.
