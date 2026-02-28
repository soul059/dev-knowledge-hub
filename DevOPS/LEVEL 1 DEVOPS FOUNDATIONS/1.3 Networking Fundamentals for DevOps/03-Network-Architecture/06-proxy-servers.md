# Chapter 6: Proxy Servers

## Overview

A **proxy server** acts as an intermediary between clients and servers. In DevOps, proxies handle load distribution, caching, SSL termination, security filtering, and API gateway functionality. Understanding forward proxies, reverse proxies, and transparent proxies is essential for designing scalable and secure infrastructure.

---

## 6.1 Proxy Types

```
┌──────────────────────────────────────────────────────────────┐
│                    PROXY TYPES                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. FORWARD PROXY (Client-side)                              │
│  ┌────────┐     ┌───────────┐     ┌──────────┐              │
│  │ Client │ ──▶ │ Forward   │ ──▶ │ Internet │              │
│  │        │     │ Proxy     │     │ Server   │              │
│  └────────┘     └───────────┘     └──────────┘              │
│  - Client knows about the proxy                              │
│  - Used for: content filtering, anonymity, caching           │
│  - Example: Squid, corporate proxy                           │
│                                                              │
│  2. REVERSE PROXY (Server-side)                              │
│  ┌────────┐     ┌───────────┐     ┌──────────┐              │
│  │ Client │ ──▶ │ Reverse   │ ──▶ │ Backend  │              │
│  │        │     │ Proxy     │     │ Server(s)│              │
│  └────────┘     └───────────┘     └──────────┘              │
│  - Client doesn't know about backend servers                 │
│  - Used for: load balancing, SSL termination, caching        │
│  - Example: Nginx, HAProxy, Traefik, AWS ALB                 │
│                                                              │
│  3. TRANSPARENT PROXY                                        │
│  ┌────────┐     ┌───────────┐     ┌──────────┐              │
│  │ Client │ ──▶ │ Transparent│──▶ │ Internet │              │
│  │        │     │ Proxy     │     │ Server   │              │
│  └────────┘     └───────────┘     └──────────┘              │
│  - Client is unaware of proxy (network-level intercept)      │
│  - Used for: monitoring, content filtering, caching          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.2 Reverse Proxy Architecture

```
┌──────────────────────────────────────────────────────────────┐
│            REVERSE PROXY IN PRODUCTION                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet                                                    │
│     │                                                        │
│     ▼                                                        │
│  ┌──────────────────────────────────────┐                    │
│  │     NGINX / TRAEFIK (Reverse Proxy)  │                    │
│  │                                      │                    │
│  │  ✓ SSL Termination (HTTPS → HTTP)    │                    │
│  │  ✓ Load Balancing                    │                    │
│  │  ✓ Caching Static Assets            │                    │
│  │  ✓ Rate Limiting                    │                    │
│  │  ✓ Request Routing                  │                    │
│  │  ✓ Compression (gzip)              │                    │
│  └──────┬────────────┬─────────────┬───┘                    │
│         │            │             │                         │
│    /api/*       /static/*     /auth/*                        │
│         │            │             │                         │
│    ┌────▼────┐  ┌────▼────┐  ┌────▼────┐                    │
│    │ API     │  │ Static  │  │ Auth    │                    │
│    │ Server  │  │ Server  │  │ Server  │                    │
│    │ :8080   │  │ :3000   │  │ :9000   │                    │
│    └─────────┘  └─────────┘  └─────────┘                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.3 Nginx as Reverse Proxy

```nginx
# /etc/nginx/conf.d/app.conf

# Upstream backend servers
upstream api_servers {
    least_conn;
    server 10.0.1.10:8080 weight=3;
    server 10.0.1.11:8080 weight=2;
    server 10.0.1.12:8080 backup;
}

upstream frontend_servers {
    server 10.0.2.10:3000;
    server 10.0.2.11:3000;
}

# Rate limiting zone
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

server {
    listen 443 ssl http2;
    server_name app.example.com;

    # SSL Configuration
    ssl_certificate     /etc/ssl/certs/app.crt;
    ssl_certificate_key /etc/ssl/private/app.key;
    ssl_protocols       TLSv1.2 TLSv1.3;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header Strict-Transport-Security "max-age=31536000" always;

    # Gzip compression
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

    # API routes → API servers
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        proxy_pass http://api_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Frontend → Frontend servers
    location / {
        proxy_pass http://frontend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Static files with caching
    location /static/ {
        root /var/www;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "OK";
        add_header Content-Type text/plain;
    }
}

# HTTP → HTTPS redirect
server {
    listen 80;
    server_name app.example.com;
    return 301 https://$server_name$request_uri;
}
```

---

## 6.4 Traefik in Docker/Kubernetes

```yaml
# docker-compose.yml with Traefik reverse proxy
version: "3.8"
services:
  traefik:
    image: traefik:v2.10
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.email=admin@example.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/certs/acme.json"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"    # Dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./certs:/certs

  api:
    image: myapp/api:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`app.example.com`) && PathPrefix(`/api`)"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
      - "traefik.http.services.api.loadbalancer.server.port=8080"
    deploy:
      replicas: 3

  frontend:
    image: myapp/frontend:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`app.example.com`)"
      - "traefik.http.routers.frontend.tls.certresolver=letsencrypt"
      - "traefik.http.services.frontend.loadbalancer.server.port=3000"
```

---

## 6.5 Forward Proxy Use Cases in DevOps

```
┌──────────────────────────────────────────────────────────────┐
│           FORWARD PROXY IN CI/CD                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────┐                                  │
│  │    Private Network     │                                  │
│  │                        │                                  │
│  │  ┌─────────┐           │     ┌───────────┐               │
│  │  │ CI/CD   │ ────────▶ │ ──▶ │ Forward   │ ──▶ Internet  │
│  │  │ Runner  │           │     │ Proxy     │     (npm,      │
│  │  └─────────┘           │     │ (Squid)   │      Docker    │
│  │                        │     │           │      Hub,      │
│  │  ┌─────────┐           │     │ - Caches  │      PyPI)    │
│  │  │ Build   │ ────────▶ │ ──▶ │   packages│               │
│  │  │ Server  │           │     │ - Logs    │               │
│  │  └─────────┘           │     │   access  │               │
│  │                        │     └───────────┘               │
│  └────────────────────────┘                                  │
│                                                              │
│  Benefits:                                                   │
│  - Cache package downloads (npm, pip, Docker images)         │
│  - Control/audit outbound internet access                    │
│  - Speed up builds with cached dependencies                  │
│  - Security: whitelist allowed external URLs                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Configure proxy for various tools
export HTTP_PROXY=http://proxy.example.com:3128
export HTTPS_PROXY=http://proxy.example.com:3128
export NO_PROXY=localhost,127.0.0.1,.internal.example.com

# Docker daemon proxy (/etc/docker/daemon.json)
{
  "proxies": {
    "http-proxy": "http://proxy.example.com:3128",
    "https-proxy": "http://proxy.example.com:3128",
    "no-proxy": "*.internal.example.com,127.0.0.0/8"
  }
}

# npm proxy
npm config set proxy http://proxy.example.com:3128
npm config set https-proxy http://proxy.example.com:3128

# pip proxy
pip install --proxy http://proxy.example.com:3128 flask
```

---

## 6.6 Comparison: Proxy Tools

| Tool | Type | Best For | Key Features |
|------|------|----------|------------|
| Nginx | Reverse proxy | Web apps, APIs | SSL, caching, load balancing |
| HAProxy | Reverse proxy | High-performance TCP/HTTP | L4/L7, health checks |
| Traefik | Reverse proxy | Docker/K8s environments | Auto-discovery, Let's Encrypt |
| Envoy | Service proxy | Microservices (sidecar) | Observability, gRPC, service mesh |
| Squid | Forward proxy | Caching, filtering | HTTP caching, ACLs |
| Apache | Reverse/forward | Legacy apps | mod_proxy, versatile |
| Caddy | Reverse proxy | Simple HTTPS setup | Auto HTTPS, zero-config TLS |

---

## 6.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| 502 Bad Gateway | Backend server down | Check backend health, upstream config |
| 504 Gateway Timeout | Backend too slow | Increase `proxy_read_timeout` |
| Real IP not logged | Missing proxy headers | Add `X-Real-IP`, `X-Forwarded-For` |
| Mixed content warnings | Incorrect `X-Forwarded-Proto` | Set `proxy_set_header X-Forwarded-Proto` |
| WebSocket not working | Upgrade header missing | Add `proxy_set_header Upgrade` |
| Proxy env vars ignored | NO_PROXY not set | Add internal hosts to `NO_PROXY` |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Forward Proxy | Client-side, controls outbound access |
| Reverse Proxy | Server-side, distributes incoming traffic |
| Transparent Proxy | Client unaware, network-level intercept |
| SSL Termination | Proxy handles HTTPS, backends use HTTP |
| Proxy Headers | X-Real-IP, X-Forwarded-For, X-Forwarded-Proto |
| Rate Limiting | Control request rates at the proxy layer |
| Caching | Proxy caches responses to reduce backend load |
| Traefik | Docker-native reverse proxy with auto-discovery |

---

## Quick Revision Questions

1. **What's the difference between a forward proxy and a reverse proxy?**
2. **Why is SSL termination typically done at the reverse proxy?**
3. **Write an Nginx config that routes `/api` to one backend and `/` to another.**
4. **How do proxy headers like `X-Forwarded-For` work and why are they important?**
5. **What environment variables must be set for Docker to use a corporate proxy?**
6. **Compare Nginx, Traefik, and Envoy — when would you choose each?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← VPN Concepts](05-vpn-concepts.md) | [README](../README.md) | [Unit 4: VPC →](../04-Cloud-Networking/01-vpc.md) |
