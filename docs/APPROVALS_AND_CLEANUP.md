# Approvals and Cleanup

This document owns `SEC-024` to `SEC-031`: state-changing approvals, the test lifecycle and cleanup ledger, cleanup across policy epochs, automatic and manual cleanup, cleanup failure handling, and cleanup under the kill switch and authorization revocation.

Which levels need approval for which class is owned by [PERMISSION_MODEL.md](PERMISSION_MODEL.md). Kill-switch semantics are owned by `SEC-022` in [POLICY_ENGINE.md](POLICY_ENGINE.md). Authorization states are owned by `SEC-006` in [SCOPE_AND_AUTHORIZATION.md](SCOPE_AND_AUTHORIZATION.md).

## SEC-024 — State-changing test-case approval bundles

State-changing work is approved per **complete test-case bundle**, not per individual HTTP request, and is required at every permission level.

A bundle describes the whole bounded operation:

- Target and resource identifiers.
- Purpose and expected outcome.
- Planned state changes.
- Concrete parameters and constraints.
- Expected external side effects.
- Cleanup steps.
- Time and request limits.
- A residual-risk statement.

The approval prompt is generated from typed proposal data by deterministic code, not from free-form AI prose, and is rendered under `SEC-040`. A verification step that would need a state change requires its own bundle (`SEC-041`).

## SEC-025 — Approval binding, single use, expiry, and timeout

An approval must be:

- Explicitly confirmed by the user through the trusted control channel (`SEC-018`).
- Bound to the exact test-case or action-bundle hash.
- Bound to the target, the parameters, and the relevant policy epoch (`SEC-021`).
- Single-use.
- Time-limited.
- Revalidated at dispatch time (`SEC-016`).

An approval for one test does not authorize a different test, target, parameter set, or later policy epoch. Approval timeout results in deny or park, never automatic approval. Unattended execution requires a separate explicit mode and must not bypass state-changing approval requirements. The definition of that mode (OD-10) and the approval time limits (OD-14) are open.

Prompt injection in a target page, tool output, file, model response, or report must not be able to approve an operation or change policy.

## SEC-026 — Test lifecycle and write-ahead cleanup ledger

A state-changing test is one lifecycle:

1. Preconditions.
2. Approved mutation.
3. Verification.
4. Cleanup.
5. Cleanup verification.

The bundle discloses the intended cleanup. **Cleanup carve-out:** cleanup may delete or revert resources that (a) were created by that approved test and (b) are recorded in the assessment's cleanup ledger. This is a narrow carve-out. It does not authorize deletion or modification of pre-existing resources, and it does not make destructive actions permissible (`SEC-020`).

**Write-ahead ledger.** Before creating a temporary resource, MXM records enough information to identify and clean it up. The ledger contains:

- Assessment and test-case identifiers.
- Resource type and target.
- Creation request or operation reference.
- Resource identifier once known.
- Cleanup procedure and constraints (as approved).
- Status and timestamps.
- Verification evidence.

The ledger must be durable before the mutation is dispatched wherever technically possible. Ledger changes are audited (`SEC-043`).

Cleanup is defined by trusted module logic and the approved bundle. **The AI must not improvise cleanup commands.**

## SEC-027 — Cleanup survives policy epochs

Cleanup of resources created by an approved test and recorded in the cleanup ledger **survives policy epoch changes**, provided all of the following hold:

1. The resource was created by the approved test.
2. Cleanup follows the original approved plan and parameters.
3. A valid authorization basis still exists (`SEC-006`).
4. Cleanup cannot affect unrelated or pre-existing resources.

Consequently, a permission-level change, a ceiling change, or any other epoch-incrementing event does not by itself cancel or invalidate cleanup of ledger-recorded resources. Cleanup still passes the kill/pause check, authorization check, destination validation, manifest conformance, and executor enforcement of `SEC-016`; it is exempt only from epoch-bound approval and level/class gating, because its approval is part of the original bundle.

How MXM proves that a recorded resource identifier, which may have been learned from an untrusted target response, really refers to a resource the test created (condition 4) is not yet specified (OD-09). Until it is, cleanup that cannot satisfy condition 4 must not delete the resource and instead leaves it pending and reported.

If the target is removed from scope while authorization remains valid, cleanup is parked pending a decision (OD-07).

## SEC-028 — Automatic cleanup and process-termination behavior

- Normal completed tests perform **automatic cleanup** of approved test-created resources.
- If the MXM process is terminated or crashes, **automatic cleanup stops**.
- The cleanup ledger is preserved and pending resources remain recorded.
- MXM does not resume cleanup silently on restart. Pending resources are visible in the assessment status and the final report, and are cleaned only through `mxm cleanup` (`SEC-029`).

Whether automatic cleanup may resume inside a still-running process after authorization is restored is open (OD-07).

## SEC-029 — Manual cleanup: `mxm cleanup`

`mxm cleanup` is the manual command for resources recorded in the ledger as pending. It must:

1. Display the pending cleanup resources.
2. Show the exact cleanup operations and parameters, taken from the ledger's approved procedure (not newly generated), rendered under `SEC-040`.
3. Require **fresh explicit user confirmation every time** it runs. Confirmations are not carried over from a previous run or from the original approval.
4. Validate authorization and the applicable policy controls (kill state, authorization, destination validation, manifest and executor enforcement, limits).
5. Record every cleanup attempt and result in the audit log (`SEC-043`).

**Manual cleanup cannot bypass the kill switch or authorization restrictions.** It runs through the same trusted control channel, executor, and policy engine as all other actions. The AI is not involved.

## SEC-030 — Cleanup failure handling

If cleanup fails:

- Record the failure and preserve evidence.
- Retry only within bounded, deterministic limits. The limits are not yet specified (OD-08).
- Stop further state-changing actions for the affected assessment.
- Read-only analysis may continue if safe.
- Alert the user visibly.
- Report residual artifacts with manual-removal instructions.
- Do not mark the assessment cleanly complete until cleanup is verified or the user explicitly acknowledges the residual state.
- Reconciliation uses `mxm cleanup` (`SEC-029`).

## SEC-031 — Cleanup under the kill switch and authorization revocation

**Kill switch** (semantics: `SEC-022`):

- Automatic cleanup stops immediately with all other network activity.
- The cleanup ledger is preserved.
- Remaining resources are reported.
- Manual cleanup must not bypass the kill switch: `mxm cleanup` refuses to perform network operations while the kill state is in effect. How the kill state is cleared is open (OD-06).

**Authorization revocation** (states: `SEC-006`):

- Automatic **and** manual network cleanup are blocked.
- The ledger is preserved.
- Remaining resources are marked *pending cleanup, requires renewed authorization*.
- Cleanup can resume only after a valid authorization is restored.
- Resumed manual cleanup still requires fresh confirmation (`SEC-029`).

Marking resources as pending is a local, non-network ledger update and is permitted in both cases.

## Destructive actions

Destructive actions are denied (`SEC-020`). The cleanup carve-out in `SEC-026` is the only deletion MXM performs of its own accord, and it is limited to resources the approved test created.
