# Prefex

**40-70% off your Claude bill. One line change. Zero dependencies.**

Prefex is a ~13MB standalone binary that sits between Claude Code and the Anthropic API. Prompt caching, intelligent model routing, and session memory — all running locally on your machine.

```bash
# Download, review, then run
curl -fsSL https://prefex.vercel.app/install.sh -o install.sh
less install.sh
bash install.sh
```

We don't do `curl | bash`. Read the script first.

---

## Quick start

### Claude Code

Add one line to `~/.claude/settings.json`:

```json
{ "env": { "ANTHROPIC_BASE_URL": "http://localhost:8019" } }
```

Restart Claude Code. Savings start immediately.

### Python / Node / cURL

```python
# Python
client = anthropic.Anthropic(base_url="http://localhost:8019")
```

```javascript
// Node
const client = new Anthropic({ baseURL: "http://localhost:8019" });
```

```bash
# cURL
curl http://localhost:8019/v1/messages -H "x-api-key: $ANTHROPIC_API_KEY" ...
```

### Dashboard

Open [localhost:8019/dashboard](http://localhost:8019/dashboard) to see savings, configure routing, and browse the marketplace.

---

## How it works

| Layer | What it does | Savings |
|-------|-------------|---------|
| **Prompt caching** | Detects repeated system prompts, injects Anthropic cache headers automatically | 30-50% |
| **Model routing** | Routes simple requests to Haiku, complex ones to Sonnet — rule-based, no ML | 10-30% |
| **Session memory** | Carries conversation context so the API doesn't re-process prior turns | 5-15% |

Total overhead: **<15ms p99**. If anything fails internally, requests pass through to Anthropic unchanged. Prefex never blocks an API call.

---

## Features

- **Standalone binary** — no Docker, no Redis, no Python, no dependencies
- **Local dashboard** — real-time savings, per-project breakdown, request log
- **Marketplace** — curated skills, MCP servers, and templates for Claude Code ([browse](https://prefex.vercel.app/marketplace))
- **Free to use** — 30-day license, renewable online, no credit card ([renew](https://prefex.vercel.app/renew))
- **Privacy-first** — runs on localhost, no telemetry, no analytics, prompts never logged by default

---

## Privacy

Prefex runs entirely on your machine. No telemetry, no analytics, no phone-home. The binary never contacts our servers.

What's logged locally (in `~/.prefex/data/prefex.db`):
- Timestamp, model, tokens, cost, cache hit rate, latency
- **Never:** prompt text, response text, API keys

Opt-in only: system prompt text (for the prompt library feature, off by default).

Verify yourself:

```bash
sudo tcpdump -i any -n 'dst port 443' | grep -v api.anthropic.com
# Should show nothing — Prefex only talks to Anthropic.
```

Full details: [prefex.vercel.app/trust](https://prefex.vercel.app/trust)

---

## Platforms

| OS | Architecture | Binary |
|----|-------------|--------|
| macOS | Apple Silicon (M1+) | `prefex-darwin-arm64` |
| macOS | Intel | `prefex-darwin-amd64` |
| Linux | x86_64 | `prefex-linux-amd64` |
| Linux | ARM64 | `prefex-linux-arm64` |
| Windows | x86_64 | `prefex-windows-amd64.exe` |

The install script auto-detects your platform.

---

## Upgrade

```bash
curl -fsSL https://prefex.vercel.app/install.sh -o install.sh && bash install.sh
```

Detects existing installation, preserves your data, downloads the latest binary.

## Uninstall

```bash
curl -fsSL https://prefex.vercel.app/uninstall.sh | bash
```

Add `--purge` to also remove `~/.prefex` (data, config, logs).

---

## Enterprise

Need team management, shared caches, budget controls, or custom routing?

**[Join the waitlist](https://prefex.vercel.app/waitlist)** — we're onboarding teams in regulated industries where data privacy matters.

---

## Feedback

This is the public home for Prefex. If you're using it — or tried and hit a wall — we want to hear from you.

**[Open an issue](https://github.com/PromptForcePrime/prefex/issues/new)**

Good things to file:
- Something didn't work (bug)
- Something you expected but doesn't exist (feature request)
- Confusing install or docs
- A use case you're not sure Prefex supports

---

**Website:** [prefex.vercel.app](https://prefex.vercel.app) | **Leaderboard:** [prefex.vercel.app/leaderboard](https://prefex.vercel.app/leaderboard) | **Marketplace:** [prefex.vercel.app/marketplace](https://prefex.vercel.app/marketplace)
