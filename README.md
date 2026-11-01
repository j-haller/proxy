# proxy

A Docker-based web server for hosting personal web applications on the local network.

## Overview

This project runs a reverse proxy in Docker that serves as the single entry point for self-hosted web apps. Each application runs in its own container, and the proxy routes incoming requests to the right one by hostname or path.

## Goals

- **One entry point** for all personal apps on the LAN
- **Isolation**: each app runs in its own container
- **Easy to extend**: adding a new app should take only a few lines of configuration
- **Local only**: not meant to be exposed to the public internet

## Requirements

- Docker
- Docker Compose

## Usage

```sh
docker compose up -d
```

Then open the configured apps from any device on the local network.
