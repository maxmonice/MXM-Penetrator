# Scope and Authorization

This document owns `SEC-001` to `SEC-006`: the authorization record and its lifecycle, scope, scope candidates, and environment classification with the production ceiling.

## SEC-001 — Authorization record required

Every assessment must have a durable authorization record **before any active testing begins**. At minimum it contains:

- Assessment identifier.
- Authorizing user or operator.
- Target identifiers and permitted asset boundaries.
- Authorization source or reference supplied by the user.
- Start and expiry time.
- Allowed assessment activities and exclusions.
- Maximum permission-level ceiling (derived per `SEC-005`).
- Environment designation (`SEC-005`); if unspecified, Production.
- Relevant rate and execution constraints.

The record format is versioned. MXM must not represent a user assertion as independently verified legal authorization. See [../RESPONSIBLE_USE.md](../RESPONSIBLE_USE.md).

## SEC-002 — Explicit scope and execution-time destination validation

Scope supports explicit domains, hostnames, IP addresses, CIDR ranges, ports, and protocol constraints as appropriate. The exact supported forms and canonicalization rules must be defined before implementation.

Scope validation must account for:

- DNS resolution and changes over time.
- CNAME chains.
- Redirects.
- Multiple A/AAAA records.
- IP/CIDR membership.
- Private, loopback, link-local, and cloud metadata addresses.
- DNS rebinding and time-of-check/time-of-use gaps.

Destinations are validated **at execution time**, not only when the user enters a hostname. Redirects and newly resolved destinations are rechecked before connection. This is enforced by the Scope Validator and the executor (`SEC-016`, `SEC-033`); the enforcement mechanisms are described in [NETWORK_ENFORCEMENT.md](NETWORK_ENFORCEMENT.md).

## SEC-003 — Newly discovered assets are scope candidates

A newly discovered domain, subdomain, host, IP address, service, or related asset is a **scope candidate**, not an authorized target.

MXM may record the discovery, attach evidence and provenance, show the candidate to the user, and explain why it may be related to the authorized target. MXM must not actively test the candidate until the user explicitly authorizes its inclusion through the trusted control channel. Authorization of a parent domain does not authorize every discovered asset unless the original scope explicitly defines that boundary.

## SEC-004 — Changes through the trusted control channel

Adding assets or changing authorization boundaries requires an explicit user action through the trusted control channel (`SEC-018`). Every change is audited and applies to subsequent actions. Existing queued proposals are revalidated against the updated scope. Whether a scope change increments the policy epoch is open (OD-05).

## SEC-005 — Environment classification and the production ceiling

- The authorization record carries an **environment designation**.
- **An unknown or unspecified environment is treated as Production.** Production safety restrictions apply until the environment is explicitly identified otherwise.
- **Production targets have a maximum permission level of Guided.** How the effective level is computed from the user's selection and this ceiling is owned by `SEC-007` in [PERMISSION_MODEL.md](PERMISSION_MODEL.md).
- Re-designating a target from Production to a non-production environment is a relaxation of controls. It is a user action through the trusted control channel and is audited.
- Environment labels never override authorization, scope, permission restrictions, or any other policy control. A non-production label does not grant scope, does not lift a hard deny, and does not change thresholds.
- Neither the AI model nor target-provided content can set or downgrade an environment designation.

How a designation is established, stored, and protected is open (OD-12).

## SEC-006 — Authorization lifecycle

An authorization record is *valid*, *expired*, or *revoked*, and can be *restored*.

- When the record is expired or revoked, no network action is permitted, including cleanup (see `SEC-031` for the cleanup consequences). Checking is done at dispatch time (`SEC-016`).
- Revocation and restoration are user actions through the trusted control channel and are audited.
- Restoration of a valid authorization does not retroactively approve any action. Approvals are governed by `SEC-025`.
- Local, non-network state changes (for example marking cleanup resources as pending) remain possible while authorization is revoked.

## Related

- Which level applies against the ceiling: [PERMISSION_MODEL.md](PERMISSION_MODEL.md)
- Cleanup when authorization is revoked: [APPROVALS_AND_CLEANUP.md](APPROVALS_AND_CLEANUP.md)
- Open items: OD-05, OD-07, OD-12
