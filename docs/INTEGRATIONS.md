# External Integrations

This document owns `SEC-037` and `SEC-038`: external adapter requirements and the initial Nmap and OWASP ZAP integration scope. Process and network isolation of adapters is owned by `SEC-036` in [NETWORK_ENFORCEMENT.md](NETWORK_ENFORCEMENT.md).

## v1 strategy

MXM's independent engine remains the primary assessment engine. External tools are optional capabilities integrated through vetted, controlled adapters.

Initial integrations:

- **Nmap:** network discovery and port scanning, beginning with TCP connect scans.
- **OWASP ZAP:** web application testing, beginning with passive scanning.

Exact supported versions, invocation modes, capabilities, and supported platforms must be pinned and documented before release (OD-20, OD-21).

## SEC-037 — Adapter requirements

Every adapter must:

- Use a versioned capability manifest (see `SEC-034` for the manifest concept; adapter manifests declare operations and side effects of the tool).
- Accept typed parameters, never arbitrary AI-generated command strings.
- Validate parameters against allowlists and bounds.
- Declare possible side effects and action classes.
- Route traffic through the strongest available enforcement boundary (`SEC-035`).
- Respect scope, rate limits, timeouts, execution budgets, and the central thresholds (`SEC-015`, `SEC-023`).
- Normalize output into MXM schemas; adapter output is untrusted data (`SEC-039`).
- Preserve provenance and raw evidence where appropriate, subject to `SEC-045` to `SEC-047`.
- Return structured errors and execution status.
- Be disabled if required controls cannot be guaranteed on the platform.

Adapters must not expose arbitrary shell execution to the AI. Use fixed executable paths, validated argument construction, controlled environment variables, bounded output, and process timeouts. Adapter control interfaces (for example a tool's local API and its key) must not be reachable by other processes beyond what the adapter requires (`SEC-036`).

## SEC-038 — Initial Nmap and OWASP ZAP scope

### Nmap

- The initial integration **prioritizes TCP connect scans (`-sT`)**. Raw SYN scans are not part of the initial integration.
- Any additional scan technique requires individual classification (`SEC-013`) and passing MXM policy before it is added to the adapter manifest.
- Scan profiles carry explicit rate, port, timing, and target limits that must not exceed the central thresholds (`SEC-015`).
- Nmap traffic is outside the Sans-IO executor path; its constraints come from adapter controls and OS-level enforcement, and its guarantees are platform-specific.

### OWASP ZAP

- The initial integration begins with **passive scanning**.
- Passive and active capabilities are separately represented in the adapter manifest.
- **Active scanning** is not part of the initial passive integration. When added, each active capability must be individually classified, must pass MXM policy checks, and requires explicit approval when its class requires it (`SEC-024`). Active scanning can be intrusive and may cause state changes depending on configuration and target behavior; it is never assumed safe merely because ZAP initiated it.
- State-changing capabilities require test-case approval. Destructive capabilities are denied (`SEC-020`).
- The intended meaning of "passive scanning" is analysis of traffic that MXM's executor has already produced under policy. Any ZAP feature that originates its own requests (for example spidering) is an active capability and falls under the rules above. How passive scanning is fed is to be confirmed (OD-21).

### Platform capabilities and limitations

Capabilities are documented **per platform and are not assumed to be identical.** "To be validated" means the intended behavior has not yet been demonstrated.

| Capability | Windows | Linux |
|---|---|---|
| Nmap TCP connect scan (`-sT`) | Intended. Enforcement guarantees to be validated (OD-17, OD-20). | Intended. Enforcement guarantees to be validated (OD-18, OD-20). |
| Nmap raw SYN scans | Not in the initial integration. | Not in the initial integration. |
| ZAP passive scanning | Intended. Isolation mechanism to be validated (OD-17, OD-21). | Intended. Isolation mechanism to be validated (OD-18, OD-21). |
| ZAP active scanning | Not in the initial integration. | Not in the initial integration. |
| Kill-switch termination of the adapter process | Mechanism to be defined and tested (OD-19). | Mechanism to be defined and tested (OD-19). |
| Adapter egress restriction | Mechanism to be defined and tested (OD-17). | Mechanism to be defined and tested (OD-18). |

If enforcement required for an adapter cannot be guaranteed on a platform, the adapter is disabled on that platform and the supported-version matrix says so (`SEC-035`, `SEC-036`).

## REA and future adapters

REA may be evaluated as an optional, isolated external adapter in a later decision. It is not the foundation of MXM's own reconnaissance or vulnerability engine. REA output is treated as leads, not verified findings; follow-up testing passes through MXM's own policy and execution boundary (`SEC-041`). Any future MCP integration is an isolated optional adapter, and an MXM MCP server is deferred.

No general-purpose dynamic plugin loader is required for v1. Begin with a small in-tree module and adapter registry behind stable interfaces.
