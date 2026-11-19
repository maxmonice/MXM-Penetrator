# V1 Roadmap

This roadmap is a proposed delivery sequence, not a claim that work has been completed. It owns the **thin vertical slice** strategy. Requirement text lives in the owner documents ([DOCUMENT_OWNERSHIP.md](DOCUMENT_OWNERSHIP.md)); phases reference requirements by ID.

## Development strategy

**Prioritize an early, end-to-end working workflow before expanding the feature set.** This is an incremental delivery strategy. It does not reduce the long-term product vision or the differentiators in [PRODUCT_SCOPE.md](PRODUCT_SCOPE.md).

Phases may overlap after Phase 1, but **exit criteria gate release claims**: nothing is represented as enforced until it has an implementation and a passing negative test (`SEC-055`).

## Phase 0 — Contracts, threat model, and governance

- Establish package structure, CLI entry point, configuration, and development tooling (OD-32).
- Define versioned schemas for authorization, scope, module manifests, request plans and responses, action proposals, policy decisions, approvals, cleanup-ledger entries, evidence, findings, and audit events (including previous-hash and own-hash fields for `SEC-044`).
- Define error and reason-code registries.
- Review and baseline [THREAT_MODEL.md](THREAT_MODEL.md); keep [SECURITY_REQUIREMENTS.md](SECURITY_REQUIREMENTS.md) current.
- Decide the items the slice depends on: the slice's reconnaissance module (OD-30), the safe-active numeric thresholds (OD-03), environment designation (OD-12), the tooling and rule set for the module-interface automated checks (OD-15; approach settled by decision S), the control-channel mechanism for the slice (OD-11), response-size handling (OD-22), redaction rules for the slice (OD-24), sanitizer rules (OD-27), and the first ground-truth application (OD-29).
- Governance files are in place: [../LICENSE](../LICENSE), [../SECURITY.md](../SECURITY.md), [../RESPONSIBLE_USE.md](../RESPONSIBLE_USE.md).

**Exit criteria:** Schemas are documented, validated, versioned, and covered by tests. The threat model is reviewed. The listed prerequisite decisions are closed.

## Phase 1 — Thin vertical slice

The first vertical slice **must include**:

1. **Authorized target input** (an authorization record and explicit scope, `SEC-001`, `SEC-002`; unknown environment treated as Production, `SEC-005`).
2. **Scope and policy validation** (`SEC-016`; the slice supports only passive and safe-active classes, and every other class is denied under `SEC-019`).
3. **One internal reconnaissance module**, written to the Sans-IO interface with a manifest (`SEC-032`, `SEC-034`).
4. **The centralized executor** (`SEC-033`).
5. **Evidence collection and sanitization** (`SEC-039`, default redaction under `SEC-045`).
6. **Basic JSON and Markdown reporting.**

The slice also emits audit events for policy decisions (`SEC-043`) and applies the central safe-active thresholds (`SEC-015`). It does not store restricted raw evidence until encryption (`SEC-046`) exists.

In parallel, the **ground-truth environment** is stood up (`SEC-056`): at least one intentionally vulnerable application runs locally and isolated, and the slice's reconnaissance module is run against it so measurement exists from the start.

**Exit criteria:** One authorized assessment runs end to end and produces a JSON and Markdown report. Negative tests show that out-of-scope, unauthorized, unclassified, and non-permitted-class actions cannot execute. The module cannot reach the network except through the executor, and the executor rejects requests outside the manifest. The ground-truth harness exists and records baseline measurements.

## Phase 2 — Safety foundation completion

- Permission profiles, effective-level computation, authorization ceilings, and environment designation (`SEC-005`, `SEC-007`, `SEC-008`).
- Full action classification and the intrusive thresholds (`SEC-013` to `SEC-015`).
- Full policy engine: capabilities, epochs, budgets, pause and kill (`SEC-016` to `SEC-023`).
- Trusted control channel and hash-bound approvals (`SEC-018`, `SEC-024`, `SEC-025`).
- Destructive hard-deny (`SEC-020`) and authorization lifecycle (`SEC-006`).
- Cleanup ledger, automatic cleanup, `mxm cleanup`, and cleanup under kill and revocation (`SEC-026` to `SEC-031`).
- Hash-chained audit log, evidence encryption and key separation, evidence tiers, and replay retention (`SEC-043` to `SEC-049`).

**Exit criteria:** Negative tests prove that unauthorized, out-of-scope, destructive, unclassified, expired, revoked, and unapproved actions cannot execute; test-case lifecycle is auditable; cleanup behaves as specified across epochs, crashes, kill, and revocation.

## Phase 3 — Independent engine expansion

- Additional passive and safe-active modules (initial module list: OD-31).
- Selected intrusive tests with bounded parameters.
- Sans-IO verification routines that run through the executor (`SEC-041`, `SEC-042`).
- Ground-truth measurement per module and category (`SEC-056`).

**Exit criteria:** Each module has a manifest, tests, scope enforcement, budget enforcement, evidence output, and ground-truth measurements. Manifest-conformance tests pass.

## Phase 4 — AI orchestration

- AI gateway and local model support; explicit cloud opt-in; rules-based routing (`SEC-052`).
- Typed proposal generation and validation (`SEC-050`, `SEC-051`).
- Centrally enforced planning limits (`SEC-053`) and bounded Autonomous planning.
- Local model capability checks gating Autonomous (`SEC-054`).
- Advisory natural-language level recommendations (`SEC-012`).

**Exit criteria:** The AI cannot bypass the policy engine, alter scope, approve actions, or exceed the Active/Autonomous invariants (`SEC-009` to `SEC-011`). A model that fails the capability checks cannot run Autonomous. Ground-truth tests validate planner behavior.

## Phase 5 — External integrations

- Nmap adapter, beginning with TCP connect scans.
- OWASP ZAP adapter, beginning with passive scanning.
- Per-platform validation (Windows and Linux separately) of enforcement and isolation.
- Normalized adapter evidence and results.

**Exit criteria:** Adapter tests demonstrate parameter validation, scope enforcement, budget enforcement, process restrictions, kill-switch termination, and fail-closed behavior on each supported platform. An adapter that cannot meet these on a platform is disabled there and documented.

## Phase 6 — Release hardening

- End-to-end tests on supported platforms.
- Prompt-injection and untrusted-output scenarios.
- DNS, redirect, IPv4/IPv6, and network-bypass cases.
- Cleanup recovery and interrupted assessments.
- Documentation review: installation, upgrade, troubleshooting.
- Publish known limitations (including the residual risks in [THREAT_MODEL.md](THREAT_MODEL.md)) and a supported-version and per-platform matrix.
- Establish the vulnerability reporting channel (OD-01), copyright statement (OD-02), and supply-chain and update strategy (OD-35).

**Exit criteria:** No unresolved critical safety defects; release claims match demonstrated controls.

## Deferred beyond v1

Complex authenticated workflows, multi-user authorization testing, cross-account privilege escalation, complex session-based testing, a dashboard, a REST API, distributed execution, and an MXM MCP server. They remain part of the long-term vision ([PRODUCT_SCOPE.md](PRODUCT_SCOPE.md)).

## Release gate

Do not label v1 safe for general use until the core policy, authorization, scope, approval, network, cleanup, audit, sanitization, and evidence-privacy controls have passed their acceptance tests, and until the ground-truth results and known limitations are published.
