# Audit and Evidence

This document owns `SEC-043` to `SEC-049`: audit events, the hash-chained audit log, evidence redaction, encryption and key separation, evidence tiers, AI replay retention, and secrets in logs and evidence.

## SEC-043 — Security-relevant audit events

The audit trail records security-relevant decisions and events, including:

- Assessment creation and authorization reference, and authorization lifecycle changes (`SEC-006`).
- Scope and scope-candidate discoveries and changes.
- Permission level and every level change.
- Policy epoch changes (`SEC-021`).
- Action proposals and deterministic classifications.
- Policy decisions and reason codes.
- Approval requests, approvers, hashes, expiry, and outcomes.
- Dispatch and execution results.
- Budget and rate-limit events.
- Network enforcement blocks.
- Cleanup ledger changes, every cleanup attempt (automatic or by `mxm cleanup`), and cleanup failures.
- Evidence capture and finding verification, including verification blocked by policy (`SEC-041`).
- AI model validation results (`SEC-054`) and cloud-transmission events (`SEC-052`).
- Pause, resume, kill, and completion events.

Each action event identifies the target, module, action class, policy epoch, selected and effective level, decision, reason code, and approval reference where relevant.

## SEC-044 — Hash-chained, tamper-evident audit log

- Each audit event includes **the hash of the previous event** and **its own calculated hash**.
- Verifying the chain detects **modification, deletion, and reordering** of events within the chain.
- The audit log is **tamper-evident, not tamper-proof.** A sufficiently privileged attacker can rewrite the entire chain and recompute every hash. The chain does not prevent tampering; it makes undetected partial tampering difficult.
- Deleting the most recent events leaves a valid chain. Such truncation is detectable only by comparing against a reference hash held elsewhere; the chain alone cannot show it.
- Audit events must not contain plaintext secrets (`SEC-049`).

The hash function, the canonical event serialization, the verification procedure, and whether to anchor the latest hash outside the local store are open (OD-23).

## Evidence

Evidence is:

- Linked to the assessment, target, module, and action.
- Timestamped with a consistent time representation.
- Stored with provenance and integrity metadata.
- Separated from mutable assessment state.
- Reproducible where practical.

Evidence content is externally supplied and untrusted. It is never rendered directly; rendering goes through the Output Sanitizer (`SEC-039`).

## SEC-045 — Evidence is redacted by default

- Evidence is redacted by default before it is stored in the sanitized tier, written to logs, included in reports, or sent to a model (`SEC-052`).
- Redaction covers, where feasible: common secrets, credentials, session cookies, authentication tokens, API keys, and personal information.
- Redaction is **best-effort and cannot guarantee** that no sensitive data remains. Documentation and reports must not claim otherwise.

The rule set, the definition of personal information, and the review process are open (OD-24).

## SEC-046 — Encryption at rest and key separation

- Sensitive locally stored evidence is **encrypted at rest**. Restricted raw evidence (`SEC-047`) is always treated as sensitive.
- **Encryption keys are managed separately from the evidence database.** A copy of the evidence store alone must not be sufficient to decrypt it.
- Key and secret handling must not place keys in plaintext configuration, logs, or the evidence database (`SEC-049`).

The encryption scheme, key storage and rotation, and the secret-management approach are open (OD-25). Local confidentiality does not survive a compromised user account (ASM-2 in [THREAT_MODEL.md](THREAT_MODEL.md)).

## SEC-047 — Sanitized evidence versus restricted raw evidence

Two tiers are distinguished:

| Tier | Purpose | Rules |
|---|---|---|
| **Sanitized evidence** | Used for findings, reports, and model context | Redacted (`SEC-045`); rendered only through the Output Sanitizer (`SEC-039`). |
| **Restricted raw evidence** | Retained only when genuinely necessary (for example when verification or reproduction cannot be supported by the redacted form) | Encrypted (`SEC-046`); access restricted; never copied into reports, logs, or prompts by default. |

The criteria for "genuinely necessary", and how raw access is restricted, are open (OD-24). Retention periods and deletion are open (OD-28).

## SEC-048 — AI replay retention defaults

**Retained by default:**

- Model and provider identifiers and versions.
- Prompt-template versions.
- Relevant model configuration.
- Prompt and response **hashes**.
- Structured AI proposals and decisions.
- Schema, module, and policy versions in force, and relevant planner decisions.

**Not retained by default:** complete raw prompts and responses.

**Raw retention** is available only if the user explicitly enables it. When enabled, redaction (`SEC-045`), encryption (`SEC-046`), and restricted access are mandatory.

Hashes and metadata support **auditing**. They do **not** independently reproduce an original model response: model output may not be reproducible from the same inputs, and a hash proves what was exchanged, not what to regenerate.

## SEC-049 — Secrets are never stored in plaintext logs or evidence

Credentials, session tokens, API keys (cloud model providers, adapter control keys), and other secrets must not appear in plaintext in evidence, audit events, ordinary logs, prompts retained under `SEC-048`, or reports. Secret management for MXM's own secrets is a mandatory control. The approach is open (OD-25).

## Findings

A finding includes:

- Stable finding identifier.
- Affected asset and endpoint.
- Severity and confidence.
- Description and impact.
- Evidence references (sanitized tier unless a documented need exists).
- Reproduction steps that remain within authorized scope.
- **Verification status** as defined in [FINDING_VERIFICATION.md](FINDING_VERIFICATION.md).
- Remediation guidance.
- First-seen and last-seen timestamps.

AI-generated assertions are not sufficient proof. Reports distinguish confirmed findings, likely findings, unverified hypotheses, and inconclusive results.

## Output formats

- JSON is the canonical machine-readable output. It is data, and consumers must treat string values derived from targets as untrusted.
- Markdown and HTML reports are rendered from the same normalized report model through the Output Sanitizer (`SEC-039`).

## Replayability

Replayability means being able to explain and audit assessment decisions: which model, template, configuration, schemas, modules, and policy version were involved, and what structured proposals and decisions resulted. It is delivered by `SEC-043` and `SEC-048`. It avoids storing secrets or unnecessary raw prompts and is not a promise of exact model-response reproduction.
