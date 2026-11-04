# vps-auth

One Google login for selected public apps on `chetan-vps`. Verified 2026-11-04.

Tinyauth uses Google OAuth only. Its allow list contains the owner's Gmail address.
The Coolify service is `lo120rlfyooszyleguajps7c` at `https://auth.chetan-vps.tech`.
Its container is `tinyauth-lo120rlfyooszyleguajps7c`.

## Access

| Service or path | Access |
|---|---|
| Kharcha public domain | Tinyauth; password login remains a fallback |
| Coolify public domain | Tinyauth, then Coolify login; Google login is enabled in Settings > Authentication |
| Coolify `/webhooks/` | Public; no Tinyauth |
| Umami dashboard | Tinyauth |
| Umami `/script.js` and `/api/send` | Public; no Tinyauth |
| `travel.chetan-vps.tech` | Public by design; share view needs access |
| Kharcha tailnet `:8790` and machine paths | Unchanged; no Tinyauth |
| Coolify tailnet `https://vps.bullhead-mine.ts.net:8000` | Bypasses Tinyauth; recovery path |
| Vaultwarden, MCPHub, 9Router, T3, Cockpit | Tailnet only; no Tinyauth |
| Mini Jellyfin and qBittorrent | Tailnet only; no Tinyauth |

Tailscale uses the Google account for tailnet access. API clients cannot follow a login redirect.

## Configuration

Shared Traefik rules are in `/data/coolify/proxy/dynamic/sso.yaml`.
Edit them in Coolify: Servers > localhost > Proxy > Dynamic Configuration.
Keep [sso.yaml](sso.yaml) in sync. Replace its placeholder with the secret for the live file.

Kharcha app `rzoma7mhudklvintirprqgvk` uses these HTTPS router middlewares:
`gzip,kharcha-headers@file,tinyauth@file,kharcha-proxy@file`.
Its Coolify environment sets `PROXY_SECRET` and `ADMIN_EMAIL`.
Kharcha trusts `Remote-Email` only with a matching `X-Kharcha-Proxy`.
See `kharcha-web/src/server/auth.ts`.

To protect a new app, add `tinyauth@file` to its HTTPS router middlewares in Coolify custom labels. Redeploy the app.
Never add Traefik middleware labels to the Tinyauth container. Coolify attached them to its own router, and every page returned 401.

## Recovery

If Google or Tinyauth is down, use tailnet Coolify at `https://vps.bullhead-mine.ts.net:8000`.
Or run `ssh vps`, then `agy`.
To bypass the wall, remove `tinyauth@file` (and for Kharcha `kharcha-proxy@file`) from the app's router middlewares, then redeploy.
Do not delete `sso.yaml` first: routers that reference its middlewares stop working until the labels change.
Coolify API on the public domain is behind the wall; use `http://127.0.0.1:8000/api/v1`.

## Secrets and host move

- Google OAuth: `~/Storage/secrets/vps-auth-google.env`.
- Kharcha proxy secret: `~/Storage/secrets/kharcha-proxy.env`.
- Coolify API token `claude-vps-auth`: `~/Storage/secrets/coolify-api.env`; expires 2026-12-04.
- Coolify API is local only: `http://127.0.0.1:8000/api/v1`.

Tinyauth environment settings live in the Coolify DB. Keep `auth.chetan-vps.tech` on host moves; the Google OAuth redirect URI depends on it.
