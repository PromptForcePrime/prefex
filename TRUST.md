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
| Private prompt record ("Keep a private record on this Mac") | Local SQLite `prompts` table | **On by default for new installs since v1.18** (`prompt_library.enabled`); an existing config keeps its value. The same switch keeps your prompt text, captured tool input (the row below) and system prompt text (`kv_cache.store_system_prompt_text`), so Coach, Prompt Doctor, session handovers, the cache warmer and your reports can use them; config.yaml can still set each alone. Pruned after `team.prompt_capture.retention_days` (default 30). On a team hub, a seat's prompts, tool input and system prompt text are kept only when the org turns on `team.prompt_capture` and the member has not opted out. Never sent to PromptForce. Turn it off in Settings, or `prompt_library: {enabled: false}`. |
| Prompt / response **text** | Not persisted by request logging | The request log stores counts and hashes, not message bodies. **However**, with the default `compression.tier: standard` (lossy) + `large_output.enabled`, the **originals of lossy-compacted tool outputs** (file contents, command output that prefex shortens) are retained locally so they stay recoverable — see the row below. |
| Compacted-output originals (CCR store) | Local SQLite `largeout_meta` at `~/.prefex/data/prefex.db` | When compression is lossy (default), the full original of each shortened tool result is kept for **`large_output.ttl_days` (default 14 days)** so `prefex_recall` can restore it on demand. Content-addressed, local-only, never exported. Set `compression.tier: safe` for lossless-only compression that persists nothing. |
| Advisor training corpus (opt-in) | Local SQLite `advisor_capture` table | Off by default. When `local_advisor.capture.enabled` is on, sampled turns record the prompt/context **hashes and lengths** plus the local advisor's candidate guidance. Raw prompt + codebase-context **text** are retained only when you additionally set `store_prompt_text: true`. Local-only; never exported without an explicit admin action. |
| Side-job training examples (opt-in) | `~/.prefex/data/auto-training.jsonl` | Off by default (`offload.auto.capture_training`). Only public jobs and text kinds that passed the PII and secret scans, never code or local-tier jobs; each capped at 4KB. Lines older than 30 days are removed. Never leaves the node. |
| Captured tool input | Local SQLite `tool_call_costs.args_text` | On for new installs since v1.18.1, as part of the private record above; off for upgrades that never set it. When `tool_capture.enabled` is on, the INPUT of allowlisted tool calls (`Bash` and `Read` by default) is retained locally so shell usage can be analysed for guardrails. Credentials are masked before write with the same redaction set used for tool output; each input is truncated to `tool_capture.max_arg_bytes` (default 4096) and pruned after `tool_capture.retention_days` (default 30). Write/Edit are NOT on the default allowlist, because their input is file content rather than an operator action. Never leaves the node. |
| Session state / prompt-cache assist | In-process memory, or your Redis if configured | Stays within your infrastructure. |
| Cross-session memory (extracted facts) | Local SQLite `project_memories` at `~/.prefex/data/prefex.db` | When session memory is enabled, short extracted facts/summaries derived from your turns are stored locally so future sessions can recall context. Derived text, not raw request bodies; local-only, never exported. Visible and deletable in the dashboard. |
| Setup Health tool inventory (opt-in) | Local SQLite `tools_schemas` table + `tools_hash` on `requests` | Off by default. When `setup_health.enabled` is on, the proxy records which tool definitions your client offers so the setup-waste scanner can price dead/bloated tools. Stored per unique tools-array hash: tool **names and byte-lengths only** — never tool descriptions, schemas, or any prompt text. Local-only. |
| Config | `~/.prefex/config.yaml` | API keys are referenced via `${ENV_VAR}`, not stored inline when you use that form. |
| Policy gateway decisions | Local SQLite `policy_gateway_events` | Written only when `policy_gateway.enabled` is on. One row per judged tool call: timestamp, dev id, org id, channel, model, **tool name**, mode, allow, severity, outcome, the policy's reason string, and `request_id` (a local `pfx-<16 hex>` id minted per decision). No tool arguments, no prompt text. |
| Guardrail verdicts | Local SQLite `guardrail_verdicts`, `guardrail_blocks`, `guardrail_masks` | Written only when `guardrails.enabled` is on. Verdicts: session id, dev id, org id, channel, model, provider name, phase, action, category, the provider's reason string, latency, error. `guardrail_blocks` and `guardrail_masks` hold a **hash** of a judged text block plus its category/reason or mask spans (character offsets), so an earlier decision is re-applied to the same block on later turns and after a restart. The text itself is not stored. |
| Tool-poisoning findings | Local SQLite `mcp_tool_admission_events` | When the policy gateway is on, tool definitions and the newest `tool_result`s are scanned locally (`internal/inspect`). Rows carry tool name, allow, severity, outcome, reason, and a hash of the tool description. Detection only: nothing is sent anywhere and nothing is removed from the body. |
| Signed audit chain (opt-in) | `~/.prefex/audit/chain.jsonl` (file sink, default) + signing key `~/.prefex/keys/audit-ed25519.pem` / `.pub` | Off by default (`audit_chain.enabled`). See [Signed audit chain](#signed-audit-chain-audit_chainenabled-default-off). Metadata only; no prompt text, no tool arguments. |
| Session taint window | **In-process memory only** | Active with the policy gateway. Per session, for 10 minutes after the event: tool name plus the path (or command head) of a sensitive read, or the domain of fetched content. Never file contents. Cleared on restart. See [Session taint window](#session-taint-window). |
| PII token maps | **In-process memory only, per request** | With `pii_scrub.enabled`, the map from placeholder (`[PII_EMAIL_1]`) to original value lives only for the one request that created it, so the response can be restored. Never written to disk; the request log keeps only a replacement **count**. |
| License key | `~/.prefex/license.json` | HMAC-signed. Contains: key id, tier, your email, an install id (random UUID generated locally in `~/.prefex/install_id`, used to bind paid keys to this install), seat count, and issue/expiry timestamps. A small `~/.prefex/.clock` file additionally records the latest clock time the proxy has seen (anti-tampering anchor for offline license expiry). All local-only; never transmitted except when you explicitly request a key from the activation page. |

**Your prompts and code stay on infrastructure you control.** One outbound
connection is on by default — an anonymous daily usage report
(counts and percentages only; never prompts, paths, model names or keys; see
[Anonymous usage reporting](#anonymous-usage-reporting-usage_reportingenabled-default-on)
for the exact fields and how to turn it off). Apart from that, the only outbound
connections Prefex makes are: (1) to the upstream LLM API (Anthropic/OpenAI/Google) to serve
your requests, (2) three **opt-in** paths to promptforce.ai — the
marketplace catalog fetch (only when `telemetry.enabled: true`, off by default),
a leaderboard submission (only when you click Join in the dashboard's Join leaderboard dialog), and the
team tier's monthly usage report (only when `team.billing.enabled: true`, off
by default — aggregate dollars only, payload shown verbatim in the dashboard
before you enable it) — and (3) an **opt-in** model-pricing refresh from
raw.githubusercontent.com (the LiteLLM pricing table; only `prefex pricing
refresh` or `pricing.autofetch: true`, both off by default — pricing otherwise
resolves entirely offline from a snapshot embedded in the binary, and the fetch
sends no data, it only downloads the public rate table), and (4) endpoints
**you** name in config for the governance features: a policy appliance, hosted
guardrail providers, an OTLP collector, and offload targets. All four are off
by default, and each sends only to the host you configured, never to us. The
next section lists exactly what each one sends. Nothing is sent
automatically outside those flags except the **licence check** below: one
small call to promptforce.ai when the daemon starts and once a day, carrying a
salted machine hash and three meter integers, never prompts, paths, email or
keys. Keys themselves are still verified entirely offline.

### Licence check (always on in release builds; dev builds never send it)

The free licence runs out by value rather than by date: once prefex has saved
you $500 (the dashboard's "Saved by prefex" figure at All, beyond native prompt
caching, priced at API rates), it warns at 80%, allows 15 days of grace, then stops
and hands your tools back to their direct connection. So that reinstalling
doesn't reset the meter, the daemon reports it to
`https://www.promptforce.ai/api/license/check` on start and every 24 hours,
and the server keeps the highest reading it has seen for this machine.

Sent, and nothing else:

| field | value |
|---|---|
| `product` | `"prefex"` |
| `machine` | sha256 of `"prefex-licence-v1:"` + the OS machine id (IOPlatformUUID / `/etc/machine-id` / MachineGuid). The raw id never leaves the machine, and the salt keeps the hash from joining any other product's |
| `version` | the binary version |
| `key_id` | `"free"` for the first-run key, otherwise the key's id |
| `lifetime_cents`, `key_used_cents`, `exhausted_at` | the meter: value added on this machine, value added under this key, and when it ran out (0 = not yet) |

The server stores those maxima per machine hash, not your IP. The check
**fails open**: if it can't be reached, prefex keeps running on its local
meter. It is separate from usage reporting and has no switch, because it is the
licence. A key with no allowance (paid, or issued) never runs out and never
sends the check. `prefex license --json` prints the local state.

---

## Endpoints you configure (governance features)

Every feature in this section is **off by default** and sends data only to a
URL the operator writes into `~/.prefex/config.yaml`. None of it goes to
Promptforce. The fields listed are taken from the request structs in the
source, and the file paths are given so a reviewer can check them.

### Policy gateway (`policy_gateway.enabled`, default **false**)

Judges each tool call the model proposes before it reaches the client. Two
ways to run it, with different egress:

- **In-process (`url: ""`)**: the shipped Rego bundle plus any user policies are
  evaluated inside the proxy. **Nothing leaves the machine.**
- **External OPA (`backend: opa`)**: one POST to
  `{url}/v1/data/{package}/decision` per decision. The `input` holds `dev_id`,
  `org_id`, `channel`, `model`, and for each proposed call the tool `name` and
  an `arg_hint` (the **first 120 characters** of its JSON arguments, not the
  full arguments). It also holds `recent_calls` (the same shape) and `taint` (see
  [Session taint window](#session-taint-window)). Source:
  `internal/policygateway/client.go` (`Input`, `ToolCallDesc`).
**Defaults and failure behavior.** `on_unavailable` defaults to `allow`, so if
an external policy server is down, times out, or returns 401/5xx, the call
proceeds. That is
fail open. Set `on_unavailable: deny` only if enforcement matters more to you
than availability. `mode: advisory` (what `config.example.yaml` ships) logs
verdicts and **never blocks**. `mode: enforce` blocks at or above
`block_threshold` (default `critical`; a `medium` verdict alerts but passes). Note that an **unset** `mode` behaves
as enforce, so write `mode: advisory` explicitly if that is what you mean.
Default timeout is 200 ms when `timeout_ms` is unset.

### Guardrail providers (`guardrails.enabled`, default **false**)

A chain of providers judges text and returns allow, mask or block. `mode`
defaults to advisory: any value other than `enforce` records verdicts and
never blocks. Each provider's `on_error` defaults to `allow` (fail open).
`on_error: block` is the fail-closed opt-in.

**What text is judged (request phase):** only the **newest user turn** of the
client's body, one content block at a time. That covers its text blocks and
the text of its `tool_result` blocks. Harness-injected `<system-reminder>`
blocks and re-sent history are skipped. This runs on the body **as the client
sent it**, before compaction **and before PII scrubbing**
(`checkRequestGuardrails`, `internal/proxy/guardrail_hook.go`). A hosted
request-phase provider therefore receives unmasked text. Use `regex` (local)
for the request phase if that matters.

| type | runs | what is sent, and where |
|---|---|---|
| `regex` | locally | nothing leaves the machine |
| `webhook` (team build) | your URL | POST `{"phase", "text", "guardrail": <provider name>}`, optional bearer `token` |
| `openai_moderation` (team build) | `api.openai.com/v1/moderations` | `{"model", "input": [text chunks]}`, 16,000-character chunks, at most 16 per text |
| `jev` (team build) | TypeSafe (`api.typesafe.ai/v1` by default) | the text (longest 16,000 characters by default; longer keeps head and tail) plus the classification question |

Verdicts are stored locally (`guardrail_verdicts`), and repeat decisions are
kept as text **hashes** (`guardrail_blocks`, `guardrail_masks`). No judged text
is stored.

### Streaming response guard (`guardrails.stream.enabled`, default **false**)

Judges a streaming answer **while it streams**. Assistant `text_delta` text is
held and checked in overlapping windows (`window_chars` 512 and `overlap_chars`
128 by default). Only Response-phase providers take part: `regex` does by
default, and a hosted provider only with `stream: true` on that provider,
because each window is a separate paid call (a 1,000-word answer is about 12
windows). **Thinking blocks are never held, judged or sent.** Response-phase
providers run only here. A non-streaming response is not judged.

The guard wraps **outside** the PII restore writer, so it sees the upstream
bytes. When PII masking is on, a hosted stream provider receives
`[PII_EMAIL_1]`, never the restored value. `TestResponseGuard_SeesMaskedTextNeverRestoredPII`
pins this. A block ends the text block cleanly and emits a non-retryable
`invalid_request_error`.

### OTLP traces (`telemetry.otlp_endpoint`, default **empty**)

When set, one trace per `/v1/messages` request is POSTed to
`{otlp_endpoint}/v1/traces` with any `otlp_headers` you configure. It is
separate from `telemetry.enabled`, which stays download-only. Attributes, in
full (`internal/proxy/otel_trace.go`, `internal/otlp/otlp.go`):

- resource: `service.name` (`prefex`), `service.version`
- root span: `gen_ai.operation.name`, `gen_ai.provider.name`,
  `gen_ai.request.model`, `gen_ai.conversation.id` (the session id),
  `prefex.channel`, `prefex.dev_id`, `prefex.org_id`, `prefex.router.decision`,
  `prefex.cost_usd`, `prefex.saving_usd`, `prefex.is_agent`, plus the usage
  fields below
- `guardrail.request` span: `prefex.guardrail.blocked` (true/false)
- `chat <model>` span: `gen_ai.response.model`, `gen_ai.usage.input_tokens`,
  `output_tokens`, `cache_read.input_tokens`, `cache_creation.input_tokens`

**No prompt text, completion text, tool arguments or file paths.**
`TestOTLP_MessagesRequestExportsTrace` puts a canary string in the prompt and
fails if it appears in the export. Fail open: a full queue drops the trace,
and collector errors are counted at `/api/instruments` under `otlp`.

### Offload and provider mode (`@prefex-<name>` targets)

A turn addressed to an offload target goes **only** to that target's
configured provider: local ollama, a hosted provider from
`internal/offload/registry.go` (OpenAI, OpenRouter, Groq, Cerebras, DeepSeek,
Gemini and others), or an OpenAI-compatible base URL you set. A failed target
never falls back to Anthropic.

- **Offload** (Claude Code, Codex, opencode): the task text plus the contents
  of any `@path` files it references. Files resolve **inside the project
  directory only**, known secret file shapes are refused, and total size is
  capped (`internal/offload/refs.go`). Session history is not sent.
- **Provider mode** (a model alias sent by any other app): the caller's own
  system prompt and **whole message history** go to the target's provider, the
  way any provider would receive them.

prefex ships no built-in targets: every `@prefex-<name>` marker is one you
define, and `@prefex-auto` uses only providers whose keys you set. A marker
with no key returns a message naming the config key rather than silently
billing elsewhere, which makes it fail closed. `offload.enabled: false`
disables every target.

**Side jobs are on by default.** `@prefex-auto`, `/pf-delegate` and the
`prefex_delegate` tool (offered to Claude Code and Codex in the tools list,
`offload.auto.delegate_tool`) hand a self-contained task to a free or local
model. Each job runs only at the privacy level it allows: **local** (your own
machines), **no_training** (providers whose stated policy is not to train on
inputs) or **any**. Jobs default to no_training, a short public question may
go to any, and anything carrying credentials, personal data or private files
is raised to local; a session once raised stays raised. A tier reports the
provider's stated policy, never a guarantee. Limit it per session with
`/pf-pin level local`, per project on the project page, or for a whole team
with `offload.auto.min_privacy`. The main conversation is never moved.

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
- The vault's TLS-intercepting proxy mints a **local root CA** stored in
  `~/.prefex/data/prefex.db`. Its private key is sealed under that same vault
  key, so reading the database does not yield a key that can forge
  certificates; the database is chmod'd `0600` when secrets are enabled.

### The root CA carries no name constraints, deliberately

A root you install in your trust store can, by construction, sign a
certificate for **any** host — not only the hosts on the proxy's allowlist.
X.509 name constraints could narrow that, and this CA does not set them.

The reason is that constraints on this root would be a **partial** control
whose failure mode is confusing rather than safe. The private key is on the
same machine as the trust store, so anyone who can read the key can also mint
a second unconstrained root and install it; the constraint stops an accident,
not an attacker with local read access. Meanwhile it would break the ordinary
case — the allowlist is edited at runtime, so a constraint frozen at CA
generation would start refusing hosts the operator has legitimately added,
with a TLS error that names the certificate rather than the policy.

The controls that actually bound this are: the CONNECT allowlist
(`secrets.allow_all` off), the sealed root key, and the fact that the proxy
listens on `127.0.0.1` only. If you want the narrower guarantee, do not
install the root — run without `secrets.enabled` and the proxy never
intercepts TLS at all.

Deferred with this reasoning during the 2026-09 `internal/secrets` audit
(FIX-PLAN.md gate G4). Revisit if the CA is ever generated per-allowlist
rather than once, or if a deployment installs this root somewhere the
operator does not control.

---

## PII handling (optional, opt-in)

With `pii_scrub.enabled: true`, Prefex redacts configurable entity types (email,
IP, phone, API keys, Stripe/GCP/Azure credentials, Slack tokens, SSN, plus your
own `custom_patterns`) from request bodies **before** they are forwarded
upstream. Each value becomes a placeholder such as `[PII_EMAIL_1]`. This is off
by default; enable it for team deployments that may carry customer data.
Implementation: `internal/pii/scrubber.go`, `internal/pii/restore_writer.go`.

**What is scrubbed:** `messages[*].content` text blocks and the text of
`tool_result` blocks. The system prompt and tool definitions are not scrubbed.

**Not masked, by design (#288).** These values are exempt, because the model
often has to write them back exactly (a commit trailer, a config file) and a
masked value would put the placeholder into a real side effect:
- exact values listed in `pii_scrub.allow` (case-insensitive), typically your
  own commit email
- `noreply` / `no-reply` / `donotreply` senders and `users.noreply.github.com`
- private, loopback, link-local and RFC 5737 documentation IP ranges

The email pattern matches the whole domain, so `.co.uk` or `.github.com` is
masked or exempted as a unit, never left partly in clear.

**Restoration (#288, #317).** The placeholder map exists only in memory for
the one request (see the table above). Placeholders the model echoes are
restored on the way back in streamed `text_delta` text **and in tool-call
arguments** (`input_json_delta`, restored JSON-escaped), including a
placeholder split across two deltas. Non-streaming JSON bodies (text and
`tool_use.input`) are restored the same way. Before this fix, a masked value the
model wrote into a tool call reached the tool as its placeholder. Thinking
deltas are never touched, because that would break their signature. A stream
that contains no placeholder passes through byte-for-byte. The scrubber
re-encodes without HTML-escaping `<`, `>`, `&` (#317), so masking does not also
change unrelated bytes of the cached prompt.

**Who sees masked text and who does not.** The upstream LLM and the streaming
response guard see placeholders only. The **request-phase guardrail runs before
scrubbing** and sees the original text (see
[Guardrail providers](#guardrail-providers-guardrailsenabled-default-false)).
The dashboard counts each masked value once per session rather than once per
re-sent turn.

---

## Signed audit chain (`audit_chain.enabled`, default **off**)

An append-only, hash-chained log in which every entry is signed with Ed25519.
One entry is written for each policy gateway decision, each guardrail verdict,
and each config reload, whether applied or refused.

- **Where:** `~/.prefex/audit/chain.jsonl` by default (`audit_chain.path`), one
  JSON object per line. `sink: s3` (team build) writes to your bucket with your
  AWS credentials, and `sink: memory` keeps it in process only. `siem` is declared
  but refuses to start, as does any unknown sink. Neither falls back to the file.
- **Signing key:** generated on first use at
  `~/.prefex/keys/audit-ed25519.pem` (private, mode 0600) and `.pub`
  (`audit_chain.key_dir` overrides the folder). `PREFEX_AUDIT_SIGNING_KEY` (base64
  PKCS8 PEM) overrides the on-disk key. The key never leaves the machine.
- **What an entry contains** (`internal/auditchain/audit.go`): `ts`,
  `request_id`, `provider` (`policy_gateway`, `guardrail:<name>`, or `config`),
  `session_id`, `org_id`, `developer_id`, `user`, `role` (the mode), `tool` (the
  tool name, guardrail phase, or `reload`), `args_summary`, `status`
  (`completed`/`failed`/`denied`), `error`, `result_hash`, `duration_ms`,
  `chain_pos`, `prev_hash`, `signature_b64`. The `args_summary` is built from
  verdict metadata only: `outcome/severity/channel/model/reason` for a policy
  decision, `action/category/channel/model/reason` for a guardrail verdict, or
  the changed top-level config sections for a reload. It **never includes
  tool arguments or prompt text** (`internal/baseline/audit_chain.go`).
- **Fails open:** a write error is logged and the request proceeds. A chain
  that cannot be built at startup or reload logs `audit chain: WARNING` and
  stays off.

**Verify:** `curl -s localhost:8019/api/audit/chain/verify` walks the local
chain, checking every signature and every `prev_hash` link, and reports the
first entry that fails. The result also appears under Activity, then
Guardrails.

## Session taint window

With the policy gateway on, prefex keeps a **10-minute, in-memory** record per
session of what earlier **allowed** tool calls touched (`internal/taint`). A
blocked call never ran, so it is not recorded. It stores metadata only:

- `sensitive`: tool name plus the path (or the head of the command) of a
  sensitive read, search or execute
- `untrusted`: tool name plus the domain, for WebFetch/WebSearch content or a
  `tool_result` the tool-poisoning scan flagged

The record is capped at 8 sources per session and never holds file contents.
It is passed to the policy as `input.taint`, so it stays local with the
in-process bundle and goes to your server with an external OPA `url`. A restart clears the record, which can drop
protection for up to 10 minutes but can never cause a false block.

---

## Telemetry & leaderboard

Two things are sent automatically: the **anonymous daily usage report**,
described first below, which you can turn off, and the **licence check**
described above, which has no switch. Everything else is opt-in or manual.

### Anonymous usage reporting (`usage_reporting.enabled`, default **on**)

**Why this exists, plainly:** prefex is built by a very small team, and we have
no other way to know whether it works outside our own machines. One summary per
day tells us how many people run it, on what kind of workload, and whether the
layers actually earn their place. It is how we decide what to build next and how
we make the case that this is worth continuing to fund. We would rather say that
than dress it up as being for your benefit.

**Exactly what is sent** — counts and percentages, once a day:

| field | example |
|---|---|
| `version`, `os`, `arch` | `v1.14.0`, `darwin`, `arm64` |
| `tokens_processed`, `turns`, `sessions`, `active_hours` | `150700000`, `290`, `29`, `0.8` |
| `avg_context_tokens`, `peak_context_tokens` | `519000`, `860000` |
| `tokens_cached`, `cache_hit_pct` | `150600000`, `91.7` |
| `tokens_compacted`, `tokens_compacted_distinct` | `3600000`, `128000` |
| `compaction_coverage_pct` | `27.9` |
| `provider_mix`, `tier_mix`, `channel_mix` | `{"Anthropic": 100.0}`, `{"opus": 79.0}` |
| `layers_on` | `["router", "cache", "compression"]` |

**What is never sent, by construction:**

- prompts, completions, or any tool output
- file paths, project names, or repository names
- model *names* — only the tier (`opus` / `sonnet` / `local`), because a model
  name can encode a private deployment or fine-tune
- API keys, session IDs, device IDs, account identifiers, IP-derived location
- absolute dollar figures

There is no account and no identifier. Two reports from the same machine on two
days cannot be linked to each other.

**The collector enforces this too, not just the client.** The endpoint stores
only fields on a fixed allowlist and discards anything else, so a future bug in
prefex cannot quietly widen what we hold. The list lives in
`site/api/usage/submit.js`; the payload it mirrors is `buildUsageReport` in
`internal/metrics/telemetry.go`. Both are in the repository — read them rather
than trusting this table.

**On by default, and you are told.** `prefex install` turns it on without
asking — from a terminal, `curl | bash`, or the PromptForce app, which uses the
defaults — and prints the disclosure above as a named install step (visible in
`--dry-run` too, so you can read it before running anything). Nothing is sent
until that notice has been shown (`consented: true`), so an upgrade cannot start
reporting silently. A re-run or upgrade keeps an earlier off.

**Turning it off**, at any time:

```bash
prefex install --no-usage-reporting     # decline up front, before any report
prefex toggle usage-reporting off       # any time afterwards
prefex toggle usage-reporting on        # and back on
```

With the install scripts, `PREFEX_USAGE_REPORTING=0` or `DO_NOT_TRACK=1` passes
`--no-usage-reporting` to `prefex install`:

```bash
PREFEX_USAGE_REPORTING=0 curl -fsSL https://promptforce.ai/install.sh | bash
```

In the dashboard: **Settings · Privacy → Anonymous Usage Reporting**. The
checkbox there is the same setting and takes effect immediately.

Or in `~/.prefex/config.yaml`:

```yaml
prefex:
  usage_reporting:
    enabled: true    # absent or false = off
```

**We do not nag.** There is no periodic re-prompt. The dashboard toggle is where
the setting lives if you change your mind.

Note that `usage_reporting` is a different setting from `telemetry`, which only
**downloads** the compaction-rule catalog and transmits nothing. Turning off
`telemetry` does not stop usage reports, and turning off `usage_reporting` does
not stop catalog downloads.

Reports are signed with a build secret to keep fabricated submissions out of
our numbers. That secret ships inside the binary, so it is a spam deterrent
rather than authentication — we would rather state that than imply a guarantee
it does not provide.

### Marketplace catalog (`telemetry.enabled`, default **false**)

A **download**, not an upload, and unrelated to the report above. When on, the
daemon fetches the optional marketplace catalog from promptforce.ai at
startup and daily. No usage data is uploaded by this flag; it controls that one
download. The two are separate settings precisely so neither can be changed by
adjusting the other.

```yaml
prefex:
  telemetry:
    enabled: false
```

- **`pricing.autofetch`** (default **false**) / **`prefex pricing refresh`**
  (manual): downloads the public LiteLLM model-pricing table from
  raw.githubusercontent.com to `~/.prefex/pricing-cache.json` so newly released
  models price correctly between prefex releases. Download-only — no usage
  data, keys, or identifiers are sent. With both off (the default), model
  pricing resolves from a snapshot embedded in the binary and there is zero
  egress from this feature.

  **Where the prices come from:** all model rates are sourced from the
  [LiteLLM](https://github.com/BerriAI/litellm) project's public
  `model_prices_and_context_window.json` (MIT-licensed community pricing
  metadata, ~3,400 models across every major provider). The copy embedded in
  each prefex release is fetched from that file at **build time**; the opt-in
  refresh above fetches the same file on demand. Your own
  `pricing.rates` config always overrides it. Verify which source priced any
  model with `prefex pricing show <model>` — it prints the tier
  (`override` / `cache` / `snapshot` / `legacy`) next to the rate, and
  unknown models are flagged `unpriced` rather than silently priced.

- **Leaderboard submission** (manual): only when you click *Join* in the
  dashboard. It sends aggregate stats (request counts, savings totals — never
  prompt/response content) **plus the username and email you enter** so your
  entry can be attributed. Don't submit if you'd rather stay off the board.

- **`team.billing.enabled`** (default **false**, team tier only): when on, the
  daemon sends one HMAC-signed usage report per calendar month to
  promptforce.ai/api/billing/report for the team tier's guarantee
  accounting. The full payload is exactly these fields — aggregate dollars
  only, **never prompt/response content, never per-developer identity** (org
  and install IDs only): `schema`, `org_id`, `install_id`, `month`,
  `baseline_usd`, `actual_usd`, `saved_usd`, `savings_pct`, `fee_usd`,
  `seats`, `version`, `ts`, `sig`. The dashboard's Savings Meter card shows
  this payload verbatim before you enable the flag, every sent report is
  stored locally in the `billing_reports` table
  (`sqlite3 ~/.prefex/data/prefex.db 'SELECT * FROM billing_reports'`), and a
  manual *Send now* path exists if you prefer to keep the automatic flag off.

  ```yaml
  prefex:
    team:
      billing:
        enabled: false
  ```

### Security advisories — a pull, not a report

`prefex advisories` checks whether your build is affected by a published
security issue. It is deliberately **one-way**:

- Prefex downloads a signed advisory file and compares it against its own
  version **locally**.
- Nothing is transmitted. Not your version, not an install ID, not a token.
  The request carries no query string and no custom headers — pinned by a test
  (`TestCheckSendsNoIdentifyingData`) rather than only promised.
- The file is signed with Ed25519. The binary embeds the **public** key only,
  so an attacker who extracts it still cannot forge an advisory. A file that
  fails verification is **discarded**, never displayed with a caveat — a
  hijacked endpoint serving "CRITICAL: download this binary" is precisely the
  attack this prevents.

The honest cost of that design: **we cannot see who is affected, and cannot
notify you.** The advisory check is a pull — prefex asks, we answer — so there
is no list of installs to reach.

**One exception, and it used to be documented wrongly here.** Unless you
turn usage reporting off at install time, the install scripts (`install.sh`, `install.ps1`) also send a single request to
`promptforce.ai/api/ping` when they finish, carrying **the version, OS,
architecture, and whether this was an install or an upgrade** — nothing else.
No hostname, no paths, no identifier, and nothing at all from the daemon or
your prompts. It fires once per install or upgrade, never afterwards.

This paragraph previously said "We chose not to build that", which was wrong:
the line had shipped in `install.sh` for some time. `PREFEX_USAGE_REPORTING=0`,
`PREFEX_NO_PING=1` or `DO_NOT_TRACK=1` suppresses it, on both scripts:

```bash
PREFEX_NO_PING=1 curl -fsSL https://promptforce.ai/install.sh | bash   # no ping
```

**Don't export `DO_NOT_TRACK` globally just to be safe.** It does more than
silence our ping: Claude Code treats it
(along with `DISABLE_TELEMETRY`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` and
`DISABLE_GROWTHBOOK`) as a signal to skip the feature-flag evaluation that
Remote Control depends on, so exporting it turns Remote Control off. Inline on
the install command, as above, it applies to that one command and has no such
effect.

**The update check is the other outbound call, and it works the same way.**
Once a day the daemon asks GitHub for the latest release tag of the public
releases repo and compares it locally. The request carries **no version, no
identifier and nothing about this install** — it reads a public tag, so it
cannot be used to tell who is running what. GitHub sees the request, which is
why this names the host rather than just saying "we check for updates". The
result appears under Settings · Diagnostics and in `prefex status`; nothing is
ever downloaded or installed without you running `prefex update` yourself.

On by default. Two ways to switch it off, because they mean different things:

```bash
prefex toggle update-check off      # this install
DO_NOT_TRACK=1                      # machine-wide, honoured here and by the installers
```

Note the same caveat as above: exported globally, `DO_NOT_TRACK` also disables
Claude Code's Remote Control, because Claude Code reads it as a signal to skip
feature-flag evaluation. Use `prefex toggle update-check off` if you only mean
to stop this check.

**Separately, and not something prefex chooses: Remote Control is off for any
session routed through a proxy.** Claude Code requires `ANTHROPIC_BASE_URL` to
be `api.anthropic.com` for it, and checks the host literally — so it is off
while prefex is running, and its error message names the variable. To use
Remote Control, run `prefex stop` and start a new Claude Code session.

**Two Claude Code calls outside the Messages API are forwarded to
`api.anthropic.com` unchanged.** Claude Code sends both to whatever
`ANTHROPIC_BASE_URL` names, so behind prefex they arrive here first:

- `HEAD /api/hello`: a connection preconnect Claude Code fires at startup.
  It has no body. prefex forwards it with **no headers from your request, and
  no credential**.
- `POST /api/oauth/file_upload`: Claude Code's attachment upload, which
  carries a file's contents. prefex forwards it **only with your own claude.ai
  OAuth bearer, byte for byte**. It streams the body without reading it,
  compacting it or logging it, and never substitutes a team or operator key. A
  request that doesn't carry your bearer is refused locally, and nothing
  leaves the machine.

prefex refuses every other `/api/*` path that reaches the proxy with its own
error and a log line naming the path, and forwards none of them. On port 8019,
`/api/*` also belongs to the unauthenticated dashboard, which refuses any
request that carries an Anthropic credential instead of serving it.

Run the advisory check periodically (or in cron) if the difference matters to
you:

```bash
prefex advisories          # exit 0 unaffected · 1 lookup failed · 3 affected
```

Dev builds carry no verification key and say so, rather than reporting a
clean bill of health they cannot actually vouch for.

---

## Fail-open guarantee

Prefex is designed to **fail open**: if any internal component errors (cache,
router, session store, scrubber), the request is logged and forwarded upstream
unchanged rather than blocked. An outage in Prefex degrades to a transparent
pass-through, never to a denied request.

The governance features follow the same rule by default, and each has one
explicit opt-in to fail closed:

| feature | default on failure | fail-closed opt-in |
|---|---|---|
| policy gateway (external server down, 401/5xx) | allow | `on_unavailable: deny` |
| guardrail provider (error, timeout, budget exceeded) | allow | per provider `on_error: block` |
| signed audit chain (write error) | request proceeds, error logged | none |
| OTLP export (queue full, collector error) | trace dropped, counted | none |
| config reload that fails validation | last good config kept, `config_synchronized: false` on `/api/health` | n/a |

Two paths are closed by design, because an open failure there would mislead:
an offload target with no credential returns a message naming the missing key
instead of silently billing Anthropic, and a Bedrock event-stream CRC failure is
returned as an error rather than guessed past.

---

## Team mode & access control

When team mode is enabled:

- Admin APIs (invites, member management, budgets, analytics, audit export) are
  gated behind a constant-time-compared `admin_token` or an OIDC admin session.
- Per-developer attribution uses an HMAC-hashed token; raw tokens are never
  stored — only their hash.
- `team.attribution` can also take identity from a request header, a JWT claim
  (the payload is read **without verifying the signature**), or
  `metadata.user_id`. Those sources are asserted identities: use them only
  behind a gateway that has already authenticated the caller. Whatever id they
  produce is what appears as `dev_id` in the local tables, the audit chain, and
  OTLP `prefex.dev_id`.
- The dashboard is intended for LAN/VPN exposure. For broader exposure, configure
  TLS (`dashboard.tls_cert_file` / `tls_key_file`) and front it with your own auth.

See `PRO_DEPLOYMENT.md` § 10 for the pre-go-live security checklist.

---

## How to verify these claims

1. **No prompt text in the request log, and what IS kept:**
   ```bash
   sqlite3 ~/.prefex/data/prefex.db ".schema requests"
   # columns are counts/hashes/metadata — no prompt/response body columns
   curl -s localhost:8019/api/privacy
   # every switch that keeps text, whether it is on, rows held, and the window
   ```
   To take your record with you, `prefex export --text`; to delete it from this
   machine, `prefex forget` (saved keys, team settings and the licence stay).
2. **Outbound connections:** run the proxy under `lsof -i` / a network monitor and
   confirm connections only to the configured upstream (and optional telemetry).
3. **Source availability:** the proxy is a single Go binary; behavior is auditable
   against the documented request flow in `CLAUDE.md`.
4. **Account for every call to `promptforce.ai`.** By default there are
   two: the licence check above (on start and daily) and exactly one anonymous
   daily usage report described above. Watch it on the wire, or read what it would
   send before it sends anything:

   ```bash
   curl -s localhost:8019/api/usage?period=today   # the same figures, verbatim
   prefex toggle usage-reporting off               # then expect zero calls
   ```

   With usage reporting off, `telemetry.enabled: false` (the default), no manual
   leaderboard submission, and `team.billing.enabled: false` (also the default),
   the only outbound traffic is to your configured LLM provider. With billing
   reports enabled, the one additional call per month is auditable byte-for-byte:
   `sqlite3 ~/.prefex/data/prefex.db 'SELECT payload_json FROM billing_reports'`.
5. **Governance features are off, or point where you expect.** Check the four
   config keys that can add a destination:
   ```bash
   grep -nE 'otlp_endpoint|^\s*url:|backend:|providers:' ~/.prefex/config.yaml
   ```
   With `policy_gateway.url` empty, `guardrails` off or `regex`-only,
   `telemetry.otlp_endpoint` empty and no offload marker used, none of them
   produces any traffic.
6. **The audit chain is intact:**
   ```bash
   curl -s localhost:8019/api/audit/chain/verify   # ok, entries_verified, first_bad
   ```
   To see what an entry holds, read the file directly
   (`tail -1 ~/.prefex/audit/chain.jsonl`). It should contain no prompt text or
   tool arguments.
7. **Policy decisions are logged locally:**
   ```bash
   sqlite3 ~/.prefex/data/prefex.db \
     'SELECT ts, tool_name, outcome, request_id FROM policy_gateway_events ORDER BY id DESC LIMIT 5'
   ```

---

## Data deletion

All state is local. To purge: stop the proxy and delete `~/.prefex/data/prefex.db`
(history) and `~/.prefex/config.yaml` (config). Archived copies, if any, live under
`~/.prefex/archive/` and are never auto-deleted — remove them manually.
