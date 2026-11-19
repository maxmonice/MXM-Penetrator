# Product Scope

## Product vision

MXM Penetrator is a unified security assessment ecosystem consisting of:

1. An independent reconnaissance and security testing engine.
2. An AI orchestration layer for analysis, planning, and explanation.
3. An extensible integration layer for vetted external tools and future modules.

The product should be useful as a standalone tool. It must not become only a thin AI wrapper around third-party scanners.

## Primary audience

Individual security researchers and developers conducting assessments on systems they own or are explicitly authorized to test. See also [../RESPONSIBLE_USE.md](../RESPONSIBLE_USE.md).

## v1 goals

- CLI-first user experience.
- Explicit authorization and scope definition.
- Deterministic policy enforcement independent of AI output.
- Internal reconnaissance and testing capabilities built as Sans-IO modules behind a centralized executor.
- **A primary focus on unauthenticated exposure and misconfiguration detection** (see "v1 detection focus" below).
- AI-assisted interpretation and bounded investigation planning.
- Three fixed permission profiles: Guided, Active, Autonomous.
- Evidence-backed findings with reproducible verification steps.
- Local-first model integration; cloud providers require explicit opt-in.
- Optional Nmap and OWASP ZAP integrations through controlled adapters, beginning with TCP connect scanning and passive scanning respectively.
- Audit logs, execution budgets, cleanup tracking, and safe failure behavior.
- An early, working end-to-end workflow (the thin vertical slice defined in [V1_ROADMAP.md](V1_ROADMAP.md)) and early ground-truth measurement ([TESTING_STRATEGY.md](TESTING_STRATEGY.md)).

## v1 detection focus

v1 concentrates on **unauthenticated exposure and misconfiguration detection**. This is a prioritization, not a statement that other capabilities are unwanted.

**Deferred from v1 (retained in the long-term roadmap):**

- Complex authenticated workflows.
- Multi-user authorization testing.
- Cross-account privilege escalation testing.
- Complex session-based security testing.

These are deferred because they depend on credential handling, multiple user contexts, session management, and state-changing test cases whose controls are not yet designed. They must not be presented as v1 capabilities. They are not permanently removed.

Note that action classification still defines how credential and login attempts are treated (`SEC-014` in [ACTION_CLASSIFICATION.md](ACTION_CLASSIFICATION.md)). Whether v1 ships any credential or login testing module is an open decision (OD-13).

## v1 non-goals

The following are not required for the first release:

- Web dashboard or graphical control panel.
- Public REST API.
- An MXM MCP server.
- Distributed or multi-agent scanning across machines.
- General-purpose dynamic plugin marketplace or unrestricted plugin loader.
- Arbitrary shell-command execution requested by an AI model.
- Multi-user approval workflows.
- Custom user-defined permission profiles.
- Autonomous destructive testing.
- Automatic expansion of authorized scope.
- Complex authenticated, multi-user, or cross-account testing (deferred; see above).
- Guaranteed semantic understanding of every protocol by the network enforcement layer.
- Identical external-adapter functionality on every platform.

## Product differentiators

The long-term differentiators are:

- Intelligent model routing.
- Independent testing engine.
- AI-assisted security reasoning.
- Unified orchestration.
- Evidence-driven verification.
- Local-first privacy with controlled cloud integration.
- Modular extensibility.
- Strong CLI and developer experience.
- Assessment intelligence informed by prior assessment results.

Not every differentiator must be fully realized in v1. The v1 release should establish safe, stable foundations for them. Assessment intelligence in particular is a long-term capability; v1 does not require it, and any use of historical assessment data will need its own privacy and redaction design (it must not weaken `SEC-045` to `SEC-048`).

## Release standard

v1 means the first usable release, not a disposable prototype. A feature is not complete until it has implementation, automated tests, documented limitations, and observable behavior. Security-critical requirements need negative tests proving that forbidden actions are blocked (`SEC-055`).

## Related documents

- Architecture and lifecycle: [ARCHITECTURE.md](ARCHITECTURE.md)
- Delivery sequence: [V1_ROADMAP.md](V1_ROADMAP.md)
- Unresolved items: [OPEN_DECISIONS.md](OPEN_DECISIONS.md)
- Document ownership: [DOCUMENT_OWNERSHIP.md](DOCUMENT_OWNERSHIP.md)
