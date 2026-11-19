# MXM Penetrator

**MXM Penetrator** is a CLI-first security assessment platform for individual security researchers and developers. Its goal is to combine an independent security testing engine with AI-assisted reasoning and controlled integrations into one evidence-driven assessment workflow.

> **Project status:** v1.1 documentation baseline. This repository describes intended behavior and architecture; it does not claim that the described controls have been implemented or independently verified. No application code exists yet.

> **Authorized use only.** MXM is for systems you own or are explicitly authorized to test. See the [Responsible Use Statement](RESPONSIBLE_USE.md).

## v1 objectives

- Provide a usable command-line assessment workflow, delivered first as an early end-to-end vertical slice ([roadmap](docs/V1_ROADMAP.md)).
- Perform authorized reconnaissance and vulnerability testing through MXM-controlled, Sans-IO modules behind a centralized executor.
- Focus v1 on unauthenticated exposure and misconfiguration detection; complex authenticated and multi-user testing is deferred, not abandoned ([scope](docs/PRODUCT_SCOPE.md)).
- Use AI for analysis, planning, and explanation without giving AI direct execution authority.
- Enforce scope, permissions, budgets, and approvals deterministically.
- Produce reproducible, evidence-backed findings, measured against ground-truth applications.
- Support local-first model use with explicit cloud-model opt-in.
- Integrate a small, vetted set of external tools through controlled adapters (Nmap TCP connect scanning and OWASP ZAP passive scanning first).

## Documentation

**Start here:** [Document ownership](docs/DOCUMENT_OWNERSHIP.md) explains which document is authoritative for which rule. [Security requirements index](docs/SECURITY_REQUIREMENTS.md) lists every `SEC-nnn` requirement.

| Document | Purpose |
|---|---|
| [Product scope](docs/PRODUCT_SCOPE.md) | Audience, goals, non-goals, deferred features |
| [Architecture](docs/ARCHITECTURE.md) | Components and assessment lifecycle |
| [Threat model](docs/THREAT_MODEL.md) | Assets, trust boundaries, abuse cases, residual risks |
| [Document ownership](docs/DOCUMENT_OWNERSHIP.md) | One authoritative owner per rule |
| [Security requirements](docs/SECURITY_REQUIREMENTS.md) | Index of stable requirement IDs |
| [Permission model](docs/PERMISSION_MODEL.md) | Guided, Active, and Autonomous |
| [Action classification](docs/ACTION_CLASSIFICATION.md) | Action classes and central thresholds |
| [Policy engine](docs/POLICY_ENGINE.md) | Evaluation order, epochs, kill switch, budgets |
| [Scope and authorization](docs/SCOPE_AND_AUTHORIZATION.md) | Authorization records, scope, environments, ceilings |
| [Approvals and cleanup](docs/APPROVALS_AND_CLEANUP.md) | Test-case approvals, cleanup ledger, `mxm cleanup` |
| [Module architecture](docs/MODULE_ARCHITECTURE.md) | Sans-IO modules, executor, manifests |
| [Output sanitization](docs/OUTPUT_SANITIZATION.md) | Terminal and report injection defenses |
| [Finding verification](docs/FINDING_VERIFICATION.md) | Verification through the policy path |
| [AI orchestration](docs/AI_ORCHESTRATION.md) | Model routing, proposals, planning limits, capability checks |
| [Integrations](docs/INTEGRATIONS.md) | Nmap, OWASP ZAP, and adapter boundaries |
| [Network enforcement](docs/NETWORK_ENFORCEMENT.md) | Enforcement and adapter isolation, per platform |
| [Audit and evidence](docs/AUDIT_AND_EVIDENCE.md) | Hash-chained audit, evidence privacy, replay retention |
| [Testing strategy](docs/TESTING_STRATEGY.md) | Negative tests and ground-truth testing |
| [V1 roadmap](docs/V1_ROADMAP.md) | Delivery phases and the thin vertical slice |
| [Open decisions](docs/OPEN_DECISIONS.md) | Unresolved decisions (`OD-nn`) |
| [Consistency review](docs/CONSISTENCY_REVIEW.md) | Second-pass review of this documentation set |
| [Changelog](CHANGELOG.md) | What changed in each documentation revision |

Project files: [LICENSE](LICENSE) (Apache License 2.0), [SECURITY.md](SECURITY.md), [RESPONSIBLE_USE.md](RESPONSIBLE_USE.md).

## Core safety principles

These summarize the design. Each is defined once in its owner document; follow the IDs.

1. **Authorization comes first.** No active testing without an authorization record and defined scope (`SEC-001`).
2. **The AI proposes; deterministic policy decides.** AI output is untrusted input (`SEC-050`, `SEC-016`).
3. **Scope is never inferred as permission.** Discovered assets are candidates (`SEC-003`).
4. **Unknown environments are treated as Production** (`SEC-005`).
5. **State changes require explicit approval of a complete test-case bundle** (`SEC-024`); cleanup follows the approved plan (`SEC-026` to `SEC-031`).
6. **Destructive actions are hard-denied in v1** (`SEC-020`).
7. **Every action is revalidated at dispatch time** (`SEC-016`).
8. **Internal modules never touch the network;** a centralized executor does (`SEC-032`, `SEC-033`).
9. **If a required safety control cannot be enforced, the action does not run** (`SEC-019`, `SEC-035`).
10. **Findings require evidence.** Verification goes through the same policy path; unverifiable findings stay likely or unverified (`SEC-041`, `SEC-042`).
11. **External content is sanitized before rendering** (`SEC-039`).
12. **The audit log is hash-chained and tamper-evident, not tamper-proof** (`SEC-044`).

## v1 permission profiles

**Guided** (green), **Active** (yellow), and **Autonomous** (red). Active and Autonomous have identical execution permissions and limits; Autonomous differs only in AI planning latitude. The matrix of what each level may run is in [Permission model](docs/PERMISSION_MODEL.md) (`SEC-007`, `SEC-010`, `SEC-011`). Production targets are capped at Guided (`SEC-005`).

## Important limitations

These documents define a design target. A control is not considered implemented merely because it is described here. Each security requirement must be mapped to code, tests, and observable enforcement before the product is represented as enforcing it. Sans-IO reduces ordinary module bypass risk but does not sandbox malicious Python code. Network and adapter enforcement guarantees are platform-specific and not yet validated. Residual risks are listed in the [threat model](docs/THREAT_MODEL.md).

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE). The copyright holder statement is not yet established (OD-02 in [open decisions](docs/OPEN_DECISIONS.md)).
