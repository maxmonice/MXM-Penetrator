# Document Ownership

## Purpose

Every security rule has exactly one **authoritative document**. Other documents must reference the rule by its stable ID (for example `SEC-016`) and the owning document instead of restating it. This prevents the drift that appeared in the baseline, where the authorization ceiling, the destructive hard-deny, and the policy-epoch rules were each described in several places.

## Conventions

1. **Rules carry IDs.** A security requirement is defined once, in its owner document, under a heading of the form `SEC-nnn — Title`. IDs are stable: they are never reused or renumbered. A withdrawn requirement is marked *Withdrawn* in place.
2. **The owner wins.** If another document appears to conflict with the owner, the owner is correct and the other document must be fixed.
3. **Reference, do not copy.** Non-owner documents may summarize *context* in a sentence, but they must not restate normative detail (thresholds, ordering, state transitions, allowed/denied lists). Use "see `SEC-nnn` in `<owner>`".
4. **Index.** [SECURITY_REQUIREMENTS.md](SECURITY_REQUIREMENTS.md) lists every ID with its owner, origin, planned enforcing component, and planned verification. It contains titles only, not rule text.
5. **Open decisions** are tracked only in [OPEN_DECISIONS.md](OPEN_DECISIONS.md) with IDs of the form `OD-nn`. Other documents point to the ID. Where a value (such as a numeric threshold) is not yet designed, the owner document names the parameter and marks it *requires design and validation*; no value is implied.
6. **Changes** to a rule are made in the owner first, then in the index, then in [../CHANGELOG.md](../CHANGELOG.md). Changing a rule that an approved decision established requires an explicit decision, not an editorial edit.

## Ownership map

| Rule domain | Authoritative document | IDs |
|---|---|---|
| Product vision, audience, v1 goals, non-goals, deferred authenticated testing | [PRODUCT_SCOPE.md](PRODUCT_SCOPE.md) | none (not security rules) |
| Components, lifecycle, process-boundary principles | [ARCHITECTURE.md](ARCHITECTURE.md) | none |
| Authorization record and lifecycle, scope, scope candidates, environment classification, production ceiling | [SCOPE_AND_AUTHORIZATION.md](SCOPE_AND_AUTHORIZATION.md) | SEC-001 – SEC-006 |
| Permission profiles, level changes, Active/Autonomous invariants and shared execution limits | [PERMISSION_MODEL.md](PERMISSION_MODEL.md) | SEC-007 – SEC-012 |
| Action classes, classification rules, numeric-threshold ownership | [ACTION_CLASSIFICATION.md](ACTION_CLASSIFICATION.md) | SEC-013 – SEC-015 |
| Policy evaluation order, capabilities, control channel, fail-closed behavior, destructive hard-deny, epochs, kill switch, budgets and rate limits | [POLICY_ENGINE.md](POLICY_ENGINE.md) | SEC-016 – SEC-023 |
| Approvals, test-case bundles, cleanup ledger, automatic/manual cleanup, cleanup under kill or revocation | [APPROVALS_AND_CLEANUP.md](APPROVALS_AND_CLEANUP.md) | SEC-024 – SEC-031 |
| Sans-IO module interface, centralized executor, module manifests | [MODULE_ARCHITECTURE.md](MODULE_ARCHITECTURE.md) | SEC-032 – SEC-034 |
| OS/network-level enforcement and adapter process isolation | [NETWORK_ENFORCEMENT.md](NETWORK_ENFORCEMENT.md) | SEC-035 – SEC-036 |
| External adapter requirements, Nmap and OWASP ZAP integration scope | [INTEGRATIONS.md](INTEGRATIONS.md) | SEC-037 – SEC-038 |
| Sanitization of externally derived content and trusted-prompt rendering | [OUTPUT_SANITIZATION.md](OUTPUT_SANITIZATION.md) | SEC-039 – SEC-040 |
| Finding verification | [FINDING_VERIFICATION.md](FINDING_VERIFICATION.md) | SEC-041 – SEC-042 |
| Audit events, audit hash chain, evidence redaction, encryption, evidence tiers, AI replay retention, secrets in logs | [AUDIT_AND_EVIDENCE.md](AUDIT_AND_EVIDENCE.md) | SEC-043 – SEC-049 |
| AI proposals, prompt-injection containment, local-first and cloud opt-in, AI planning limits, local capability checks | [AI_ORCHESTRATION.md](AI_ORCHESTRATION.md) | SEC-050 – SEC-054 |
| Negative-test obligations and ground-truth testing | [TESTING_STRATEGY.md](TESTING_STRATEGY.md) | SEC-055 – SEC-056 |
| Delivery phases and the thin vertical slice | [V1_ROADMAP.md](V1_ROADMAP.md) | none |
| Threats, assets, trust boundaries, assumptions, residual risks | [THREAT_MODEL.md](THREAT_MODEL.md) | none (maps to IDs) |
| Unresolved decisions | [OPEN_DECISIONS.md](OPEN_DECISIONS.md) | OD-01 – OD-36 |
| Change history and revision-decision key (A–R) | [../CHANGELOG.md](../CHANGELOG.md) | none |
| Second-pass consistency review of the documentation set | [CONSISTENCY_REVIEW.md](CONSISTENCY_REVIEW.md) | none |
| Vulnerability reporting and disclosure process | [../SECURITY.md](../SECURITY.md) | none |
| Responsible-use statement | [../RESPONSIBLE_USE.md](../RESPONSIBLE_USE.md) | none |
| License | [../LICENSE](../LICENSE) | none |

## Cross-document dependencies that must stay consistent

The following topics span several documents. Each has one owner; the others only reference it.

| Topic | Owner | Referenced from |
|---|---|---|
| Active and Autonomous share execution permissions and limits | PERMISSION_MODEL (`SEC-010`, `SEC-011`) | AI_ORCHESTRATION, ARCHITECTURE, README |
| Which class each permission level may run automatically | PERMISSION_MODEL (`SEC-007`) | ACTION_CLASSIFICATION, POLICY_ENGINE |
| Numeric thresholds for safe-active and intrusive limits | ACTION_CLASSIFICATION (`SEC-015`) | PERMISSION_MODEL, POLICY_ENGINE, MODULE_ARCHITECTURE |
| Kill switch semantics | POLICY_ENGINE (`SEC-022`) | APPROVALS_AND_CLEANUP, NETWORK_ENFORCEMENT |
| What happens to cleanup under kill or revocation | APPROVALS_AND_CLEANUP (`SEC-031`) | POLICY_ENGINE, SCOPE_AND_AUTHORIZATION |
| Production default and unknown-environment rule | SCOPE_AND_AUTHORIZATION (`SEC-005`) | PERMISSION_MODEL, ARCHITECTURE |
| Policy epoch definition | POLICY_ENGINE (`SEC-021`) | PERMISSION_MODEL, APPROVALS_AND_CLEANUP |
| Executor and manifest enforcement | MODULE_ARCHITECTURE (`SEC-033`, `SEC-034`) | POLICY_ENGINE, ACTION_CLASSIFICATION, INTEGRATIONS |
