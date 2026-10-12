# Hostinger VPS deployment

Deployed from this checkout on 2026-10-11.

- SSH: `root@100.72.254.110` (Tailscale).
- App: https://srv1320412.hstgr.cloud:8443
- Source, Compose configuration and persistent data: `/opt/opengym` on the VPS.
- Data: `/opt/opengym/data`; retain this directory across updates and back it up.
- nginx site: `/etc/nginx/sites-available/opengym`, enabled via `sites-enabled/opengym`.
- Public HTTPS port: 8443. Internal web port: `127.0.0.1:8084`. API has no published port.
- Environment: `/opt/opengym/.env`, mode 600. `env.example` documents the settings.
- AI Coach disabled. Passkey accounts and browser guest mode enabled.
- First registered account becomes the admin. Create your own profile first.

## Startup and recovery

Both containers use `restart: always`; Docker and nginx are enabled at boot. No separate
systemd wrapper is needed. Intentional `docker compose down` removes the containers and
therefore disables their automatic startup until `up` is run again.

The existing Let's Encrypt certificate for `srv1320412.hstgr.cloud` is reused. The existing
certbot timer is enabled with the nginx installer for renewal. UFW allows TCP 8443.

## Maintenance

Run on the VPS:

```sh
cd /opt/opengym
docker compose -f compose.vps.yaml ps
docker compose -f compose.vps.yaml logs --tail 100
docker compose -f compose.vps.yaml up -d --build --wait
curl -fsS https://srv1320412.hstgr.cloud:8443/api/health
```

Update source without overwriting `.env` or `data/`; rebuild with the command above.
Back up `data/` before updates; it includes account credentials, session secret, and uploads.
For a consistent backup, stop only openGym's API while copying its data, then restart it.

## Existing services inspected

GolfDay (3000/8000), Marketing Hub (3001 behind nginx 443), Buzz (3200 behind nginx 443),
n8n (5678), Golf Physics (3970), Roth IRA (8090), Peptide Calculator (8091), and another
Node listener (4173), plus database/cache containers and a Bitbucket runner. These were
left running. Ports 8443 and 8084 were unused before deployment.

## Validation

Source images built successfully; both containers passed healthchecks. Public HTTPS
validated without bypassing certificate checks; `/api/health` reported `ok: true` and
`writable: true`. Browser sign-in screen and guest home UI loaded successfully.
Simulated process crashes recovered automatically for both containers (restart count 1),
and the public API remained healthy afterwards. Boot-enabled services verified. Host reboot and actual
iPhone passkey registration were not performed.
