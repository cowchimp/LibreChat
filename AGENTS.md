See CLAUDE.md.

When adding or changing code that mutates user documents, invalidate the auth user document cache for affected users. This includes single-user updates and bulk role/user mutations; otherwise OpenID JWT request burst caching can serve a stale `req.user` until its TTL expires.

## Base44 Dev Setup Notes

- Vite runs from live source on port 3090 (published as port 3000) and proxies `/api` and `/oauth` to the separate `backend` service on port 3080.
- `BACKEND_HOST` separates the Vite proxy target from Vite's `HOST=0.0.0.0` listen address; `VITE_ALLOWED_HOSTS=all` permits external preview hostnames.
- MongoDB and MeiliSearch are local compose services. RAG API and vectordb are optional and are not needed for the preview.
- The backend requires `client/dist/index.html` even with Vite serving the UI, so setup copies a placeholder after building workspace packages.
- Node modules are kept in named volumes. AI provider keys default to `user_provided`, allowing users to enter keys in LibreChat instead of requiring boot-time secrets.
- Verify with `curl -H 'Host: external-preview.example.com' http://localhost:3000/` and `curl http://localhost:3000/api/config`.
