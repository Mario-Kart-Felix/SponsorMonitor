# SponsorMonitor


## traefik.yml

```yaml
# Docker configuration backend
providers:
    docker:
      defaultRule: "Host(`{{ trimPrefix `/` .Name }}.docker.localhost`)"

# API and dashboard configuration
api:
    insecure: true

entryPoints:
    web:
      address: ":80"

    websecure:
      address: ":443"

certificatesResolvers:
    myresolver:
        acme:
            email: admin@example.com
            storage: acme.json
            # Staging server for testing
            caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
            #tlsChallenge:

            # Optional
            httpChallenge:
                entryPoint: web
```

## docker-compose

```yaml
version: "3"
services:
  traefik:
    image: traefik
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
      - ./traefik/acme:/etc/traefik/acme
    labels:
      # global redirect to https
      - traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)
      - traefik.http.routers.http-catchall.entrypoints=web
      - traefik.http.routers.http-catchall.middlewares=redirect-to-https
      # middleware redirect
      - traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https
    ports:
      - 80:80
      - 443:443
      - 8080:8080

  sponsormonitor:
    environment:
      - GITHUB_ACCESS_TOKEN
      - SECRET_TOKEN
    image: byt3bl33d3r/sponsormonitor
    volumes:
      - ./sm.conf:/etc/sponsormonitor/sm.conf
    labels:
      - traefik.http.routers.sponsormonitor.rule=Host(`${DOMAIN}`) && Path(`/`)
      - traefik.http.routers.sponsormonitor.tls=true
      - traefik.http.routers.sponsormonitor.tls.certresolver=myresolver
      - traefik.http.routers.sponsormonitor.entrypoints=websecure
    ports:
      - 5000:5000
```
