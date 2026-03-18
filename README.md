# app-letsencrypt
An SSL-terminating reverse proxy for semantic.works apps, it manages Let's Encrypt SSL certificates with automated renewal.

## Getting started
### Prerequisites
- Docker and Docker Compose installed
- Domain names pointing to your server
- Ports 80 and 443 accessible

### Basic Setup
   ```bash
   git clone https://github.com/redpencilio/app-letsencrypt.git
   cd app-letsencrypt
   docker compose up -d
   ```

## How-to guides
### How to proxy to a container
This guide assumes your DNS (e.g. `example.redpencil.io`) is setup to resolve to the server `app-letsencrypt` is running on.

Add the following environment variables to the container you want to proxy to. Make sure the container also joins the Docker network of `app-letsencrypt`.

``` yaml
services:
  identifier:
    environment:
      VIRTUAL_HOST: "example.redpencil.io"
      LETSENCRYPT_HOST: "example.redpencil.io"
      LETSENCRYPT_EMAIL: "info@redpencil.io"
    networks:
      - proxy
      - default
networks:
  proxy:
    name: app-letsencrypt_default
    external: true
```

Recreate the container

``` bash
docker compose up -d
```

`app-letsencrypt` will automatically generate a SSL certificate and proxy rules.

### How to setup basic auth for a domain
Install `apache2-utils` and generate a htpasswd file in `./config/htpasswd/<VIRTUAL_HOST>`. The name of the file must be equivalent to the `VIRTUAL_HOST` environment variable set on the proxied container. Pass a username to the command and provide a password interactively.

``` bash
apt-get install apache2-utils
cd app-letsencrypt
htpasswd -c ./config/htpasswd/example.redpencil.io admin
```

Restart the proxy container to pick up the changes
``` bash
docker compose restart nginx-proxy
```

### How to configure allowed upload size for a domain
Generate a config file in `./config/vhost.d/<VIRTUAL_HOST>`. The name of the file must be equivalent to the `VIRTUAL_HOST` environment variable set on the proxied container. 

```
# ./config/vhost.d/example.redpencil.io
client_max_body_size	4096m;
```

You can add any custom Nginx config to this file.

Restart the proxy container to pick up the changes
``` bash
docker compose restart nginx-proxy
```

### How to monitor and log traffic
Monitor your nginx-proxy traffic with GoAccess reports (writes report to $PWD/report.html):

```bash
# Generate HTML report from nginx-proxy logs
docker compose logs --no-log-prefix nginx-proxy | sed 's/\x1b\[[0-9;]*m//g; s/^nginx\.1[[:space:]]*|[[:space:]]*//' | docker run --rm -i -v $PWD:/report -e LANG=$LANG allinurl/goaccess -a -o /report/report.html --log-format='%v %h %^ %^ [%d:%t %^] "%r" %s %b "%R" "%u" "%^"' --date-format='%d/%b/%Y' --time-format='%T' -
```

For a real-time interactive terminal dashboard using GoAccess:

```bash
mkfifo /tmp/nginx_pipe && \
docker compose logs --no-log-prefix nginx-proxy -f | sed 's/\x1b\[[0-9;]*m//g; s/^nginx\.1[[:space:]]*|[[:space:]]*//' > /tmp/nginx_pipe & \
docker run --rm -it -v /tmp:/logs allinurl/goaccess -a -f /logs/nginx_pipe --log-format='%v %h %^ %^ [%d:%t %^] "%r" %s %b "%R" "%u" "%^"' --date-format='%d/%b/%Y' --time-format='%T' --real-time-html
```

Log all 500 errors with timestamp, host, error and path

```bash
drc logs --no-log-prefix nginx-proxy | sed 's/\x1b\[[0-9;]*m//g; s/^nginx\.1[[:space:]]*|[[:space:]]*//' | awk '$10 ~ /^500/ {print $5, $1, $10, $8}'
```

Breakdown of 5xx status code

```bash
# Breakdown by 5xx status codes
drc logs --no-log-prefix nginx-proxy | sed 's/\x1b\[[0-9;]*m//g; s/^nginx\.1[[:space:]]*|[[:space:]]*//' | awk '$10 ~ /^5/ {status[$10]++} END {for (s in status) print s ":", status[s]}'
```

## Reference
### Configuration and usage
See [nginx-proxy documentation](https://github.com/nginx-proxy/nginx-proxy/tree/main/docs)
### Related Projects
- [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy)
- [docker-letsencrypt-nginx-proxy-companion](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion)
- [certbot](https://certbot.eff.org/)
