# Network Enforcement

This document owns `SEC-035` and `SEC-036`: OS and network-level enforcement, and isolation of external adapter processes.

## Objective

Assessment traffic must pass through an enforcement boundary capable of restricting destinations, ports, rates, concurrency, time, and execution budgets. Enforcement is cross-platform in architecture but uses platform-specific mechanisms, and **guarantees are documented per platform; they are not assumed to be identical.**

## SEC-035 — OS/network-level enforcement; fail closed when unavailable

- Network-level enforcement supplements, and does not replace, application-level checks by the policy engine and executor (`SEC-016`, `SEC-033`).
- No internal module may use an unmediated network path for assessment traffic (`SEC-032`, `SEC-033`).
- External tools are restricted through adapter controls and OS, process, or container isolation (`SEC-036`).
- Destinations are resolved and validated at execution time; redirects and DNS resolution changes are revalidated (`SEC-002`).
- Request, connection, time, concurrency, and per-class budgets are enforced (`SEC-015`, `SEC-023`).
- Enforcement decisions and blocked attempts are logged (`SEC-043`).
- **Fail closed.** If a required control cannot be enforced on the current platform, the dependent action does not run, and an adapter that needs the control is disabled (`SEC-019`).

### Honest guarantees

The network layer can generally enforce destination, port, connection, time, and volume constraints when all traffic is mediated. It may not understand application semantics for every protocol. HTTP method semantics alone do not determine whether an operation changes state, and encrypted traffic is not inspectable by a layer that does not terminate it. Action classification therefore also depends on trusted module manifests, concrete parameters, and application-level knowledge (`SEC-014`, `SEC-034`).

Because Sans-IO modules share the MXM process with the executor, OS-level controls apply to the process as a whole, not to individual modules (see [MODULE_ARCHITECTURE.md](MODULE_ARCHITECTURE.md)).

## SEC-036 — External adapter process isolation

External adapters run tools with their own network stacks and parsers. Sans-IO does not apply to them.

- Each adapter runs as a separate process under the strongest practical OS, process, or container controls.
- Its network egress is constrained to what the assessment permits.
- The adapter must not be able to reach the trusted control channel (`SEC-018`).
- The adapter must be terminable by the kill switch (`SEC-022`).
- If isolation or egress constraints required for the adapter cannot be guaranteed on a platform, the adapter is disabled there.

Whether adapters run in containers, restricted processes, or both is open (OD-19).

## Platform notes

The mechanisms below are **candidates to investigate and validate**. None is a commitment, and no guarantee is claimed until it is demonstrated against bypass tests.

### Windows

Windows is a first-class development and target platform. The implementation must not assume Linux network namespaces exist.

Investigate and validate:

- Windows Firewall rules and their lifecycle.
- Restricted process execution and Windows user/token boundaries.
- Container-based isolation where supported and configured.
- A local proxy or mediated network client for supported protocols.
- Process-tree termination for the kill switch.
- How Nmap and OWASP ZAP traffic can be constrained without creating a bypass.

Windows Firewall alone may not provide sufficient per-assessment or per-process control for every tool and protocol. The selected design must be demonstrated with tests that attempt to bypass the intended restrictions. Open: OD-17.

### Linux

The architecture does not assume Linux and Windows provide the same guarantees. Candidates include OS-level network isolation, per-user or per-process egress rules, and containers. The specific mechanism, and what it can and cannot guarantee for each adapter, is open (OD-18).

### macOS

Support, if any, depends on the same investigation (OD-18). It is not a v1 commitment in this documentation.

## Common enforcement interface

The architecture exposes a common enforcement interface with platform-specific implementations. Documented guarantees and supported adapter features are stated **separately for each platform**; see [INTEGRATIONS.md](INTEGRATIONS.md).

## Required verification

Before v1 release, test:

- Out-of-scope IP and hostname blocking.
- Redirect to out-of-scope destination.
- DNS changes and rebinding scenarios.
- IPv4 and IPv6 behavior.
- Loopback, private, link-local, and metadata address restrictions.
- Port and rate limits.
- Process and adapter bypass attempts.
- Kill switch termination of adapter processes and in-flight connections.
- Enforcement failure and recovery behavior (fail closed).
