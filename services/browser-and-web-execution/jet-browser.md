# Jet Browser

> **"Jet Browser — the browser runtime agents can carry"**

| | |
|---|---|
| **Website** | https://masaka-ai.vercel.app/jet-browser/ |
| **Docs** | https://github.com/masakaai/jet-browser/tree/v0.8.0#readme |
| **GitHub** | https://github.com/masakaai/jet-browser |
| **Stars** | [![GitHub Stars](https://img.shields.io/github/stars/masakaai/jet-browser?style=social)](https://github.com/masakaai/jet-browser) |
| **Classification** | `agent-native` |
| **Category** | [Browser & Web Execution Services](README.md) |
| **License** | Apache-2.0 |
| **Latest-month signal** | [v0.8.0](https://github.com/masakaai/jet-browser/releases/tag/v0.8.0) released 2026-10-10 with a versioned OCI runtime, an Agent Skill, and the bounded Jet Browser Verifier MCP package |
| **Verified at** | 2026-10-11 |

---

## Official Website

https://masaka-ai.vercel.app/jet-browser/

---

## Official Repo

https://github.com/masakaai/jet-browser

---

## How to Use (Agent Onboarding)

**Interaction pattern:** `Coding-time Skill + local container runtime`

Install the published repository skill, then let the coding agent check the runtime requirements and run the repository's acceptance flow:

```bash
npx skills add masakaai/jet-browser --skill jet-browser --yes
git clone https://github.com/masakaai/jet-browser.git
cd jet-browser
npm ci
npm run standalone
```

Docker and Node.js 24+ are required. The acceptance flow starts a real WPE WebKit browser against a loopback fixture, exercises native input, verifies matching DOM state, captures a PNG, and cleans up. Jet Browser does not attach to a personal Chrome profile.

---

## Agent Skills

**Status:** ✅ Available in the official repository.

```bash
npx skills add masakaai/jet-browser --skill jet-browser --yes
```

| Skill | What It Teaches the Agent |
|---|---|
| [`jet-browser`](https://github.com/masakaai/jet-browser/tree/v0.8.0/.agents/skills/jet-browser) | Check Docker and Node.js requirements, run the standalone acceptance, bind the versioned tool declarations, preserve one-session-per-container isolation, and keep personal Chrome/CDP work outside Jet Browser. |

Codex and Claude Code users can also install the project's native plugin from the `masakaai/jet-browser` marketplace repository.

---

## MCP

**Status:** ✅ Available — a bounded runtime verifier, not a general-purpose browser MCP server.

| Detail | Value |
|---|---|
| **MCP Repo** | [`integrations/mcp`](https://github.com/masakaai/jet-browser/tree/v0.8.0/integrations/mcp) |
| **Registry identity** | [`io.github.masakaai/jet-browser`](https://registry.modelcontextprotocol.io/v0.1/servers?search=io.github.masakaai%2Fjet-browser), version 0.8.0 |
| **Transport / package** | stdio via `ghcr.io/masakaai/jet-browser-mcp:0.8.0` |
| **Tool boundary** | Capability inspection and one network-disabled runtime verification; no URL, profile, credential, or arbitrary-command input |
| **Compatible clients** | Trusted local MCP hosts that can launch a Docker-backed stdio server; VS Code and Cursor install links are published in the official README |

The verifier mounts the Docker socket, which is host-level trust. Both its outer server and the browser acceptance run with networking disabled; broader browsing uses the standalone JSONL runtime under the embedding harness's policy.

---

## What It Does

Jet Browser packages WPE WebKit, a headless Wayland compositor, and a Rust WebDriver bridge as a self-contained browser runtime. One container owns one isolated browser session. A harness sends ordered JSONL commands over stdin and receives machine-readable results without adopting a hosted control plane, account system, model, or agent loop.

Native pointer, keyboard, text, touch, wheel, tabs, DOM inspection, JavaScript results, downloads, and screenshots share the same runtime. This lets an agent system bind an action to semantic and visual evidence from one session rather than reconciling separate replay paths.

---

## Why It Is Agent-Native

| Criterion | Evidence |
|---|---|
| **Agent-first positioning** | The [official product page](https://masaka-ai.vercel.app/jet-browser/) says: “Jet Browser is MASAKA's portable, self-contained WPE WebKit runtime for agents: a real browser, native input, screenshots, and an ordered JSONL protocol.” |
| **Agent-specific primitive** | [One-session-per-container ownership](https://github.com/masakaai/jet-browser/blob/v0.8.0/docs/architecture.md#standalone-core), ordered command/result lines, [versioned agent tool declarations](https://github.com/masakaai/jet-browser/blob/v0.8.0/docs/agent-tools.md), and same-runtime native-input/DOM/PNG evidence form a browser boundary designed for an agent harness. |
| **Autonomy-compatible control plane** | The [documented browser lifecycle](https://github.com/masakaai/jet-browser/blob/v0.8.0/docs/architecture.md#browser-lifecycle) lets a supervisor create, navigate, interact, inspect, capture evidence, and close a session without an account, API key, model, hosted control plane, or per-action human confirmation. |
| **M2M integration surface** | [Ordered JSONL over stdin/stdout](https://github.com/masakaai/jet-browser/blob/v0.8.0/docs/architecture.md#standalone-core) is the core interface; versioned JSON Schema tool declarations, an OCI image, a GitHub Action, Agent Skill/plugin manifests, and a bounded stdio MCP verifier are also published. |
| **Identity / delegation** | The [container and explicit profile path](https://github.com/masakaai/jet-browser/blob/v0.8.0/docs/architecture.md#state-and-isolation) define the session boundary. Jet deliberately does not mint credentials or inherit a daily browser profile; the embedding application owns principals, delegated authorization, encrypted persistence, retention, egress policy, and durable audit storage. |

---

## Primary Primitives

| Primitive | Description |
|---|---|
| **Isolated browser session** | One WPE WebKit session, compositor, process tree, runtime directory, and profile path per container |
| **Ordered JSONL operations** | One command and one machine-readable response per line, processed in order |
| **Native input** | Pointer, keyboard, text, touch, wheel, drag, navigation, and tab operations delivered through the real runtime |
| **Semantic and visual evidence** | DOM snapshots, document state, JavaScript results, and fresh screenshots from the same session |
| **Versioned tool declarations** | Stable tool identities and schemas that an embedding harness can bind to its own model-facing names |
| **Explicit browser state** | Cookie and web-storage import/export helpers plus explicit profile paths, with persistence policy left to the host |
| **Lifecycle cleanup** | Explicit close plus container-level teardown for the compositor, driver, and browser process tree |

---

## Autonomy Model

1. A supervisor starts a resource-bounded Jet Browser container for one task or trust domain.
2. The harness sends `create`, then ordered navigation and native-input operations over JSONL.
3. The agent reads DOM/document state and fresh screenshots from that same browser session to verify outcomes.
4. The harness applies its own URL, credential, approval, timeout, and egress policy around privileged operations.
5. The supervisor sends `close` in guaranteed cleanup; terminating the container also terminates the owned browser process tree.

No Jet-hosted account or human dashboard is required for this loop.

---

## Identity and Delegation Model

- The runtime unit is one explicit container and one browser session; separate users, agents, or trust domains should receive separate containers and writable profile paths.
- Jet never silently reuses a person's signed-in daily Chrome profile and does not expose CDP or Chrome-extension compatibility.
- Imported cookies and browser storage are delegated capabilities. The embedding application must authenticate the caller, authorize the task, encrypt persisted state, enforce retention, and control which credentials enter the container.
- Command/result ordering and session-bound evidence support attribution, but Jet does not claim a hosted agent identity, credential broker, role system, or tamper-evident audit ledger.
- The optional distributed worker supports session-bound streams and monotonically increasing control epochs so stale controllers can be rejected after ownership changes.

---

## Protocol Surface

| Interface | Detail |
|---|---|
| **stdio JSONL** | Core, framework-neutral create/navigate/input/inspect/screenshot/state/download/close operations, one ordered command and response per line |
| **Agent tool schemas** | Versioned declarations from [`sdk/tools.mjs`](https://github.com/masakaai/jet-browser/blob/v0.8.0/sdk/tools.mjs), with local validation and collision checks |
| **OCI runtime** | Public `linux/amd64` image `ghcr.io/masakaai/jet-browser:0.8.0`; production deployments should pin its digest |
| **MCP verifier** | Two bounded stdio tools for capability inspection and network-disabled runtime verification, not arbitrary web browsing |
| **GitHub Action** | Versioned runtime smoke test for CI without repository or account secrets |
| **Optional distributed layer** | Session-bound control, preview, state, usage/action records, and takeover primitives around the standalone core |

---

## Human-in-the-Loop Support

The standalone core has no required approval UI and normally runs unattended inside policies enforced by the calling harness. Optional distributed components can provide preview and control handoff, but the embedding application remains responsible for user consent, high-risk action approvals, authentication, and authorization. Jet Browser is intentionally separate from workflows that must operate an already signed-in personal Chrome profile.

---

## Why Generic Alternatives Do Not Qualify

| Alternative | Why It Fails |
|---|---|
| **Personal desktop Chrome / raw CDP attachment** | Inherits a human account and long-lived profile instead of giving each task an explicit disposable container, process tree, profile path, and ordered evidence boundary. |
| **Generic browser container** | Provides a browser process but not Jet's versioned agent tool contract or its ordered native-input, semantic-state, and screenshot receipt from one runtime. |
| **High-level browser-agent framework** | Bundles the model or agent loop with browsing; Jet keeps the execution layer replaceable so the same runtime can serve any harness or deterministic test runner. |

---

## Use Cases

- **Disposable browser workers** — isolate each agent task or trust domain in its own WPE WebKit container
- **Repository and CI verification** — prove startup, native input, DOM state, screenshot capture, and cleanup with a reproducible acceptance flow
- **Harness-neutral browser tools** — bind stable schemas to a custom agent loop without adopting a vendor control plane
- **Cross-engine testing** — add a non-Chromium browser runtime where Playwright/CDP coverage is not the required contract
- **Evidence-oriented web actions** — keep native action, semantic state, and visual confirmation attributable to one ordered session
