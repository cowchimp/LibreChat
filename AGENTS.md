See CLAUDE.md.

When adding or changing code that mutates user documents, invalidate the auth user document cache for affected users. This includes single-user updates and bulk role/user mutations; otherwise OpenID JWT request burst caching can serve a stale `req.user` until its TTL expires.

## Base44 development runtime

- Start the full stack with `docker compose -f docker-compose.base44.yml up -d`.
- The one-shot `setup` service installs workspace dependencies and builds package/client artifacts required by the backend; the `app` service then runs Vite and nodemon from the bind-mounted source.
- Verify live-source serving with `curl http://localhost:3000/` (the response includes `/@vite/client`) and backend readiness with `curl http://localhost:3000/api/config`.
- MongoDB and Meilisearch run locally. AI-provider credentials are optional for boot and are only needed when configuring the corresponding model endpoint.
