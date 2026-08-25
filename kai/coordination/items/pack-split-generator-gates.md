---
type: work-item
id: pack-split-generator-gates
title: Harden the pack generator and make validate/release gates multi-manifest aware
initiative: pack-split
milestone: dependency-guarantees
delivery_class: product-change
state: in-progress
resume_state: null
priority: 10
owner: principal-swe-infra
next_role: principal-swe-infra
target: pack-split build tooling — generator + multi-manifest gates (foundation)
artifact_target: null
context_artifacts:
  - kai/initiatives/pack-split/artifacts/decisions/pack-split-engineering-decomposition.md
  - kai/initiatives/pack-split/artifacts/docs/pack-split-partition-lock.md
  - scripts/pack-preview.mjs
  - scripts/validate-plugin.mjs
  - scripts/release-guard.mjs
touches:
  - scripts/pack-preview.mjs
  - scripts/lib/pack-plan.mjs
  - scripts/validate-plugin.mjs
  - scripts/release-guard.mjs
  - .github/workflows/validate.yml
  - package.json
depends_on:
  - item: pack-split-partition-lock
    requires: completed
waiting_on_questions: []
required_for_milestone: true
review_requirements:
  - role: principal-swe-architect
    kind: independent-architecture
completed_reviews:
  - role: principal-swe-architect
    kind: independent-architecture
    change_ref: 457254b973fb58b129332ffaa609fb5febfdd412
    verdict: ratified
    timestamp: 2026-08-24-2231
change_ref: 457254b973fb58b129332ffaa609fb5febfdd412
version: 11
lease: null
updated: 2026-08-24-2244
---

## Outcome

`scripts/pack-preview.mjs` becomes the deterministic authoritative pack generator
(stable ordering, per-pack `plugin.json`, a single machine-readable partition with
explicit committed-slice control), and `scripts/validate-plugin.mjs` +
`scripts/release-guard.mjs` iterate over N manifests instead of assuming one root
`plugin.json` — the shared foundation every downstream dependency-guarantee item
sits on. Ships on `0.x`.

**Explicitly not in this foundation (steward correction at acceptance, 2026-08-24-2240).**
This item delivers partition/generator/multi-manifest **plumbing only**. Asset-ownership
routing (a non-markdown asset travelling with its sole invoking pack, promotion to core
when invoked across >1 pack) and the hooks-exactly-once rule are **not** delivered here
and remain owned by `pack-split-crosspack-validator` (validation of asset references and
`hooks.json` assigned to exactly one pack) and `pack-split-generated-pack-trees`
(materializing that ownership into the committed trees). The prior wording
("asset/hooks-ownership routing hooks") credited this foundation with a downstream
guarantee it does not provide.

## Acceptance

*Reconciled by the steward at acceptance 2026-08-24-2240 against the ratified review at
`change_ref 457254b973fb58b129332ffaa609fb5febfdd412` and the operator's verification run.
Criterion 4 as originally written bundled four local commands with "new CI steps green";
those are two different claims with two different evidence sources, so it is **split** —
the local half is met, the remote half cannot be true before the branch is pushed. The bar
is not lowered: nothing is ticked that is not evidenced, and the CI half stays open.*

- [x] The generator materializes core + a department tree deterministically from root,
      with a per-pack `plugin.json`; re-running is byte-stable.
      — 35-check self-test proves determinism, per-pack `plugin.json`, committed-tree
      round-trip, and a core+personal slice emitting no `kai-engineering/` keys; architect
      verified verbatim body copy, `normalizeLF` on emit **and** on read, fixed JSON key
      order, sorted rosters. On-disk committed materialization is deliberately gated
      (`COMMITTED_PACKS` empty) and belongs to `pack-split-generated-pack-trees`.
- [x] `validate-plugin.mjs` validates N manifests (per-pack version agreement; marketplace
      index may list multiple plugins) without regressing the single-manifest monolith checks.
      — `discoverManifests` returns the root manifest alone when `packs/` is absent (N=1
      byte-for-byte unchanged); `manifestParityErrors` enforces lockstep; marketplace
      generalised to N entries, monolith entry still required. Operator run: valid
      (56 agents, 50 skills).
- [x] `release-guard.mjs` classifies the committed pack-tree directory as behavior-sensitive
      so a generated-tree change still requires a bump + changelog + README.
      — `BEHAVIOR_PREFIXES = ['agents/', 'skills/', 'scripts/', 'packs/']` with self-test
      cases and path spot-checks; `--self-test` passed. The `packs/` root is confirmed
      (see the decision in Evidence), so this is no longer contingent on an assumption.
- [x] `node scripts/pack-preview.mjs --self-test`, `node scripts/validate-plugin.mjs`,
      `node scripts/release-guard.mjs --self-test`, and `npm test` all pass **locally**
      (operator-executed 2026-08-24, pre-binding; all exit 0).
- [ ] **The new CI steps run green on the pushed PR.** The steps exist in
      `.github/workflows/validate.yml` (`Pack generator self-test` → `pack-preview --self-test`;
      `Committed pack trees match the generator` → `pack-preview --check`), but remote CI has
      **never executed** — the change is uncommitted, unpushed, and has no PR. **Open by
      construction**; it closes at PR delivery on real workflow-run evidence, not by assertion.
- [x] Version bumped on `0.x` with CHANGELOG + README stamp (release-guard).
      — `0.57.0 -> 0.58.0` coherent across `plugin.json`, `package.json`, `package-lock.json`
      (both fields), `marketplace.json` (both fields), CHANGELOG dated section + compare link,
      README `## Status`, AGENTS.md release-path list; architect verified coherence.

## Evidence

**Implemented on branch `kai/feat/29-pack-generator-gates` (working tree; uncommitted).**
`principal-swe-infra`, with operator verification and review fixes, 2026-08-24.

Increment 1 — generator engine:
- `scripts/lib/pack-plan.mjs` (new): the single machine-readable partition/manifest source —
  `PACKS`, the ratified `SKILL_OWNER_OVERRIDES`, `planPacks`, deterministic
  `planManifests`/`materializePacks`, `discoverManifests`, and the pure gate helpers
  `manifestParityErrors` + `marketplaceConsistencyErrors`. Imported by the preview/generator
  and validator so inherited and non-inherited skill ownership is defined once.
- `scripts/pack-preview.mjs`: imports the partition from `pack-plan.mjs` (re-exports the names the
  locked partition doc references); promoted to the deterministic generator with `--write` and
  `--check` (regenerate-and-diff), LF-normalised byte-stable output. `COMMITTED_PACKS` is empty
  in this foundation, so `--write` refuses until the downstream extraction item selects
  `core + personal`; self-tests prove an unselected department is not materialised.

Increment 2 — validate/release multi-manifest:
- `scripts/validate-plugin.mjs`: iterates all discovered manifests (monolith + any
  `packs/<name>/plugin.json`) for structure + per-pack version parity; marketplace generalised to
  N plugins (every entry agrees with its own manifest; monolith entry still required; an
  unpublished pack with no entry is allowed). With no `packs/` present, N=1 behaviour is
  byte-for-byte unchanged — the current single-plugin marketplace is unaffected.
- `scripts/release-guard.mjs`: `packs/` added to `BEHAVIOR_PREFIXES` (+ self-test cases), so a
  committed pack-tree change requires bump + CHANGELOG + README.
- `.github/workflows/validate.yml`: new CI steps `pack-preview --self-test` and `pack-preview --check`.

Release (`0.57.0 -> 0.58.0`, pre-1.0 minor = feature per AGENTS.md): `plugin.json`, `package.json`,
`package-lock.json` (both fields), `.github/plugin/marketplace.json` (both fields); CHANGELOG entry
+ compare link; README `## Status` stamp; AGENTS.md release-path list (+`packs/`).

**Deliberately NOT done (item boundaries / other items' touches):** no committed `packs/` trees
(WS#7 `pack-split-generated-pack-trees`), no preflight injection (WS#4), no degraded-mode block
(WS#5), no cross-pack reference validation (WS#3), no marketplace pack entries / publication. Packs
remain absent from the public install surface; `kai-core` identity and uninstall-first/coexistence
decisions untouched.

**Committed-tree root `packs/` — CONFIRMED by the steward, 2026-08-24-2240.** Decomposition
Open Question 2 is **closed**; this is no longer an open assumption. Evidence judged sufficient
without further investigation: the northstar's generate-and-commit-unpublished principle needs a
committed root but does not name one; the accepted decomposition (WS#1/#7) assumes `packs/` so
`release-guard` behavior-classification and the marketplace `source` can point at it;
`pack-split-generated-pack-trees` already declares `touches: packs/kai-core/`,
`packs/kai-personal/`; the ratified implementation encodes it in exactly two places
(`PACKS_DIR` in `scripts/lib/pack-plan.mjs`, `BEHAVIOR_PREFIXES` in `scripts/release-guard.mjs`);
and `.gitignore` does not exclude it. `dist/` was the named alternative and is rejected: it
conventionally signals build output that is *not* committed, which is the opposite of the
committed-unpublished contract. `packs/` is now the initiative's convention — downstream items
inherit it and do not re-litigate it.

**Verification executed by the operator after review fixes:**
- `node scripts/pack-preview.mjs --self-test` — 35 checks passed.
- `node scripts/pack-preview.mjs --check` — passed; no committed packs configured and
  `packs/` is intentionally absent.
- `node scripts/validate-plugin.mjs` — valid (56 agents, 50 skills).
- `node scripts/release-guard.mjs --self-test` — passed.
- `node scripts/check-syntax.mjs` — 22 JS/MJS + 1 PowerShell helper parsed.
- `npm test` — passed.

**Review fixes before binding:** the first implementation silently swept all nine
mechanical orphans into core and allowed `--write` to generate all five packs. The
operator corrected both: the ratified 1-core/4-personal/4-engineering dispositions
are explicit in the partition source, and committed generation remains disabled
until the core-plus-personal extraction item selects that slice.

## Notes

- **DoD gate run 2026-08-24-2244 (`workflow-ship` PREPARE) — verdict `BOUNCE`, one Gap.**
  Five of six dimensions Clear at `change_ref 457254b973fb58b129332ffaa609fb5febfdd412`;
  **dim-2 (verified) is a Gap** — the item's own open acceptance criterion ("the new CI
  steps run green on the pushed PR") has **no evidence and cannot acquire any** until the
  branch is committed, pushed, and a PR exists. Under `kai-core-definition-of-done` an
  unbacked Clear is a Gap, and this is not waivable: the dimension genuinely applies.
  The gap is **not** procedural bookkeeping — it is the only check that can still fail:
  1. **Different platform.** Every green run to date is Windows (`C:\src\kai`);
     `.github/workflows/validate.yml` runs `ubuntu-latest`. This change's central claim
     is byte-stable, LF-normalised, forward-slash-keyed generation — precisely the class
     that passes on one platform and fails on the other, and precisely what architect
     findings A2/A3 flag.
  2. **A step that has never executed in any form.** The `Release-guard (behavior change
     requires a bump + release notes)` step is `if: github.event_name == 'pull_request'`
     and runs `release-guard --base <base.sha> --head <head.sha>`. The local run was
     `--self-test`, which exercises the classifier, **not** the real base/head gate over
     this diff. `scripts/` is a `BEHAVIOR_PREFIX`, so this diff will be classified
     behavior-sensitive and the gate will demand the bump + CHANGELOG + README that are
     present — but that demand has never actually been evaluated.
  3. **Repo-wide blast radius.** The two new steps (`Pack generator self-test`,
     `Committed pack trees match the generator`) become mandatory gates on *every* PR in
     this repository once merged. A wrong or platform-fragile `--check` blocks all
     contributors, not just this initiative.
  State returned `in-review -> in-progress`, `next_role: principal-swe-infra` for PR
  delivery under `kai-core-pr-delivery`. No ship record was written (none is written for a
  bounced item), nothing was committed, pushed, PR'd, merged, tagged, released, or
  published, and no implementation, release-metadata, or downstream-scope file was touched.

- **The ratified diff must not be edited (binding constraint on PR delivery).** The
  `independent-architecture` verdict binds to `change_ref
  457254b973fb58b129332ffaa609fb5febfdd412`, and the operator has confirmed byte-identity
  between that object and the current implementation/release files via
  `git diff <change_ref> -- <files>`. PR delivery must commit **that tree unchanged** —
  no A5/A6 fixes, no CHANGELOG rewording, no opportunistic tidying. Any content change
  invalidates the ratified review, requires a new `change_ref`, and sends the item back
  through `principal-swe-architect`. The PR head commit will necessarily have a different
  SHA than the stash object; that alone is **not** an implementation change. If
  `change_ref` is re-pointed to the PR head SHA, record the byte-identity proof (an empty
  `git diff <ratified-ref> <pr-head>`) in Evidence so the existing verdict still binds.
  Note also that a `git stash create` object is dangling and garbage-collectable — commit
  the branch before it is lost.

- **Manager split (reviewability).** Extracted from the steward's descriptions, which had
  folded the multi-manifest gates into `pack-split-crosspack-validator` and the generator into
  `pack-split-generated-pack-trees`. Built that way, three guarantee items would re-implement the
  same plumbing and two would be XL. This is the single first implementation item.
- Delivered as 2 PR increments: (1) generator engine; (2) validate/release multi-manifest refactor.
- **Proposed** for the `dependency-guarantees` `required_items` (steward's call at grooming).

- **Steward-accepted 2026-08-24-2240 (`principal-product-manager`, acceptance at
  `change_ref 457254b973fb58b129332ffaa609fb5febfdd412`).** Product acceptance **granted**:
  the ratified foundation is the thing that was commissioned, it held to foundation scope
  (no pack trees, no preflight, no degraded block, no cross-pack validator, no publication),
  and it does not overclaim downstream guarantees. Steward actions: (1) corrected the Outcome
  prose, which credited this item with asset/hooks-ownership routing it does not deliver;
  (2) reconciled the five acceptance criteria — four met, and the bundled local-plus-CI
  criterion split so "remote CI green" stays honestly open until a PR exists;
  (3) **confirmed `packs/`** as the committed-tree root, closing decomposition Open Question 2;
  (4) routed the architect's carry-forward findings into owning records rather than prose.
  **State stays `in-review`; `next_role: workflow-ship`.** Per the lifecycle contract, all
  `review_requirements` are satisfied for a `product-change` item, so routing goes to
  `workflow-ship`, which alone owns the DoD gate and the `in-review -> release-ready`
  transition. The steward does not move it there and explicitly does **not** mark it
  `shipped`: nothing is committed, pushed, PR'd, merged, tagged, released, or published.

- **Carry-forward routing of the architect's findings (2026-08-24-2240).** None reopens this
  item; none is fixed in this PR. Each is now owned by the record that first makes it reachable:
  - **A1** (`checkCommitted` ENOENT before the `--write` hint when a slice is selected and
    `packs/` is absent), **A2** (OS artifacts under `packs/` failing local `--check` while CI
    stays green), **A3** (`.gitattributes` pin `packs/** text eol=lf`) →
    `pack-split-generated-pack-trees`, added as acceptance criteria (A3 also added to its
    `touches`). All three are unreachable until a committed tree exists, which is that item.
  - **A4** (marketplace publication sits outside release enforcement; entry `name` not asserted
    against the `name` in the `plugin.json` at its `source`) → `pack-split-release-12b`, added
    as acceptance criteria with `scripts/release-guard.mjs` + `scripts/validate-plugin.mjs`
    added to its `touches`. Cross-referenced from `pack-split-generated-pack-trees`, because
    committing the first tree is what makes the unpublished→published flip a pure
    `marketplace.json` edit — the guard must land no later than the flip.
  - **A5** (`PACK_AGENTS` duplicate legacy roster truth; four self-test checks running the
    legacy `planSkills(PACK_AGENTS)` path) → `pack-split-ci-partition-checks`, added as an
    acceptance criterion. Steward reasoning: that item turns the partition self-test into a
    hard CI gate, so collapsing the duplicate truth and re-pointing the legacy checks belongs
    there, immediately before the checks become load-bearing. Not fixed here — the authoritative
    assertions are unaffected today and touching `pack-preview.mjs` again would expand a
    ratified diff for a non-defect.
  - **A6** (`planManifests` always sets `manifest.skills = 'skills'` while `materializePacks`
    only creates `skills/` for a pack owning ≥1 skill) → **parked as a PROPOSAL** in
    `kai/initiatives/pack-split/backlog.md`. Unreachable in this initiative: all five locked
    departments own ≥1 skill, and the northstar forbids adding packs beyond the agreed five.
    Trigger to revisit is recorded with the proposal.

- **Not expanding this PR.** A5 and A6 are the only findings that touch files already in this
  item's diff, and neither is required for acceptance. Under `scope-discipline` they are
  refinements with no reachable failure today; re-opening a ratified `change_ref` for them
  would invalidate the binding review for no gain.

- **Steward-groomed 2026-08-24-2013 (`proposed -> ready`).** `principal-product-manager`
  accepted the decomposition, added this item to the `dependency-guarantees` `required_items`
  (`shipped`) as the foundational split, confirmed scope against repository evidence
  (`scripts/pack-preview.mjs`/`validate-plugin.mjs`/`release-guard.mjs` are single-manifest today),
  and promoted this to `ready` at priority 10 — the **single first implementation item** for the
  initiative. `next_role: principal-swe-infra`, version 1 -> 2. Its dependency
  (`pack-split-partition-lock`) is `completed`, so it is executable now. The other 13 records stay
  `proposed` for milestone-by-milestone grooming; `pack-split-host-semantics-spike` is recommended
  to start in parallel (operator-run). Dispatch is the director's.
