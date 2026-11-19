# Module Architecture

This document owns `SEC-032` to `SEC-034`: the Sans-IO interface for internal modules, the centralized executor, and module manifests.

It applies to **internal MXM modules**: reconnaissance modules, vulnerability test modules, and finding-verification routines. External tools (Nmap, OWASP ZAP) are not Sans-IO modules; they are adapters governed by [INTEGRATIONS.md](INTEGRATIONS.md) and [NETWORK_ENFORCEMENT.md](NETWORK_ENFORCEMENT.md).

## The Sans-IO model

A Sans-IO module contains protocol and test logic but no I/O. It interacts with the outside world only through typed data exchanged with the executor.

```text
Orchestrator ──inputs──▶ Module ──typed request plan──▶ Executor
                            ▲                              │
                            └──────typed response──────────┘
Module ──observations / evidence candidates / hypotheses──▶ Orchestrator
```

Conceptually, a module is a state machine:

1. It is initialized with typed inputs (a target reference, prior evidence references, parameters).
2. It emits one or more **typed request plans**. A plan describes what is to be requested (protocol, destination reference, method or operation, parameters, purpose, expected evidence). It is data, not an action.
3. The executor decides whether and how to perform each request (`SEC-033`) and returns a **typed response** (or a typed refusal or error).
4. The module processes the response and either emits further plans or finishes with observations, evidence candidates, and hypotheses.

Because modules are pure logic over typed inputs, they should be testable with recorded responses and without network access. This is a design intent that supports testing and replay, not a security guarantee.

The Sans-IO model fits the staged asyncio pipeline ([ARCHITECTURE.md](ARCHITECTURE.md)): the executor owns concurrency, and modules do not need to be asynchronous.

## SEC-032 — Sans-IO internal modules

All internal MXM modules must use the Sans-IO interface.

- Modules **do not own sockets** and do not perform network I/O, directly or indirectly. This includes opening connections, resolving DNS, and invoking other programs for network activity.
- The module interface provides **no network handle, client object, or credential** that could be used to reach the network.
- Modules receive responses only from the executor.
- Modules emit typed request plans and process executor-provided responses.
- Finding-verification routines follow the same interface (`SEC-041`).

### What Sans-IO does and does not provide

Sans-IO **reduces ordinary module bypass risk**: a module written to the interface has no supported way to reach the network except through the executor, which makes accidental bypass structurally unlikely and simplifies review.

Sans-IO **does not sandbox malicious Python code**. A module is Python code running in the MXM process. Code that is deliberately hostile (or a compromised dependency) can import network libraries, open sockets, read local files, or tamper with in-process state regardless of the interface. Therefore:

- Sans-IO is not a security boundary against hostile code. Review, dependency control, and trust in in-tree modules remain assumptions (see ASM-3 and ASM-4 in [THREAT_MODEL.md](THREAT_MODEL.md)).
- OS-level enforcement (`SEC-035`) operates on processes. If modules and the executor share a process, OS controls cannot distinguish module traffic from executor traffic.
- **External adapters are different**: they run tools MXM does not control and therefore require OS or process-level isolation (`SEC-036`).

### Detecting violations (decision S)

- **v1 detects interface violations with code review plus automated checks.** The checks flag network and subprocess use in module code (for example through import restrictions or lint rules), and module tests run with no network access. These catch **accidental** violations. They do not stop deliberately hostile code.
- **Modules and the executor share a process in v1.** Separate-process containment is deferred, not rejected.
- **The interface must stay separable.** Request plans and responses are plain, serializable typed data, and a module must not depend on shared in-process objects with the executor. This keeps a later move to a separate module process possible without rewriting module logic.

The exact tooling, the banned-import and banned-call rule set, and the policy for module file-system access remain open (OD-15).

## SEC-033 — A single centralized executor

- A centralized executor performs **all** network operations for internal modules and verification routines. There is no alternate path.
- The executor validates every request against: the module manifest, scope, authorization, the policy decision and capability, permission level, and budgets and rate limits. The evaluation order is owned by `SEC-016` in [POLICY_ENGINE.md](POLICY_ENGINE.md).
- The executor owns DNS resolution, connection establishment, and redirect following. A redirect is a new request and is validated before connection, including its resolved destination (`SEC-002`).
- Each concrete request is checked against the capability that authorized its action (`SEC-017`). An action's capability does not authorize additional requests beyond its constraints.
- The executor enforces concurrency, rate, and budget limits centrally (`SEC-015`, `SEC-023`). Modules cannot raise them.
- Responses returned to modules are untrusted data. Anything later rendered goes through the Output Sanitizer (`SEC-039`).
- The executor emits audit events for dispatches, blocks, and results (`SEC-043`).
- Limits on response size and decompression are not yet specified (OD-22).

## SEC-034 — Module manifests: required and enforced at runtime

Every module declares a **versioned manifest**. A manifest contains:

- Module identifier and version.
- Declared capabilities and purpose.
- Supported protocols.
- **Allowed methods or operations.**
- **Allowed destinations**, expressed as target-scope-bound destinations or explicitly declared third-party passive sources.
- **Declared request rates**, which may not exceed the central thresholds in `SEC-015`.
- Expected side effects.
- Parameter constraints.
- Possible action classes.

Rules:

- A manifest is **input to validation, not permission to execute.** The classifier evaluates it together with concrete parameters and context (`SEC-013`).
- Manifests are **enforced at runtime** by the executor: a request that uses a method, destination, or rate outside the manifest is rejected, whatever the module asks for.
- Unknown, missing, or inconsistent manifests fail closed (`SEC-019`).
- A manifest cannot lower an action's class below what its concrete effect warrants (`SEC-014`). A claim that a method is read-only is a manifest declaration that is reviewed and tested; it is not self-certifying.
- Manifest changes are versioned and reviewed like code.
- A conformance test confirms that the traffic a module generates in test runs is a subset of what its manifest allows (`SEC-055`).

## Relationship to other components

| Topic | Authoritative document |
|---|---|
| Class definitions and numeric thresholds | [ACTION_CLASSIFICATION.md](ACTION_CLASSIFICATION.md) |
| Evaluation order and capabilities | [POLICY_ENGINE.md](POLICY_ENGINE.md) |
| Network-layer enforcement | [NETWORK_ENFORCEMENT.md](NETWORK_ENFORCEMENT.md) |
| Verification routines | [FINDING_VERIFICATION.md](FINDING_VERIFICATION.md) |
