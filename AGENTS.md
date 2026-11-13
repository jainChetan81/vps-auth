# VPS authentication

Tinyauth and Traefik rules protect selected public Coolify services.

- Shared rules: `~/AGENTS.md`.
- `docker-compose.yml` defines the Tinyauth service.
- `sso.yaml` defines shared Traefik middlewares and routers.
- The live rules are managed in Coolify's proxy dynamic configuration.
- Keep `sso.yaml` in sync with the live rules without copying secrets.
- Public webhook and tracking paths use separate routers without Tinyauth.
- Do not attach Traefik middleware labels to the Tinyauth container.
- Tailnet Coolify access bypasses Tinyauth for recovery.
- No repository build or test command is documented.
- The container health check runs `tinyauth healthcheck`.
- See `~/Services/coolify/README.md` and this repository's README.
