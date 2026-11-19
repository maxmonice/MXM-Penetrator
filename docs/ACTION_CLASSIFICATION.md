# Action Classification

This document owns `SEC-013` to `SEC-015`: action classes, classification rules, and the central numeric thresholds. Which class may run at which permission level is owned by [PERMISSION_MODEL.md](PERMISSION_MODEL.md). The order in which classification feeds policy evaluation is owned by `SEC-016` in [POLICY_ENGINE.md](POLICY_ENGINE.md). Module manifests are owned by `SEC-034` in [MODULE_ARCHITECTURE.md](MODULE_ARCHITECTURE.md).

## Classes

### 1. Passive

The operation sends no traffic to the target or target-owned infrastructure. Examples include analysis of existing local evidence, public certificate-transparency data, passive DNS sources, and public archives.

A third-party service that actively probes the target on MXM's behalf is not passive, because it generates traffic to the target. Governance of third-party data sources, including what target identifiers are disclosed to them, is open (OD-16).

### 2. Safe-active

The operation interacts with the target using ordinary client behavior, at ordinary volume, without crafted inputs or authentication attempts, and without an intended state change. Examples may include a basic HTTP fetch, TLS handshake, direct DNS query, or limited banner check. "Ordinary volume" is defined numerically by the thresholds in `SEC-015`.

### 3. Intrusive

The operation uses crafted or malformed inputs, elevated request volume, or techniques likely to create meaningful availability, alerting, or operational impact, but has no intended persistent state change. Examples may include injection probes, directory brute force, port sweeps, and fuzzing. "Elevated request volume" is defined numerically by `SEC-015`.

### 4. State-changing

The operation may create, modify, delete, submit, or otherwise change target or third-party state. Examples include registration, uploads, form submissions, password-reset triggers, email/SMS/webhook side effects, and authentication attempts that can affect counters, alerts, sessions, or lockout state.

State-changing operations require approval of a complete test-case bundle at every permission level (`SEC-024`).

### 5. Destructive

The operation is intended or reasonably likely to cause irreversible data loss, deliberate denial of service, significant resource exhaustion, or disruption to real users. Examples include deleting pre-existing data, intentionally locking out real users, or high-volume credential attempts likely to cause account lockouts.

Destructive actions are hard-denied in v1 with no user override (`SEC-020`). The deterministic definition of "high-volume" and "likely to cause lockouts" for credential attempts is open (OD-13).

## SEC-013 — Deterministic classification; AI never assigns the class

- Every proposed operation receives a deterministic classification from the Action Classifier before policy evaluation.
- The classifier works from the module manifest, the concrete parameters, and the execution context.
- AI models never assign, override, or lower the authoritative class. The AI may provide context, which the classifier may ignore.
- Unknown or ambiguous behavior is assigned the more restrictive plausible class.
- Unclassified actions are denied.
- When more than one class could apply, the **most restrictive applicable class wins**. The relation among classes is not a simple ladder (for example, fuzzing is intrusive but non-mutating; a single account-creation request is state-changing but low-volume), so classification considers effect and volume independently and takes the most restrictive result.

## SEC-014 — Classify by effect; credential/login testing is state-changing

- Classify by actual effect and concrete parameters, not by tool name, technique label, or HTTP method alone.
- An HTTP GET can be state-changing if the application performs a mutation. A GraphQL POST can be read-only. A module may only claim a method is read-only through its manifest (`SEC-034`); otherwise the more restrictive class applies.
- Failed login attempts may alter counters or trigger lockouts; **credential and login testing is state-changing**.
- High-volume credential attempts that risk locking out real users or causing substantial disruption are **destructive**.
- Third-party side effects (email, SMS, webhooks) are at least state-changing.
- A parameter-dependent change (for example, a larger target list or higher volume) can escalate an action to a more restrictive class.

## SEC-015 — Central numeric thresholds for safe-active and intrusive limits

Safe-active and intrusive testing have explicit numeric thresholds. They are:

1. **Defined in this document only.** No other document, module manifest, adapter profile, or planner configuration defines or overrides them. A manifest may only declare limits at or below the central thresholds.
2. **Enforced centrally** by the Action Classifier, the Budget / Rate Limiter, and the executor (`SEC-023`, `SEC-033`), not by modules.
3. **Per class, not per level.** Permission levels cannot change them, which is how Active and Autonomous remain identical (`SEC-011`).

**The numeric values have not been designed. They are not specified here. Each requires design and validation before implementation** (see OD-03). In particular, none of the values below should be inferred from tool defaults or examples.

| Parameter | Applies to | Value |
|---|---|---|
| Maximum request rate | Safe-active | *Requires design and validation* |
| Maximum request rate | Intrusive | *Requires design and validation* |
| Maximum concurrency | Safe-active | *Requires design and validation* |
| Maximum concurrency | Intrusive | *Requires design and validation* |
| Per-test request budget | Safe-active | *Requires design and validation* |
| Per-test request budget | Intrusive | *Requires design and validation* |
| Assessment-wide request budget | Safe-active | *Requires design and validation* |
| Assessment-wide request budget | Intrusive | *Requires design and validation* |

Passive actions send no traffic to the target and have no target-side thresholds. Limits on third-party source queries are covered by OD-16.

Open design questions for these parameters, all tracked in OD-03:

- The granularity of each limit (per host, per target, per assessment, per port).
- Whether additional parameters (timeouts, payload size, maximum response size) belong in this table. Response-size handling is OD-22.
- How a request that exceeds the **intrusive** ceiling is treated. It is not executable in v1 under either interpretation (denied as out-of-bounds, or classified destructive).

A request that exceeds the safe-active thresholds is no longer safe-active; it is at least intrusive.

## Tests

The classifier and threshold enforcement require unit and property tests covering:

- Method-versus-effect mismatches.
- Authentication attempts and lockout side effects.
- Third-party side effects.
- Ambiguous and unknown actions.
- Parameter-dependent escalation to a more restrictive class.
- Attempts by AI output to claim a lower action class.
- A manifest declaring limits above the central thresholds is rejected.
- Active and Autonomous receive identical thresholds for identical actions (`SEC-011`).
