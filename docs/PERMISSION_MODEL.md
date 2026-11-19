# Permission Model

This document owns `SEC-007` to `SEC-012`: what each permission level allows, how levels change, and the relationship between Active and Autonomous.

Permission levels define which action classes may execute without per-action approval and how much planning latitude the AI has. They do not replace target authorization, scope checks, hard denies, budgets, or approval requirements. The order in which all of these are applied is owned by `SEC-016` in [POLICY_ENGINE.md](POLICY_ENGINE.md).

## Profiles

| Level | Name | Color |
|---|---|---|
| 1 | Guided | Green |
| 2 | Active | Yellow |
| 3 | Autonomous | Red |

Colors indicate decreasing human supervision, not additional execution privilege. Active and Autonomous have identical execution permissions (`SEC-010`, `SEC-011`).

## Permission matrix

This is the single authoritative statement of which class may run at which level. "Automatic" means the action may execute without per-action user approval **when every other policy control passes** (authorization, scope, ceiling, hard denies, manifest conformance, limits, budgets). Class definitions are in [ACTION_CLASSIFICATION.md](ACTION_CLASSIFICATION.md).

| Action class | Level 1 Guided | Level 2 Active | Level 3 Autonomous |
|---|---|---|---|
| Passive | Automatic | Automatic | Automatic |
| Safe-active | Automatic | Automatic | Automatic |
| Intrusive (non-state-changing) | Requires user approval | Automatic | Automatic |
| State-changing | Requires approval of a complete test-case bundle (`SEC-024`) | Same | Same |
| Destructive | Denied (`SEC-020`) | Denied | Denied |

## Planning latitude

Planning latitude is **not** an execution permission. It controls how much the AI may choose and sequence on its own. Every proposal it produces is checked independently against the same execution policy.

| Level | AI planning latitude |
|---|---|
| Guided | Limited and transparent. |
| Active | AI may select from registered tests within the assessment plan and budgets. |
| Autonomous | Greater independent planning, sequencing, and bounded replanning. May select and order registered modules and queue bounded follow-up investigations. |

For Active and Autonomous, the limits in `SEC-053` ([AI_ORCHESTRATION.md](AI_ORCHESTRATION.md)) apply, and Autonomous is additionally gated by the model capability checks in `SEC-054`. Autonomous mode additionally requires reviewable decision traces and checkpoints (see `SEC-053`). How the planning parameters differ by level, and whether Guided uses the same parameter set, are open (OD-04).

What the AI can never do at any level is owned by `SEC-009` below and `SEC-050` in AI_ORCHESTRATION.

## SEC-007 — Fixed profiles, Guided default, and effective level

- Exactly three profiles exist: Guided, Active, Autonomous. They are fixed and are not individually customizable in v1.
- Every new assessment starts with Guided preselected.
- The user must confirm the selected level for every new assessment.
- The active level is prominently visible during an assessment.
- The **effective level** is the most restrictive of the user-selected profile and the authorization ceiling defined by `SEC-005` ([SCOPE_AND_AUTHORIZATION.md](SCOPE_AND_AUTHORIZATION.md)). A ceiling can only restrict; selecting a higher level cannot override it. The user is told when the effective level differs from the selection.

## SEC-008 — Permission-level changes

- Users may change levels during an active assessment, **only** through the trusted control channel (`SEC-018`), never through AI output, target content, or an untrusted file.
- A downgrade applies immediately to queued and pending actions; those actions are re-evaluated.
- An upgrade applies only to newly proposed actions and never retroactively authorizes a pending or previously denied proposal.
- Every change generates a user-visible notification, an audit event (`SEC-043`), and a policy epoch change (`SEC-021`).
- In-flight non-mutating actions may finish subject to existing limits. In-flight state-changing test cases follow `SEC-027`; a level change does not cancel their approved cleanup.
- Pause and kill are separate controls (`SEC-022`).

## SEC-009 — AI cannot change its level or approve actions

The AI cannot change the active permission level, change its ceiling, approve its own proposals, or approve any state-changing action. The AI may recommend a different level (see `SEC-012`); only the user can confirm one.

## SEC-010 — Active/Autonomous execution-permission invariant

For identical assessment context, scope, authorization ceiling, action proposal, and execution budget, the set of actions that may execute under Autonomous must never exceed the set allowed under Active. Equivalently:

`AllowedActions(Autonomous) = AllowedActions(Active)`

Planning latitude may differ; execution permission does not. The planner may generate different proposals in Autonomous, but each proposal is independently checked against the same execution policy. This invariant must be covered by automated tests (property tests, `SEC-055`).

## SEC-011 — Active and Autonomous share identical execution limits

Level 2 (Active) and Level 3 (Autonomous) must have identical:

- execution permissions,
- network rate ceilings,
- concurrency limits,
- per-test request budgets, and
- assessment-wide execution budgets.

Autonomous differs from Active only in greater AI planning, sequencing, and bounded replanning freedom. It never receives more aggressive testing permissions or network execution capacity. Because the numeric thresholds are defined per action class and not per level (`SEC-015`), a level change cannot alter them. Any future setting that would give Autonomous different execution limits contradicts this requirement and requires an explicit decision to change it.

AI planning limits (replans, follow-ups, concurrent tasks, duration, tokens) are a different category, are governed by `SEC-053`, and may differ in value by level. They must remain separate from network execution budgets.

## SEC-012 — Natural-language selection is advisory

The user may select a level manually or describe desired behavior in natural language. AI may map the description to **one of the three fixed profiles** and explain its recommendation. The CLI shows a deterministic description of the profile (generated by trusted code, not by the model) and requires explicit user confirmation. A model response can never activate or elevate a profile, and natural language cannot carry additional permissions or custom behavior.
