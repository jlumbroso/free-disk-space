# ADR-0002: Reclaim space by deletion, not by repartitioning

- **Date of decision**: 2022-06-22 (implicit in the first working version)
- **Date of record**: 2026-08-31  |  **Iteration**: 2 (2026-09-16)
- **Status**: Accepted — and load-bearing for open items PR #49 and issue #21
- **Deciders**: Jérémie Lumbroso

> ### 📜 Partial reconstruction — and this one is genuinely mixed
>
> The **principle** below is Record: the author states side-effect caution as his
> approach, in his own words. The **specific application** — that this is why
> `easimon/maximize-build-space` was cited and not adopted — is Reconstruction,
> with confidence and falsifier stated, and an open question raised with him
> rather than resolved here.
>
> Marked this finely because the two halves have different evidentiary weight and
> collapsing them would overstate the record.

**TL;DR**: There are two ways to give a CI runner more usable disk: **delete
things**, or **repartition the disk**. This action deletes. The alternative was
known and cited at the time and deliberately not taken. Deletion's side effects
are *local and legible* — a package is gone — whereas repartitioning's are
*systemic*: it changes where the build runs, destroys swap, constrains workflow
ordering, and leaves kernel state behind.

---

## Context

**Record.** The problem, from ADR-0001: a musicology pipeline processing scanned
manuscript facsimiles in CI, against a runner with ~14 GB free.

**Record.** Two approaches existed in 2022, and the README cited both:

| approach | representative | mechanism |
|---|---|---|
| **deletion** | `apache/flink`'s `free_disk_space.sh`, `ShubhamTatvamasi/free-disk-space-action` | remove packages and directories that are installed but unused |
| **repartitioning** | `easimon/maximize-build-space` | build an LVM volume spanning the root and temp disks, mount it as the build path |

Both are in this repository's README Acknowledgement section. Only one is in the
code.

---

## Decision

**Delete. Do not touch the filesystem's structure.**

### Why — the principle

**Record**, the author, 2026-08-31:

> "I think part of my approach was **epistemic caution** […] knowing whether
> there would be unexpected side effects or not to the suggestions."

and, of the cited sources specifically:

> "I think a lot of these had some side effects."

### Why — the application to this choice

> **Reconstruction.** The author has not stated that this is *why* the
> repartitioning approach was declined. What follows is inference from the
> artifact and from what the alternative actually does.

`easimon/maximize-build-space` reclaims space by:

```
fallocate → losetup → pvcreate → vgcreate → lvcreate → mkfs → mount
```

Its consequences reach beyond disk usage:

1. **It changes where the build runs.** `build-mount-path` defaults to
   `$GITHUB_WORKSPACE`, so the workspace becomes a mounted LVM volume. Anything
   that inspects the filesystem, relies on hardlinks across devices, or assumes
   the default layout behaves differently.
2. **It destroys swap non-optionally** — `swapoff -a`, in order to reuse `/mnt`.
3. **It constrains the caller's workflow.** It must run *before* checkout,
   because it creates the mount point the checkout lands in.
4. **It leaves kernel-level state** — loop devices and volume groups — in place
   for every subsequent step in the job.

Deletion has none of these properties. After this action runs, the runner is
structurally identical; only files are absent. A user reasoning about a failure
downstream has one fact to hold — *that package is gone* — rather than a changed
device topology.

*Confidence: **medium-high**. The two approaches were both cited, only one was
implemented, and the author independently describes side-effect caution as his
governing concern. What I cannot evidence is the counterfactual — whether he
evaluated LVM and declined it, or never seriously considered it.*

*Falsifier: **if wrong** — if repartitioning was skipped for effort or
unfamiliarity rather than judgment — then this ADR credits a deliberate boundary
where there was an accident, and the triage guidance below rests on nothing.
**What would settle it**: one sentence from the author. Raised as
`QST-DELETION-VS-LVM` in the onboarding seed rather than assumed here.*

---

## Consequences — and this is the operative part

**This decision draws a boundary, and the boundary is what makes triage cheap.**
Two open items sit directly on it:

### PR #49 — "add chroot to make `rm`'s relative to it" (@milkpirate, 2026-01-10)

Introduces a `chroot`, changing the filesystem semantics under which removals
happen. Its author writes: *"please review carefully, I might have missed
something!"*

**Under this ADR the question is no longer open-ended.** It is: *does this keep
side effects local and legible?* A `chroot` changes the frame of reference for
every path in the script — which is a structural change of exactly the class
this action exists to avoid. That is not an automatic rejection, but it relocates
the burden: the PR must argue why this case is different, rather than the
maintainer having to reconstruct from scratch what the design was protecting.

### Issue #21 — "free github runner space in container env" (@wuwentao, 2024-03-11)

Asks the action to work inside a job running in a container. This is the same
boundary seen from the other side: the container *already* changes the
filesystem frame, so the action's assumptions about paths no longer hold.

**Under this ADR it becomes a scope question with a stated principle to answer
it** — is a container environment inside the design's boundary or outside it? —
rather than an open-ended feature request.

### The general consequence

Without this ADR, both items require rediscovering the design's intent before
they can be judged. With it, both become *askable* in a sentence — which is less
than answered and is the whole gain. The ADR does not decide PR #49; it converts
it from "what was this even for?" into "does this keep side effects local?", a
question the maintainer can hold in one hand. **That gap is the
mechanism the author named as epistemic paralysis**: *"I don't remember all my
choices, and so I'm just filled with dread of breaking something I don't
understand."* PR #49 has been open since January; issue #21 since 2024.

---

## What this ADR does not settle

- **Whether the boundary should hold.** It records where the line was drawn and
  why it plausibly was. Moving it is a live decision, and PR #49 is a reasonable
  request to move it. This ADR exists so that moving it is *deliberate*.
- **The counterfactual** (see the falsifier). Open as `QST-DELETION-VS-LVM`.

---

## Questions

### QST-DELETION-VS-LVM: Was the repartitioning approach evaluated and declined, or never seriously considered?
- Status: unanswered
- Why asking: This is the falsifier for this ADR. It credits a deliberate design boundary; if the boundary was an accident, the triage guidance below it rests on nothing.
- Need: one sentence; "never thought about it" is complete and useful
- Options:
  - **A — Declined on the merits.** You looked at `easimon/maximize-build-space`, understood that it repartitions, and judged the side-effect profile unacceptable for your use. The boundary is a principle.
  - **B — Never seriously considered.** You found a deletion-shaped solution first, it worked, and the repartitioning approach was cited as prior art without ever being a live candidate. The boundary is a path taken.
  - **C — Considered and deferred.** It was plausible but looked like more work than you had time for; not rejected, just never reached.

*Handle note: this question is `QST-DELETION-VS-LVM` throughout — the handle it
carries in the onboarding seed, where it was first raised. An earlier draft of
this ADR retitled the heading to QST-LVM-CONSIDERED (unbackticked here so it
reads as history, not as a live citation) while the body kept citing
the original, so a reader grepping the cited handle would have found nothing.
Corrected 2026-09-16 on Shipwright 5's catch; the original handle wins because it
was cited first and cited most.*

**Recommendation**: (by Sherd 5, Claude Opus 5)

**A — Declined on the merits.** Because both approaches are cited in the README and only one is
implemented, and the author independently describes side-effect caution as his
governing method. If wrong: this ADR turns an accident into a principle — the
flattering direction of error, and therefore the one to distrust.

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

### QST-BOUNDARY-MOVE: Should the deletion-only boundary move to admit filesystem manipulation?
- Status: unanswered
- Why asking: **Two open contributions are asking exactly this** — PR #49 (`chroot`) and issue #21 (container environments). This ADR makes the question answerable; it does not answer it. Contributors should be able to argue it here rather than rediscovering the boundary each time.
- Need: a ruling, or a contributor's argument in ORRCF form
- Options:
  - **A — Hold the line.** Deletion only. Filesystem manipulation is out of scope permanently; PR #49 and issue #21 are closed with the boundary stated as the reason.
  - **B — Hold, with a stated burden.** The boundary stands, but it is written down along with what an argument to move it would have to show. Contributors can argue *within* a frame instead of guessing at one.
  - **C — Move the boundary behind an opt-in.** Admit filesystem manipulation as a non-default input, so the blast radius is chosen rather than inherited — the same shape as the `swap-storage` question in #12.
  - **D — Move it fully.** The action becomes "reclaim space by whatever means," and the reasoning-locally property is traded for reach.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**B — Hold the boundary, and make the burden explicit rather than implicit.** Because
the action's value is that a user can reason about its effects locally — *this
package is gone* — and `chroot` or container support changes the frame of
reference for every path in the script. If wrong: the action stays unusable in
container-based jobs, which is a real and growing class of CI, and issue #21 goes
unanswered for a fourth year.

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

## Evidence

| claim | source |
|---|---|
| Both approaches cited in the README | `README.md`, Acknowledgement, since `6111681`, 2022-06-22 |
| `easimon`'s mechanism and side effects | `evidence/free-disk-space/cited-sources/easimon-action.yml`, read 2026-08-30 |
| Deletion-only implementation from v1.0 | `action.yml` @ `3a49c22`, 2022-06-22 |
| Side-effect caution as the author's approach | Author's testimony, 2026-08-31 |
| PR #49, issue #21 as boundary cases | the issue and PR threads themselves — [#49](../../pull/49), [#21](../../issues/21) |

*Reconstructed by Sherd 5 (Claude Opus 5), 2026-08-31. The cited-but-unused
source was fetched and read before this was written; see
`evidence/cited-sources/` for the archived sources this comparison used.*
