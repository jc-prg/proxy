# jc://proxy/

Simple combination of an NGINX reverse proxy and Let's Encrypt (certbot) for docker compose, e.g. on a Raspberry Pi.

## requirements

* Docker with the compose v2 plugin (`docker compose`)
* `envsubst` (package `gettext-base` on Debian / Raspberry Pi OS)
* ports 80 and 443 forwarded to the host

## setup

*1. Create Basic Configuration*

```bash
$ cp .env.sample .env
$ nano .env                          # edit for your needs
$ ./create
```

Alternatively just run `./start`: without a `.env` it offers to copy `.env.sample`, opens it in nano and runs `./create`. Later, menu entry `c` edits `.env` and recreates the configuration (restart the proxy afterwards to apply it).

This creates one NGINX config per domain in [config/nginx-domains](config/nginx-domains/INFO.md) – run it again whenever PROXY\_DOMAINS changes. `start`, `docker-compose.yml` and `docker-compose-certbot.yml` are not generated: docker compose and `./start` read their values directly from `.env`, so keep it in dotenv format (no `export`, no arrays, comments on their own lines).

*2. Domain Configurations for NGINX*

  * define the routes (PROXY\_DOMAINS, one entry per line) in [.env](.env.sample) and run *./create* or ...
  * use [config/templates/domain.conf.sample](config/templates/domain.conf.sample) to create a configuration for your domain and copy it to a file per domain into `config/nginx-domains` or ...
  * use your own NGINX configuration file

Entry format: `host[/folder];forward|@page[;snippet]`

```
PROXY_DOMAINS="
cloud.example.com;192.168.1.20:80;nextcloud
apps.example.com;192.168.1.20:81
apps.example.com/folder;192.168.1.50:81
soon.example.com;@placeholder
"
```

  * `host;forward` – `https://host/...` is forwarded to `http://forward/...`
  * `host/folder;forward` – `https://host/folder/...` is forwarded to `http://forward/...` (the prefix is stripped, `/folder` redirects to `/folder/`). Redirects and cookie paths of the backend are rewritten, `X-Forwarded-Prefix: /folder` is sent. Links inside the HTML are *not* rewritten – the application must use relative URLs or support a base path. A host with only folder entries returns 404 on `/`.
  * `host[/folder];@page` – a static placeholder page is served instead of forwarding, from the subfolder `NGINX_PAGES/page/` (default `config/pages/page/`). It needs an `index.html`, further files (css, images) in the subfolder are served as well, every other path returns the `index.html`. Snippets are not supported for pages. See [config/pages/placeholder](config/pages/placeholder/index.html) for an example; changes to the page are live without restart, a new `NGINX_PAGES` requires a restart.
  * all entries of a host share one NGINX config and one certificate

*3. Request Certificates*

Test first with a dry run (no certificates are changed), then request the certificates:

```bash
$ ./start dryrun
$ ./start renew
```

The proxy is stopped while certbot (standalone) is running on port 80, then started again.

*4. Start Proxy*

```bash
$ ./start start
```

*5. Automate*

Add the entries from [config/templates/crontab](config/templates/crontab) to the crontab of root (`sudo crontab -e`). `./start renew` runs once a week, pulls updated images and renews certificates if needed.

## security notes

* TLS 1.2/1.3 only, HTTP/2, HSTS
* unknown hosts / direct ip requests are rejected (`00-default.conf`)
* `X-Forwarded-For` is set to the real client ip, spoofed values are dropped
* rate limit per client ip: 20 req/s, burst 100, max. 50 connections – adjust `limit_req_zone` in [config/nginx/nginx.conf](config/nginx/nginx.conf) and `limit_req` / `limit_conn` in the domain templates if needed
* optional per-entry snippets (root and folder entries): `host[/folder];forward;snippet` in PROXY_DOMAINS inlines `config/templates/snippets/<snippet>.conf` – `nextcloud` hides the version (`status.php`, `ocs/.../cloud/capabilities`) from requests without Nextcloud client user agent, cookie or login (404)
* the proxy container runs read-only, without docker socket and with reduced capabilities
* certificates (`config/letsencrypt/`) and generated domain configs are excluded from git

## logs

NGINX logs to stdout, the docker log is rotated (5 x 10 MB):

```bash
$ docker compose logs -f proxy
```
