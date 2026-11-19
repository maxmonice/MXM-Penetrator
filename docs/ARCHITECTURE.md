# Architecture

This document describes components and lifecycle. It does not define security rules; those are owned by the documents listed in [DOCUMENT_OWNERSHIP.md](DOCUMENT_OWNERSHIP.md) and are referenced here by ID.

## Architectural principles

- CLI-first and local-first.
- Python is the primary implementation language.
- AI is advisory and proposal-producing, never an execution authority (`SEC-050`).
- A deterministic policy engine is the final authority for every action (`SEC-016`).
- Internal modules are Sans-IO: they produce typed request plans and process executor-provided responses; they do not perform network I/O (`SEC-032`).
- One centralized executor performs all network operations (`SEC-033`).
- External tools are isolated behind typed, vetted adapters (`SEC-036`, `SEC-037`).
- The assessment runs as a **staged asynchronous (asyncio) pipeline**, not a general-purpose DAG scheduler. Sans-IO modules do not dictate the concurrency model; the executor owns concurrency.
- Assessment state (SQLite) and evidence are stored separately.
- Components communicate through versioned schemas.
- Fail closed when required authorization or enforcement cannot be established (`SEC-019`).
- Language, license, and interface decisions: Python, Apache License 2.0 ([../LICENSE](../LICENSE)), CLI-first. A dashboard, REST API, distributed execution, and an MXM MCP server are deferred ([PRODUCT_SCOPE.md](PRODUCT_SCOPE.md)).

## High-level components

```text
CLI  (trusted control channel, SEC-018)
 ├── Assessment Manager
 │    ├── Authorization & Scope
 │    ├── Permission Profile
 │    ├── Execution / AI Budgets
 │    └── Assessment State (SQLite)
 │
 ├── Orchestrator
 │    ├── Reconnaissance Planner
 │    ├── Test Planner
 │    ├── AI Gateway / Model Router
 │    └── Internal Module Registry   (Sans-IO modules + manifests)
 │          ├── Recon Modules
 │          └── Vulnerability Test Modules
 │
 ├── Policy & Execution Boundary
 │    ├── Action Classifier
 │    ├── Policy Engine
 │    ├── Approval Manager
 │    ├── Scope Validator
 │    ├── Budget / Rate Limiter
 │    └── Mediated Network Executor   (sole network path for internal modules)
 │
 ├── External Adapter Host
 │    ├── Nmap Adapter
 │    └── OWASP ZAP Adapter
 │
 ├── Cleanup Ledger
 │
 └── Evidence & Reporting
      ├── Evidence Store
      ├── Audit Log  (hash-chained)
      ├── Finding Verifier  (Sans-IO; uses the same executor path)
      ├── Output Sanitizer  (all rendering goes through it)
      └── JSON / Markdown / HTML report renderers
```

This is a logical architecture. The final process boundaries, IPC mechanism, and storage implementation must be decided and tested during implementation (see OD-11, OD-19, OD-32, OD-34). Trust boundaries are described in [THREAT_MODEL.md](THREAT_MODEL.md).

## Component responsibilities in one line each

| Component | Responsibility | Authoritative rules |
|---|---|---|
| Action Classifier | Assigns the action class from manifest and concrete parameters | [ACTION_CLASSIFICATION.md](ACTION_CLASSIFICATION.md) |
| Policy Engine | Decides ALLOW / REQUIRE_APPROVAL / DENY / PARK | [POLICY_ENGINE.md](POLICY_ENGINE.md) |
| Approval Manager | Test-case bundles and trusted approvals | [APPROVALS_AND_CLEANUP.md](APPROVALS_AND_CLEANUP.md) |
| Scope Validator | Authorization, scope, and destination validation | [SCOPE_AND_AUTHORIZATION.md](SCOPE_AND_AUTHORIZATION.md) |
| Mediated Network Executor | Validates and performs every internal network operation | [MODULE_ARCHITECTURE.md](MODULE_ARCHITECTURE.md) |
| External Adapter Host | Runs vetted external tools in isolation | [INTEGRATIONS.md](INTEGRATIONS.md), [NETWORK_ENFORCEMENT.md](NETWORK_ENFORCEMENT.md) |
| Cleanup Ledger | Records test-created resources and cleanup state | [APPROVALS_AND_CLEANUP.md](APPROVALS_AND_CLEANUP.md) |
| Output Sanitizer | Sanitizes externally derived content before rendering | [OUTPUT_SANITIZATION.md](OUTPUT_SANITIZATION.md) |
| Finding Verifier | Verification plans, executed through the same boundary | [FINDING_VERIFICATION.md](FINDING_VERIFICATION.md) |
| Audit Log / Evidence Store | Tamper-evident audit and protected evidence | [AUDIT_AND_EVIDENCE.md](AUDIT_AND_EVIDENCE.md) |
| AI Gateway / Orchestrator | Model routing, typed proposals, bounded planning | [AI_ORCHESTRATION.md](AI_ORCHESTRATION.md) |

## Assessment lifecycle

1. The user creates an assessment.
2. MXM collects the target authorization, explicit scope, and environment designation. An unspecified environment is treated as Production (`SEC-005`).
3. MXM derives the authorization ceiling from the record (`SEC-005`) and shows which levels are available.
4. The user selects Guided, Active, or Autonomous. Guided is preselected, and the user confirms the selection (`SEC-007`).
5. Optional natural-language input may be interpreted by AI as a recommendation; the user must confirm (`SEC-012`).
6. The assessment manager creates the authorization record, the initial policy epoch (`SEC-021`), and execution and AI budgets (`SEC-023`, `SEC-053`).
7. Reconnaissance modules emit request plans; the executor validates and performs permitted actions through the policy boundary.
8. Newly discovered assets are recorded as scope candidates (`SEC-003`).
9. The planner proposes bounded tests based on authorized assets and collected evidence.
10. The policy engine classifies and evaluates every action (`SEC-016`). Approval requirements follow the permission matrix in [PERMISSION_MODEL.md](PERMISSION_MODEL.md); destructive actions are denied (`SEC-020`).
11. Evidence is sanitized, redacted, and stored; findings are verified through the same executor and policy path where possible (`SEC-041`). Findings whose verification is not currently permitted remain likely or unverified (`SEC-042`).
12. Cleanup obligations are reconciled; pending resources are reported (`SEC-026`, `SEC-028`, `SEC-030`). `mxm cleanup` is the manual path (`SEC-029`).
13. MXM generates reports through the Output Sanitizer and preserves a hash-chained audit trail (`SEC-044`).

## Execution boundary

All internal modules submit typed request plans to the single controlled execution boundary. The module interface gives modules no network handle (`SEC-032`). The executor validates every request against the module manifest, scope, authorization, policy, permissions, and budgets (`SEC-033`, `SEC-034`). Details: [MODULE_ARCHITECTURE.md](MODULE_ARCHITECTURE.md).

External adapters receive typed, validated parameters, must not expose arbitrary shell command construction to the AI planner, and must be constrained by OS or process-level isolation (`SEC-036`, `SEC-037`). Sans-IO does **not** sandbox malicious Python code, so it is not a substitute for adapter isolation.

## State and evidence

- SQLite is the initial assessment-state store.
- Evidence is stored separately from assessment metadata (`SEC-046`, `SEC-047`).
- JSON is the canonical machine-readable output; Markdown and HTML are rendered from the same report model.
- Schemas are versioned to support future migrations.

## Failure behavior

Failure behavior is owned by `SEC-019` ([POLICY_ENGINE.md](POLICY_ENGINE.md)). The system must not silently downgrade safety checks or continue with an unbounded fallback.
