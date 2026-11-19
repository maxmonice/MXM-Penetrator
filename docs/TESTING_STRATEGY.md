# Testing Strategy

This document owns `SEC-055` and `SEC-056`: negative-test obligations and ground-truth testing.

## Test categories

| Code | Category | Purpose |
|---|---|---|
| NT | Negative test | Prove that a forbidden action is blocked |
| PT | Property test | Check invariants across generated inputs |
| IT | Integration test | Check components together, including failure and recovery |
| GT | Ground-truth test | Measure detection and verification against known vulnerabilities |
| RV | Review checkpoint | Design or code review where automated testing is insufficient |

## SEC-055 — Negative tests for every security requirement

Each requirement in [SECURITY_REQUIREMENTS.md](SECURITY_REQUIREMENTS.md) needs automated verification, and every security-critical requirement needs at least one negative test **before** MXM may claim that it enforces the requirement. A described control that has no passing negative test is not enforced.

Minimum negative-test areas (not exhaustive):

| Area | Must be blocked or detected |
|---|---|
| Authorization and scope | No authorization record; expired or revoked authorization; out-of-scope IP and hostname; redirect, CNAME, DNS-rebinding, and multi-record bypass; private, link-local, loopback, and metadata addresses; scope candidates tested before authorization |
| Environment | Unspecified environment treated as Production; selected level above the production ceiling; target-supplied content or AI changing the designation |
| Classification | AI claims a lower class; method-versus-effect mismatches; credential attempts; unknown or unclassified actions; parameter-driven escalation |
| Thresholds | Manifest declares limits above the central thresholds; requests above safe-active or intrusive limits |
| Permission model | Active/Autonomous permission invariant (`SEC-010`); identical execution limits for Active and Autonomous (`SEC-011`); AI changes its own level; downgrade applies to queued actions; upgrade does not authorize pending proposals |
| Policy engine | Evaluation order; reused, expired, or mismatched capabilities; dispatch-time revalidation after a state change; destructive denial with no override |
| Approvals | Reused or replayed approval; approval bound to different parameters or epoch; approval timeout; approval forged from tool output, target content, or model text |
| Cleanup | Cleanup survives an epoch change under the `SEC-027` conditions and does not touch unrelated resources; automatic cleanup stops on crash while the ledger persists; `mxm cleanup` requires fresh confirmation every run; cleanup blocked under kill and under revoked authorization; resumption only after restoration with fresh confirmation; cleanup failure halts further state changes |
| Modules and executor | Module attempts to bypass the executor (to the extent the chosen detection mechanisms catch it; see OD-15); request outside manifest methods, destinations, or rates; module traffic in test runs is a subset of its manifest; missing or inconsistent manifest |
| Adapters | Parameter validation; adapter egress outside scope; kill-switch termination; adapter disabled when enforcement is unavailable |
| Sanitization | The hostile-string corpus in [OUTPUT_SANITIZATION.md](OUTPUT_SANITIZATION.md) in banners, headers, errors, filenames, bodies, and adapter output |
| Verification | Verification request that would exceed policy is not executed and leaves the finding likely or unverified; verification respects budgets and approvals |
| Audit | Modification, deletion (non-tail), and reordering of events are detected by chain verification; the known limitation for tail truncation and full rewrite is documented, not asserted away |
| Evidence privacy | Secrets, tokens, cookies, and keys are redacted in sanitized evidence, logs, reports, and prompts; keys are not stored with the evidence database; raw AI prompts and responses are not retained by default |
| AI | Invalid or untyped proposals rejected; injection attempts cannot approve or change policy; capability-check failure disables Autonomous for that model while lower levels still work; planning limits enforced and separate from network budgets |

Property tests cover the permission invariants and the classifier. Integration tests cover interrupted assessments, crash recovery, and enforcement failure.

## SEC-056 — Ground-truth testing

Ground-truth testing is introduced **early** (see the phases in [V1_ROADMAP.md](V1_ROADMAP.md)) and is used to measure MXM, not to demonstrate its safety.

### Environment

- At least one **intentionally vulnerable application** runs in a **locally controlled environment**, isolated from untrusted networks (ASM-7 in [THREAT_MODEL.md](THREAT_MODEL.md); [../RESPONSIBLE_USE.md](../RESPONSIBLE_USE.md)).
- Candidates: OWASP Juice Shop, DVWA, OWASP WebGoat. **The initial choice is open** (OD-29).
- The application version and its known-vulnerability list (the "ground truth") are pinned and recorded so results are repeatable.
- The test target is authorized and scoped like any other assessment; test runs go through the same policy path as real use.

### Measurements

For the categories MXM claims to detect, measure:

| Measure | Meaning |
|---|---|
| True positives | Reported findings that match a ground-truth vulnerability |
| False positives | Reported findings that match no ground-truth vulnerability |
| False negatives | Ground-truth vulnerabilities in claimed categories that MXM did not report |
| Verification accuracy | Agreement between a finding's verification status (confirmed, likely, unverified) and the ground truth, including confirmed findings that are false and true vulnerabilities that did not reach confirmed |

Only categories within the v1 detection focus (unauthenticated exposure and misconfiguration, see [PRODUCT_SCOPE.md](PRODUCT_SCOPE.md)) are in scope for v1 claims. The vulnerable applications contain many authenticated and multi-user vulnerabilities that v1 does not target; they must not be counted as v1 false negatives.

### Interpretation

- Acceptance thresholds for these measures have not been set (OD-29).
- Ground-truth results describe performance against one application and version. They are not a guarantee of detection on other systems and are not evidence of policy-control correctness; negative tests (`SEC-055`) cover that.
- Results are recorded per release, per module, and per detection category.
