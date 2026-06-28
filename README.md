# Prefex

**Spend less on Claude, with receipts.**

Prefex is a local-first proxy for Claude Code and the Anthropic / OpenAI APIs. The best ideas for cutting
LLM cost go viral one at a time, scattered across the ecosystem as sidecars, plugins, and cloud services.
Prefex distills a dozen of them, rebuilds each one natively in Go, and stacks them into a single
self-contained binary that runs behind one `base_url` change. No code changes, no data leaving your
machine, and an honest dashboard that shows what Prefex actually saved you beyond the native prompt
caching you already get for free.

Prefex is a [Promptforce.ai](https://promptforce.ai) product. Promptforce.ai is a Day Two AI company.

> Prefex is in **Early Access**: free for 60 days, no obligation. Get your install link at
> [promptforce.ai](https://promptforce.ai).

---

## Install

```bash
# macOS / Linux. Download the script and review it before running.
curl -fsSL https://promptforce.ai/install.sh -o install.sh
less install.sh
bash install.sh
```

```powershell
# Windows PowerShell
iwr https://promptforce.ai/install.ps1 -OutFile install.ps1; .\install.ps1
```

We do not do `curl | bash`. Download the script, read it, then decide. The installer starts Prefex on
`:8019`, connects Claude Code, and is reversible with `prefex stop` or the uninstaller.

Then point your client at it:

```json
// Claude Code: ~/.claude/settings.json
{ "env": { "ANTHROPIC_BASE_URL": "http://localhost:8019" } }
```

Dashboard: http://localhost:8019/dashboard

---

## What it does

A dozen optimizations across six layers, in one binary:

- **Smart routing**: an explainable rules router sends simple turns to a cheaper model and keeps the
  hard ones on the strong one. You set the threshold.
- **Prompt-cache assist**: prefix detection, a cache warmer, and prefix-stabilization keep your cached
  context hot, extending the native prompt cache further than it goes alone.
- **Compression**: lossless-first compaction of tool output, logs, and JSON. Cache-boundary-safe and
  recoverable. Never lossy on code.
- **Cross-session memory**: typed facts extracted from your sessions and re-injected by project, so a
  new session starts with what the last one learned.
- **Spend guardrails**: token traps, loop guards, think-caps, and lean reasoning catch runaway turns
  before they bill.
- **Receipts and coaching**: savings attributed per mechanism, in honest incremental terms, then
  surfaced inline as you work so the numbers actually change how you spend.

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
3. **Your keys stay home.** Everything runs on localhost. No telemetry, no analytics, no external
   logging. The dashboard has no auth because nothing can reach it but you.

Verify it yourself:

```bash
# All outbound traffic should go only to api.anthropic.com / api.openai.com
sudo tcpdump -i any -n 'dst port 443' 2>/dev/null | grep -v api.anthropic.com
```

---

## Credits and attribution

Prefex is a **distillery**: we study the best ideas in AI tooling and re-author the *algorithm* natively
in Go, inside our own principles. Copyright protects code, not ideas, so for the projects below **no
source code is copied into Prefex**: these are courtesy credits, not license obligations. We name them
because the ideas are theirs and the credit should be too.

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
| [OpenHands](https://github.com/All-Hands-AI/OpenHands) | Condense a task span into a structured progress note | MIT (core) |
| [GraphRAG](https://github.com/microsoft/graphrag) | Bi-temporal pattern: don't lose history on a lossy edit | MIT |
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

**Bundled Go libraries.** The Prefex binary itself links three third-party Go modules:
[go-redis](https://github.com/redis/go-redis) (BSD-2-Clause), [yaml.v3](https://github.com/go-yaml/yaml)
(MIT and Apache-2.0), and [modernc.org/sqlite](https://gitlab.com/cznic/sqlite) (BSD-3-Clause). The full
set of 22 bundled modules (direct and transitive) is in
[`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md); see also [`NOTICE`](NOTICE).

If we have miscredited or mislabeled anything, please open an issue and we will correct it.

---

## License and terms

Prefex is distributed as a binary, in Beta, provided as-is with no warranty. Source code is not included
in this repository (binary releases only). Early Access is free for 60 days; Pro and Team plans are
available on request at [contact@daytwoai.com](mailto:contact@daytwoai.com).

Third-party components bundled or invoked by the optional sidecar retain their own licenses, recorded in
[`NOTICE`](NOTICE).

---

## A note on Anthropic

Day Two AI is an Anthropic partner. **Prefex is built independently, with no Anthropic participation,
funding, or endorsement.** It is not an Anthropic product and is not affiliated with Anthropic. "Claude"
and "Anthropic" are trademarks of Anthropic; they are used here only to describe interoperability. Prefex
sits in front of the Anthropic API as a local proxy and never sends your prompts or keys anywhere but the
provider you are already calling.
