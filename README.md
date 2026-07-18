# Hermes Agent — Advanced Railway Template

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/hermes-agent-4)

Production-ready Railway deployment template for **[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)** (pinned to `v2026.7.7.2`).

One-click deploy a full Hermes Agent instance with a polished admin UI, reverse-proxied native dashboard, metrics, skills management, MCP editor, cron viewer, workspace browser, pairing controls, backup/restore, and more.

## Deploy

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/hermes-agent-4)

1. Click the button above
2. Set `ADMIN_PASSWORD` (or let Railway generate one)
3. Deploy → open the generated public URL
4. Log in with username `admin` and your `ADMIN_PASSWORD`

The template attaches a persistent volume at `/data` automatically.

## Features

- **Starlette admin server** that supervises `hermes gateway` + native `hermes dashboard`
- Cookie-based auth + setup wizard
- Reverse proxy to the real Hermes React dashboard (with "← Setup" escape hatch)
- Live **CPU / Memory / Disk** metrics
- **Skills browser** (list / view SKILL.md / delete)
- **MCP Servers** editor (JSON view of `mcp_servers` in config.yaml)
- **Cron Jobs** file viewer
- **Workspace** file browser for `~/.hermes/workspace`
- Pairing approval UI for messaging platforms
- Full backup & restore (zip of config, keys, history, skills, memories…)
- xAI OAuth / SuperGrok support
- Pre-built dashboard + TUI assets for instant first load
- Proper zombie reaping via `tini`
- Persistent volume on `/data`

## Quick Deploy

1. Click the Railway button (or create a new service from this repo)
2. Attach a volume to `/data`
3. Set `ADMIN_PASSWORD` (required)
4. Deploy → open the generated domain → log in → complete the setup wizard

## Required Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `ADMIN_PASSWORD` | **Yes** | Password for the admin UI |
| `HERMES_REF` | No | Override Hermes git ref (default: `v2026.7.7.2`) |
| `HERMES_AUTH_JSON_BOOTSTRAP` | No | One-time bootstrap of `auth.json` (e.g. xAI tokens) |

All other provider keys, channel tokens, etc. are configured through the web UI and stored on the volume.

## Volume

Mount a persistent volume at **`/data`**. Everything under `/data/.hermes` (config, sessions, skills, memories, logs, cron, workspace) lives there.

## Architecture

```
Railway Service
├── tini (PID 1)
└── start.sh
    └── python server.py          # Starlette admin + reverse proxy
        ├── hermes gateway        # the agent itself (subprocess)
        └── hermes dashboard      # native React UI on 127.0.0.1:9119
```

The admin UI lives at `/setup`. Once configured, `/` proxies to the native Hermes dashboard.

## Local Development

```bash
# Build
docker build -t hermes-railway --build-arg HERMES_REF=v2026.7.7.2 .

# Run with a local volume
docker run --rm -it \
  -p 8080:8080 \
  -e ADMIN_PASSWORD=changeme \
  -v $(pwd)/data:/data \
  hermes-railway
```

## Updating Hermes

Bump the `HERMES_REF` build arg (or set the Railway variable) to a newer tag from  
https://github.com/NousResearch/hermes-agent/releases  
then redeploy. The image stamps itself as a Docker install so the in-dashboard "Update" button correctly refuses.

## License

This template is MIT. Hermes Agent itself is MIT — see the upstream repository.
