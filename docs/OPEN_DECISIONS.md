# Open Decisions and Design Questions

This document owns the register of **unresolved decisions** (`OD-nn`). Other documents point here instead of restating them. Settled decisions are recorded below and in [../CHANGELOG.md](../CHANGELOG.md); security rules are owned by the documents in [DOCUMENT_OWNERSHIP.md](DOCUMENT_OWNERSHIP.md).

## Settled v1 product decisions (baseline)

The following nine decisions were explicitly agreed and are preserved:

1. **Level 3 — Autonomous is included in v1.** It has bounded planning and replanning and exactly the same execution permissions as Level 2.
2. **Level 2 is named Active.** The three profiles are Guided (green), Active (yellow), and Autonomous (red).
3. **Production targets default to a Guided authorization ceiling.** Higher selected levels cannot bypass the ceiling.
4. **Newly discovered assets are scope candidates.** They require explicit authorization before active testing.
5. **State-changing tests use test-case approval bundles.** Approval covers the bounded test lifecycle and disclosed cleanup.
6. **Credential/login testing is state-changing.** High-volume attempts that risk real-user lockout or major disruption are destructive and denied.
7. **Initial external integrations are Nmap and OWASP ZAP.** Both are optional, vetted adapters; MXM's own engine remains primary. (Refined by revision decision I: connect scans and passive scanning first.)
8. **Network enforcement is cross-platform with platform-specific implementations.** Windows enforcement must be investigated and verified; fail closed when required restrictions cannot be guaranteed.
9. **Natural-language permission selection is advisory.** AI recommends; the user explicitly confirms.

Decisions A–R, approved for the v1.1 documentation revision, are listed with their owner documents in [../CHANGELOG.md](../CHANGELOG.md). Earlier foundational decisions (Python, Apache License 2.0, CLI-first, local-first AI with cloud opt-in, rules-based router, typed proposals, independent policy engine, REA as an isolated optional adapter, staged asyncio pipeline, SQLite plus a separate evidence store, JSON canonical output, deferred dashboard, REST API, distributed execution, and MXM MCP server) are preserved in [ARCHITECTURE.md](ARCHITECTURE.md) and [PRODUCT_SCOPE.md](PRODUCT_SCOPE.md).

The full list of security requirements is the index in [SECURITY_REQUIREMENTS.md](SECURITY_REQUIREMENTS.md).

## Open decisions

"Needed before" is the earliest phase of [V1_ROADMAP.md](V1_ROADMAP.md) that depends on the decision. No decision here has a proposed value, and none of the documents assumes one.

### Governance

| ID | Decision | Needed before | Related |
|---|---|---|---|
| OD-01 | Vulnerability reporting channel and response commitments | Phase 6 (before first public release) | [../SECURITY.md](../SECURITY.md) |
| OD-02 | Copyright holder and any NOTICE statement for the Apache License 2.0 | Phase 6 | [../LICENSE](../LICENSE) |

### Policy, limits, and controls

| ID | Decision | Needed before | Related |
|---|---|---|---|
| OD-03 | Numeric thresholds for safe-active and intrusive testing: request rate, concurrency, per-test request budget, assessment-wide request budget; their granularity; how requests above the intrusive ceiling are treated (denied, or classified destructive). Safe-active values are a prerequisite for the vertical slice. | Phase 1 (safe-active), Phase 2 (intrusive) | `SEC-015` |
| OD-04 | Numeric AI planning limits (replans, follow-up actions, concurrent tasks, duration, tokens), how they differ between Active and Autonomous, and whether Guided uses the same parameter set | Phase 4 | `SEC-053` |
| OD-05 | Complete list of events that increment the policy epoch beyond permission-level changes (scope, authorization, environment designation, budgets) | Phase 2 | `SEC-021`, `SEC-004` |
| OD-06 | Kill and pause lifecycle: how the kill state is cleared, whether `mxm cleanup` may run in a new process after a kill, and pause semantics for in-flight actions | Phase 2 | `SEC-022`, `SEC-031` |
| OD-07 | Cleanup edge cases: (a) a target removed from scope while authorization remains valid (currently parked); (b) whether automatic cleanup may resume inside a running process after authorization is restored | Phase 2 | `SEC-027`, `SEC-028`, `SEC-031` |
| OD-08 | Cleanup execution limits: whether cleanup draws on a reserved execution budget, and retry bounds | Phase 2 | `SEC-023`, `SEC-030` |
| OD-09 | How MXM proves, before cleanup, that a recorded resource identifier (possibly learned from an untrusted response) refers to a resource the test created | Phase 2 | `SEC-027` |
| OD-10 | Definition of the explicit unattended-execution mode | Phase 2 (or post-v1) | `SEC-025` |
| OD-11 | Control-channel mechanism (local TTY, authenticated local IPC) and protection from other processes of the same user and from adapters | Phase 1 (basic), Phase 2 | `SEC-018` |
| OD-12 | How the environment designation is established, stored, changed, and protected | Phase 1 | `SEC-005` |
| OD-13 | Deterministic definition of "high-volume" and lockout-risk credential attempts, and whether v1 ships any credential or login testing module | Phase 3 | `SEC-014`, `SEC-020` |
| OD-14 | Time limits: approval expiry and execution-capability lifetime | Phase 2 | `SEC-025`, `SEC-017` |

### Modules, enforcement, and integrations

| ID | Decision | Needed before | Related |
|---|---|---|---|
| OD-15 | *Narrowed by decision S:* detection is review plus automated checks, and separate-process containment is deferred. Still open: the checking tooling, the banned-import and banned-call rule set, and the policy for module file-system access | Phase 0 / Phase 1 | `SEC-032` |
| OD-16 | Governance of third-party passive data sources: allowed sources, rate limits, and what target identifiers may be disclosed to them | Phase 1 (if the slice module uses any) | `SEC-034`, [ACTION_CLASSIFICATION.md](ACTION_CLASSIFICATION.md) |
| OD-17 | Windows enforcement mechanism and the guarantees it can support | Design before Phase 2; validation by Phase 5 | `SEC-035`, `SEC-036` |
| OD-18 | Linux and macOS enforcement mechanisms and support matrix | Phase 5 | `SEC-035`, `SEC-036` |
| OD-19 | Whether adapters run in containers, restricted processes, or both; kill-switch termination mechanism | Phase 5 | `SEC-036` |
| OD-20 | Nmap: supported versions, scan profiles, and per-platform guarantees | Phase 5 | `SEC-038` |
| OD-21 | ZAP: supported versions; how passive scanning is fed; classification of any later active capability | Phase 5 | `SEC-038` |
| OD-22 | Response-size, decompression, and output limits applied by the executor | Phase 1 | `SEC-033` |

### Evidence, AI, and output

| ID | Decision | Needed before | Related |
|---|---|---|---|
| OD-23 | Audit-chain hash function, canonical serialization, verification procedure, and whether to anchor the latest hash externally | Phase 1 / Phase 2 | `SEC-044` |
| OD-24 | Redaction rule set, definition of personal information, and criteria for retaining restricted raw evidence | Phase 1 | `SEC-045`, `SEC-047` |
| OD-25 | Secret storage, evidence-encryption scheme, and key management | Phase 2 | `SEC-046`, `SEC-049` |
| OD-26 | Capability-check suite design, pass criteria, re-validation triggers, and whether Active needs any check | Phase 4 | `SEC-054` |
| OD-27 | Sanitizer rule set per output context, library versus custom implementation, and whether report templates may contain script | Phase 1 | `SEC-039` |
| OD-28 | Retention and deletion policy for evidence, reports, and audit data | Phase 2 | `SEC-047` |

### Testing and implementation

| ID | Decision | Needed before | Related |
|---|---|---|---|
| OD-29 | Initial ground-truth application (OWASP Juice Shop, DVWA, or OWASP WebGoat), ground-truth labeling, and acceptance thresholds | Phase 1 (application), later (thresholds) | `SEC-056` |
| OD-30 | *Settled by decision T:* the slice module is HTTP fingerprinting and security-header analysis (safe-active). Still open: its exact manifest, the set of URLs it may fetch, and its parameters, which depend on `OD-03` | Phase 0 | [V1_ROADMAP.md](V1_ROADMAP.md) |
| OD-31 | Initial module list for later phases | Phase 3 | [V1_ROADMAP.md](V1_ROADMAP.md) |
| OD-32 | Python package structure, CLI framework, schema format and versioning strategy | Phase 0 | [ARCHITECTURE.md](ARCHITECTURE.md) |
| OD-33 | Local model runtime and the providers supported at launch | Phase 4 | `SEC-052` |
| OD-34 | SQLite schema and evidence-store layout | Phase 0 / Phase 1 | [ARCHITECTURE.md](ARCHITECTURE.md) |
| OD-35 | Update, signing, and supply-chain verification strategy | Phase 6 | [THREAT_MODEL.md](THREAT_MODEL.md) AC-29 |
| OD-36 | How blocked verification is presented and how the user requests a retry | Phase 3 | `SEC-042` |

## Recording decisions

Each technical decision is recorded with its rationale, trade-offs, threat implications, and tests before implementation is considered complete. When an `OD-nn` is closed, update the owner document first, then this register, the index, and the changelog.
