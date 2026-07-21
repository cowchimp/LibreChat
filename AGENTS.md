# LibreChat — Base44 Dev Setup Notes

## Architecture
- **Monorepo** (npm workspaces): `api/` (Express backend), `client/` (Vite React frontend), `packages/` (shared libs)
- **Backend** runs on port 3080; **Frontend** (Vite dev server) runs on port 3090, proxies `/api` and `/oauth` to backend
- Port 3000 → maps to Vite dev server (3090 inside container)

## Key Quirks

### client/dist/index.html must exist
The backend (`api/server/index.js`) reads `client/dist/index.html` synchronously at startup. If the file is missing, the server crashes immediately. A minimal stub at `client/dist/index.html` is enough for dev — the Vite dev server serves the real frontend.

### Vite allowedHosts patch
`client/vite.config.ts` was patched to support `VITE_ALLOWED_HOSTS=all` → `allowedHosts: true` (needed for the preview proxy). The original code only accepted a comma-separated list.

### HOST env var dual use
In the client container, `HOST` controls the backend proxy URL (via `backendURL` in vite.config.ts). Set `HOST=api` (the compose service name) so the Vite dev server proxies `/api` requests to `http://api:3080`, not `http://0.0.0.0:3080`. Vite listens on `0.0.0.0` via `--host 0.0.0.0` CLI flag.

### Package build order
`npm run build:packages` must run before starting api/client. It builds:
1. `packages/data-provider`
2. `packages/data-schemas`  
3. `packages/api`
4. `packages/client`

These are shared libraries consumed by both the api and client workspaces.

### Missing librechat.yaml
The backend logs an error about missing `/app/librechat.yaml` — this is expected and non-fatal. LibreChat falls back to defaults.

### No RAG API in dev setup
The RAG API (vector DB + rag_api service) is omitted for simplicity. The app warns about it on startup but works fine without it.

## Services (docker-compose.base44.yml)
| Service | Image | Purpose |
|---|---|---|
| mongodb | mongo:8.0 | Primary database |
| meilisearch | getmeili/meilisearch:v1.35.1 | Full-text search |
| npm-install | node:24 | One-shot: install deps + build packages |
| api | node:24 | Express backend (nodemon) on port 3080 |
| client | node:24 | Vite dev server on host port 3000 |

## Dev Credentials
- All AI provider keys (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GOOGLE_KEY`) default to `user_provided` — users enter their own API keys in the UI settings.
- Crypto keys in `.env` are from the `.env.example` defaults — generate new ones for any real deployment.

## Verify App Works
```bash
# Frontend live
curl -sf http://localhost:3000/ | head -5

# External preview proxy
curl -sf -H "Host: external-preview.example.com" http://localhost:3000/ | head -5

# API health (inside container)
docker compose -f docker-compose.base44.yml exec api curl -sf http://localhost:3080/health
```

## Restart
```bash
docker compose -f docker-compose.base44.yml up -d
```
