# Threat Model

**Status:** initial formal threat model, written before implementation. It must be reviewed whenever an architecture decision changes a trust boundary, and again before the release gate in [V1_ROADMAP.md](V1_ROADMAP.md).

This document does not define security rules. It identifies what is protected, from whom, and where controls are expected to hold or fail. Rules are defined in the owner documents (see [DOCUMENT_OWNERSHIP.md](DOCUMENT_OWNERSHIP.md)); this document maps threats to requirement IDs and records **residual risk** honestly.

## 1. System overview

MXM is a local, CLI-driven assessment tool. A user supplies an authorization record and scope. Internal **Sans-IO modules** produce typed request plans. A **centralized executor** validates each request against the policy engine, module manifest, scope, budgets, and rate limits, then performs the network operation. **External adapters** (Nmap, OWASP ZAP) run as separate processes. An **AI gateway** talks to local or (opt-in) cloud models, which return typed proposals. Results go to an evidence store, an audit log, and reports.

## 2. Assets

| ID | Asset | Why it matters |
|---|---|---|
| AST-01 | Target systems, their data, and their users, and third parties affected by side effects | Primary thing MXM must not harm |
| AST-02 | Authorization records and scope definitions | Define what MXM may touch |
| AST-03 | Policy engine, permission profiles, thresholds, and manifests | A bypass defeats every other control |
| AST-04 | Cleanup ledger | Records resources MXM created on a target |
| AST-05 | Audit log | Accountability and incident reconstruction |
| AST-06 | Evidence (raw and sanitized) | May contain secrets and personal data |
| AST-07 | Secrets and keys: cloud-provider API keys, adapter API keys, evidence-encryption keys | Compromise enables data theft or impersonation |
| AST-08 | AI prompts, responses, and model configuration | May contain sensitive target data |
| AST-09 | Assessment state (SQLite) | Integrity of plans, approvals, budgets |
| AST-10 | Reports | Shared outward; may leak data or carry injected content |
| AST-11 | The user's host and accounts | MXM runs with the user's privileges |
| AST-12 | Trustworthiness of findings | A fabricated or inflated finding harms the user's decisions |
| AST-13 | MXM source, dependencies, and updates | Supply-chain integrity |

## 3. Trust boundaries

| ID | Boundary | Trust assumption |
|---|---|---|
| TB-1 | User ↔ trusted control channel | Trusted. The only source of approvals, level changes, scope changes, pause, kill, and manual cleanup (`SEC-018`). Mechanism undecided (OD-11). |
| TB-2 | MXM core: policy engine, executor, approval manager, scope validator, audit log | Trusted computing base. Correctness is assumed; compromise of this code is outside what MXM can defend against. |
| TB-3 | Internal modules ↔ core | Semi-trusted: reviewed in-tree code that may contain bugs. Sans-IO removes the network handle from ordinary modules (`SEC-032`). It does **not** sandbox hostile Python. |
| TB-4 | AI gateway / models ↔ core | Model output is untrusted input (`SEC-050`). A local model runs on the host; a cloud provider is an external party. |
| TB-5 | External adapters ↔ core | Untrusted processes with their own network stacks and parsers (`SEC-036`). |
| TB-6 | MXM ↔ target and network | Hostile. Every byte of response is attacker-controlled. |
| TB-7 | MXM ↔ third-party data sources (passive recon sources) | Untrusted data; also an information-disclosure channel (OD-16). |
| TB-8 | MXM ↔ local storage | Same-user processes can read and write unless OS controls prevent it. |
| TB-9 | MXM ↔ renderers (terminal emulator, report viewers) | Interpret control sequences and markup; targets of injection (`SEC-039`). |

## 4. Threat actors

| ID | Actor | Capability |
|---|---|---|
| TA-1 | Malicious target operator | Controls responses, banners, redirects, DNS, error messages, filenames, and certificates |
| TA-2 | Third party who can place content on a target | Injects user-generated content that MXM later reads |
| TA-3 | On-path network attacker | Observes or alters traffic MXM sends over untrusted networks |
| TA-4 | Malicious or vulnerable dependency, adapter, or contributor code | Runs inside or beside MXM |
| TA-5 | Unprivileged malware running as the same OS user | Reads/writes MXM state, evidence, and keys unless prevented |
| TA-6 | Privileged attacker or compromised host | Can rewrite local state, including recomputing a hash chain |
| TA-7 | Cloud model provider (honest-but-curious or compromised) | Sees whatever is transmitted after opt-in |
| TA-8 | Misled or misbehaving AI model | Produces malicious, malformed, or manipulated proposals |
| TA-9 | Careless or unauthorized user | Misconfigures scope, mislabels environments, or tests systems without authority |
| TA-10 | Recipient of a report | Opens reports in unknown viewers; may receive sensitive data unintentionally |

## 5. Attack surfaces

| ID | Surface |
|---|---|
| SRF-1 | All target-derived text: responses, headers, banners, errors, filenames, certificates |
| SRF-2 | Content sent to AI models (prompts), and model responses |
| SRF-3 | Terminal output and interactive prompts (approvals, level changes, manual cleanup) |
| SRF-4 | HTML and Markdown report generation |
| SRF-5 | Module interface (request plans and manifests) |
| SRF-6 | External adapter invocation, adapter control APIs, and adapter output parsing |
| SRF-7 | DNS resolution, redirects, and IP address handling |
| SRF-8 | Local files: assessment database, evidence store, cleanup ledger, audit log, configuration, keys |
| SRF-9 | Trusted control channel implementation |
| SRF-10 | Cleanup operations that act on identifiers learned from target responses |
| SRF-11 | Third-party passive data source queries |
| SRF-12 | Dependency, packaging, and update path |

## 6. Security assumptions

| ID | Assumption | If it fails |
|---|---|---|
| ASM-1 | The user is authorized to test the declared scope. MXM cannot verify this. | Unauthorized testing; MXM's controls do not make it lawful. |
| ASM-2 | The host OS and the user account are not compromised during an assessment. | Local state, evidence, and keys can be read or altered (TA-5, TA-6). Guarantees degrade as described in residual risks. |
| ASM-3 | The Python runtime and pinned dependencies are not malicious. | In-process code can bypass any in-process control. |
| ASM-4 | In-tree modules are reviewed and not adversarial. | A hostile module is not contained by Sans-IO. |
| ASM-5 | Terminal emulators and report viewers behave according to their standards. | Sanitizer rules may miss emulator-specific behaviors. |
| ASM-6 | The user reads approval prompts. | Approval fatigue weakens the human gate. |
| ASM-7 | Ground-truth applications run isolated on a locally controlled environment. | An exposed vulnerable app becomes a liability. |
| ASM-8 | The trusted control channel is not writable by the AI, target content, or adapters. | Approval and policy changes could be forged. Depends on OD-11. |

## 7. Abuse cases

Mitigation IDs refer to requirements in [SECURITY_REQUIREMENTS.md](SECURITY_REQUIREMENTS.md). "Open" refers to an item in [OPEN_DECISIONS.md](OPEN_DECISIONS.md).

| ID | Abuse case | Actor | Assets | Mitigations | Residual risk |
|---|---|---|---|---|---|
| AC-01 | Target content contains instructions that steer the model toward out-of-scope or higher-class actions | TA-1, TA-2, TA-8 | AST-01, AST-12 | SEC-050, SEC-051, SEC-013, SEC-016, SEC-002, SEC-003 | The model can still be misled into wasting allowed budget or producing misleading hypotheses within permitted actions. |
| AC-02 | Banner, error message, or filename carries terminal control sequences that forge or hide an approval prompt | TA-1 | AST-01, AST-11 | SEC-039, SEC-040, SEC-018 | Sanitizer gaps; emulator-specific behavior (ASM-5); rule set open (OD-27). |
| AC-03 | Target content injects HTML/script into a report | TA-1, TA-10 | AST-10, AST-11 | SEC-039 | Viewers outside MXM's control; Markdown rendering differs by viewer. |
| AC-04 | Redirect, CNAME chain, or DNS rebinding sends traffic to an out-of-scope, private, link-local, or metadata address | TA-1 | AST-01, AST-11 | SEC-002, SEC-016, SEC-035 | Enforcement strength per platform is undecided (OD-17, OD-18). |
| AC-05 | Target response lures MXM into testing related hosts | TA-1 | AST-01 | SEC-003 | None expected if scope candidates are never actively tested without user authorization. |
| AC-06 | Target content or a mislabeled record claims a non-production environment to lift the production ceiling | TA-1, TA-9 | AST-01 | SEC-005 | A user can still deliberately mislabel; the label mechanism is open (OD-12). |
| AC-07 | A model claims a lower action class for a risky action | TA-8 | AST-01 | SEC-013, SEC-034 | Classification errors in manifests are possible and need review and tests. |
| AC-08 | AI-written text persuades the user to raise the level or approve a risky action | TA-8, TA-1 | AST-01 | SEC-009, SEC-012, SEC-040, SEC-018 | Social-engineering of the user cannot be fully removed (ASM-6). |
| AC-09 | A module performs network I/O directly | TA-4 | AST-01, AST-03 | SEC-032, SEC-033, SEC-035 | Sans-IO does not stop deliberately malicious in-process code; detection mechanisms are open (OD-15). |
| AC-10 | A module emits requests outside its manifest (stray method, destination, or rate) | TA-4 | AST-01 | SEC-034, SEC-033 | Manifest content itself may be wrong. |
| AC-11 | An external adapter's vulnerability or malicious output compromises MXM | TA-1, TA-4 | AST-11, AST-03 | SEC-036, SEC-037, SEC-039 | Parser and API vulnerabilities in third-party tools; isolation strength per platform (OD-19). |
| AC-12 | An adapter opens connections that bypass mediation | TA-4 | AST-01 | SEC-035, SEC-036, SEC-038 | Windows and Linux enforcement guarantees are undecided and may differ (OD-17, OD-18, OD-20, OD-21). |
| AC-13 | Secrets or personal data in HTTP responses are stored or reported | TA-1 | AST-06, AST-10 | SEC-045, SEC-047, SEC-049 | Redaction is "where feasible" and cannot be complete. |
| AC-14 | Same-user malware reads evidence, keys, or approvals | TA-5 | AST-06, AST-07 | SEC-046 (key separation) | High. If the user account is compromised, local confidentiality is not assured (ASM-2). |
| AC-15 | A cloud provider receives sensitive target data | TA-7 | AST-06, AST-08 | SEC-052, SEC-045, SEC-048 | After opt-in, transmitted data leaves the user's control. |
| AC-16 | Retained raw prompts or responses leak | TA-5, TA-7 | AST-08 | SEC-048 | Applies only if the user opts in to raw retention. |
| AC-17 | An approval is replayed or reused for different parameters | TA-8, TA-4 | AST-01 | SEC-025, SEC-017, SEC-016 | None expected if binding is implemented as specified. |
| AC-18 | A level or scope change races with in-flight or queued actions | TA-9 | AST-01 | SEC-008, SEC-016, SEC-021 | Complete epoch-event list is open (OD-05). |
| AC-19 | A verification step performs an action the policy would not allow | TA-8 | AST-01 | SEC-041, SEC-042 | None expected if verification uses the same executor path. |
| AC-20 | Cleanup acts on a resource identifier supplied by the target, deleting a pre-existing or unrelated resource | TA-1 | AST-01, AST-04 | SEC-027, SEC-026 | The mechanism that proves a recorded identifier is test-created is open (OD-09). |
| AC-21 | The cleanup ledger is altered to broaden cleanup | TA-5, TA-6 | AST-04 | SEC-043, SEC-044 | A privileged attacker can alter local state; changes are only tamper-evident. |
| AC-22 | The audit log is modified, truncated, or reordered | TA-5, TA-6 | AST-05 | SEC-044 | Tamper-evident only; a privileged attacker can rewrite and re-chain the whole log. |
| AC-23 | Actions or adapter processes continue after the kill switch | TA-4 | AST-01 | SEC-022, SEC-036 | Requests already sent cannot be recalled; adapter termination guarantees are platform-specific. |
| AC-24 | Autonomous planning drives excessive load on a fragile target | TA-8, TA-9 | AST-01 | SEC-011, SEC-015, SEC-023, SEC-053 | Thresholds are not yet designed (OD-03, OD-04); target fragility is unpredictable. |
| AC-25 | Oversized or hostile responses exhaust MXM resources | TA-1 | AST-11 | SEC-023, SEC-053 | Response-size and decompression limits are open (OD-22). |
| AC-26 | The user tests systems without authorization | TA-9 | AST-01 | SEC-001 (accountability only), [RESPONSIBLE_USE.md](../RESPONSIBLE_USE.md) | Accepted. MXM cannot verify authority (ASM-1). |
| AC-27 | Passive source queries disclose target identifiers or return poisoned data | TA-1, TA-7 | AST-02, AST-12 | SEC-003, SEC-039 | Third-party source governance is open (OD-16). |
| AC-28 | A weak local model emits malformed or unsafe proposals | TA-8 | AST-12 | SEC-050, SEC-054 | Capability checks establish compatibility, not safety. |
| AC-29 | A compromised dependency or update | TA-4 | AST-13 | None specified | Update, signing, and supply-chain strategy is open (OD-35). |
| AC-30 | Reports shared with sensitive data included | TA-10 | AST-10 | SEC-045, SEC-047 | Human review of reports is still needed. |
| AC-31 | Actions continue after authorization expiry or revocation | TA-9 | AST-01 | SEC-006, SEC-016 | None expected if checked at dispatch time. |
| AC-32 | Credential testing locks out real users | TA-8, TA-9 | AST-01 | SEC-014, SEC-020 | The deterministic definition of lockout-risk volume is open (OD-13). |

## 8. Residual-risk summary

These risks remain even if every specified control is implemented correctly:

1. **Compromised local environment.** Evidence confidentiality, key secrecy, and local state integrity depend on the host (ASM-2). The audit hash chain is tamper-evident, not tamper-proof.
2. **Malicious in-process code.** Sans-IO prevents ordinary bypass; it does not sandbox hostile Python (ASM-3, ASM-4).
3. **Adapter and platform enforcement strength** differs between Windows and Linux and is not yet decided.
4. **Incomplete redaction.** Redaction is "where feasible" and cannot guarantee that no secret or personal data remains.
5. **Human factors.** Authorization honesty, approval fatigue, and environment mislabeling are not fully controllable.
6. **Model unreliability.** Capability checks do not prove a model is safe; a misled model can still waste allowed budget or mislead the user.
7. **Unbounded unknowns until designed.** Numeric limits, epoch-event lists, and several enforcement mechanisms are open decisions; protections that depend on them are not yet specified.

## 9. Review process

- Re-review when any trust boundary changes, when an adapter is added, or when a mitigation relies on an open decision that has since been resolved.
- Every abuse case with a "None expected" residual must have a negative test (`SEC-055`) before release claims are made.
- Residual risks must be repeated in the published known-limitations statement required by the release gate.
