---
name: cloudflare-tunnel-caddy
description: >-
  Use this skill when implementing remote access for the Webapp (`hub.<domain>`) and
  preview environments (`*.preview.<domain>`) using Cloudflare Tunnel (`cloudflared`),
  Cloudflare Access (Zero Trust email policy), and Caddy dynamic reverse proxy (M4 & M5).
---

# Cloudflare Tunnel + Access & Caddy Reverse Proxy (M4 & M5)

Guides implementation of secure remote access without public open ports (SPEC D-17, US-M4-01 🛡, US-M5-01 🛡, ARCH §12, §13).

## When to Activate
- Implementing `hub setup webapp` and `hub doctor` tunnel/Access checks (`AC-M4-01.2`).
- Configuring `cloudflared` systemd service and ingress rules for `hub.<domain>` and `*.preview.<domain>`.
- Implementing Caddy dynamic configuration via Caddy Admin API (`http://127.0.0.1:2019/config/...`) for Docker task previews (`US-M5-01`).
- Implementing Vercel and Cloudflare Pages/Workers preview & production deployment targets (`US-M5-02`, `US-M5-04`).

## Core Security & Routing Architecture

### 1. Zero Public Ports Invariant (🛡 D-17, AC-M4-01.1, AC-M5-01.5)
- `hubd` binds **only** `127.0.0.1:<hub_port>`.
- Caddy binds **only** `127.0.0.1:<caddy_port>` (and admin API on `127.0.0.1:2019`).
- `cloudflared` runs as a local daemon connecting outbound to Cloudflare Edge:
  ```yaml
  tunnel: <tunnel-uuid>
  credentials-file: /home/ntchub/.ntc-hub/cloudflared/<tunnel-uuid>.json
  ingress:
    - hostname: hub.<domain>
      service: http://127.0.0.1:<hub_port>
    - hostname: "*.preview.<domain>"
      service: http://127.0.0.1:<caddy_port>
    - service: http_status:404
  ```

### 2. Cloudflare Access Policy (🛡 AC-M4-01.2, AC-M4-01.5, AC-M5-01.2)
- Provision Cloudflare Access Applications covering both `hub.<domain>` and `*.preview.<domain>`.
- Policy: `Include -> Emails -> [owner_email]`.
- Unauthenticated requests are blocked at Cloudflare Edge and **never reach `hubd` or Caddy**.
- `hubd` Webapp still enforces its own second-layer authentication (Password + TOTP or Telegram magic link — `AC-M4-01.3`).

### 3. Dynamic Preview Routing via Caddy Admin API (US-M5-01, US-M5-03)
- When `/preview <task_id>` runs on a `docker` target project:
  1. Build and start the preview container on an internal port/network.
  2. Register a route in Caddy via `POST /config/apps/http/servers/srv0/routes` matching host `<task_id>.preview.<domain>` → upstream `127.0.0.1:<container_port>`.
  3. Schedule automatic teardown after `preview_ttl_hours` (default 24h) or on `/down <task_id>` / task completion (`AC-M5-03.1`), removing both the Caddy route and the Docker container.
