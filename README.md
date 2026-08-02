# mywant-caddy-plugin

A [MyWant](https://github.com/onelittlenightmusic/MyWant) plugin that adds the **Caddy** want type.

Runs Caddy as a local HTTPS reverse proxy (using its built-in internal CA) in
front of mywant-gui, so LAN clients — e.g. the Web Inspector bookmarklet on an
iPhone — can reach it over `https://` without hitting mixed-content blocks.
Uses the same built-in `live_server_management` agent as the cloudflare/ngrok
plugins.

Unlike cloudflare/ngrok, Caddy doesn't print a URL that needs regex-extracting
from its log output — once the process is confirmed running, this want
persists two fixed, already-known values straight into `~/.mywant/config.yaml`:

- `web_inspector_ca_cert_path` — filesystem path to Caddy's internal CA root
  cert (served to phones at `GET /api/v1/web-wants/ca-cert`)
- `https_path` — a certificate-confirmed `https://` origin (e.g.
  `https://localhost:8443`)

## Requirements

- [MyWant](https://github.com/onelittlenightmusic/MyWant) installed and running
- [Caddy](https://caddyserver.com/) installed and available in `PATH` (`brew install caddy`)
- A Caddyfile that reverse-proxies to mywant-gui, with hostnames listed
  explicitly (a bare `:8443` blocks Caddy's on-demand cert issuance) — see
  `mywant-gui/docs/WebInspectorIPhone.md` for a working example:

  ```
  localhost:8443, 127.0.0.1:8443, <LAN IP>:8443 {
      reverse_proxy localhost:8081
  }
  ```

## Installation

```sh
git clone https://github.com/onelittlenightmusic/mywant-caddy-plugin \
  ~/.mywant/custom-types/mywant-caddy-plugin
```

Then restart MyWant (`mywant stop && mywant start -D`).

## Usage

```sh
# List available want types (confirm "caddy" appears)
mywant types list

# Deploy with defaults (Caddyfile at /opt/homebrew/etc/Caddyfile)
mywant wants create -t caddy

# Deploy with a custom Caddyfile / cert path
mywant wants create -t caddy \
  -p caddyfile_path=/opt/homebrew/etc/Caddyfile \
  -p ca_cert_path=/opt/homebrew/var/lib/caddy/pki/authorities/local/root.crt \
  -p https_path=https://localhost:8443
```

## Parameters

| Name | Default | Description |
|------|---------|--------------|
| `caddyfile_path` | `/opt/homebrew/etc/Caddyfile` | Path to the Caddyfile |
| `command` | `caddy` | Path to caddy binary |
| `log_file` | `""` | Path to capture caddy output (auto if empty) |
| `ca_cert_path` | `/opt/homebrew/var/lib/caddy/pki/authorities/local/root.crt` | Persisted to config.yaml's `web_inspector_ca_cert_path`. Set to `""` to skip. |
| `https_path` | `https://localhost:8443` | Persisted to config.yaml's `https_path`. Set to `""` to skip. |

## State

After the want is achieved (`process_status == "running"`):

- `current.process_pid` — PID of the running caddy process
- `current.ca_cert_path` / `current.https_path` — mirror what was written to config.yaml
- `current.launch_summary` — JSON summary of the running process
- `final_result` — the persisted `https_path`
