# OpenClaw on Mac Mini — Local & Secure

> **Goal:** Run [OpenClaw](https://openclaw.ai/) as a fully **local, agentic AI assistant** on a dedicated Mac Mini, backed by **Gemma 4** served by **Ollama** — no cloud inference, no inbound ports, reproducible from a clean macOS install.

Companion repo and write-up of one engineer's path from a cloud-hosted OpenClaw to a self-hosted, single-purpose Mac Mini.

---

## Why this project

This project is the local-hardware evolution of an earlier OpenClaw deployment on AWS:

- **Previous setup (cloud):** [retinskiy/openclaw-aws](https://github.com/retinskiy/openclaw-aws) — OpenClaw on EC2 wired to Anthropic models via AWS Bedrock. It worked great, but it ran through credits fast.
- **LinkedIn write-up of the AWS setup:** [post by @mikhail-retinski](https://www.linkedin.com/posts/mikhail-retinski_ai-agenticai-aiagents-activity-7449390824154451968-Rwbi/)

For a 24/7 personal AI assistant the economics flip in favour of a small, dedicated machine running an open-weights model locally — once paid for, the marginal cost of inference is zero, and no data leaves the device.

---

## Architecture

```
┌─────────────────────── Mac Mini (loopback only) ───────────────────────┐
│                                                                        │
│   You ──► OpenClaw CLI / UI ──► OpenClaw Gateway ──► Ollama 11434      │
│                                        │                  │            │
│                                        │                  └► Gemma 4   │
│                                        │                               │
│                                        └► Skills · Workspace · Channels│
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

Design choices, with the reasoning behind each:

| Choice | Why |
|---|---|
| Dedicated Mac Mini | Single-purpose, always-on, low power, Apple Silicon Metal acceleration for Ollama. |
| **No Docker** | Saves RAM and avoids a hypervisor between Ollama and Metal. The Mac is single-purpose, so isolation by container is unnecessary. |
| **Single admin user** | One human, one machine, one purpose. No multi-tenant concerns. iCloud not signed in, to keep the box decoupled from a personal Apple ID. |
| **Gemma 4 via Ollama** | Open weights (Apache 2.0), 256K context, multimodal, runs comfortably on Apple Silicon, no API key. |
| **Loopback-only binds** | Both Ollama and the OpenClaw gateway bind to `127.0.0.1` — the box is invisible to the LAN even before the firewall kicks in. |
| **Tailscale (later phase)** | Remote access over a private mesh rather than opening any port to the LAN or the internet. |

---

## Repository layout

```
openclaw-macmini/
├── README.md          # this file — project overview
└── INSTALL.md         # step-by-step install guide (the meat)
```

The actual setup lives in [INSTALL.md](INSTALL.md), structured in phases so each step is reproducible from a clean macOS install.

---

## Install phases (overview)

1. **Phase 0 — Reset.** Erase All Content and Settings → fresh macOS, single local admin, no iCloud.
2. **Phase 1 — Base hardening.** FileVault, application firewall (stealth + block-all), SSH key-only, Wake-for-network.
3. **Phase 2 — Runtime stack.** Homebrew → Node 24 → Ollama → pull Gemma 4 (loopback-only).
4. **Phase 3 — OpenClaw.** `npm install -g openclaw@latest` → `openclaw onboard` → wire to local Ollama.
5. **Phase 4 — Operations.** Background services, log paths, update workflow.
6. **Phase 5 — Assistant identity.** Email, GitHub, social accounts (configured once the base is stable).
7. **Phase 6 — Remote access.** Tailscale for secure access from other devices.

Phases 0–4 produce a working local assistant. 5–6 are quality-of-life additions on top.

---

## Status

🚧 **Work in progress.** This repo is being built up alongside the actual install on a fresh Mac Mini. Each phase in [INSTALL.md](INSTALL.md) is updated as it's executed, so what you see is what's been verified on real hardware — not theory.

A long-form Medium article with trade-offs, conclusions, and the AWS-vs-local comparison will follow once the install is complete.

---

## Resources

**OpenClaw**

- Official site: <https://openclaw.ai/>
- GitHub: <https://github.com/openclaw/openclaw>
- Showcase (community deployments): <https://openclaw.ai/showcase>
- `SOUL.md` template & docs: <https://docs.openclaw.ai/reference/templates/SOUL>
- `soul.md` standard / spec: <https://soul.md/>

**Local LLM stack**

- Ollama: <https://ollama.com/>
- Gemma 4 on Ollama: <https://ollama.com/library/gemma4>

---

## License

TBD — will likely be MIT for the documentation in this repo. The OpenClaw project itself is governed by its own license at the upstream repository.
