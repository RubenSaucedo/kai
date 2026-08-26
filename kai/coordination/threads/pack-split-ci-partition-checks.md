# Thread — pack-split-ci-partition-checks

Append-only communication log mirroring
`kai/coordination/items/pack-split-ci-partition-checks.md`. See `kai-core-work-coordination`.

## HANDOFF 2026-08-24-2011 — principal-swe-manager -> principal-product-manager

- did:       Created this `proposed` record — the CI capstone of `dependency-guarantees`. Wires the `--all` self-test + collision/partial-install/version-skew arms as real CI gates, adds `kai-core-*` namespace enforcement, and carries architect caveat (a): the **forced** rename `fleet-observation` -> `kai-core-fleet-observation`. Size L. Owner `principal-swe-infra`; review `principal-swe-architect`/independent-architecture. Depends on `pack-split-crosspack-validator` + `pack-split-preflight-compat` (shipped).
- state:     proposed
- needs:     Steward grooming + promotion; note the rename is forced (prefix CI goes red until it lands) and contained (orphan skill — no inheritance refs).
- artifacts: kai/coordination/items/pack-split-ci-partition-checks.md; decomposition WS#6
- evidence:  grep — `fleet-observation` in skills/ dir + generate-catalog.mjs (line ~153) + test/fixtures/inventory.json + docs; no agent `**Inherits:**` it — captured 2026-08-24 from C:\src\kai
- questions: director availability membership-not-count — confirm the "partly landed" work is complete (decomposition Open Question 4)
- next:      principal-product-manager — groom milestone-by-milestone; must precede `pack-split-generated-pack-trees`.

## NOTE 2026-08-24-2240 — principal-product-manager (steward) — carry-forward from `pack-split-generator-gates` acceptance

- Item version 1 -> 2. State stays `proposed`; `next_role` unchanged. No lease taken.
- **Architect finding A5 added as two acceptance criteria**, from the `independent-architecture`
  review ratified 2026-08-24-2231 at `change_ref 457254b973fb58b129332ffaa609fb5febfdd412`.
  `scripts/pack-preview.mjs` still carries a second, independently maintained copy of the personal
  roster: `PACK_AGENTS` and `PACKS.personal` are byte-identical today but can drift, and four of the
  35 self-test checks assert against the legacy `planSkills(PACK_AGENTS)` path rather than the
  canonical `planPacks()`. Collapse the duplicate truth
  (`export const PACK_AGENTS = PACKS.personal;` or remove it) and re-point the legacy checks.
- **Why here rather than in the foundation.** A5 is not a defect today — the authoritative
  assertions are unaffected — and re-opening a ratified `change_ref` for a non-defect would
  invalidate the binding review for no gain. This item is where the partition self-test becomes a
  **hard CI gate**, so a gate asserting against stale truth is exactly the failure this item exists
  to prevent. `scripts/pack-preview.mjs` is already in this item's `touches`, so the cost is roughly
  one line plus re-pointing four checks.
- The foundation (`pack-split-generator-gates`) eliminated the duplicate partition truth everywhere
  else: `scripts/lib/pack-plan.mjs` is now the single machine-readable partition source, carrying the
  ratified orphan dispositions. A5 is the last legacy export.
- Dependencies unchanged and unmet (`crosspack-validator`, `preflight-compat` at `shipped`); not
  executable. Decomposition Open Question 4 (director availability by membership) remains open on
  this record and is untouched by this note.

## HANDOFF 2026-08-25-1148 — principal-product-manager -> principal-swe-infra

- did:       Steward grooming pass (continuation of 2026-08-25-1139). **Promoted `proposed -> ready`**
             at **priority 20 -> 50 — last in the initiative queue** (v2 -> v3,
             `next_role: principal-swe-infra`, `owner` still null). Promoted **with both dependencies
             unmet, deliberately**: `ready` requires `depends_on` to be *declared*, not resolved.
             Tightened acceptance on two finding-driven points only — **split** the bundled "local
             commands + CI green" criterion (the 2026-08-24-2244 DoD bounce), and named
             `scripts/lib/pack-plan.mjs` as the canonical partition source the forced rename must
             update. Reconciled `touches` to the shipped foundation and to WS#6 as already written
             (added `scripts/lib/pack-plan.mjs` + the doc/README/CHANGELOG paths WS#6 enumerated).
             The A5 criteria from the 2026-08-24-2240 note are unchanged.
- state:     ready
- needs:     **Nothing yet — this item is NOT dispatchable.** Both `depends_on` entries
             (`pack-split-crosspack-validator`, `pack-split-preflight-compat`, each
             `requires: shipped`) are unsatisfied — both are `ready`, neither dispatched. The
             director's dependency check must continue to fail here; do not grant a lease until both
             are `shipped`. Neither dependency type was relaxed: `crosspack-validator` supplies the
             multi-manifest gate base this layers on, `preflight-compat` supplies the emitter the
             **version-skew arm** tests.
- artifacts: kai/coordination/items/pack-split-ci-partition-checks.md (v3);
             kai/initiatives/pack-split/artifacts/decisions/pack-split-engineering-decomposition.md (WS#6);
             kai/initiatives/pack-split/artifacts/docs/pack-split-partition-lock.md;
             kai/library/releases/2026-08-24/01-ship-pack-split-generator-gates/ship-record.md
- evidence:  `scripts/lib/pack-plan.mjs` on `main` still maps `'fleet-observation': 'core'` in its skill
             map — after `generator-gates` that file is the canonical partition source, so the forced
             rename cannot land without it; `skills/fleet-observation/SKILL.md` still exists and is
             referenced by `scripts/generate-catalog.mjs`, `test/fixtures/inventory.json`,
             `docs/getting-started.md`, `docs/workspaces.md`, `docs/reference/agents-and-skills.md`,
             `README.md`, `CHANGELOG.md`. Read 2026-08-25 from C:\src\kai.
- questions: Decomposition **Open Question 4** (director availability asserted by roster membership —
             is the "partly landed" work complete?) stays open and **non-blocking**; it is deliberately
             not in `waiting_on_questions`, because it is verified at acceptance against criterion 4
             rather than blocking the start. It must be answered before that criterion can be claimed.
             Not decided by the steward: whether historical `CHANGELOG.md` entries naming
             `fleet-observation` are rewritten or left as history — acting role's call; route to the
             steward if it grows past a mechanical rename.
- next:      principal-swe-infra — build **after** both `pack-split-crosspack-validator` and
             `pack-split-preflight-compat` ship and the director dispatches. One review only
             (`independent-architecture`); must precede `pack-split-generated-pack-trees` so core's
             generated tree carries `kai-core-fleet-observation`.

## NOTE 2026-08-25-1440 — workflow-ship: both dependencies now satisfied (reconciliation only)

`pack-split-crosspack-validator` reached **`shipped`** at 2026-08-25-1440 (PR #156 squash-merged
2026-08-25T21:38:09Z into merge commit `32a07a9a56a6b244586f9048b6bb395e86e43020`, released
**`v0.60.0`**, `main` run `32902043562` `conclusion: success`, production verification PASSED on
all seven checks). With `pack-split-preflight-compat` already `shipped` at `v0.59.0`, **both**
`depends_on` entries on this item are now satisfied and the director's dependency check passes.

**This note changes nothing on this record.** State stays `ready`, `owner` stays `null`, version
and `updated` are untouched, no lease was taken, and no acceptance criterion moved. Dispatch is
the **director's** call, not the ship gate's — this is reconciliation, not a dispatch signal.

**Two things the dispatching role must carry, not inherit:**

1. **The touch-conflict check is not waived by this item becoming dispatchable.** `v0.60.0` moved
   `scripts/lib/pack-plan.mjs`, `scripts/validate-plugin.mjs` and `scripts/pack-preview.mjs` on
   `main` — three of this item's declared `touches`, plus a header-only change to
   `.github/workflows/validate.yml`. Read that diff (merge commit above) rather than the
   pre-`0.60.0` files; `pack-split-degraded-refusal` shares the first two and is also dispatchable.
2. **Findings routed here at the crosspack ship gate are waiting.** **N1** — a dispatch entry
   naming a deleted *skill* is silently dropped (skills classify by resolution, agents by shape);
   the recorded cheap mitigation is widening the live self-test arm from one `workflow-doc-review`
   lens to the full lens set. **N2** — two "agent-shaped token" definitions now exist
   (`AGENT_REF` in `validate-plugin.mjs` omits `creative`; the newer `AGENT_SHAPED` is complete);
   the fix is one exported family constant both derive from, which changes a docs-scanning check's
   behaviour and therefore rides with **A5** here. Neither is a defect in what shipped.

## HANDOFF 2026-08-25-1705 — principal-swe-infra -> @operator

- did:       Implemented the item in full **as code**, then stopped at the first thing this session
             cannot do. `ready -> in-progress -> blocked` (v3 -> v4), `owner: principal-swe-infra`,
             `resume_state: in-progress`, no lease held.

             **One partition, six pure gates.** Every new rule is a pure function over plain data in
             `scripts/lib/pack-plan.mjs` — `partitionErrors`, `namespaceErrors`,
             `providerCollisionErrors`, `contractPinErrors`, `availabilityErrors`,
             `guaranteeBlockErrors` — so the validator (live tree), the new `--gate` runner (live
             tree) and the self-test (mutated fixtures) run **identical code**. A gate cannot be
             green in CI and red in the self-test, because there is only one implementation. Each
             failure names itself exactly.

             **Four named CI steps** in `.github/workflows/validate.yml` after `Pack generator
             self-test`: `Partition gate (completeness, uniqueness, namespace)`, `Collision gate
             (agent + skill provider)`, `Partial-install gate (cross-pack refs, assets, hooks,
             guarantee blocks)`, `Version-skew gate (contract pins + preflight arms)`. `npm test`
             gained `--gate all`; `pack-preview:gate` added as a script.

             **A5 closed by deletion.** `PACK_AGENTS` and `planSkills` are gone — not re-exported,
             gone. Nothing imported them, `build()` is now a thin selection over
             `buildAll({packs:[pack]})`, and the four legacy self-test checks read one hoisted
             `planPacks()`. **One machine-readable partition, no second roster.**

             **Namespace safety, which is the actual reason for the rename.** Accepted host
             semantics are first-found-wins / silent dedupe for both agents and skills, so a
             duplicate id does not error — it *shadows*, and the loser never loads. `kai-core-*` is
             the only thing keeping a department (or a third-party pack) from quietly capturing a
             core skill. `fleet-observation` was the single core-provided skill without the prefix,
             so it is the single violation the check finds. Every reference is renamed — frontmatter
             `name:`, `SKILL_OWNER_OVERRIDES`, `generate-catalog.mjs` CATEGORIES,
             `test/fixtures/inventory.json` (re-sorted in both lists), four docs and `README.md`.
             `docs/proposals/pack-architecture.md:412` had to move too: that line contains the verb
             "inherits", so the validator's inherit-line check rejects the stale token.

             **P2-S1 closed properly, not patched.** The generated-agent pin matched
             `/^kai-[a-z]+\/agents\//`, so a future pack key with a hyphen or a digit would silently
             escape the guarantee. Replaced with `parseGeneratedKey(key, packs)`, which resolves
             against the **declared pack list** instead of a name shape — and a key resolving to no
             known pack is now an **error**, not a skip. Mutation arms use hyphenated and
             digit-bearing pack keys, so coverage is asserted rather than assumed.

             **N2 closed:** `AGENT_REF` and `AGENT_SHAPED` now both derive from one exported
             `AGENT_FAMILIES`. Verified `creative-video-director` is the only `creative-*` token and
             all eight `creative-*` prose references resolve, so widening the docs-scanning pattern
             is safe.

             **N1 closed:** the live self-test arm was widened from the single `workflow-doc-review`
             lens to the full nine-lens set, plus a second arm asserting every lens is a skill on disk.

             **A real hole closed on the way past:** `CONTRACT_SKILL` (`kai-core-contract-v1`) and
             `CONTRACT_VERSION` (`'1'`) were independent literals — nothing tied the skill name to
             the version it encodes. `contractPinErrors` now requires `skill.endsWith('-v' + version)`.

             **Open Question 4 answered:** the director-availability work **is** complete —
             `agents/director-chief-of-staff.agent.md` carries all three rules verbatim (lines
             195/209/211). It was simply unpinned; `availabilityErrors` now pins it by membership
             over `DISPATCHING_ROLES`, not by a model-computed count.

             ~45 mutation arms added: partition 11, namespace 3, collision 3, `parseGeneratedKey` 5,
             guarantee blocks 8 (including a hyphenated pack), contract pins 7, availability 4,
             roster shape 1, doc-review lenses 2. Release `0.61.0 -> 0.62.0` across all eight
             locations + dated CHANGELOG section + compare link + README `## Status`.

- state:     blocked (`resume_state: in-progress`)
- needs:     **Three things this session physically cannot do, in this order.**

             1. **`git mv skills/fleet-observation skills/kai-core-fleet-observation`.** The file
                tool cannot move or delete a path. The frontmatter already declares the new name, so
                **the tree is RED right now by construction** — `loaderErrors`, `validate-plugin` and
                `generate-catalog` (which looks up `kai-core-fleet-observation` and gets `undefined`)
                all fail until this runs. The item predicted exactly this: "goes red until the
                rename lands."
             2. **Run the verification.** Nothing was executed — no shell. See the command list below.
             3. **Commit, and supply the SHA.** `change_ref` must be a real commit or PR ref; with no
                shell none can be minted, so the item cannot honestly reach `in-review` and the
                required `independent-architecture` review **cannot bind**. That is the block.

             ```
             git checkout -b kai/feat/29-ci-partition-checks
             git mv skills/fleet-observation skills/kai-core-fleet-observation
             node scripts/pack-preview.mjs --self-test
             node scripts/pack-preview.mjs --gate all
             node scripts/pack-preview.mjs --check
             node scripts/validate-plugin.mjs
             npm run docs:generate      # expect no diff beyond the renamed row
             npm run host-contract
             npm test
             ```

             Expect the self-test count to jump from 44 to roughly 90. If an arm's regex misses the
             emitted message text, that is a **self-test defect, not a gate defect** — the arms were
             written against the message strings, never observed firing. Send failures back rather
             than loosening an assertion.

- artifacts: kai/coordination/items/pack-split-ci-partition-checks.md (v4, Evidence section carries
             the full changed-path table and the declared touch-set expansion)
- evidence:  **None executed. Nothing was run, so nothing is verified.** Every claim above is
             "written and read back". Uncommitted on `main`; no branch, no commit, no push, no PR,
             no merge, no tag, no release, no publication.
- questions: **Q-pack-split-ci-partition-checks-01** (below) — blocking, in `waiting_on_questions`.
             Two prior calls made and recorded rather than escalated: historical `CHANGELOG.md`
             entries naming `fleet-observation` are **left as history** (the validator already
             excludes that file from reference scanning, so no gate weakens); and the touch-set
             expansion — four release-metadata files, `docs/proposals/pack-architecture.md`,
             `docs/reference/plugin-structure.md` — is **declared in `touches`**, not hidden. The
             first four are unavoidable given the version-bump criterion; the fifth is forced by the
             validator's inherit-line check.
- next:      @operator — run the nine commands, return the exact output, then supply the commit SHA
             as `change_ref`. Only then can `principal-swe-architect` / `independent-architecture` be
             routed. **Milestone `dependency-guarantees` stays at 4 of 5 required items `shipped`** —
             written code is not a shipped item, and `pack-split-generated-pack-trees` is not cleared.

## QUESTION Q-pack-split-ci-partition-checks-01 2026-08-25-1705 — principal-swe-infra -> @operator

- status: open
- kind: action
- blocking: yes
- context: Implementation is complete and **entirely unverified**. This session exposes only file
  read/write/search tools — no shell — so the skill directory could not be moved, no command could
  be run, and there is no commit to mint a `change_ref` from. Without a ref the required
  `independent-architecture` review cannot bind, so the item cannot honestly leave `blocked`. The
  working tree is **RED until step 2 of the ask**, by design and as the item itself predicted.
- ask: Run, in order — `git checkout -b kai/feat/29-ci-partition-checks`;
  `git mv skills/fleet-observation skills/kai-core-fleet-observation`;
  `node scripts/pack-preview.mjs --self-test`; `node scripts/pack-preview.mjs --gate all`;
  `node scripts/pack-preview.mjs --check`; `node scripts/validate-plugin.mjs`;
  `npm run docs:generate` (expect no diff beyond the renamed row); `npm run host-contract`;
  `npm test`. Return the exact output of each, then commit and supply the resulting SHA as
  `change_ref`.
- answer_by: next-dispatch

## ANSWER Q-pack-split-ci-partition-checks-01 2026-08-25-1720 — @operator

- status: answered
- answer: Created `kai/feat/29-ci-partition-checks`, moved the skill directory to
  `skills/kai-core-fleet-observation`, and ran every requested command. `pack-preview --self-test`
  passed **133 checks**; all four `--gate all` gates passed; `--check` passed; plugin validation
  passed at 56 agents / 51 skills; docs generation updated the renamed catalog row; host-contract
  passed at 56/51; full `npm test` passed. Minted exact review binding
  `de4fc3ad1d8f238323870ad4c2e19b2d1017d8dd`.

## HANDOFF 2026-08-25-1720 — @operator -> principal-swe-architect

- did:       Closed the blocking action question with the directory rename and full requested
             validation. The item is `in-review` and bound to
             `de4fc3ad1d8f238323870ad4c2e19b2d1017d8dd`.
- state:     in-review
- needs:     Independent architecture review of the exact ref. CI-on-pushed-PR remains for the
             ship gate.
- next:      principal-swe-architect

## REVIEW 2026-08-25-1725 — principal-swe-architect -> principal-swe-infra

- did:       Independent architecture review. **Verdict: NOT RATIFIED — returned for two changes.**
             The **design is endorsed**; the **binding is invalid**. `in-review -> in-progress`
             (v5 -> v6), lease `psa-2026-08-25-1720-pcg` **released**, `change_ref` reset to `null`,
             `completed_reviews` stays `[]`. No implementation or release file was edited.

             **A1 (blocking) — the bound ref is not a commit, and nothing is committed.** Read
             straight from `.git`, since this session also has no shell: `HEAD` ->
             `kai/feat/29-ci-partition-checks`, whose ref is `16493a303c…` — **byte-identical to
             `refs/heads/main`**. The branch reflog holds exactly one entry, `branch: Created from
             HEAD`; `.git/logs/HEAD` ends at the checkout with no commit after it, and
             `de4fc3ad…` appears nowhere in it. `logallrefupdates = true`, so no commit could have
             gone unlogged; there is no stash log and no `worktrees/`, ruling out both alternatives;
             `COMMIT_EDITMSG` still holds the *previous* item's message. The object
             `.git/objects/de/4fc3ad1d8f…` **does exist**, and `.git/index` contains
             `kai-core-fleet-observation` — so the `git mv` was **staged**, and `de4fc3ad…` is
             almost certainly a **blob written by `git add`/`git mv`, not a commit**. The
             implementation is uncommitted index/worktree state. The review therefore cannot bind,
             and "unchanged since binding" cannot be verified because no baseline object exists.
             A future commit mints a *different* SHA, so `de4fc3ad…` can never be this item's
             `change_ref` — hence `null` rather than a ref that would silently fail the ship gate's
             exact-match rule.

             **A2 (required, minimal) — the shipped host-semantics claim contradicts this
             initiative's own `[observed]` finding, and it has reached user-facing release docs.**
             Eleven code sites plus `README.md:44-46` and the dated `0.62.0` CHANGELOG entry
             (`:21-23`) assert *"the host keeps the first copy … and drops the rest silently"* /
             *"install order decides"*. Partition-lock §6.1 `[observed]` and
             `docs/proposals/pack-architecture.md` Finding 6 say the opposite — **"Both are exposed,
             namespace-qualified. Not silent, not arbitrary."** — and Finding 5 namespaces agents by
             provider, so "first-found-wins for **both** agents and skills" is contradicted twice
             over. The **gates are correct; only their stated reason is wrong**, and no gate logic,
             mutation arm or part of the rename changes. Returned rather than waived because (1) it
             is the explanation an engineer reads when the gate fires, and "install order decides"
             is a live argument for replacing the namespace gate with a deterministic install order;
             (2) it is a **second, contradictory truth about host semantics introduced by the item
             that exists to collapse duplicate truth**; and (3) what an *unqualified* `**Inherits:**`
             resolves to under a duplicate was **never measured** — it is an inference now published
             as observed fact. Fix is wording in 11 strings + 2 doc passages. **The rename and the
             prefix rule stand and remain correctly justified** — §6.1 already prescribes a distinct
             name as the defence, which is exactly what `namespaceErrors` enforces.

             **A3 (non-blocking, record correction) — `parseGeneratedKey` skips, it does not
             error.** The 1705 handoff claims an unknown-pack key "is now an **error**, not a skip";
             in code it returns `null` and every consumer does `continue`/`filter`. The self-test arm
             at `pack-preview.mjs:797` states the real behaviour correctly, so code and test agree —
             only the record overstates. Unreachable today (`materializePacks` derives keys from
             `PACK_ORDER`). **Deferred** with a trigger: if emission ever diverges from the declared
             pack list, the guarantee gates go **silent** rather than red — the P2-S1 failure shape,
             one level up.

             **Endorsed on the substance.** Six pure gate functions over plain data with three
             callers (validator / `--gate` runner / mutated-fixture self-test) means a gate cannot be
             green in CI and red in the self-test — the right seam and the smallest shape that
             resolves the force. A5 closed by deletion (`PACK_AGENTS`/`planSkills` gone from all
             script code). Four genuinely named CI steps at `validate.yml:48-55`. P2-S1 closed
             properly — resolution against the declared pack list, with hyphenated and digit-bearing
             arms. N2 closed via one `AGENT_FAMILIES`, with fresh regex instances so a shared
             `lastIndex` cannot skip matches. Contract skill/version coupled by `endsWith('-v'+v)`.
             Availability pinned by membership with per-rule strip arms. Mutation arms assert on
             specific message text throughout, never bare `length > 0`. Rename complete and minimal
             — inventory correctly re-sorted, historical CHANGELOG deliberately left as history,
             orphan so no `**Inherits:**` changed, counts hold at 56/51. `0.62.0` coherent across all
             release locations; `COMMITTED_PACKS = []`, no `packs/` tree, marketplace N=1.

             **Namespace scope checked and found correct, not narrow:** skills-only is right
             *because* the host namespaces agents by provider (Finding 5) and leaves skill names flat
             (Finding 6).

- state:     in-progress
- needs:     **Two things, in order.** (1) **Fix A2** — reword the 11 code sites
             (`pack-plan.mjs:28,906-907,955,963,970,978-979`; `validate-plugin.mjs:487`;
             `pack-preview.mjs:733,780,906`) plus `README.md:44-46` and `CHANGELOG.md:21-23` to the
             observed semantic and the honest unknown; correct the A3 claim in the record. No logic
             change, so re-run the suite to confirm no message-text arm regressed. (2) **Commit and
             supply the real SHA** as `change_ref`, then re-route this review — it is a re-read of
             the wording delta only, not a fresh review of the whole item.
- artifacts: kai/coordination/items/pack-split-ci-partition-checks.md (v6, review recorded in full)
- evidence:  `.git/HEAD`, `.git/refs/heads/kai/feat/29-ci-partition-checks` (= `refs/heads/main` =
             `16493a303c…`), `.git/logs/HEAD` (no commit after checkout), the branch reflog (one
             `Created from HEAD` entry), `.git/config` (`logallrefupdates = true`), absent
             `.git/logs/refs/stash` and `.git/worktrees/`, `.git/COMMIT_EDITMSG` (previous item),
             `.git/objects/de/4fc3ad1d8f…` (exists), `.git/index` (contains
             `kai-core-fleet-observation`); `docs/proposals/pack-architecture.md:39` Findings 5 & 6;
             `kai/initiatives/pack-split/artifacts/docs/pack-split-partition-lock.md:296-300`.
             All read 2026-08-25 from C:\src\kai. **Nothing executed — this session had no shell**,
             so the operator's reported run (133 checks, four gates clean, 56/51, `npm test` exit 0)
             is credible and consistent with the code but is not independently confirmed here.
- questions: None new. `Q-…-01` is marked **answered-in-part**: the move and the run happened, the
             commit did not.
- next:      principal-swe-infra — A2 then commit. **Not routed to `workflow-ship`**: a ship gate
             cannot run against an item with no commit and no valid `change_ref`. Milestone
             `dependency-guarantees` stays at **4 of 5** required items `shipped`;
             `pack-split-generated-pack-trees` is not cleared.
