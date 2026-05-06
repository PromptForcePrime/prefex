# Prefex

LLM inference proxy for Claude. Reduce API costs by 40-70% through prompt caching, smart routing, and session memory.

## Quick Start

### Install

```bash
# macOS / Linux
curl -fsSL https://prefex.vercel.app/install.sh | bash

# Windows PowerShell
iwr https://prefex.vercel.app/install.ps1 -OutFile install.ps1; .\install.ps1
```

Then configure Claude Code:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:8019"
  }
}
```

Open the dashboard: http://localhost:8019/dashboard

### Or Docker

```bash
docker run -d \
  -p 8019:8019 \
  -e ANTHROPIC_API_KEY=sk-... \
  -v ~/.prefex:/root/.prefex \
  promptforce/prefex:latest
```

## What It Does

Prefex sits between Claude Code and the Anthropic API. It optimizes every request:

1. **Prompt Caching** — Detects repeated context, caches at API level (0.1x read cost)
2. **Smart Routing** — Automatically downgrades to Claude Haiku for simple tasks (0.2x cost)
3. **Session Memory** — Reuses conversation state across Claude Code restarts
4. **Request Logging** — SQLite dashboard tracks cost, latency, cache hits

## Key Stats

Total savings across active users: $25k+ (as of May 2026)
Average cost reduction: 40-70% per user
Cache hit rates: 15-35% on typical coding workflows

## Architecture

```
Claude Code → Prefex (localhost:8019) → Anthropic API
                     ↓
              SQLite dashboard
```

Single Go binary, optional Redis (fallback: in-memory store), no auth on dashboard (localhost only).

## Config

Configuration lives at `~/.prefex/config.yaml`. Edit via dashboard or CLI:

```yaml
prefex:
  mode: live
  upstream:
    anthropic_api_key: ${ANTHROPIC_API_KEY}
  router:
    enabled: true
    strong_model: claude-sonnet-4-6
    weak_model: claude-haiku-4-5-20251001
    cost_threshold: 0.3
  kv_cache:
    enabled: true
    ttl_seconds: 86400
  session_store:
    enabled: true
    ttl_seconds: 7200
  dashboard:
    port: 8019
```

## Privacy

Prefex logs request metadata (tokens, model, latency) but never logs prompt or response text. All data stays on your machine unless you explicitly sync.

## Docs

Read the full guide: https://prefex.vercel.app/guide

## License

Proprietary. See LICENSE file.
