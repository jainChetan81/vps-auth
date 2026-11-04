# vps-auth

One Google login for public Coolify apps on `chetan-vps`.

- Tinyauth at `https://auth.chetan-vps.tech`, Google OAuth, allow list of one email.
- Public apps opt in with the Traefik label `traefik.http.routers.<router>.middlewares=tinyauth@docker`.
- Tailnet-only services (Vaultwarden, MCPHub, 9Router, T3, Cockpit, Jellyfin, qBittorrent) stay off this wall. Tailscale (Google login) is their login.
- Machine paths (kharcha ingest on tailnet `:8790`, `/api/jobs/*` timers) never go behind the wall.

## Recovery

Tinyauth or Google down: `ssh vps` over Tailscale (or password + TOTP from outside), then `agy` CLI. Remove the middleware label in Coolify, or stop the service.

## Host move

Tinyauth keeps no state worth copying; its env lives in the Coolify DB. Keep the domain `auth.chetan-vps.tech`: the Google OAuth redirect URI depends on it.

## Live setup (2026-11-04)

- Coolify service `tinyauth` (uuid `lo120rlfyooszyleguajps7c`, project Apps/production). Container name `tinyauth-lo120rlfyooszyleguajps7c` on network `lo120rlfyooszyleguajps7c`.
- Env in Coolify: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `ALLOWED_EMAIL`. Local copy: `~/Storage/secrets/vps-auth-google.env` (0600).
- Google OAuth client "vps-auth (Tinyauth + Coolify)" in Cloud project `vps-mcp-513611`. Redirect URIs: `https://auth.chetan-vps.tech/api/oauth/callback/google`, `https://coolify.chetan-vps.tech/auth/google/callback`.
- Coolify login: Settings > Authentication > Google enabled, same client, auto-join off. Local password login stays as fallback.
- Do not put Traefik middleware labels on the Tinyauth container itself: Coolify applied them to Tinyauth's own router and every page returned 401. Define the middleware on the protected app instead:

```
traefik.http.middlewares.tinyauth.forwardauth.address=http://tinyauth-lo120rlfyooszyleguajps7c:3000/api/auth/traefik
traefik.http.middlewares.tinyauth.forwardauth.authResponseHeaders=Remote-User,Remote-Email,Remote-Name
traefik.http.routers.https-0-<app-uuid>.middlewares=gzip,tinyauth
```

- Coolify API: local only (`http://127.0.0.1:8000/api/v1`, allow list is private IPs). Token `claude-vps-auth` (Read, Write, Deploy; expires 2026-12-04) in `~/Storage/secrets/coolify-api.env`.
