# nginx reverse proxy for *.yehudab.com

Dockerized nginx that terminates TLS and reverse-proxies the services running
on this VPS. (Historically this repo hosted a WordPress install for
yehudab.com — the main site has since moved to 11ty on GitHub Pages, and the
WordPress pieces were removed. A final archive lives in
`backblaze:pico-vps-backup/wordpress-archive/`.)

## What it serves

| Host | Upstream |
|---|---|
| mycar.yehudab.com | `:4000` (car-location-react) |
| staticman.yehudab.com | `:6000` (staticman) |
| apps.yehudab.com | static files from `apps/`, incl. `/solver-images/` from picoclaw |

Server blocks for forecast/ackee remain in `nginx/default.conf` but those
domains are served by fly.io now.

## Layout

- `docker-compose.yml` — single `nginx` service, host networking, pinned via
  `NGINX_VERSION` in `.env` (gitignored; see `.env_example`)
- `nginx/default.conf` — all server blocks; TLSv1.2/1.3, `http2 on`,
  hybrid post-quantum key exchange (X25519MLKEM768) via OpenSSL ≥ 3.5 defaults
- `certs/` → mounted at `/etc/letsencrypt`; active lineage:
  `mycar.yehudab.com-0002` (covers mycar, staticman, apps)
- `letsencrypt/letsencrypt-renew.sh` — renews via dockerized certbot (webroot)
  and HUPs nginx; run every 12h by `/etc/cron.d/certbot`

## Operations

Upgrade nginx: bump `NGINX_VERSION` in `.env`, then

```
sudo docker compose up -d
```

Test config before reloading:

```
sudo docker exec nginx nginx -t
sudo docker kill -s HUP nginx
```

Force a cert renewal check:

```
sudo ./letsencrypt/letsencrypt-renew.sh
```
