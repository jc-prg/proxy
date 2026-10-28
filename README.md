# jc://proxy/

Simple combination of an NGINX reverse proxy and Let's Encrypt (certbot) for docker compose, e.g. on a Raspberry Pi.

## requirements

* Docker with the compose v2 plugin (`docker compose`)
* `envsubst` (package `gettext-base` on Debian / Raspberry Pi OS)
* ports 80 and 443 forwarded to the host

## setup

*1. Create Basic Configuration*

```bash
$ cd config/
$ cp config_prod.sample config_prod
$ nano config_prod                   # edit for your needs
$ ./create prod
```

This creates `start`, `docker-compose.yml` and `docker-compose-certbot.yml` in the project root and one NGINX config per domain in [config/nginx-domains](config/nginx-domains/INFO.md).

*2. Domain Configurations for NGINX*

  * define domain and forward server (PROXY\_DOMAINS) in the config file [config/config_prod](config/config_prod.sample) and run *./create prod* or ...
  * use [config/templates/domain.conf.sample](config/templates/domain.conf.sample) to create a configuration for your domain and copy it to a file per domain into `config/nginx-domains` or ...
  * use your own NGINX configuration file

*3. Request Certificates*

Test first with a dry run (no certificates are changed), then request the certificates:

```bash
$ cd ../
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
* optional per-domain snippets: `"domain;forward;snippet"` in PROXY_DOMAINS inlines `config/templates/snippets/<snippet>.conf` – `nextcloud` hides the version (`status.php`, `ocs/.../cloud/capabilities`) from requests without Nextcloud client user agent, cookie or login (404)
* the proxy container runs read-only, without docker socket and with reduced capabilities
* certificates (`config/letsencrypt/`) and generated domain configs are excluded from git

## logs

NGINX logs to stdout, the docker log is rotated (5 x 10 MB):

```bash
$ docker compose logs -f proxy
```
