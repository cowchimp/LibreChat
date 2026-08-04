See CLAUDE.md.

When adding or changing code that mutates user documents, invalidate the auth user document cache for affected users. This includes single-user updates and bulk role/user mutations; otherwise OpenID JWT request burst caching can serve a stale `req.user` until its TTL expires.

## Base44 Dev Setup Notes

- Vite dev server (port 3090) proxies `/api` and `/oauth` to the backend (port 3080). Both run in the same container so the proxy targets `localhost:3080`.
- The `VITE_ALLOWED_HOSTS=all` env var is mapped to `allowedHosts: true` in `client/vite.config.ts` to allow external preview hostnames.
- MongoDB and MeiliSearch run as separate compose services. RAG API and vectordb are optional and not included in the dev setup.
- Node modules are stored in named Docker volumes to persist across container recreations. The setup service runs `npm ci` + turborepo build for all packages except the frontend client.
- AI provider API keys default to `user_provided` which means users enter them in the LibreChat UI settings.
