# proxy

A Docker-based web server for hosting personal web applications on the local network.

## Overview

This project runs a reverse proxy in Docker that serves as the single entry point for self-hosted web apps. Each application runs in its own container, and the proxy routes incoming requests to the right one by hostname: each app gets its own subdomain, e.g. `dashboard.<server>.lan`.

## Goals

- **One entry point** for all personal apps on the LAN
- **Isolation**: each app runs in its own container
- **Easy to extend**: adding a new app should take only a few lines of configuration
- **Local only**: not meant to be exposed to the public internet

## Requirements

- Docker
- Docker Compose

## Apps

| Hostname | App | Config |
|----------|-----|--------|
| `dashboard.<server>.lan` | home-dashboard | `conf.d/dashboard.conf` |

Unknown hostnames get a 404 (`conf.d/default.conf`).

## DNS

Every app hostname must resolve to the server. Easiest is one wildcard entry in the local DNS (Pi-hole, AdGuard Home, OpenWrt): `*.<server>.lan → <server-ip>`. Without wildcard support, add one entry per app, or use `/etc/hosts` on each client.

## Usage

```sh
docker network create proxy   # once per server
docker compose up -d
```

Apps join the external `proxy` network and are addressed by their container alias (e.g. `home-dashboard:8080`).

## Adding an app

1. Add `conf.d/<app>.conf` with `listen 80;` and `server_name ~^<app>\.;`, proxying to the app's alias on the `proxy` network.
2. Add it to the table above and run `docker compose exec nginx nginx -s reload`.
