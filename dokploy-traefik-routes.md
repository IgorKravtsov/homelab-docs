# Dokploy / Traefik Routes

This file tracks live public routes terminated on the VPS by Dokploy / Traefik and forwarded to Home Lab services over the Tailscale subnet route.

## Routing Model

```text
public domain
  → VPS public IP
  → Dokploy / Traefik on VPS
  → Tailscale subnet route
  → GL.iNet LAN
  → http://192.168.8.x:port
```

## Current Validated State (Apr 2026)

- **Actual GL.iNet LAN**: `192.168.8.0/24`
- **Actual router IP**: `192.168.8.1`
- **Validated VPN path**: VPS can reach `192.168.8.1` through Tailscale subnet routing
- **Validated public edge**: Dokploy / Traefik on VPS is the chosen reverse proxy layer

## Domain Mapping

| Domain | Service | Internal Target | Notes |
|--------|---------|-----------------|-------|
| `n8n.shatori.com` | n8n | `http://192.168.8.142:5678` | Runs on Raspberry Pi, 8 GB RAM |
| `draw.shatori.com` | Excalidraw | `http://192.168.8.142:8081` | Runs alongside n8n |
| `open-web-ui.shatori.com` | Open WebUI | `http://192.168.8.142:3000` | Runs alongside n8n and Excalidraw |

## Shared Middleware

```yaml
http:
  middlewares:
    redirect-to-https:
      redirectScheme:
        scheme: https
        permanent: true
```

## Combined Example in `homelab.yaml`

```yaml
http:
  routers:
    n8n-http:
      rule: "Host(`n8n.shatori.com`)"
      entryPoints:
        - web
      middlewares:
        - redirect-to-https
      service: n8n-service

    n8n-https:
      rule: "Host(`n8n.shatori.com`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt
      service: n8n-service

    draw-http:
      rule: "Host(`draw.shatori.com`)"
      entryPoints:
        - web
      middlewares:
        - redirect-to-https
      service: draw-service

    draw-https:
      rule: "Host(`draw.shatori.com`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt
      service: draw-service

    open-web-ui-http:
      rule: "Host(`open-web-ui.shatori.com`)"
      entryPoints:
        - web
      middlewares:
        - redirect-to-https
      service: open-web-ui-service

    open-web-ui-https:
      rule: "Host(`open-web-ui.shatori.com`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt
      service: open-web-ui-service

  middlewares:
    redirect-to-https:
      redirectScheme:
        scheme: https
        permanent: true

  services:
    n8n-service:
      loadBalancer:
        servers:
          - url: "http://192.168.8.142:5678"
        passHostHeader: true

    draw-service:
      loadBalancer:
        servers:
          - url: "http://192.168.8.142:8081"
        passHostHeader: true

    open-web-ui-service:
      loadBalancer:
        servers:
          - url: "http://192.168.8.142:3000"
        passHostHeader: true
```
