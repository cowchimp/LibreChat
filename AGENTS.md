See CLAUDE.md.

When adding or changing code that mutates user documents, invalidate the auth user document cache for affected users. This includes single-user updates and bulk role/user mutations; otherwise OpenID JWT request burst caching can serve a stale `req.user` until its TTL expires.

## Base44 Dev Setup

- **Stack**: Node.js 24 monorepo (Express API + Vite React client) with MongoDB and MeiliSearch.
- **Dev compose**: `docker-compose.base44.yml` runs everything from source with hot reload.
- **Setup service**: Runs `npm ci` and `npm run build:packages` to install deps and build shared workspace packages into a named volume.
- **API**: Runs via `nodemon` on port 3080 (internal). A placeholder `client/dist/index.html` is created at startup so the API doesn't crash looking for it.
- **Client**: Vite dev server on port 3090 (mapped to host 3000). Proxies `/api` and `/oauth` to the API container. Uses `BACKEND_HOST` env var (added to vite.config.ts) to target the API service.
- **allowedHosts**: Modified Vite config to accept `VITE_ALLOWED_HOSTS=all` for `allowedHosts: true`.
- **AI keys**: Set to `user_provided` by default — users enter their own keys in the UI settings. No external secrets required to boot.
- **RAG API**: Not included (optional; needs external API keys for embeddings). The API warns about it but works fine without it.
- **Verify**: `curl http://localhost:3000/api/config` should return JSON config.
