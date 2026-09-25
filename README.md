I build the layers between what AI agents claim and what systems can verify.

<br>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/map-dark.svg">
  <img src="assets/map-light.svg" alt="Engineering map: QUESTION → OBSERVE → VERIFY → CONSTRAIN → AUTHORIZE, with requirement-zero, fornax-core, agent-assembly, glomeris, and eltanin mapped to each phase" width="760">
</picture>

<br>

---

## What exists today

### 🧪 Fornax — evidence-first agent integrity

**What should I believe about what this agent told me, given the evidence actually available?**

Fornax watches a coding agent session in real time, captures immutable evidence (tool calls, exit codes, transcripts), and checks the agent's own claims against that evidence — surfacing `VERIFIED` / `UNVERIFIED` / `CONTRADICTED` / `REVIEW` / `UNAVAILABLE`. Never a made-up trust score.

The five-state vocabulary matters: missing evidence is `UNAVAILABLE`, not a pass. A claim with no supporting evidence is `UNVERIFIED`, not believed. A claim that contradicts the tool output record is `CONTRADICTED`.

```
CLAIM: "All tests passed."
EVIDENCE: exit_code=1, stderr="test failed"
VERDICT: ✕ CONTRADICTED
```

```
STATUS      🐕 Dogfooding
RELEASE     v0.0.7  ·  2026-09-24
STACK       Rust  ·  local daemon  ·  SQLite/WAL  ·  Claude Code + Codex adapters
VALIDATED   Working daemon; contradiction detection; immutable evidence store; Quick Start reproduces the core claim on a local machine; 22 ADRs; ablation benchmark with pinned assertions
LIMITS      Only exit-code evidence is produced by shipped adapters today — the ablation benchmark measures 22% recall on ground-truth-positive cases; other evidence kinds (file diffs, process observations) are declared but not yet captured; v0.0.1 series; no external adoption
```

[source](https://github.com/horonomy/fornax-core) · [architecture invariants](https://github.com/horonomy/fornax-core/blob/main/docs/adr/0001-architecture-invariants.md) · [22 ADRs](https://github.com/horonomy/fornax-core/tree/main/docs/adr)

---

### ⚙️ Agent Assembly — governance-native AI agent runtime

**What is this agent allowed to do? What did it actually do? Is that audit trail verifiable?**

Agent Assembly provides runtime governance for AI agents: policy enforcement on every action, a hash-chained tamper-evident audit trail, and layered mechanisms (SDK shim, sidecar proxy, eBPF) deployable independently. An absent mechanism is reported as absent — nothing underneath silently picks up what it would have done.

A design note on honesty: where a mechanism intercepts but does not block before execution, the README says so. ADR 0033 logs a truthfulness bug — a CONNECT-level event being written as `allow` when the payload was not inspected — awaiting a code fix. That a specification document contains its own known defects is a deliberate engineering choice.

```
STATUS      🟠 Developer Preview
RELEASE     v0.0.1-rc.7  ·  2026-09-17
STACK       Rust  ·  CLI (aasm)  ·  runtime + proxy + eBPF  ·  gRPC + PolicyBundle
CHANNELS    crates.io · Homebrew tap · PyPI · npm · GHCR  (all at rc version)
VALIDATED   CLI + runtime on Linux and macOS; sidecar proxy (Linux+macOS); eBPF observation on Linux; hash-chained audit; developer-tool integration lifecycle
LIMITS      v0.0.1-rc — API and wire protocol not stable; eBPF terminates processes asynchronously, not pre-execution; no prebuilt macOS proxy binary on any channel; no macOS host-level enforcement
```

[source](https://github.com/ai-agent-assembly/agent-assembly) · [architecture + enforcement matrix](https://github.com/ai-agent-assembly/agent-assembly/blob/main/docs/src/architecture/README.md) · [limitations and known bypasses](https://github.com/ai-agent-assembly/agent-assembly/blob/main/docs/src/devtools/limitations.md)

---

### 🛡️ Eltanin — compute authorization and enforcement

**No protected compute without authorization.**

Eltanin is a local-first authorization and enforcement layer for protected compute (NVIDIA GPUs on Linux; Apple Silicon Metal on macOS for functional validation). A workload must go through the `eltanin run` authorization path — with a scoped, expiring lease — or the device stays protected. No cloud control plane in the hot path.

A note on what has and has not been proven: `SECURITY_MODEL.md` opens with "No claim below is validated yet. Every 'Claimed: denied' here is the *target* F-M1-007 is built to, pending HORO-841's hardware spike." Apple Silicon / Metal provides real functional evidence (E2), but device-level enforcement is the hardware spike on bare-metal Linux / NVIDIA (E3) — that is the mandatory security gate and it is not replaceable by macOS evidence. An Apple-only PASS is `BLOCKED ON E3`, never `READY`.

"Compatibility is not protection" — a platform being functionally supported never means device-level enforcement is claimed for it.

```
STATUS      🚧 MVP Development
RELEASE     v0.2.0-preview  ·  2026-09-20
STACK       Rust  ·  cgroup v2 device-BPF  ·  local IPC protocol  ·  CLI
VALIDATED   Authorization happy path; ALLOW/DENY flow; IPC protocol; Apple Silicon Metal functional (E2); privileged daemon; 8 ADRs
LIMITS      The NVIDIA backend crate (eltanin-nvidia) and the eBPF device guard (eltanin-device-guard) have not yet been merged into the workspace — the enforcement code is still in progress; NVIDIA/Linux device-level enforcement (E3 — the mandatory security gate) is not yet proven on hardware; single-host scope only
EXPLICIT NON-GOALS  AMD/Intel, Windows, SaaS control plane, enterprise RBAC, Kubernetes
```

[source](https://github.com/horonomy/eltanin) · [security model](https://github.com/horonomy/eltanin/blob/main/docs/product/SECURITY_MODEL.md) · [ADR-0007 Apple Silicon boundary](https://github.com/horonomy/eltanin/blob/main/docs/adr/0007-apple-silicon-metal-backend.md)

---

### 🧰 Glomeris — policy-constrained developer storage autopilot

**AI can recommend. Policy decides. Executor verifies. Filesystem reality wins.**

When a developer machine enters disk pressure, Glomeris discovers reclaimable storage, explains why each candidate is or is not safe to remove, and executes only policy-approved typed cleanup actions — re-measuring actual freed bytes until a target is reached or no safe action remains.

An optional BYOK LLM planner can rank and explain ambiguous candidates, but it can only select from a typed, closed set of known actions — it can never invent a raw command and can never upgrade a `PROTECTED` resource to safe. The policy layer is not a soft advisory; it is the execution gate.

```
STATUS      🐕 Dogfooding
RELEASE     v0.2.0  ·  2026-09-13
STACK       Rust  ·  Swift (macOS menu-bar app)  ·  BYOK LLM planner (optional)
VALIDATED   detect / scan / explain / clean / autopilot on macOS; menu-bar app; policy-constrained execution; founder dogfood reports (2026-09-13, 2026-09-21)
LIMITS      Experimental MVP; macOS-only; Homebrew cache cleanup is permanently refused at execution (brew cleanup has no path-scope argument — fail-closed design); Docker cache is detect-only, no registered cleanup action; the LLM planner is advisory-only and optional
```

[source](https://github.com/Chisanan232/glomeris) · [safety model](https://chisanan232.github.io/glomeris/safety_model.html) · [known limitations](https://chisanan232.github.io/glomeris/known_limitations.html) · [docs](https://chisanan232.github.io/glomeris/)

---

### 📐 Requirement Zero — engineering methodology as agent skill

**Every requirement must earn its right to exist.**

Before Requirement #1 comes Requirement #0: prove the requirement deserves to exist. Two Claude Code skills delivered as Markdown (no package, server, or runtime):

- **Requirement Zero** — challenges a requirement before planning or code. Five verdicts: DELETE, REDUCE, DEFER, BUILD, BUILD HARD.
- **Codebase Zero** — audits an existing artifact and asks whether it still deserves to. Six verdicts: DELETE, CONSOLIDATE, SIMPLIFY, DEFER CLEANUP, KEEP, INVEST.

The evaluation is worth examining for how it is reported as much as for what it found. The first evaluation run was discovered to be contaminated by an ambient `CLAUDE.md` file in the working directory — it was superseded with a clean re-run that produced less favorable numbers. The less favorable numbers are the ones published.

Aggregate across 36 calls (claude-sonnet-4-6, 6 cases × 2 arms × 3 runs): skill arm improved from 4/6 to 4/6 majority-correct cases — same headline number, with one case flipped each direction. The documented twelve-point limitations section includes: tiny N, single model, no downstream cost measured, baseline deliberately primed.

```
STATUS      🟠 Developer Preview
RELEASE     v0.2.0  ·  2026-08-16
TECH        Claude Code skills (Markdown) · eval harness (Python stdlib)
VALIDATED   36-call eval (Requirement Zero); 42-call eval (Codebase Zero); contamination-controlled; results and full harness published
LIMITS      N=3 per cell — no statistical power; single model; downstream implementation saving is explicitly unmeasured; Codebase Zero eval is newer with less validation history
```

[source](https://github.com/Chisanan232/requirement-zero) · [evaluation results — Requirement Zero](https://github.com/Chisanan232/requirement-zero/blob/main/eval/results/2026-08-15-claude-sonnet-4-6.md) · [evaluation results — Codebase Zero](https://github.com/Chisanan232/requirement-zero/blob/main/eval/codebase-zero/results/2026-08-15-claude-sonnet-4-6.md)

---

## How I tend to build

**Evidence before interpretation.**
Raw observations are recorded before any inference runs against them. In Fornax, adapter events are persisted (append-only) before claim extraction or verification. In Eltanin, authorization evidence is classified by class (E1 simulated / E2 functional / E3 device-level enforcement) — a higher evidence class cannot be substituted for a lower one.

**Limitations belong in the same document as the claims.**
Eltanin's `SECURITY_MODEL.md` opens with "No claim below is validated yet." Agent Assembly's canonical ADR logs a known truthfulness bug and names the pending fix. Requirement Zero's evaluation superseded its own favorable numbers with less favorable ones after discovering a confound. The limitation section is not a footnote.

**Security boundaries must be stated explicitly, not implied by architecture.**
"Compatibility is not protection" (Eltanin). An absent enforcement mechanism is reported as absent — nothing underneath silently picks up what it would have done (Agent Assembly). `UNAVAILABLE` is a real verdict state, not silence (Fornax).

**Local-first critical path.**
Evidence capture → verification → local verdict must work with all cloud access disabled. Cloud sync, if it exists, is async and best-effort after local validation approves it. This is not a preference; it is a design gate.

**Reviewable changes over history rewrites.**
Fornax's current implementation was replayed ticket-by-ticket as reviewed PRs after a one-time history normalization. The bootstrap is preserved at `archive-v0.0.1-bootstrap`. The commit history is the design history.

**Negative and inconclusive results are still results.**
Publishing an evaluation that shows no statistically meaningful effect, after correcting for a confound that made the earlier numbers look better, is harder than publishing the favorable version. It is also the only version that justifies future claims.

---

## What I'm currently working through

- Can a coding agent's claims be verified without modifying the agent's own behavior? The Fornax daemon is a test of this: the adapter is a hook, not a wrapper, and the verification happens after the fact.

- Which enforcement point in an agent system actually catches more: in-process SDK shim, sidecar proxy, or kernel-level eBPF? Each reaches further but costs more and has different failure modes. Agent Assembly's layered design is an experiment in making this distinction observable.

- How do you prove "no GPU without authorization" on real hardware, rather than configure a policy that assumes good-faith execution paths? Eltanin has a code answer; it does not yet have a hardware-evidence answer. That gap is the blocking item.

- When a coding agent skill improves verdict accuracy on one evaluation case while degrading it on another — and the net headline number is identical to the baseline — what does that actually tell you about skill efficacy? The Requirement Zero eval results contain exactly this scenario.

- At what point does "local-first, cloud-async" stop being a sound architectural principle and start being a way to defer decisions about trust boundaries that will need to be made eventually?

---

## Older systems — still how I build

### PyFake-API-Server  `·`  🛠️ Maintenance paused

> [!IMPORTANT]
> Active development of this project is currently paused.

A configurable mock HTTP server (Flask or FastAPI) that lets you define API responses in YAML/JSON — or import them from an OpenAPI spec — without writing application code. Useful for frontend development, contract testing, and CI pipelines. Published as `fake-api-server` on PyPI.

What this shows about engineering discipline from 2023–2025: a solo Python project with 12 active GitHub Actions workflows, four test layers (unit, integration, live-MySQL compatibility, E2E using the packaged GitHub Action against itself), SonarCloud, Codecov, and a versioned documentation site. Most of this infrastructure existed before the project reached v0.1.0.

```
RELEASE   v0.4.2  ·  2025-03-31   (maintenance paused since; active dev was 2023–2025)
PYPI      fake-api-server  ·  fake-api-server-surveillance
```

[source](https://github.com/Chisanan232/PyFake-API-Server) · [PyPI](https://pypi.org/project/fake-api-server/)

### multirunnable  `·`  🗃️ Legacy

> [!WARNING]
> This project is no longer actively maintained.

A Python library that unified multiprocessing, threading, gevent, and asyncio behind a single `RunningMode` enum and executor API — plus synchronization primitives, a retry decorator, and a parallel persistence layer. The `study/` directory still shows the pre-implementation research pattern: write the exploration scripts before committing to the design.

```
RELEASE   v0.17.0  ·  2022-05-27
PYPI      multirunnable
```

[source](https://github.com/Chisanan232/multirunnable) · [PyPI](https://pypi.org/project/multirunnable/)

---

## Elsewhere

**[@horonomy](https://github.com/horonomy)** — Fornax · Eltanin  
**[@ai-agent-assembly](https://github.com/ai-agent-assembly)** — Agent Assembly  
**LINE corp.** — Software Engineer (day job)
