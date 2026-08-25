# Active initiatives

Operational focus pointer, not the permanent record. Each row names an
initiative `slug` currently receiving attention and, briefly, why. This file
lists only the current focus — `kai/initiatives/INDEX.md` is the permanent
all-status catalog, and removing a terminal initiative from this file must
never make it undiscoverable there.

| slug | why active |
|------|------------|
| pack-split | `partition-lock` **completed**; `pack-split-engineering-decomposition` **completed**. Focus is `dependency-guarantees`. The foundational item **`pack-split-generator-gates`** is steward-ACCEPTED and architecture-RATIFIED at `change_ref 457254b97…`, but the **`workflow-ship` DoD gate BOUNCED it (2026-08-24-2244): 5 dimensions Clear, 1 Gap** — remote CI has never run, so acceptance criterion 5 has no evidence and cannot get any until the branch is pushed. State **`in-progress`**, **`next_role: principal-swe-infra` for PR delivery** under `kai-core-pr-delivery`; the reviewed tree must be committed byte-identically or the ratified review stops binding. **Nothing is committed, pushed, PR'd, merged, tagged, released, or shipped.** Architect findings routed: A1–A3 → `generated-pack-trees`, A4 → `release-12b`, A5 → `ci-partition-checks`, A6 parked in the backlog. The other 12 records stay `proposed`; `pack-split-host-semantics-spike` recommended for parallel operator dispatch. Non-blocking: **3** open questions remain (review-lens binding, first-department `shipped` semantics, director-availability completeness) — the `packs/` committed-tree root is now **confirmed**. |

The pack-split initiative is the current focus.
