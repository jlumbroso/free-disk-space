# Decision records

This directory holds the reasoning behind `free-disk-space` — why it exists, what
it inherited, what it deliberately does not do, and what is still open.

**Most of it was written four years after the fact.** The action was built in
2022 and maintained without a decision log; these records were reconstructed in
2026 from the repository, its pull requests, the sources it cites, and the
author's own answers to questions the artifacts could not settle.

That reconstruction was **mostly unsuccessful**, and saying so is the point of
this page.

**One record here is not a reconstruction at all.**
[0007](0007-swap-storage-defaults-to-false-the-policy-acquires-a-boundary.md) was
decided and written on the same day. Read it next to any of the others and the
difference in what survives is the whole argument.

---

## How to read these

Every section of every record is marked with what kind of claim it is:

| mark | means |
|---|---|
| **Record** | The reasoning is in the evidence. A commit message, an inline comment, a pull-request thread, or the author's own words. Cited, not inferred. |
| **Reconstruction** | Inferred from evidence. Carries the evidence, a confidence, and a **falsifier** — what would show it wrong. |
| **Absent** | A decision demonstrably happened and its reasoning is **not recoverable**. Marked as a gap. No record is written, because writing a plausible one is worse than admitting the gap. |

**Where a record says "Absent", nobody knows.** That is not an invitation to
guess; it is the honest state of the evidence.

## What is still open

**Twelve questions across seven records are unanswered** (fifteen are posed;
three are answered). They are marked `Status: unanswered` and they are real — several
are decisions a contributor could reasonably argue, not just gaps in the author's
memory:

- Should the deletion-only boundary move to admit filesystem manipulation?
  (bears on [#49](../../pull/49), [#21](../../issues/21))
- Should package isolation be systematic rather than case-by-case?
  (bears on [#41](../../issues/41), [#42](../../pull/42))
- Should measurement extend from space to time? ([#25](../../issues/25))
- Should the helper functions be renamed, and by what principle?
  (bears on [#28](../../pull/28), [#37](../../pull/37), [#46](../../pull/46))
- Now that `swap-storage` has flipped to `false`, does the same reasoning reach
  `tool-cache`? ([0007](0007-swap-storage-defaults-to-false-the-policy-acquires-a-boundary.md))

If you have a view, the records are the place to put it.

---

## The records

| | what it decides | mostly |
|---|---|---|
| [0001](0001-why-this-action-exists-and-why-it-is-public.md) | Why this exists, and why it is public rather than a private script | Record |
| [0002](0002-reclaim-space-by-deletion-not-by-repartitioning.md) | **Delete, don't repartition** — and why the alternative was declined | Mixed |
| [0003](0003-inherited-foundations-what-came-from-where.md) | What came from `apache/flink`, what came from `ShubhamTatvamasi`, what is original | Reconstruction |
| [0004](0004-tolerate-per-command-failure-rather-than-aborting.md) | Failure is non-fatal per command, and reported rather than swallowed | Record |
| [0005](0005-report-what-each-option-actually-saved.md) | Report what each option actually freed | Record |
| [0006](0006-the-option-set-grew-it-was-not-designed.md) | The option set accreted; it was not designed | Record |
| [0007](0007-swap-storage-defaults-to-false-the-policy-acquires-a-boundary.md) | **`swap-storage` defaults to `false`** — and the defaults policy gains a stated limit | Record *(written the day it was decided)* |

**Start with 0003 if you are here to change something.** Of three long-standing
defects, two turn out to be inherited rather than authored, and the third exists
only in the composition — which changes how all three should be judged.

**0007 is different from the rest, and deliberately so.** It was decided and
written on the same day, 2026-09-16, with no reconstruction in it. It is what the
other six were supposed to look like.

---

## What the reconstruction cost, stated plainly

Eight substantive reconstructions were attempted across this repository and one
other in the same estate. **Two were right.**

Each wrong one was plausible, cited real evidence, and carried a stated
confidence — and each was wrong about the thing that mattered, until the author
was asked directly. The recurring failure was reaching for a generic engineering
explanation where the truth was specific and external: an adversary's move, a
change of employer that freed up time, a reciprocity ethic.

**This is the argument for writing decisions down at the time, and it is
empirical rather than rhetorical.** The information that would let you recover a
decision from its artifacts is destroyed by the act of building. Some of it is
not merely hard to recover but *unrecoverable in principle* — a decision **not**
to do something leaves no line of code to attach to, so no amount of reading the
source will find it. [0002](0002-reclaim-space-by-deletion-not-by-repartitioning.md)
is exactly that kind of decision, and it exists here only because someone was
asked.

### And the scoreboard above is probably flattering

On 2026-09-16 the author read all six records and corrected one: the `true`
defaults, recorded here as *never deliberated*, had in fact been decided under a
policy he could still name. See
[0001](0001-why-this-action-exists-and-why-it-is-public.md) §4.

What makes that worth putting on the front page is **how the error survived**. It
was not an unchecked inference. It was asked about, in writing, and the author
answered *"Yes, perfect read, all throughout"* — to a question that had bundled
three separate claims under one heading. A blanket yes to a compound question
cannot say which part it agrees with. The wrong part was promoted to **Record**,
the strongest mark in the table above, and stopped looking like a guess.

**So the two-of-eight tally counts only the reconstructions that stayed marked as
reconstructions.** A wrong one that collected an assent left the denominator
entirely. At least one did. There is no way to know how many others would, which
is a more uncomfortable result than the tally and a more honest one.

The same day produced the correction in the other direction: the author believed
the action had never been published to the GitHub Marketplace. It had been
(0001 §2a). **Memory is strong on reasons and weak on acts; artifacts are the
reverse.** Neither is authoritative, and a method that trusted either one
categorically would have shipped one of these two errors.

---

*Reconstructed by Sherd 5 (Claude Opus 5) with Jérémie Lumbroso, August–September
2026. Method, including its wrong turns: the campaign's protocol log. Sources
cited by these records — including two that have changed since 2022 — were
retrieved at the revisions in force when this action was written, not at their
current state.*
