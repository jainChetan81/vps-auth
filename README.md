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

### Protect a new app

`tinyauth@file` is the `tinyauth` middleware in `sso.yaml`. Traefik sends each request to Tinyauth first.
No valid Google session: Tinyauth redirects to Google login. Valid session: the request reaches the app with `Remote-Email`.

1. In Coolify, open the app > Configuration > General > Container Labels.
2. Find the line `traefik.http.routers.https-0-<uuid>.middlewares=`.
3. Append `,tinyauth@file` to its value. Example: `gzip,tinyauth@file`.
4. Save, then Redeploy.
5. In a private window, open the app domain. Expect a redirect to `auth.chetan-vps.tech`.

For a one-click service (no label field), add a router in `sso.yaml` instead. Copy `umami-sso`.
Keep public paths, such as webhooks or tracking scripts, in a separate router with priority 1000 and no middleware.
Never add Traefik middleware labels to the Tinyauth container. Coolify attached them to its own router, and every page returned 401.

## Recovery

If Google or Tinyauth is down, use tailnet Coolify at `https://vps.bullhead-mine.ts.net:8000`.
Or run `ssh vps`, then `agy`.
To bypass the wall, remove `tinyauth@file` (and for Kharcha `kharcha-proxy@file`) from the app's router middlewares, then redeploy.
Do not delete `sso.yaml` first: routers that reference its middlewares stop working until the labels change.
Coolify API on the public domain is behind the wall; use `http://127.0.0.1:8000/api/v1`.

## Secrets and host move

Secret values are in the host's private credentials file, section "Single login". Never commit them.
The Google OAuth client, Kharcha proxy secret and Coolify API token (expires 2026-12-04) are there.

Tinyauth environment settings live in the Coolify DB. Keep `auth.chetan-vps.tech` on host moves; the Google OAuth redirect URI depends on it.
