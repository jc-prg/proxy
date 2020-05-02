# jc://proxy/

Simple combination of NGINX container and letsencrypt for docker-compose on Raspberry Pi ... not ready yet.

## status quo

This configuration already can be used with a bit of manual effort ...

1. Create Basic Configuration

```bash
$ cd config/
$ cp config_prod.sample config_prod
$ nano config_prod                   # edit for your needs
$ ./create prod
```

2. Create Domain Configurations for NGINX

  * use [config/templates/domain.conf](config/templates/domain.conf) to create configuration for your domain
  * copy to [data/nginx-domain](data/nginx-domain) ...

3. Start

```bash
$ cd ../
$ ./start start
```

## open ...

* create domain configurations based on config file and template
* ensure correct link to key files for different domains created with letsencrypt
