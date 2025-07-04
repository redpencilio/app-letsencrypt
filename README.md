# app-letsencrypt
An SSL-terminating reverse proxy for semantic.works apps, it manages Let's Encrypt SSL certificates with automated renewal.

## Quick Start
### Prerequisites
- Docker and Docker Compose installed
- Domain names pointing to your server
- Ports 80 and 443 accessible

### Basic Setup
   ```bash
   git clone https://github.com/redpencilio/app-letsencrypt.git
   cd app-letsencrypt
   docker-compose up -d
   ```
### Configuration and usage
See [nginx-proxy documentation](https://github.com/nginx-proxy/nginx-proxy/tree/main/docs)

## Monitoring and Logging
### Generate Access Reports

Monitor your nginx-proxy traffic with GoAccess reports:

```bash
# Generate HTML report from nginx-proxy logs
docker compose logs --no-log-prefix nginx-proxy | sed 's/\x1b\[[0-9;]*m//g; s/^nginx\.1[[:space:]]*|[[:space:]]*//' | docker run --rm -i -v $PWD:/report -e LANG=$LANG allinurl/goaccess -a -o /report/report.html --log-format='%v %h %^ %^ [%d:%t %^] "%r" %s %b "%R" "%u" "%^"' --date-format='%d/%b/%Y' --time-format='%T' -
```

### Live Terminal Dashboard
For a real-time interactive terminal dashboard using GoAccess:

```bash
mkfifo /tmp/nginx_pipe && \
docker compose logs --no-log-prefix nginx-proxy -f | sed 's/\x1b\[[0-9;]*m//g; s/^nginx\.1[[:space:]]*|[[:space:]]*//' > /tmp/nginx_pipe & \
docker run --rm -it -v /tmp:/logs allinurl/goaccess -a -f /logs/nginx_pipe --log-format='%v %h %^ %^ [%d:%t %^] "%r" %s %b "%R" "%u" "%^"' --date-format='%d/%b/%Y' --time-format='%T' --real-time-html
```

## Related Projects

- [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy)
- [docker-letsencrypt-nginx-proxy-companion](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion)
- [certbot](https://certbot.eff.org/)
