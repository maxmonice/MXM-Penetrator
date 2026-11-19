# Security Policy

**Status:** pre-release documentation baseline. MXM Penetrator has no released versions and no implementation that claims to enforce the controls described in this repository. This policy describes how security reports about MXM itself will be handled once code exists, and what counts as a security issue in MXM.

This document owns the **vulnerability-reporting and disclosure process** only. Security requirements for the product are owned by the documents listed in [docs/DOCUMENT_OWNERSHIP.md](docs/DOCUMENT_OWNERSHIP.md) and indexed in [docs/SECURITY_REQUIREMENTS.md](docs/SECURITY_REQUIREMENTS.md).

## Reporting a vulnerability

The private reporting channel (contact address or hosted private-advisory mechanism) has **not been established yet**. It must be defined and published here before the first public release. See [docs/OPEN_DECISIONS.md](docs/OPEN_DECISIONS.md), item **OD-01**.

Until a channel exists, do not publish exploit details for unreleased or pre-release code in public issue trackers.

When a channel is defined, a useful report contains:

- The affected component, document, or requirement ID (for example `SEC-016`).
- The version or commit.
- Steps to reproduce within a controlled, authorized environment.
- The observed impact and the expected behavior.
- Whether the issue depends on a compromised local environment (see the residual-risk statements in [docs/THREAT_MODEL.md](docs/THREAT_MODEL.md)).

## What counts as a security issue in MXM

Examples of issues in scope:

- Any way for an action to reach the network without passing the centralized executor and the policy engine.
- Bypass or weakening of scope, authorization, permission-level, approval, budget, or kill-switch enforcement.
- Execution of actions that exceed a module manifest.
- Incorrect action classification that lowers the required approval.
- Prompt injection, terminal-control-sequence injection, or HTML/script injection that reaches a trusted prompt, the control channel, or a rendered report.
- Disclosure of unredacted secrets or personal data through logs, evidence, reports, prompts, or cloud model requests.
- Defects that defeat audit hash-chain verification or hide a modification, deletion, or reordering from the verifier.
- Cleanup behavior that can affect pre-existing or unrelated target resources.
- Escape from an external adapter's intended isolation.

## What is out of scope

- Vulnerabilities in targets that MXM discovers during an authorized assessment. Handle those under the assessment's own disclosure arrangements.
- Misuse of MXM against systems the user is not authorized to test. See [RESPONSIBLE_USE.md](RESPONSIBLE_USE.md).
- Attacks that require a privileged attacker on the user's host rewriting local state. The audit log is **tamper-evident, not tamper-proof**; see `SEC-044` in [docs/AUDIT_AND_EVIDENCE.md](docs/AUDIT_AND_EVIDENCE.md) and the residual risks in the threat model.
- Malicious in-process Python code. Sans-IO modules reduce ordinary bypass risk but do not sandbox hostile code; see `SEC-032` in [docs/MODULE_ARCHITECTURE.md](docs/MODULE_ARCHITECTURE.md).

## Handling process (intended)

The intended process is: acknowledge the report, reproduce and assess it against the requirement it affects, fix it with a negative test that fails before the fix, and disclose coordinated with the reporter. Response-time commitments are deliberately not stated here because no maintainer capacity or timeline has been established (**OD-01**).

## Supported versions

None. A supported-version matrix will be published with the first release, together with the per-platform limitations required by `docs/V1_ROADMAP.md`.
