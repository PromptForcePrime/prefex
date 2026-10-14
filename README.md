<table border="0"><tr>
<td valign="middle"><img src="img/promptforce-mark.png" width="78" alt="Prefex"></td>
<td valign="middle"><pre>
██████╗ ██████╗ ███████╗███████╗███████╗██╗  ██╗
██╔══██╗██╔══██╗██╔════╝██╔════╝██╔════╝╚██╗██╔╝
██████╔╝██████╔╝█████╗  █████╗  █████╗   ╚███╔╝ 
██╔═══╝ ██╔══██╗██╔══╝  ██╔══╝  ██╔══╝   ██╔██╗ 
██║     ██║  ██║███████╗██║     ███████╗██╔╝ ██╗
╚═╝     ╚═╝  ╚═╝╚══════╝╚═╝     ╚══════╝╚═╝  ╚═╝
</pre></td>
</tr></table>

[![Release verified](https://github.com/PromptForcePrime/prefex/actions/workflows/verify-release.yml/badge.svg)](https://github.com/PromptForcePrime/prefex/actions/workflows/verify-release.yml)
[![Release](https://img.shields.io/github/v/release/PromptForcePrime/prefex)](https://github.com/PromptForcePrime/prefex/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/PromptForcePrime/prefex/total)](https://github.com/PromptForcePrime/prefex/releases)
[![Platforms](https://img.shields.io/badge/platforms-macOS%20%7C%20Linux%20%7C%20Windows-blue)](VERIFICATION.md)
[![Checksums](https://img.shields.io/badge/releases-SHA256%20checksums-informational)](VERIFICATION.md)
[![Cache-safe](https://img.shields.io/badge/cache--safe-self--verified-success)](VERIFICATION.md#cache-safe)
[![Trust](https://img.shields.io/badge/trust-data%20handling-blueviolet)](TRUST.md)
[![Internal suite](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2FPromptForcePrime%2Fprefex%2Fbadges%2Ftests.json)](VERIFICATION.md)
[![Coverage](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2FPromptForcePrime%2Fprefex%2Fbadges%2Fcoverage.json)](VERIFICATION.md)


**Spend less on Claude, with receipts.**

Prefex is a local-first proxy for Claude Code and the Anthropic / OpenAI APIs. The best ideas for cutting
LLM cost go viral one at a time, scattered across the ecosystem as sidecars, plugins, and cloud services.
Prefex distills a dozen of them, rebuilds each one natively in Go, and stacks them into a single
self-contained binary that runs behind one `base_url` change. No code changes, your prompts go only
to the models you use, and an honest dashboard that shows what Prefex actually saved you beyond the native prompt
caching you already get for free.

Prefex is a [Promptforce.ai](https://promptforce.ai) product. Promptforce.ai is a Day Two AI company.

> **Pro mode is free until Prefex saves you $500**: the first $500 of savings *beyond* native prompt
> caching is included, no card, no obligation. Get your install link at
> [promptforce.ai](https://promptforce.ai). See [Pro mode and Team mode](#pro-mode-and-team-mode).
>
> **Community:** [Discord](https://discord.gg/Ke5BdFB9Au) · [GitHub](https://github.com/PromptForcePrime/prefex)

---

## Install

The easiest way is the **PromptForce app** (macOS 13+ and Windows, beta): it installs Prefex and
ReadyBase, connects Claude Code and Codex, and lives in the menu bar. Download it from
[promptforce.ai](https://promptforce.ai). Mac builds are signed and notarized; Windows builds are
Authenticode-signed.

Or install Prefex alone from the command line:

```bash
# macOS / Linux. Download the script and review it before running.
curl -fsSL https://promptforce.ai/install.sh -o install.sh
less install.sh
bash install.sh
```

```powershell
# Windows PowerShell
iwr https://www.promptforce.ai/install.ps1 -OutFile install.ps1; .\install.ps1
```

We recommend downloading the script and reading it before you run it. The installer starts Prefex on
`:8019`, connects Claude Code, and is reversible with `prefex stop` or the uninstaller.

Then point your client at it:

```json
// Claude Code: ~/.claude/settings.json
{ "env": { "ANTHROPIC_BASE_URL": "http://localhost:8019" } }
```

Dashboard: http://localhost:8019/dashboard

---

## What it does

A dozen-plus optimizations across twelve layers, in one binary:

- **Smart routing**: an explainable rules router sends simple turns to a cheaper model and keeps the
  hard ones on the strong one. You set the threshold.
- **Prompt-cache assist**: prefix detection, a cache warmer, prefix-stabilization, and prewarm-on-switch
  keep your cached context hot — including pre-paying the re-write before a model switch or a proxy
  stop, so you never eat the cold-turn spike.
- **Compression**: lossless-first compaction of tool output, logs, and JSON. Cache-boundary-safe and
  recoverable. Never lossy on code.
- **Cross-session memory**: typed facts extracted from your sessions and re-injected by project, so a
  new session starts with what the last one learned. Strictly project-scoped — one project's memory
  never surfaces in another.
- **Session handover**: when a session ends, Prefex distills a structured note — goal, progress, key
  files, blockers, next steps — and hands it to your next session in that project. Injected once,
  cache-safely, so continuity costs nothing per turn.
- **Spend guardrails**: token traps, loop guards, think-caps, and lean reasoning catch runaway turns
  before they bill.
- **Policy gateway**: an embedded OPA engine evaluates every tool call against your Rego policies —
  advisory logging or hard enforcement. MCP tool descriptions are scanned at session start and
  suspicious ones flagged on a security activity view (detection and audit; the scan flags, it doesn't
  block). Policies evaluate in-process on your machine.
- **Output shaper**: trims model verbosity on mechanical turns — measured ~11% fewer output tokens,
  the ones that bill at ~5× input and never cache.
- **Side jobs on free and local models**: `@prefex-auto`, `/pf-delegate`, and a delegate tool your
  agent can call hand self-contained work (a summary, a first-pass review) to a free or local model
  instead of Claude. Each job runs only at the privacy level it allows (your machines, providers that
  state they don't train on inputs, or any provider); credentials and personal data stay on your
  machines, and your main conversation never changes model.
- **Receipts and coaching**: savings attributed per mechanism, in honest incremental terms, then
  surfaced inline as you work so the numbers actually change how you spend.
- **Encoder routing** (optional): an on-device ONNX encoder — a frozen ModernBERT trunk that runs
  fully offline — scores prompt complexity for sharper routing than rules alone. Opt-in, default-off.
- **Spend watcher**: a background sentinel reads the live request stream and flags the expensive stuff
  the moment it happens — big cold cache writes, prefix flapping, rate limits, spend spikes — to your
  status line and dashboard.
- **Every tool, one daemon**: `prefex wrap codex` (or aider, cursor, zed) routes any tool through the
  same shared daemon — one dashboard, one savings ledger, per-tool attribution. Codex rides the OpenAI
  Responses API natively. Pricing for ~3k models resolves from an embedded snapshot; unknown models are
  flagged `unpriced`, never silently guessed.

Every layer is independent and fail-open. If any component errors, the request passes straight through to
the upstream API unchanged.

---

## Honest savings (no fake percentages)

There is no honest fixed percentage. It depends entirely on your workload, so Prefex shows you two numbers
and is clear about which is which:

- **Saved by Prefex** (the headline): what routing, trimming, and guardrails added *beyond* the native
  prompt caching Claude Code already gives you for free.
- **vs no optimization at all** (a labeled counterfactual): every token repriced at full strong-model
  rate with no caching discount. The bigger number, shown only for context.

Costs are read from the provider's own billing fields, not computed by Prefex. Don't trust a headline
number, including ours; check your own dashboard, request by request.

---

## Pro mode and Team mode

- **Pro mode** (one developer, one machine): every layer above, the full dashboard, and the first
  **$500 of savings beyond native prompt caching included**. It is measured by the same "Saved by
  Prefex" figure the dashboard headlines, so what you've used is always on your own screen. Prefex warns
  at 80%; when the $500 is reached you get 15 days to add a licence before it stops and hands your tools
  back to the provider directly.
- **Team mode** (an organization): everything in Pro for each seat, plus a team dashboard with per-developer
  spend and savings attribution, org-scoped reporting, per-developer monthly quotas, and shared
  guardrail and policy settings. Seats join the team's Prefex with one command.

Licences for continued Pro use and for Team mode: [promptforce.ai](https://promptforce.ai) or
[contact@daytwoai.com](mailto:contact@daytwoai.com).

---

## Connect your client

Anything that lets you set a custom API base URL works out of the box.

| Tool | How to connect |
|------|----------------|
| Claude Code | `ANTHROPIC_BASE_URL` in `~/.claude/settings.json` (auto-configured by the installer) |
| Cursor | `ANTHROPIC_BASE_URL` env var in your shell |
| Continue.dev | `apiBase` per model in `~/.continue/config.json` |
| Aider | `--api-base http://localhost:8019` |
| Codex CLI | `prefex wrap codex` (routes this session only) |
| Python / Node SDK | `base_url="http://localhost:8019"` |

---

## Principles

1. **Fail open.** Internal errors never block a request. On bad input, a missing dependency, or a slow
   path, Prefex logs and forwards upstream unchanged.
2. **Binary, not source.** One CGO-free Go binary. No runtime fan-out of processes, no sidecar to
   supervise. Trained weights are the one exception, behind an opt-in, default-off seam (see Attribution).
3. **Your prompts and keys stay home.** Everything runs on localhost. Prompts, code and keys go only to
   the providers you use: the one you already call, plus, for side jobs you hand off, a local model or a
   provider whose key you added. Beyond that, Prefex sends two things to promptforce.ai: an anonymous
   daily usage report (counts and percentages only, on by default, off with
   `prefex toggle usage-reporting off`), and a small licence check carrying a salted machine hash,
   version, key id and three numbers, never a prompt, path or identity. It also checks GitHub daily for a
   newer release. [promptforce.ai/trust](https://promptforce.ai/trust) lists every field. The dashboard
   has no auth because nothing can reach it but you.

Verify it yourself:

```bash
# Outbound traffic: your provider(s), any side-job provider you added, api.github.com, promptforce.ai
sudo tcpdump -i any -n 'dst port 443' 2>/dev/null | grep -v api.anthropic.com
```

---

## Credits and attribution

Prefex is a **distillery**: we study the best ideas in AI tooling and re-author the *algorithm* natively
in Go, inside our own principles. Copyright protects code, not ideas, so for the projects below **no
source code is copied into Prefex**: these are courtesy credits, not license obligations. We name them
because the ideas are theirs and the credit should be too.

The 16 GitHub projects below carry 447k+ stars between them (counted 2026-09-29; papers, runtimes and
platform features are credited but not counted).

### Ideas distilled into native Go (clean-room, no code used)

| Source | The idea we learned from | License |
|--------|--------------------------|---------|
| [RouteLLM](https://github.com/lm-sys/RouteLLM) | A cheap classifier routes simple queries to a weak model | Apache-2.0 |
| [LLMLingua / LLMLingua-2](https://github.com/microsoft/LLMLingua) | Token-importance scoring to drop low-value tokens | MIT |
| [caveman-compression](https://github.com/wilpel/caveman-compression) | Strip filler from natural-language text for cheap token shaving | MIT |
| [RTK](https://github.com/rtk-ai/rtk) | Intercept tool output and compress it before it reaches context | Apache-2.0 |
| [headroom](https://github.com/headroomlabs-ai/headroom) | Content-routed compaction, recover-on-demand CCR, and output shaping (terse-steer + effort-down on routine turns) | Apache-2.0 |
| [Mem0](https://github.com/mem0ai/mem0) | Cross-session memory via extracted facts and entities | Apache-2.0 |
| [MemGPT / Letta](https://github.com/letta-ai/letta) | Tiered memory and self-managed context window | Apache-2.0 |
| [OpenHands](https://github.com/OpenHands/OpenHands) | Condense a task span into a structured progress note | MIT (core) |
| [codeburn](https://github.com/getagentseal/codeburn) | Dead-MCP-tool detection thresholds, cache-aware waste pricing, embedded pricing snapshot | MIT |
| [Token Meter](https://github.com/splunk/token-meter) (Splunk) | Evidence semantics: every displayed number carries its provenance (covered / partial / estimated / unavailable), and a period with no comparable prior says so instead of showing a zero delta | MIT |
| [Switchyard](https://github.com/NVIDIA-NeMo/Switchyard) (NVIDIA) | A typed classifier taxonomy for routing (task type, complexity, reasoning depth, risk) rather than one score | Apache-2.0 |
| [Invariant Scanner](https://github.com/invariantlabs-ai/invariant) | Typed violation taxonomy for agent tool-call traces — theirs audits after the fact, our policy gateway gates live on the wire | Apache-2.0 |
| [DefenseClaw](https://github.com/cisco-ai-defense/defenseclaw) | Tool-result scanning, MCP tool-description admission scanning, severity-tiered outcomes, observe-then-enforce workflow — re-authored on prefex's wire position, keeping our fail-open posture and Rego (not their YAML schema) | Apache-2.0 |
| [GraphRAG](https://github.com/microsoft/graphrag) | Bi-temporal pattern: don't lose history on a lossy edit | MIT |
| [llmfit](https://github.com/AlexsJones/llmfit) | Scoring candidate local models against detected hardware by weighted fit (VRAM, RAM headroom, disk, GPU class) | MIT |
| [Experiential](https://github.com/experientiallabs/experiential) | Paired-difference estimation with a variance floor (a constant sample cannot report zero uncertainty), and coerce-with-disclosure — every request substitution named as `path->effective`, logged and counted rather than applied silently | Apache-2.0 |
| [Chain of Draft](https://arxiv.org/abs/2502.18600) (paper) | Draft-then-refine prompting to cut reasoning tokens, applied after the last cache breakpoint so the cached prefix never moves | paper |
| [vLLM](https://github.com/vllm-project/vllm) / [Ollama](https://github.com/ollama/ollama) | Local inference for cheap auxiliary tasks (judge, distillation, router scoring) | Apache-2.0 / MIT |
| Anthropic (native) | Prompt cache and server-side context editing, the things we extend | platform |

ReadyBase (also known as code-drift), the co-change and blast-radius graph that gates code compaction, is
a sister Promptforce project.

### Components we actually package (real license obligations)

The one place Prefex distributes or runs third-party code and weights is the **optional, default-off
Kompress text-compaction sidecar** (the "headroom" model track). It runs out of process; the Go binary
stays model-free and fails open if the sidecar is absent. These obligations apply **only if you opt into
the sidecar** (`compression.kompress: true`). See [`NOTICE`](NOTICE) and `deploy/kompress-sidecar/`.

| Component | Role | License |
|-----------|------|---------|
| **[headroom / Kompress](https://github.com/headroomlabs-ai/headroom)** | The keep/drop model + ONNX inference our sidecar runs (headroom's *algorithms* are reimplemented in Go and credited above; this entry is the *model*) | Apache-2.0 |
| [`chopratejas/kompress-v2-base`](https://huggingface.co/chopratejas/kompress-v2-base) | The keep/drop model weights the sidecar serves | Apache-2.0 |
| [ModernBERT](https://github.com/AnswerDotAI/ModernBERT) | The base encoder those weights are fine-tuned from | Apache-2.0 |
| [ONNX Runtime](https://github.com/microsoft/onnxruntime) | Inference runtime for the model (no PyTorch) | MIT |
| [Hugging Face Transformers](https://github.com/huggingface/transformers) | Tokenization for the sidecar | Apache-2.0 |

**Bundled Go libraries.** The most notable third-party modules linked into the Prefex binary:
[Open Policy Agent](https://github.com/open-policy-agent/opa) (Apache-2.0, the embedded policy-gateway
engine), [go-redis](https://github.com/redis/go-redis) (BSD-2-Clause),
[yaml.v3](https://github.com/go-yaml/yaml) (MIT and Apache-2.0), and
[modernc.org/sqlite](https://gitlab.com/cznic/sqlite) (BSD-3-Clause). The binary also embeds LiteLLM's
[model-pricing snapshot](https://github.com/BerriAI/litellm) (MIT) as data. The full set of 45 bundled
modules (direct and transitive) is in [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md); see also
[`NOTICE`](NOTICE).

If we have miscredited or mislabeled anything, please open an issue and we will correct it.

---

## License and terms

Prefex is distributed as a binary, in Beta, provided as-is with no warranty. Source code is not included
in this repository (binary releases only). Pro mode is free until Prefex saves you $500 beyond native
prompt caching; licences and Team mode at [promptforce.ai](https://promptforce.ai) or
[contact@daytwoai.com](mailto:contact@daytwoai.com).

Third-party components bundled or invoked by the optional sidecar retain their own licenses, recorded in
[`NOTICE`](NOTICE).

---

## A note on Anthropic

Day Two AI is an Anthropic partner. **Prefex is built independently, with no Anthropic participation,
funding, or endorsement.** It is not an Anthropic product and is not affiliated with Anthropic. "Claude"
and "Anthropic" are trademarks of Anthropic; they are used here only to describe interoperability. Prefex
sits in front of the Anthropic API as a local proxy and never sends your prompts or keys anywhere but the
provider you are already calling.
