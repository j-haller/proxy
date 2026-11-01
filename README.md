# proxy

A Docker-based web server for hosting personal web applications on the local network.

## Overview

This project runs a reverse proxy in Docker that serves as the single entry point for self-hosted web apps. Each application runs in its own container, and the proxy routes incoming requests to the right one by path prefix: each app lives under its own path, e.g. `http://<server>/home-dashboard/`. Only one hostname has to resolve, so no custom DNS is needed.

## Goals

- **One entry point** for all personal apps on the LAN
- **Isolation**: each app runs in its own container
- **Easy to extend**: adding a new app should take only a few lines of configuration
- **Local only**: not meant to be exposed to the public internet

## Requirements

- Docker
- Docker Compose

## Apps

| URL | App | Config |
|-----|-----|--------|
| `http://<server>/home-dashboard/` | home-dashboard | `conf.d/apps/home-dashboard.conf` |

The path prefix is stripped before forwarding, so each app serves from `/` internally (e.g. `/home-dashboard/sse` → `/sse`). Everything else gets a 404 (`conf.d/default.conf`).

### App requirements

Since the prefix is stripped, the app does not know it lives under `/<app>/`. Its frontend must therefore use **relative URLs only** (`sse`, `api/foo`, assets via Vite `base: './'`), never absolute ones like `/sse`. Absolute redirects or cookies with `Path=/` must also be avoided or made prefix-aware (the proxy sends `X-Forwarded-Prefix`).

## Name resolution

Clients only need to resolve the server's name, e.g. via the router's DHCP hostname registration, mDNS (`<server>.local`), or the IP directly.

## Usage

```sh
docker network create proxy   # once per server
docker compose up -d
```

Apps join the external `proxy` network and are addressed by their container alias (e.g. `home-dashboard:8080`).

## Adding an app

1. Add `conf.d/apps/<app>.conf` with `location` blocks for `/<app>` (redirect to `/<app>/`) and `/<app>/`, stripping the prefix via `rewrite ^/<app>(/.*)$ $1 break;` before `proxy_pass` to the app's alias on the `proxy` network. Copy `home-dashboard.conf` as a template.
2. Add it to the table above and run `docker compose exec nginx nginx -s reload`.
