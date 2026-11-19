# Consistency Review (second pass)

**Scope:** `README.md`, `SECURITY.md`, `RESPONSIBLE_USE.md`, `CHANGELOG.md`, and every file in `docs/` after the v1.1 revision.
**Method:** scripted cross-checks (ID definitions and references, link targets, numeric content) plus a manual read of the permission, cleanup, authorization, epoch, kill-switch, and enforcement rules against each other.
**Limit:** this review was performed by the same author who wrote the revision. It is a consistency check, not independent assurance. An independent review of the documents, especially of `THREAT_MODEL.md`, is still recommended before implementation.

## Results of the requested checks

| # | Check | Result |
|---|---|---|
| 1 | Contradictions across documents | None found after the fixes listed below. |
| 2 | Permission model, cleanup, authorization, epochs, kill switch, and network enforcement agree | They agree. Kill: `SEC-022` stops all network activity including automatic cleanup; `SEC-031` governs the ledger. Revocation: `SEC-006` blocks network actions; `SEC-031` blocks automatic and manual cleanup and requires renewed authorization. Epochs: `SEC-021` defines epochs; `SEC-025` binds approvals to them; `SEC-027` exempts cleanup. `SEC-035` and `SEC-036` supply enforcement and adapter termination for the kill path. |
| 3 | Modules cannot bypass the executor through their documented interface | The documented interface (`SEC-032`) gives modules no network handle, client, or credential, and `SEC-033` routes every request through the executor. **This does not stop deliberately hostile Python code**, and the documents say so in `MODULE_ARCHITECTURE.md`, `ARCHITECTURE.md`, the threat model (AC-09), and `SECURITY.md`. Since decision S, violations are detected by review and automated checks (accidents only), and process separation is deferred; tooling is open (OD-15). |
| 4 | Verification actions cannot bypass policy | `SEC-041` requires the same executor and policy path, the same budgets, and approvals; `SEC-016` states that its order applies to verification; `SEC-024` requires a bundle for state-changing verification; `SEC-042` leaves blocked findings likely or unverified. |
| 5 | Active and Autonomous execution budgets remain identical | `SEC-011` states identical permissions, rate ceilings, concurrency, per-test budgets, and assessment-wide budgets; thresholds are defined per class, not per level (`SEC-015`); `AI_ORCHESTRATION.md` keeps AI planning limits separate and removed "action counts and concurrency" from them; `SEC-010` and `SEC-011` are covered by property and negative tests in `TESTING_STRATEGY.md`. |
| 6 | No undocumented numeric limits invented | A scripted scan found no numeric limit, rate, timeout, retry count, or duration in the documents. Every such value is marked *requires design and validation* or points to an open decision (OD-03, OD-04, OD-08, OD-14, OD-22). Only identifiers, phase numbers, and version references appear as numbers. |
| 7 | Deferred features are not presented as v1 requirements | Authenticated, multi-user, cross-account, and session-based testing appear only as deferred (`PRODUCT_SCOPE.md`, `V1_ROADMAP.md`, `TESTING_STRATEGY.md`). Dashboard, REST API, distributed execution, MXM MCP server, custom profiles, and multi-user approvals appear only as non-goals. ZAP active scanning and Nmap raw scans are marked "not in the initial integration". Assessment intelligence is a long-term differentiator only. |
| 8 | Every accepted decision is represented | All 18 revision decisions (A–R) map to an owner document and requirement IDs in the `CHANGELOG.md` key and in [SECURITY_REQUIREMENTS.md](SECURITY_REQUIREMENTS.md) (the P, Q, and R decisions are non-security and carry no `SEC` ID). All preserved baseline decisions are confirmed present; see "Preserved decisions" below. |
| 9 | Remaining open decisions identified | `OD-01` to `OD-36` in [OPEN_DECISIONS.md](OPEN_DECISIONS.md). A scripted check confirmed that every `OD` referenced anywhere is defined and every defined `OD` is referenced from at least one other document. |

## Scripted integrity checks

- 56 requirement IDs (`SEC-001` to `SEC-056`) are each defined exactly once, in the document the index names as owner.
- No ID is referenced without a definition, and no definition is missing from the index.
- All relative links resolve.

## Defects found during the second pass, and fixed

1. **Duplicated ceiling rule.** "Effective level is the most restrictive of the selection and the ceiling" appeared in both `PERMISSION_MODEL.md` and `SCOPE_AND_AUTHORIZATION.md`. It is now stated only in `SEC-007`; `SEC-005` references it.
2. **Guided and the planning limits.** `PERMISSION_MODEL.md` said planning limits apply "in every case", while decision O covers Active and Autonomous only and `OD-04` leaves Guided open. The sentence now says Active and Autonomous.
3. **Unreferenced open decision.** `OD-31` (initial module list) was not referenced elsewhere; it is now referenced from Phase 3 of the roadmap.
4. **A rule added without approval.** An earlier draft of `SEC-008` required a "what will change" confirmation on upgrade. That was not an approved decision, so it was removed.
5. **Changelog gaps.** Expired-authorization handling and the minimal pause definition were interpretive additions that were not listed; they are now items 18 and 19 of the changelog's interpretive list.

## Preserved decisions: spot-check

Python (`ARCHITECTURE.md`); Apache License 2.0 (`LICENSE`, `README.md`); CLI-first; local-first AI with explicit cloud opt-in (`SEC-052`); AI gateway; rules-based router (`AI_ORCHESTRATION.md`); typed action proposals (`SEC-050`); independent policy engine (`SEC-016`); network-level enforcement with platform-specific implementations (`SEC-035`); REA as an isolated optional adapter (`INTEGRATIONS.md`); staged asyncio pipeline, not a DAG scheduler (`ARCHITECTURE.md`); SQLite plus separate evidence store; JSON canonical with Markdown and HTML; deferred dashboard, REST API, distributed execution, and MXM MCP server; the three permission levels with Guided preselected (`SEC-007`); approval of state-changing bundles at every level (`SEC-024`); hard-denied destructive actions (`SEC-020`); AI cannot change its own level (`SEC-009`); audited mid-assessment level changes (`SEC-008`); production ceiling of Guided (`SEC-005`); scope candidates (`SEC-003`); credential/login testing as state-changing and high-volume attempts as destructive (`SEC-014`); kill switch, audit logging, rate limits, secret management, and versioned schemas (`SEC-022`, `SEC-043`, `SEC-023`, `SEC-049`, `ARCHITECTURE.md`).

## Residual weaknesses and unresolved questions

These are not contradictions; they are places where the documents are honest about not yet knowing.

1. **Several controls depend on open decisions.** Safe-active and intrusive thresholds (OD-03) gate the vertical slice; the complete epoch-event list (OD-05) affects approval binding; the control-channel mechanism (OD-11) underpins every approval; and the proof that a cleanup identifier belongs to the test (OD-09) underpins the cleanup carve-out.
2. **Windows and Linux enforcement strength is unknown** (OD-17 to OD-21). The roadmap permits disabling an adapter on a platform that cannot meet the requirements.
3. **Cleanup exemptions are an interpretation.** Exempting cleanup from epoch-bound approval and level gating (while keeping kill, authorization, destination, manifest, and executor checks) is derived from decisions B and D, not stated in them. The project owner confirmed it on 2026-11-19, together with all other interpretive additions listed in the changelog.
4. **Cleanup and budgets.** If budgets can be exhausted before cleanup runs, test-created resources could be stranded (OD-08).
5. **Pause semantics are minimal** (OD-06).
6. **Hash-chain limits.** Tail truncation and a full privileged rewrite are not detected by the chain alone (OD-23).
7. **Redaction is best-effort.** The documents state this and avoid promising completeness (OD-24).
8. **Sans-IO is not a sandbox.** Hostile in-process code is out of scope for that control. Decision S defers separate-process containment and keeps the interface serializable so it can be added later (OD-15 covers the remaining tooling).
9. **Third-party passive sources** are an information-disclosure channel not yet governed (OD-16).
10. **Governance placeholders.** The vulnerability reporting channel (OD-01) and the copyright holder statement (OD-02) are undefined, so `SECURITY.md` cannot yet accept reports.
11. **The threat model is initial.** It maps threats to requirements but has not been reviewed independently, and several mitigations rely on mechanisms not yet designed.
