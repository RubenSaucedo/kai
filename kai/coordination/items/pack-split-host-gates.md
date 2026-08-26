---
type: work-item
id: pack-split-host-gates
title: Host gates — macOS + cloud + install-order + fresh-session verification evidence
initiative: pack-split
milestone: first-pack-extracted
delivery_class: knowledge
state: blocked
resume_state: in-progress
priority: 10
owner: principal-swe-infra
next_role: @operator
target: pack-split host-gate certification evidence
artifact_target: kai/initiatives/pack-split/artifacts/reliability/pack-split-host-gates.md
context_artifacts:
  - kai/initiatives/pack-split/artifacts/decisions/pack-split-engineering-decomposition.md
  - kai/initiatives/pack-split/artifacts/decisions/pack-split-host-gates.md
  - docs/proposals/pack-architecture.md
  - kai/initiatives/pack-split/artifacts/reliability/pack-split-host-semantics-spike.md
  - kai/initiatives/pack-split/artifacts/reliability/pack-split-first-department-install.md
  - kai/initiatives/pack-split/artifacts/reliability/pack-split-migration-doctor.md
touches:
  - kai/initiatives/pack-split/artifacts/reliability/pack-split-host-gates.md
depends_on:
  - item: pack-split-first-department
    requires: shipped
  - item: pack-split-migration-doctor
    requires: shipped
waiting_on_questions:
  - Q-pack-split-host-gates-01
  - Q-pack-split-host-gates-03
required_for_milestone: true
review_requirements:
  - role: principal-sre
    kind: independent-reliability
completed_reviews: []
change_ref: null
version: 10
lease:
  holder: null
  token: null
  version_at_grant: null
  acquired: null
  expires: null
updated: 2026-08-26-1650
---

## Outcome

The formal verification evidence that the real `kai-core` + `kai-personal` install behaves on macOS,
on the cloud host, under real install order, marketplace-vs-direct, and across fresh sessions — the
hard gate that must pass before the `1.0.0` flip. Closes `completed` (evidence record).

## Acceptance

- [ ] Install-order + fresh-session + collision + cross-plugin resolution verified on macOS + one cloud host.
- [ ] Marketplace-vs-direct install paths both produce the correct bound contract (no stale legacy copy).
- [ ] Marketplace and direct install evidence records whether each pack root
      receives npm dependency installation or a `node_modules` tree.
- [ ] A generated department agent carrying both guarantee blocks replies with
      exactly `KAI-CORE-MISSING` and nothing else in both the no-core and
      contract-skew (`--contract 2`) arms.
- [ ] Evidence recorded at the artifact target; `principal-sre` reviews host/platform behavior.
- [ ] Result is an explicit go / no-go for `pack-split-release-12b`.

## Evidence

- Prepared canonical gate and execution packets:
  `kai/initiatives/pack-split/artifacts/reliability/pack-split-host-gates.md`.
- Corrected the packet from the architecture decision at
  `kai/initiatives/pack-split/artifacts/decisions/pack-split-host-gates.md`:
  persistent direct macOS installs in both orders, a run-local directory
  marketplace, run-start `main` pin verification, and a throwaway-branch cloud
  spike using repository `enabledPlugins`.
- External macOS and genuine cloud-host transcripts are not yet available.
  Release 12b is explicitly **NO-GO**.

## Notes

- **Scope (steward decision):** scoped as a **minimal smoke gate**, not full certification — the
  northstar defers full macOS/cloud certification (decomposition *Scope negotiations*).
- Host runs are **operator-executed**; this role designs the gate + records evidence. Closes `completed`.
- **Steward promotion 2026-08-26-1558:** promoted `proposed -> ready` after both typed
  dependencies reached their required `shipped` state. Priority 10 makes this the sole
  committed queue head. `principal-swe-infra` designs and records the bounded gate;
  `@operator` executes only the external host actions. Full fleet certification and any
  observer redesign remain out of scope.
- **Architecture correction 2026-08-26-1645:** the public-marketplace and
  enterprise-managed prerequisites were false constraints in the prior packet,
  not item Acceptance. `Q-pack-split-host-gates-02` is answered and withdrawn;
  `Q-pack-split-host-gates-01` now points to the corrected macOS packet, and
  `Q-pack-split-host-gates-03` requests the bounded cloud branch spike. A
  default-branch-only cloud result routes to steward escalation E1 without an
  infra decision.
