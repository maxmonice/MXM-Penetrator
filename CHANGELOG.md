# Changelog

All notable documentation changes are recorded here. No application code exists; every entry describes documentation only.

## 1.1.0 — v1 documentation revision (2026-11-19)

Revises the 14-document baseline (README plus 13 documents in `docs/`) to incorporate the architecture review and the decisions made since.

### Summary of architectural changes

1. **Sans-IO modules and a centralized executor.** Internal modules produce typed request plans and process executor-provided responses; they do not do network I/O. The executor validates every request, and module manifests are enforced at runtime (methods, destinations, request rates). Sans-IO does not sandbox hostile Python, so external adapters still need OS or process isolation.
2. **A defined cleanup model.** Cleanup of test-created, ledger-recorded resources survives policy epoch changes under four conditions. Normal tests clean up automatically; after a crash automatic cleanup stops and the ledger is preserved; `mxm cleanup` is the manual path and needs fresh confirmation every run. The kill switch stops everything including automatic cleanup; authorization revocation blocks both automatic and manual network cleanup until authorization is restored.
3. **Deterministic classification with central thresholds.** Numeric limits for safe-active and intrusive testing live only in `ACTION_CLASSIFICATION.md`. Their values are not yet designed. Active and Autonomous share identical execution limits; Autonomous differs only in AI planning latitude, and AI planning limits are separate from network budgets.
4. **Production by default.** Unknown environments are Production, which is capped at Guided; labels never override other controls.
5. **Verification cannot bypass policy.** Every verification request uses the same executor and policy path. A finding that cannot be verified under current permissions stays likely or unverified.
6. **Output sanitization.** All externally derived content is sanitized at the rendering boundary, and trusted prompts are protected against spoofing.
7. **Evidence and audit integrity and privacy.** Hash-chained audit log (tamper-evident, not tamper-proof); redaction by default; encryption at rest with separate key management; sanitized versus raw evidence tiers; AI replay retention without raw prompts by default.
8. **AI safeguards.** Local model capability checks gate Autonomous mode; AI planning limits are centrally enforced.
9. **Integration scope.** Nmap begins with TCP connect scans; ZAP begins with passive scanning; platform capabilities are documented separately for Windows and Linux.
10. **Delivery and measurement.** Thin end-to-end vertical slice first; ground-truth testing against an intentionally vulnerable application from the start; v1 focuses on unauthenticated exposure and misconfiguration.
11. **Governance and traceability.** Formal threat model, stable `SEC-nnn` requirement IDs with a single authoritative owner each, license, security policy, and a responsible-use statement.

### Revision decisions (key for the "Origin" column in the requirements index)

| Letter | Decision | Authoritative document |
|---|---|---|
| A | Sans-IO internal module architecture | [MODULE_ARCHITECTURE.md](docs/MODULE_ARCHITECTURE.md) |
| B | Cleanup survives policy epochs | [APPROVALS_AND_CLEANUP.md](docs/APPROVALS_AND_CLEANUP.md) |
| C | Automatic and manual cleanup (`mxm cleanup`) | [APPROVALS_AND_CLEANUP.md](docs/APPROVALS_AND_CLEANUP.md) |
| D | Kill switch and authorization revocation | [POLICY_ENGINE.md](docs/POLICY_ENGINE.md), [SCOPE_AND_AUTHORIZATION.md](docs/SCOPE_AND_AUTHORIZATION.md), [APPROVALS_AND_CLEANUP.md](docs/APPROVALS_AND_CLEANUP.md) |
| E | Output sanitization | [OUTPUT_SANITIZATION.md](docs/OUTPUT_SANITIZATION.md) |
| F | Environment classification (unknown = Production) | [SCOPE_AND_AUTHORIZATION.md](docs/SCOPE_AND_AUTHORIZATION.md) |
| G | Centralized numeric testing thresholds | [ACTION_CLASSIFICATION.md](docs/ACTION_CLASSIFICATION.md) |
| H | Identical Active/Autonomous execution budgets | [PERMISSION_MODEL.md](docs/PERMISSION_MODEL.md) |
| I | Nmap `-sT` first; ZAP passive first; per-platform notes | [INTEGRATIONS.md](docs/INTEGRATIONS.md) |
| J | Finding verification through the policy path | [FINDING_VERIFICATION.md](docs/FINDING_VERIFICATION.md) |
| K | Hash-chained audit log | [AUDIT_AND_EVIDENCE.md](docs/AUDIT_AND_EVIDENCE.md) |
| L | Evidence privacy | [AUDIT_AND_EVIDENCE.md](docs/AUDIT_AND_EVIDENCE.md) |
| M | AI replayability and privacy | [AUDIT_AND_EVIDENCE.md](docs/AUDIT_AND_EVIDENCE.md) |
| N | Local AI capability checks before Autonomous | [AI_ORCHESTRATION.md](docs/AI_ORCHESTRATION.md) |
| O | AI planning limits | [AI_ORCHESTRATION.md](docs/AI_ORCHESTRATION.md) |
| P | Thin vertical slice | [V1_ROADMAP.md](docs/V1_ROADMAP.md) |
| Q | Ground-truth testing | [TESTING_STRATEGY.md](docs/TESTING_STRATEGY.md) |
| R | Authenticated testing deferred | [PRODUCT_SCOPE.md](docs/PRODUCT_SCOPE.md) |

### Added

- [LICENSE](LICENSE): the official Apache License 2.0 text (unmodified, including the standard appendix; the copyright holder statement is OD-02).
- [SECURITY.md](SECURITY.md): reporting scope and process. The reporting channel itself is not yet defined (OD-01).
- [RESPONSIBLE_USE.md](RESPONSIBLE_USE.md).
- [docs/THREAT_MODEL.md](docs/THREAT_MODEL.md): assets, trust boundaries, actors, attack surfaces, assumptions, 32 abuse cases, residual risks.
- [docs/SECURITY_REQUIREMENTS.md](docs/SECURITY_REQUIREMENTS.md): index of `SEC-001` to `SEC-056`.
- [docs/DOCUMENT_OWNERSHIP.md](docs/DOCUMENT_OWNERSHIP.md).
- [docs/MODULE_ARCHITECTURE.md](docs/MODULE_ARCHITECTURE.md), [docs/OUTPUT_SANITIZATION.md](docs/OUTPUT_SANITIZATION.md), [docs/FINDING_VERIFICATION.md](docs/FINDING_VERIFICATION.md), [docs/TESTING_STRATEGY.md](docs/TESTING_STRATEGY.md).
- [docs/CONSISTENCY_REVIEW.md](docs/CONSISTENCY_REVIEW.md): the second-pass review.

### Changed

| File | Change |
|---|---|
| README.md | Status, documentation table, principles now reference IDs; license and governance links |
| PRODUCT_SCOPE.md | v1 detection focus; deferred authenticated testing; MXM MCP server added to non-goals |
| ARCHITECTURE.md | Sans-IO and asyncio principles; Cleanup Ledger and Output Sanitizer components; lifecycle reordered so authorization and ceiling precede level selection |
| PERMISSION_MODEL.md | Now the single owner of the class-by-level matrix; identical execution limits (`SEC-011`); ceiling rule moved to `SEC-005` |
| ACTION_CLASSIFICATION.md | Central thresholds table (values not designed); precedence rule; policy mapping removed (owned by the permission matrix and the policy engine) |
| POLICY_ENGINE.md | Evaluation order now includes classification first and manifest conformance; epochs, kill switch, budgets consolidated |
| SCOPE_AND_AUTHORIZATION.md | Environment classification; authorization lifecycle; ceiling rule is now only here |
| APPROVALS_AND_CLEANUP.md | Rewritten around `SEC-024` to `SEC-031` |
| AI_ORCHESTRATION.md | Planning limits and capability checks; action counts and network concurrency removed from AI limits because they are execution budgets |
| AUDIT_AND_EVIDENCE.md | Hash chain, redaction, encryption, tiers, replay retention |
| INTEGRATIONS.md, NETWORK_ENFORCEMENT.md | Initial Nmap/ZAP scope; per-platform tables; adapter isolation |
| V1_ROADMAP.md | Restructured around the thin vertical slice and early ground-truth testing |
| OPEN_DECISIONS.md | Replaced with the `OD-01` to `OD-36` register; duplicated security requirements removed in favor of the index |

### Interpretive additions (confirmed by the project owner, 2026-11-19)

These follow from the approved decisions but were not stated in them. The project owner confirmed all of them. Reversing any requires an explicit decision:

1. The evaluation order places classification before step 1 and adds manifest conformance (a consequence of decision A).
2. Cleanup is exempt from epoch-bound approval and level gating (its approval is the original bundle) but is not exempt from kill, authorization, destination validation, manifest, or executor checks (B with D).
3. Interim fail-closed rule: if MXM cannot show a recorded resource was created by the test, cleanup does not delete it and leaves it pending (OD-09).
4. If a target is removed from scope while authorization remains valid, cleanup is parked pending a decision (OD-07).
5. After a crash or restart MXM does not resume cleanup silently; pending resources are cleaned only by `mxm cleanup` (C).
6. "Passive" excludes a third-party service that actively probes the target.
7. The hash chain cannot by itself reveal deletion of the most recent events; this limitation is stated.
8. Externally supplied filenames are never used as filesystem paths without validation (E).
9. Trusted prompts never contain model-generated text.
10. Re-designating a target from Production to non-production is a trusted-channel, audited action.
11. "Concurrent tasks" in AI planning limits means planning or analysis concurrency, not network concurrency.
12. ZAP "passive scanning" is interpreted as analysis of traffic MXM's executor produced (OD-21).
13. The effective level is shown to the user when it differs from the selection.
14. Verification actions consume the same execution budgets as other actions.
15. Roadmap detail: the slice supports only passive and safe-active classes; hash chaining completes in Phase 2.
16. "Verification accuracy" is defined in `TESTING_STRATEGY.md`.
17. `SECURITY.md` response commitments are deliberately omitted because none exist.
18. An expired authorization is treated like a revoked one for network actions, including cleanup (decision D names revocation only).
19. Pause is defined minimally (no new dispatch) with in-flight semantics left open (OD-06).

### Not changed

All baseline decisions are preserved, including Python, Apache License 2.0, CLI-first, local-first AI with explicit cloud opt-in, the AI gateway, the rules-based model router, typed action proposals, the independent policy engine, network-level enforcement with platform-specific implementations, REA as an optional isolated adapter, the staged asyncio pipeline, SQLite plus a separate evidence store, JSON as canonical output with Markdown and HTML reports, deferred dashboard, REST API, distributed execution, and MXM MCP server, the three permission levels with Guided preselected, mandatory approval of state-changing test-case bundles, hard-denied destructive operations, no AI self-promotion, audited mid-assessment level changes, the production Guided ceiling, scope candidates, and credential-testing classification.

### Remaining open decisions

See [docs/OPEN_DECISIONS.md](docs/OPEN_DECISIONS.md): `OD-01` to `OD-36`.
