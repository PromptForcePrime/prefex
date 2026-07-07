# Trust & Data Handling

How Prefex handles your data, what it stores, what leaves your infrastructure, and
how to verify these claims. Written for security and procurement review.

---

## What Prefex is

Prefex is a self-hosted proxy that sits between Claude Code (or any
Anthropic-compatible client) and the Anthropic API. It runs **on your own
machine or your own server** — there is no Prefex-operated cloud in the request
path. Your prompts and responses flow from your client, through your proxy, to
Anthropic. Prefex never proxies them through a third party.

---

## Where data lives

| Data | Location | Notes |
|---|---|---|
| Request metadata (tokens, model, latency, cost, cache hits) | Local SQLite at `~/.prefex/data/prefex.db` | Never leaves the node unless you export it. |
| Prompt / response **text** | Not persisted by request logging | The request log stores counts and hashes, not message bodies. **However**, with the default `compression.tier: standard` (lossy) + `large_output.enabled`, the **originals of lossy-compacted tool outputs** (file contents, command output that prefex shortens) are retained locally so they stay recoverable — see the row below. |
| Compacted-output originals (CCR store) | Local SQLite `largeout_meta` at `~/.prefex/data/prefex.db` | When compression is lossy (default), the full original of each shortened tool result is kept for **`large_output.ttl_days` (default 14 days)** so `prefex_recall` can restore it on demand. Content-addressed, local-only, never exported. Set `compression.tier: safe` for lossless-only compression that persists nothing. |
| Advisor training corpus (opt-in) | Local SQLite `advisor_capture` table | Off by default. When `local_advisor.capture.enabled` is on, sampled turns record the prompt/context **hashes and lengths** plus the local advisor's candidate guidance. Raw prompt + codebase-context **text** are retained only when you additionally set `store_prompt_text: true`. Local-only; never exported without an explicit admin action. |
| Session state / prompt-cache assist | In-process memory, or your Redis if configured | Stays within your infrastructure. |
| Cross-session memory (extracted facts) | Local SQLite `project_memories` at `~/.prefex/data/prefex.db` | When session memory is enabled, short extracted facts/summaries derived from your turns are stored locally so future sessions can recall context. Derived text, not raw request bodies; local-only, never exported. Visible and deletable in the dashboard. |
| Config | `~/.prefex/config.yaml` | API keys are referenced via `${ENV_VAR}`, not stored inline when you use that form. |
| License key | `~/.prefex/license.json` | HMAC-signed. Contains: key id, tier, your email, an install id (random UUID generated locally in `~/.prefex/install_id`, used to bind paid keys to this install), seat count, and issue/expiry timestamps. A small `~/.prefex/.clock` file additionally records the latest clock time the proxy has seen (anti-tampering anchor for offline license expiry). All local-only; never transmitted except when you explicitly request a key from the activation page. |

**All data stays on infrastructure you control.** The only outbound connections
Prefex makes are: (1) to the upstream LLM API (Anthropic/OpenAI/Google) to serve
your requests, and (2) two **user-initiated, opt-in** paths to prefex.vercel.app — the
marketplace catalog fetch (only when `telemetry.enabled: true`, off by default)
and a leaderboard submission (only when you click Submit in the dashboard).
Nothing is sent automatically, and there is **no license-activation call** —
licensing is verified entirely offline.

---

## Credentials

- Prefex **validates and forwards** API keys; it does not store them. In
  passthrough mode, your client's own credentials are forwarded upstream
  untouched.
- For OAuth (Claude Code login), the bearer token is forwarded as-is. Prefex
  does not persist it.
- The optional **secrets vault** (`secrets.enabled`) encrypts brokered
  credentials at rest with an AES key generated locally at
  `~/.prefex/keys/vault.key` and never transmitted.

---

## PII handling (optional, opt-in)

With `pii_scrub.enabled: true`, Prefex redacts configurable entity types (email,
IP, phone, API keys, cloud credentials, Slack tokens, SSN) from request bodies
**before** they are forwarded upstream. This is off by default; enable it for
team deployments that may carry customer data. Implementation:
`internal/pii/scrubber.go`.

---

## Telemetry & leaderboard

Prefex sends **nothing automatically**. Two opt-in paths exist:

- **`telemetry.enabled`** (default **false**): when on, the daemon fetches the
  optional marketplace catalog from prefex.vercel.app at startup and daily. No usage
  data is uploaded by this flag — it only controls that one download. Leave it
  false to keep egress limited to the provider APIs.

  ```yaml
  prefex:
    telemetry:
      enabled: false
  ```

- **Leaderboard submission** (manual): only when you click *Submit* in the
  dashboard. It sends aggregate stats (request counts, savings totals — never
  prompt/response content) **plus the username and email you enter** so your
  entry can be attributed. Don't submit if you'd rather stay off the board.

---

## Fail-open guarantee

Prefex is designed to **fail open**: if any internal component errors (cache,
router, session store, scrubber), the request is logged and forwarded upstream
unchanged rather than blocked. An outage in Prefex degrades to a transparent
pass-through, never to a denied request.

---

## Team mode & access control

When team mode is enabled:

- Admin APIs (invites, member management, budgets, analytics, audit export) are
  gated behind a constant-time-compared `admin_token` or an OIDC admin session.
- Per-developer attribution uses an HMAC-hashed token; raw tokens are never
  stored — only their hash.
- The dashboard is intended for LAN/VPN exposure. For broader exposure, configure
  TLS (`dashboard.tls_cert_file` / `tls_key_file`) and front it with your own auth.

See `PRO_DEPLOYMENT.md` § 10 for the pre-go-live security checklist.

---

## How to verify these claims

1. **No prompt text in the DB:**
   ```bash
   sqlite3 ~/.prefex/data/prefex.db ".schema requests"
   # columns are counts/hashes/metadata — no prompt/response body columns
   ```
2. **Outbound connections:** run the proxy under `lsof -i` / a network monitor and
   confirm connections only to the configured upstream (and optional telemetry).
3. **Source availability:** the proxy is a single Go binary; behavior is auditable
   against the documented request flow in `CLAUDE.md`.
4. **Keep `telemetry.enabled: false`** (the default) and confirm **zero** calls
   to `prefex.vercel.app` — with telemetry off and no manual leaderboard submission,
   the only outbound traffic is to your configured LLM provider.

---

## Data deletion

All state is local. To purge: stop the proxy and delete `~/.prefex/data/prefex.db`
(history) and `~/.prefex/config.yaml` (config). Archived copies, if any, live under
`~/.prefex/archive/` and are never auto-deleted — remove them manually.
