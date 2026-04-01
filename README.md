# Prefex

**40–70% off your LLM bill. One line of config.**

Prefex is a local proxy for Anthropic and OpenAI APIs. Drop it in front of your existing API client — no code changes, no data leaving your machine, savings from the first request.

```bash
# Download the install script and review it before running
curl -fsSL https://prefex.vercel.app/install.sh -o install.sh
less install.sh

# When you're satisfied, run it
bash install.sh
```

We don't do `curl | bash`. Download the script, read it, then decide.

---

## How it works

Prefex intercepts your API calls locally and applies three cost-reduction layers:

**Prefix cache** — detects repeated system prompt tokens and injects Anthropic's caching headers automatically. Cached tokens cost 90% less. No configuration needed.

**Session memory** — stores conversation turns in local Redis and sends only the delta on each request, not the full history. Eliminates the biggest source of redundant input tokens in multi-turn workflows.

**Model routing** — a local RouteLLM sidecar scores each prompt and routes simple requests to a cheaper model (Haiku) while keeping complex ones on Sonnet/Opus. Threshold is configurable.

All three layers are independent. Each works without the others. Together they compound.

---

## Connect your client

Change one value:

```python
# Anthropic SDK
client = anthropic.Anthropic(api_key="sk-ant-...", base_url="http://localhost:8000")

# OpenAI SDK
client = OpenAI(api_key="sk-...", base_url="http://localhost:8000/v1")
```

```json
// Claude Code: ~/.claude/settings.json
{ "env": { "ANTHROPIC_BASE_URL": "http://localhost:8000" } }
```

---

## Privacy

Prefex runs entirely on localhost. No telemetry, no analytics, no external logging. Your prompts go directly from your machine to Anthropic/OpenAI — Prefex only touches them locally to inject caching headers and route models.

If any Prefex component fails, requests pass through unchanged. It never blocks an API call due to its own failure.

Verify: `sudo tcpdump -i any -n 'dst port 443' | grep -v api.anthropic.com`

---

## Feedback and feature requests

This is the public home for Prefex feedback. If you're using Prefex — or tried to and hit a wall — we want to hear from you.

**[Open an issue →](https://github.com/PromptForcePrime/prefex/issues/new)**

Good things to file:
- Something didn't work (bug report)
- Something you expected to exist but doesn't (feature request)
- Something confusing in the install or docs
- A use case you're not sure Prefex supports
- Anything that made you close the tab

Security issues: email security@prefex.dev rather than filing a public issue.
