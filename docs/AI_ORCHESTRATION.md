# AI Orchestration

This document owns `SEC-050` to `SEC-054`: typed proposals, prompt-injection containment, local-first privacy, AI planning limits, and local model capability checks. What each permission level allows is owned by [PERMISSION_MODEL.md](PERMISSION_MODEL.md). Retention of AI replay data is owned by `SEC-048` in [AUDIT_AND_EVIDENCE.md](AUDIT_AND_EVIDENCE.md).

## Role of AI

AI assists with:

- Interpreting assessment context and existing evidence.
- Generating test hypotheses.
- Selecting among registered modules.
- Ordering and prioritizing bounded investigations.
- Proposing follow-up tests.
- Summarizing evidence and explaining remediation.
- Recommending a permission profile based on natural-language intent (`SEC-012`).

AI does not:

- Execute shell commands or network requests directly.
- Assign authoritative action classes (`SEC-013`).
- Grant or change permissions, or change its own level (`SEC-009`).
- Approve state-changing operations (`SEC-024`).
- Expand scope (`SEC-003`).
- Change authorization ceilings, budgets, or limits (`SEC-023`).
- Register or invoke unapproved modules.
- Override deterministic policy decisions.
- Invent evidence or mark an unverified hypothesis as a confirmed finding (`SEC-042`).

## SEC-050 — Typed action proposals; AI output is untrusted

The model returns structured proposals conforming to versioned schemas. A proposal contains a registered module identifier, typed parameters, a target reference, a rationale, and expected evidence. Proposals are untrusted until validated; invalid proposals are rejected. Free-form model text is never interpreted as an executable command. A validated proposal becomes an action that the classifier and policy engine evaluate (`SEC-013`, `SEC-016`); the AI's rationale never substitutes for that evaluation.

## SEC-051 — Prompt-injection containment

All target-derived content, external tool output, files, and model responses are untrusted data. They may inform hypotheses but cannot issue trusted control instructions.

- The control channel remains separate from model context (`SEC-018`).
- Policy checks operate on typed data and trusted configuration, not on instructions embedded in retrieved content.
- Content sent to a model is treated as data, is subject to redaction (`SEC-045`), and is bounded by the transmission rules in `SEC-052`.
- Model-generated text never appears inside trusted prompts (`SEC-040`).
- The containment limits what an injected model can cause (it can only emit proposals that policy evaluates). It does not prevent a model from being misled into producing poor hypotheses or spending allowed budget on them (see AC-01 in [THREAT_MODEL.md](THREAT_MODEL.md)).

## SEC-052 — Local-first AI; explicit cloud opt-in

- A model gateway provides a common interface to local and cloud models.
- Local models are the default.
- Cloud use requires explicit user opt-in and clear disclosure of what data will be sent.
- Secrets, credentials, and unnecessary sensitive evidence must be redacted before any external transmission (`SEC-045`). Redaction is best-effort ("where feasible").

## Model routing

A **rules-based** model router is the initial implementation. Routing decisions may consider task type, model capability, latency, resource constraints, and user privacy settings. The router never routes data to a cloud model without the opt-in required by `SEC-052`. Which local runtimes and providers are supported at launch is open (OD-33).

## SEC-053 — Centrally enforced AI planning limits

Both Active and Autonomous planning are bounded by centrally enforced limits on:

- Maximum AI replans.
- Maximum follow-up actions.
- Maximum concurrent tasks.
- Maximum assessment duration.
- Maximum AI token consumption.

These limits are **separate from network execution budgets and rate limits** (`SEC-023`, `SEC-015`). "Concurrent tasks" here means concurrent planning or analysis tasks, not network concurrency; action counts and network concurrency are execution limits owned by `SEC-023`, and they are identical for Active and Autonomous (`SEC-011`).

The limits are enforced by the Orchestrator and the Budget / Rate Limiter, not by prompt text. When a bound is reached, the planner pauses or parks work and explains why. The planner cannot expand scope, change budgets or limits, alter its permission level, bypass approvals, or use unregistered modules.

Autonomous planning additionally requires checkpoints and decision traces so that autonomous choices are reviewable, and it is restricted to registered modules. Checkpoint frequency and the allowed-module set are configuration of the assessment, not capabilities the planner can change.

**The numeric values are not yet designed and require design and validation (OD-04).** That open item also covers how values differ between Active and Autonomous and whether Guided uses the same parameter set.

## SEC-054 — Local model capability checks before enabling Autonomous

Before Level 3 (Autonomous) is enabled, MXM validates the selected model's:

- **Structured-output compliance.**
- **Action-schema compliance.**
- **Refusal and safety-handling behavior.**

If the model fails the required checks, Autonomous mode is **disabled for that model**. Lower-permission operation and deterministic testing remain available where supported.

These checks establish **compatibility and reliability**, not proof that a model is inherently safe. A model that passes can still be misled (`SEC-051`). Check results are recorded against the model identity and configuration (`SEC-043`, `SEC-048`). The design of the check suite, pass criteria, re-validation triggers, and whether Active requires any check are open (OD-26).

## Finding verification

AI-generated findings begin as hypotheses. Verification status and the rule that verification passes through the same policy path are owned by [FINDING_VERIFICATION.md](FINDING_VERIFICATION.md).
