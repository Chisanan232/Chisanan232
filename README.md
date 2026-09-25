I build the layers between what AI agents claim and what systems can verify.

<br>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/map-dark.svg">
  <img src="assets/map-light.svg" alt="Four engineering questions mapped to projects: DECIDE → requirement-zero, VERIFY → fornax-core, GOVERN → agent-assembly + glomeris, AUTHORIZE → eltanin" width="760">
</picture>

<br>

---

## What exists today

### 🧪 Fornax — evidence-first agent integrity

![Dogfooding](https://img.shields.io/badge/status-Dogfooding-3b7a57?style=flat-square) ![release](https://img.shields.io/github/v/release/horonomy/fornax-core?include_prereleases&style=flat-square&label=release) ![Rust](https://img.shields.io/badge/Rust-orange?style=flat-square&logo=rust&logoColor=white)

Coding agents can claim success even when their own tool output says otherwise. Fornax watches a session in real time, captures immutable evidence, and checks each claim against it — returning `VERIFIED`, `UNVERIFIED`, `CONTRADICTED`, or `UNAVAILABLE`. Missing evidence is `UNAVAILABLE`, not a pass.

**Current boundary:** Only exit-code evidence is captured today; the ablation benchmark confirms 22% recall on ground-truth-positive cases.

[source](https://github.com/horonomy/fornax-core) · [architecture invariants](https://github.com/horonomy/fornax-core/blob/main/docs/adr/0001-architecture-invariants.md)

---

### ⚙️ Agent Assembly — governance-native AI agent runtime

![Developer Preview](https://img.shields.io/badge/status-Developer%20Preview-4a6fa5?style=flat-square) ![release](https://img.shields.io/github/v/release/ai-agent-assembly/agent-assembly?include_prereleases&style=flat-square&label=release) ![Rust](https://img.shields.io/badge/Rust-orange?style=flat-square&logo=rust&logoColor=white)

Agents need execution freedom without unconstrained authority. Agent Assembly enforces policy on every agent action, records a tamper-evident audit trail, and exposes layered enforcement mechanisms — each one explicitly stating its capability boundary. An absent mechanism is reported absent, not silently replaced.

**Current boundary:** RC series — API not stable; eBPF terminates processes after the fact, not before.

[source](https://github.com/ai-agent-assembly/agent-assembly) · [limitations and known bypasses](https://github.com/ai-agent-assembly/agent-assembly/blob/main/docs/src/devtools/limitations.md)

---

### 🛡️ Eltanin — compute authorization and enforcement

![MVP Development](https://img.shields.io/badge/status-MVP%20Development-c05820?style=flat-square) ![release](https://img.shields.io/github/v/release/horonomy/eltanin?include_prereleases&style=flat-square&label=release) ![Rust](https://img.shields.io/badge/Rust-orange?style=flat-square&logo=rust&logoColor=white)

Protected compute should not depend on users voluntarily following an approved execution path. Eltanin is a local-first authorization layer for GPU compute — a workload must hold a scoped, expiring lease or the device stays closed. No cloud control plane in the enforcement hot path.

**Current boundary:** The Linux/NVIDIA enforcement crates have not been merged; device-level enforcement is not yet proven on hardware.

[source](https://github.com/horonomy/eltanin) · [security model](https://github.com/horonomy/eltanin/blob/main/docs/product/SECURITY_MODEL.md)

---

### 🧰 Glomeris — policy-constrained developer storage autopilot

![Dogfooding](https://img.shields.io/badge/status-Dogfooding-3b7a57?style=flat-square) ![release](https://img.shields.io/github/v/release/Chisanan232/glomeris?style=flat-square&label=release) ![Rust + Swift](https://img.shields.io/badge/Rust%20%2B%20Swift-orange?style=flat-square&logo=rust&logoColor=white)

Disk cleanup is easy to automate badly. Glomeris discovers reclaimable storage on macOS, explains why each candidate is or isn't safe to remove, and executes only policy-approved typed actions. An optional LLM planner may rank candidates; it cannot invent or authorize a deletion.

**Current boundary:** macOS-only experimental MVP; Homebrew and Docker cache cleanup have architectural constraints that limit what can be safely executed.

[source](https://github.com/Chisanan232/glomeris) · [safety model](https://chisanan232.github.io/glomeris/safety_model.html) · [known limitations](https://chisanan232.github.io/glomeris/known_limitations.html)

---

### 📐 Requirement Zero — engineering methodology as agent skill

![Developer Preview](https://img.shields.io/badge/status-Developer%20Preview-4a6fa5?style=flat-square) ![release](https://img.shields.io/github/v/release/Chisanan232/requirement-zero?style=flat-square&label=release) ![Claude Code skill](https://img.shields.io/badge/Claude%20Code%20skill-black?style=flat-square)

Coding agents can efficiently build things that should never have existed. Requirement Zero challenges a requirement before planning or writing code; Codebase Zero audits existing artifacts and asks the same question. Both are Claude Code skills — no package, server, or runtime.

**Current boundary:** Evaluation on six cases shows modest accuracy improvement; downstream cost savings are unmeasured.

[source](https://github.com/Chisanan232/requirement-zero) · [evaluation results](https://github.com/Chisanan232/requirement-zero/blob/main/eval/results/2026-08-15-claude-sonnet-4-6.md)

---

## How I tend to build

**Evidence over claims.** Observations are recorded before any interpretation runs. Missing capability is `UNAVAILABLE`, not a pass.

**Failure paths are part of the design.** What happens when enforcement is absent, evidence is missing, or authorization is refused is specified — not left implicit.

**Security boundaries stay explicit.** Compatibility is not protection. An absent mechanism is reported as absent.

**Negative results stay visible.** An evaluation superseded because its confound made the numbers too favorable is published with the corrected, less favorable results.

---

## What I'm currently working through

- Can a coding agent's claims be verified without modifying the agent's behavior? The Fornax hook model is a test of this.
- Where in the stack does enforcement actually stop more: in-process SDK, sidecar proxy, or kernel-level eBPF — and what does each genuinely catch vs. observe?
- How do you prove "no GPU without authorization" on real hardware, not just in code?

---

## Older systems

**[PyFake-API-Server](https://github.com/Chisanan232/PyFake-API-Server)** · 🛠️ Maintenance paused · `v0.4.2` · [PyPI](https://pypi.org/project/fake-api-server/)  
Configurable mock HTTP server; define API responses in YAML or import from an OpenAPI spec.

**[multirunnable](https://github.com/Chisanan232/multirunnable)** · 🗃️ Legacy · `v0.17.0` · [PyPI](https://pypi.org/project/multirunnable/)  
Unified Python API across multiprocessing, threading, gevent, and asyncio.

---

## Elsewhere

**[@horonomy](https://github.com/horonomy)** — Fornax · Eltanin  
**[@ai-agent-assembly](https://github.com/ai-agent-assembly)** — Agent Assembly  
Software Engineer · LINE corp.
