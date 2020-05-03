# jc://proxy/

Simple combination of NGINX container and letsencrypt for docker-compose on Raspberry Pi ... not ready yet.

## status quo

This configuration already can be used with a bit of manual effort ...

*1. Create Basic Configuration*

```bash
$ cd config/
$ cp config_prod.sample config_prod
$ nano config_prod                   # edit for your needs
$ ./create prod
```

*2. Install Letsencrypt*

```bash
$ cd ../
$ cd certbot/
$ ./letsencrypt install
```


*3. Create Domain Configurations for NGINX*

  * define domain and forward server (PROXY\_DOMAINS) in the config file [config/config_prod](config/config_prod.sample) and *./create prod* or ...
  * use [config/templates/domain.conf.sample](config/templates/domain.conf.sample) to create configuration for your domain and copy to a file per domain into the directory [data/nginx-domains](#) or ...
  * use your own NGINX configuration file

*4. Start Proxy*

```bash
$ cd ../
$ ./start start
```

## open ...

* ensure correct link to key files for different domains created with letsencrypt
* install crontab per script
