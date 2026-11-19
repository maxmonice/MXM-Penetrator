# Responsible Use Statement

MXM Penetrator is a security assessment tool for individual security researchers and developers. It is intended only for systems you own or are explicitly authorized to test. This statement is guidance, not legal advice, and it does not modify the Apache License 2.0 that governs the software (see [LICENSE](LICENSE)).

## Your responsibilities

1. **Test only what you are authorized to test.** You are responsible for obtaining authorization from the system owner and for staying within its terms.
2. **MXM cannot verify authorization.** The authorization record MXM requires (`SEC-001`) is a durable statement made by you. MXM does not independently verify legal authority and must not be represented as doing so.
3. **Controls reduce accidents; they are not a legal shield.** Scope, permission, approval, and budget controls are designed to prevent mistakes and limit what an AI model can do. They do not make an unauthorized test authorized, and they do not stop a determined user from misusing a tool they control.
4. **Treat unknown environments as production.** MXM does so by default (`SEC-005`). Do not label a production system as non-production to obtain more permissive behavior.
5. **Understand the data you send.** Local models are the default. Cloud models require explicit opt-in and may receive target-derived data after redaction (`SEC-052`). Redaction is best-effort ("where feasible") and cannot be guaranteed to remove everything sensitive.
6. **Handle evidence and reports carefully.** Evidence can contain secrets and personal data even after redaction. Protect stored evidence, and review reports before sharing them (`SEC-045`, `SEC-046`, `SEC-047`).
7. **Run intentionally vulnerable test applications in isolation.** Ground-truth targets such as OWASP Juice Shop, DVWA, or OWASP WebGoat must run in a locally controlled environment and must not be exposed to untrusted networks (`SEC-056`).
8. **Follow applicable laws, contracts, and platform rules.** This includes rules of cloud providers, hosting providers, and any third-party data sources you query.

## What MXM will not do in v1

- Destructive testing. Destructive actions are hard-denied (`SEC-020`).
- Test newly discovered assets without your explicit authorization (`SEC-003`).
- Perform state-changing tests without your explicit confirmation of the complete test-case bundle (`SEC-024`).

## Disclosure of findings

Findings produced during an authorized assessment belong to your engagement. Follow the disclosure terms agreed with the system owner. Vulnerabilities in MXM itself should be reported as described in [SECURITY.md](SECURITY.md).
