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
