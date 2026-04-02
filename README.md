# Prefex

**40–70% off your Claude bill. One line in Claude Code settings.**

Prefex is a local proxy built for Claude Code and the Anthropic API. It optimizes your API calls invisibly — your workflow does not change, your keys never leave your machine, and savings start from the first request.

```bash
# Download the install script and review it before running
curl -fsSL https://prefex.vercel.app/install.sh -o install.sh
less install.sh

# When you are satisfied, run it
bash install.sh
```

We do not do `curl | bash`. Download the script, read it, then decide.

---

## Quick start

### Claude Code (recommended)

Add one line to `~/.claude/settings.json`:

```json
{ "env": { "ANTHROPIC_BASE_URL": "http://localhost:8019" } }
```

Restart Claude Code. Done.

### Python (Anthropic SDK)

```python
client = anthropic.Anthropic(
    api_key="sk-ant-...",
    base_url="http://localhost:8019",  # add this
)
```

### Dashboard

Open [http://localhost:8019/dashboard](http://localhost:8019/dashboard) to see savings in real time.

---

## What it does

Prefex sits between your code and the Anthropic API. It applies multiple optimization layers to reduce your token costs by 40–70%, depending on your usage pattern. The proxy adds <15ms overhead per request.

All optimization happens locally. If any internal component fails, requests pass through to Anthropic unchanged. Prefex never blocks an API call.

---

## Privacy

Prefex runs entirely on localhost. No telemetry, no analytics, no external logging. Your prompts go directly from your machine to Anthropic — Prefex only touches them locally.

Verify with tcpdump:

```bash
sudo tcpdump -i any -n 'dst port 443' | grep -v api.anthropic.com
```

Full details: [https://prefex.vercel.app/trust](https://prefex.vercel.app/trust)

---

## Upgrade

Run the install script again — it detects the existing installation, preserves your data, and pulls the latest version.

```bash
curl -fsSL https://prefex.vercel.app/install.sh -o install.sh && bash install.sh
```

## Uninstall

```bash
curl -fsSL https://prefex.vercel.app/uninstall.sh | bash
```

Add `--purge` to also remove `~/.prefex` (data, config, logs).

---

## Feedback and feature requests

This is the public home for Prefex feedback. If you are using Prefex — or tried to and hit a wall — we want to hear from you.

**[Open an issue](https://github.com/PromptForcePrime/Prefex/issues/new)**

Good things to file:
- Something did not work (bug report)
- Something you expected to exist but does not (feature request)
- Something confusing in the install or docs
- A use case you are not sure Prefex supports

Security issues: email security@prefex.dev rather than filing a public issue.

---

**Website:** [https://prefex.vercel.app](https://prefex.vercel.app)
**Leaderboard:** [https://prefex.vercel.app/leaderboard](https://prefex.vercel.app/leaderboard)

