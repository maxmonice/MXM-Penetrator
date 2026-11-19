# Security Requirements Index

This is an **index**, not a rulebook. Each requirement is defined once, in its authoritative document (see [DOCUMENT_OWNERSHIP.md](DOCUMENT_OWNERSHIP.md)). The table lists the ID, a short title, the owning document, the origin of the requirement, the logical component expected to enforce it, and the planned verification. For the normative text, read the owner document.

**Status of every requirement: specified, not implemented.** No requirement may be represented as enforced until it is mapped to code, tests, and observable enforcement (see [TESTING_STRATEGY.md](TESTING_STRATEGY.md) and the release gate in [V1_ROADMAP.md](V1_ROADMAP.md)).

## Legend

**Origin**
- *Baseline*: present in the original 14-document baseline or in an earlier approved decision.
- *A–R*: the revision decision (see [../CHANGELOG.md](../CHANGELOG.md)) that introduced or changed the requirement. Letters refer to the approved decision list of the v1.1 documentation revision.

**Planned verification**
- **NT**: negative test proving a forbidden action is blocked.
- **PT**: property-based test.
- **IT**: integration test.
- **GT**: ground-truth test (see `SEC-056`).
- **RV**: design or code review checkpoint where automated testing is not sufficient.

**Components** are the logical components from [ARCHITECTURE.md](ARCHITECTURE.md). Final process boundaries are not yet decided.

## Index

| ID | Title | Owner | Origin | Enforcing component | Planned verification |
|---|---|---|---|---|---|
| SEC-001 | Authorization record required before active testing | SCOPE_AND_AUTHORIZATION | Baseline | Assessment Manager, Policy Engine | NT, IT |
| SEC-002 | Explicit scope and execution-time destination validation | SCOPE_AND_AUTHORIZATION | Baseline | Scope Validator, Mediated Network Executor | NT, IT |
| SEC-003 | Newly discovered assets are scope candidates | SCOPE_AND_AUTHORIZATION | Baseline | Scope Validator, Assessment Manager | NT |
| SEC-004 | Authorization and scope changes only through the trusted control channel | SCOPE_AND_AUTHORIZATION | Baseline | Assessment Manager, Policy Engine | NT |
| SEC-005 | Unknown environment defaults to Production; production ceiling | SCOPE_AND_AUTHORIZATION | F | Assessment Manager, Policy Engine | NT, PT |
| SEC-006 | Authorization lifecycle: expiry, revocation, restoration | SCOPE_AND_AUTHORIZATION | D | Policy Engine, Assessment Manager | NT, IT |
| SEC-007 | Fixed permission profiles, Guided default, effective level | PERMISSION_MODEL | Baseline | Assessment Manager, Policy Engine | NT |
| SEC-008 | Permission-level changes | PERMISSION_MODEL | Baseline | Policy Engine, Approval Manager | NT, IT |
| SEC-009 | AI cannot change its level or approve actions | PERMISSION_MODEL | Baseline | Policy Engine, AI Gateway | NT |
| SEC-010 | Active/Autonomous execution-permission invariant | PERMISSION_MODEL | Baseline | Policy Engine | PT |
| SEC-011 | Active/Autonomous identical execution limits | PERMISSION_MODEL | H | Policy Engine, Budget / Rate Limiter | PT, NT |
| SEC-012 | Natural-language level selection is advisory | PERMISSION_MODEL | Baseline | CLI, AI Gateway | NT |
| SEC-013 | Deterministic classification; AI never assigns the class | ACTION_CLASSIFICATION | Baseline | Action Classifier | NT, PT |
| SEC-014 | Classify by effect; credential/login testing is state-changing | ACTION_CLASSIFICATION | Baseline | Action Classifier | PT, NT |
| SEC-015 | Central numeric thresholds for safe-active and intrusive limits | ACTION_CLASSIFICATION | G | Action Classifier, Budget / Rate Limiter | PT, IT |
| SEC-016 | Dispatch-time evaluation order and revalidation | POLICY_ENGINE | Baseline | Policy Engine | NT, IT |
| SEC-017 | Short-lived single-use execution capabilities | POLICY_ENGINE | Baseline | Policy Engine, Mediated Network Executor | NT |
| SEC-018 | Trusted control channel | POLICY_ENGINE | Baseline | CLI, Approval Manager | NT, RV |
| SEC-019 | Fail-closed behavior | POLICY_ENGINE | Baseline | Policy Engine | NT |
| SEC-020 | Destructive actions hard-denied | POLICY_ENGINE | Baseline | Policy Engine | NT |
| SEC-021 | Policy epochs | POLICY_ENGINE | Baseline, B | Policy Engine | PT, NT |
| SEC-022 | Kill switch and pause | POLICY_ENGINE | D | Policy Engine, Mediated Network Executor | NT, IT |
| SEC-023 | Budgets and rate limits; execution budgets separate from AI budgets | POLICY_ENGINE | Baseline, O | Budget / Rate Limiter | NT, PT |
| SEC-024 | State-changing test-case approval bundles | APPROVALS_AND_CLEANUP | Baseline | Approval Manager | NT |
| SEC-025 | Approval binding, single use, expiry, timeout | APPROVALS_AND_CLEANUP | Baseline | Approval Manager | NT |
| SEC-026 | Test lifecycle and write-ahead cleanup ledger | APPROVALS_AND_CLEANUP | Baseline | Cleanup Ledger, Orchestrator | NT, IT |
| SEC-027 | Cleanup survives policy epochs | APPROVALS_AND_CLEANUP | B | Cleanup Ledger, Policy Engine | NT, IT |
| SEC-028 | Automatic cleanup and process-termination behavior | APPROVALS_AND_CLEANUP | C | Cleanup Ledger, Orchestrator | IT |
| SEC-029 | Manual cleanup (`mxm cleanup`) | APPROVALS_AND_CLEANUP | C | CLI, Cleanup Ledger, Policy Engine | NT, IT |
| SEC-030 | Cleanup failure handling | APPROVALS_AND_CLEANUP | Baseline | Cleanup Ledger, Orchestrator | IT |
| SEC-031 | Cleanup under kill switch and authorization revocation | APPROVALS_AND_CLEANUP | D | Cleanup Ledger, Policy Engine | NT, IT |
| SEC-032 | Sans-IO internal modules | MODULE_ARCHITECTURE | A | Internal Module Registry | RV, NT |
| SEC-033 | Single centralized executor | MODULE_ARCHITECTURE | A | Mediated Network Executor | NT, IT |
| SEC-034 | Module manifests required and enforced at runtime | MODULE_ARCHITECTURE | A | Mediated Network Executor, Action Classifier | NT, PT |
| SEC-035 | OS/network-level enforcement; fail closed when unavailable | NETWORK_ENFORCEMENT | Baseline | Mediated Network Executor, platform enforcement | NT, IT |
| SEC-036 | External adapter process isolation | NETWORK_ENFORCEMENT | A | External Adapter Host | NT, RV |
| SEC-037 | External adapter requirements | INTEGRATIONS | Baseline | External Adapter Host | NT, IT |
| SEC-038 | Initial Nmap and OWASP ZAP integration scope | INTEGRATIONS | I | External Adapter Host | NT, IT |
| SEC-039 | Sanitize all externally derived content before rendering | OUTPUT_SANITIZATION | E | Output Sanitizer | NT, PT |
| SEC-040 | Trusted prompts resistant to spoofing | OUTPUT_SANITIZATION | Baseline, E | Approval Manager, Output Sanitizer | NT |
| SEC-041 | Verification passes through the executor and policy engine | FINDING_VERIFICATION | J | Finding Verifier, Policy Engine | NT |
| SEC-042 | Unverifiable findings remain likely/unverified | FINDING_VERIFICATION | J | Finding Verifier | NT, GT |
| SEC-043 | Security-relevant audit events | AUDIT_AND_EVIDENCE | Baseline | Audit Log | IT |
| SEC-044 | Hash-chained, tamper-evident audit log | AUDIT_AND_EVIDENCE | K | Audit Log | NT, PT |
| SEC-045 | Evidence redacted by default | AUDIT_AND_EVIDENCE | L | Evidence Store | NT, PT |
| SEC-046 | Encryption at rest and key separation | AUDIT_AND_EVIDENCE | L | Evidence Store | NT, RV |
| SEC-047 | Sanitized evidence versus restricted raw evidence | AUDIT_AND_EVIDENCE | L | Evidence Store | NT |
| SEC-048 | AI replay retention defaults and raw-retention opt-in | AUDIT_AND_EVIDENCE | M | AI Gateway, Audit Log | NT |
| SEC-049 | Secrets never stored in plaintext logs or evidence | AUDIT_AND_EVIDENCE | Baseline | Evidence Store, Audit Log | NT |
| SEC-050 | Typed action proposals; AI output is untrusted | AI_ORCHESTRATION | Baseline | AI Gateway, Orchestrator | NT |
| SEC-051 | Prompt-injection containment | AI_ORCHESTRATION | Baseline | AI Gateway, Orchestrator | NT, GT |
| SEC-052 | Local-first AI; explicit cloud opt-in; redaction before transmission | AI_ORCHESTRATION | Baseline | AI Gateway | NT |
| SEC-053 | Centrally enforced AI planning limits | AI_ORCHESTRATION | O | Orchestrator, Budget / Rate Limiter | NT, PT |
| SEC-054 | Local model capability checks before Autonomous | AI_ORCHESTRATION | N | AI Gateway | IT |
| SEC-055 | Negative tests for every security requirement | TESTING_STRATEGY | Baseline | Test suite | RV |
| SEC-056 | Ground-truth testing | TESTING_STRATEGY | Q | Test suite | GT |

## Traceability fields not yet populated

The index deliberately does not yet name code modules or test identifiers, because no implementation exists. When implementation starts, each row gains: the implementing module path, the test IDs, and the date a control was first demonstrated against its negative test. A requirement without a passing negative test is *not enforced* for the purposes of any release claim.
