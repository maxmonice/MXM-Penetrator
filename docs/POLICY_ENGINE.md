# Policy Engine

This document owns `SEC-016` to `SEC-023`: evaluation order, capabilities, the trusted control channel, fail-closed behavior, the destructive hard-deny, policy epochs, the kill switch, and budgets and rate limits.

The policy engine is the deterministic authority that decides whether an action may execute, requires approval, or must be denied. It is independent of the AI planner and of external tool output.

## Decisions

The policy engine returns a structured decision:

- `ALLOW`: all policy checks pass without additional approval.
- `REQUIRE_APPROVAL`: the action is otherwise eligible but needs a valid user approval.
- `DENY`: the action is prohibited or required controls are missing.
- `PARK`: the action is not currently executable and may be reconsidered after a user or system event.

The exact schema and reason-code registry must be versioned.

## SEC-016 — Dispatch-time evaluation order and revalidation

Before evaluation, the Action Classifier assigns the action class (`SEC-013`). An action that cannot be classified is denied.

Every action is then evaluated **at dispatch time**, immediately before execution, in this order. Each layer may restrict the action; no later layer may override an earlier denial.

1. Kill/pause state (`SEC-022`).
2. Authorization record is valid, unexpired, and not revoked (`SEC-006`).
3. Target and all resolved destinations are within current scope (`SEC-002`).
4. Hard-deny rules, including destructive actions (`SEC-020`).
5. Module manifest conformance for the concrete request (`SEC-034`).
6. Authorization ceiling and the effective permission level (`SEC-007`, `SEC-005`).
7. Class rule for the effective level (matrix in [PERMISSION_MODEL.md](PERMISSION_MODEL.md)).
8. Rate limits, concurrency limits, timeouts, and execution budgets (`SEC-023`, `SEC-015`).
9. Approval validity, where required (`SEC-025`).
10. Issue a narrowly scoped execution capability and dispatch (`SEC-017`).

A planning-time decision is never sufficient. Immediately before execution MXM re-checks the current policy epoch and permission level, authorization validity and scope, resolved destination IPs and redirect targets, the action class and concrete parameters, budget and rate-limit availability, and approval token validity and binding.

This order applies to **every** action, including verification requests (`SEC-041`). It applies to cleanup operations with the specific differences stated in `SEC-027` and `SEC-031`.

## SEC-017 — Short-lived execution capabilities

After an action passes policy, the engine issues a short-lived, single-use capability bound to:

- the action hash,
- the target and resolved-destination constraints,
- the action class,
- the parameter set or its hash,
- the relevant policy epoch,
- budget and rate constraints, and
- an expiration and a single-use identifier.

The executor rejects missing, expired, reused, or mismatched capabilities, and it checks each concrete request against the capability's constraints (`SEC-033`). The lifetime value is open (OD-14).

## SEC-018 — Trusted control channel

Approvals, permission-level changes, scope and authorization changes, pause, kill, and manual cleanup must arrive through a trusted local control channel, such as the local CLI TTY or authenticated local IPC. AI output, tool output, target content, web pages, adapter output, and untrusted files cannot act as control messages.

The control channel must remain separate from model context and must not be reachable by external adapters. The concrete mechanism, and how it is protected from other processes running as the same user, is open (OD-11). How trusted prompts are rendered is owned by `SEC-040`.

## SEC-019 — Fail-closed behavior

- Unknown action: deny.
- Unclassified action: deny.
- Missing, invalid, or inconsistent manifest: deny.
- Missing or invalid authorization: deny.
- Scope ambiguity: deny or park pending user action.
- Approval timeout: deny or park; never auto-approve.
- Required network enforcement unavailable: deny (`SEC-035`).
- Budget exhausted: deny or park.
- Policy engine error: deny execution.

MXM must not silently downgrade safety checks or continue with an unbounded fallback.

## SEC-020 — Destructive actions are hard-denied

Destructive actions (defined in [ACTION_CLASSIFICATION.md](ACTION_CLASSIFICATION.md)) are denied in v1. There is no override, setting, level, approval, or natural-language instruction that enables them. The cleanup of resources created by the same approved test is not destructive in this sense; its narrow carve-out is defined once in `SEC-026`.

## SEC-021 — Policy epochs

A **policy epoch** is a version number of the effective policy state for an assessment.

- A permission-level change increments the epoch (`SEC-008`).
- Every policy decision records the epoch in force (`SEC-043`).
- Approvals and capabilities are bound to an epoch (`SEC-025`, `SEC-017`); an approval from an earlier epoch does not authorize a later one.
- **Exception:** cleanup of ledger-recorded, test-created resources survives epoch changes under the conditions in `SEC-027`.

The complete list of events that increment the epoch beyond level changes (for example scope changes, authorization changes, environment re-designation, and budget changes) is not yet decided (OD-05).

## SEC-022 — Kill switch and pause

**Kill switch.** When activated:

- Testing stops immediately.
- All network activity stops immediately, **including automatic cleanup**.
- Adapter processes are terminated according to their isolation mechanism (`SEC-036`).
- Pending actions are not dispatched.
- The kill state cannot be overridden by an approval, a level change, or manual cleanup.
- The kill event is audited (`SEC-043`).

The cleanup ledger is preserved and remaining resources are reported as specified in `SEC-031`. Requests already sent cannot be recalled; MXM does not claim otherwise.

**Pause.** Pause stops dispatch of new actions. It is separate from kill and from level changes. The treatment of in-flight actions on pause is not yet specified.

The lifecycle of the kill state (how it is cleared, whether `mxm cleanup` can run in a new process afterwards) and the in-flight semantics of pause are open (OD-06).

## SEC-023 — Budgets and rate limits

- **Execution budgets and rate limits** cover requests, concurrency, time, and per-class counts. Safe-active and intrusive numeric thresholds are owned by `SEC-015`.
- **AI budgets** (replans, follow-ups, tasks, duration, tokens) are separate and owned by `SEC-053`.
- Limits are enforced centrally by the executor and the Budget / Rate Limiter, never by modules, adapters, or the AI.
- Budget exhaustion results in deny or park, with an explanation to the user.
- Extending a budget or raising a limit is a user action through the trusted control channel and is audited. The AI cannot change budgets or limits.
- Execution limits are identical for Active and Autonomous (`SEC-011`).
- Whether cleanup operations draw from a reserved budget, so that exhaustion cannot strand test-created resources, is open (OD-08).

## Auditability

Every policy decision produces an audit event with the content defined by `SEC-043` in [AUDIT_AND_EVIDENCE.md](AUDIT_AND_EVIDENCE.md).
