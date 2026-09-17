# ADR-0007: `swap-storage` defaults to `false` — and the defaults policy acquires a boundary

- **Date of decision**: 2026-09-16  |  **Date of record**: 2026-09-16  |  **Iteration**: 1
- **Status**: Accepted — decided; implementation pending (a release change)
- **Deciders**: Jérémie Lumbroso, on the argument of **@zaikunzhang** (issue #12, 2023-09-10)

> ### 📜 This one is not a reconstruction. It is the first record in this repository written *at the time*.
>
> ADRs 0001–0006 were reassembled four years after the fact, and the campaign that
> produced them exists because that is expensive and lossy. **This decision was
> made on 2026-09-16 and written down on 2026-09-16.** There is no inference in
> it, no confidence line on the rationale, and no falsifier on the *reasoning* —
> only on the prediction. The reasoning is quoted from the person who did it.
>
> That difference is the whole argument of the preceding six records, so it is
> marked rather than left for a reader to notice.

**TL;DR**: `swap-storage` now defaults to **`false`**. Not because the
forgiveness-not-permission policy (ADR-0001 §4) was wrong, but because
@zaikunzhang's three-year-old report identified the condition under which it
fails: **the policy assumes over-deletion is cheap because the user can tell what
happened, and removing swap produces an OOM-killed runner that looks like
anything at all.** The policy keeps its scope and gains a stated limit. A FAQ
carries the side-effect information for every option.

---

## Context

**Record.** ADR-0001 §4 records the defaults policy, in the author's words:

> "I was mirroring *ask for forgiveness, not permission* — which is delete
> everything you can. And then as you find out that things are missing, you can
> switch them back on."

**Record.** Issue #12 (@zaikunzhang, 2023-09-10, 11 👍 — the repository's
most-supported item) argued for flipping `swap-storage`:

> "Otherwise, some memory-consuming actions may get the runner shut down due to
> memory starvation — it is not uncommon for a space-intensive action to be
> memory-hungry as well. This did happen in [PRIMA], and **it took a couple of
> days to figure out that `swap-storage: true` was the reason.**"

He also proposed the alternative himself: *"If `swap-storage: true` is going to be
retained, then careful documentation may be necessary to highlight the
above-mentioned caveat."*

**The sentence that decides it is the one about two days.** The policy's premise
is a working feedback loop — delete aggressively, *because discovery is cheap*.
For most options discovery is cheap: a removed package produces an error naming
the package. For swap it is not — the failure is not "something is missing" but
"the job died," and nothing connects the death to this action. Discovery cost a
competent maintainer two days.

*Care with the generalisation, because an earlier draft of this ADR overstated it
and Parallax 6 caught that: it is **not** true that every other option names
itself. `tool-cache` does not — which is exactly why it was already defaulted
`false` in 2022 (ADR-0001 §4a). The claim is about most options, and the
exceptions are the point.*

---

## Decision

**`swap-storage` defaults to `false`.**

**Record** — the author's ruling, 2026-09-16, verbatim:

> "FLIP the default + document the problem that @zaikunzhang experienced in a
> FAQ, like *'what are possible side-effects of these settings?'* That way we get
> the best of both worlds. **I agree that the defaults shouldn't break something
> so elementary as OOM.** We should close @zaikunzhang's issue with this, link to
> their issue in our fix, and make sure they are credited."

### The boundary, stated

The clause *"the defaults shouldn't break something so elementary as OOM"* is the
general principle. **It is newly *stated*, not newly *held*** — ADR-0001 §4a shows
it applied to `tool-cache` in 2022. ADR-0001 §4's policy is unchanged in scope and
now carries its limit in writing:

> **Delete aggressively by default — where the failure is attributable.** An
> option is not a candidate for an aggressive default if its failure mode is
> (a) a *fundamental resource failure* rather than a missing artifact, and
> (b) *non-attributable* — presenting as something with no visible connection to
> this action.

Both conditions are needed. A missing package is attributable and its absence is
not fundamental: `true` is right. Removing swap is both fundamental and
non-attributable: `true` was wrong. **This is the policy applied, not overridden**
— its premise simply does not hold for this option, and the author is the one who
noticed.

### Options considered

- **A — Keep `true`, document the caveat.** @zaikunzhang's own fallback. Breaks no
  existing workflow; leaves the trap armed for everyone who does not read the
  README.
- **B — Flip to `false`.** Fixes the trap; silently reclaims less space for every
  existing consumer who relied on the default.
- **C — Flip to `false` *and* document side effects for every option in a FAQ.**
  ← **chosen.** His words: *"the best of both worlds."*
- **D — Flip, and emit a warning when `swap-storage: true` is set.** Considered
  here, not chosen: the action already has a severity convention (ADR-0004) and a
  user who explicitly opts in has made an informed choice; warning them is noise
  of exactly the kind the `::debug::` refinement exists to avoid.

### Rationale

Four reasons, in the order they carry weight:

1. **The failure is out of proportion to the benefit.** Swap removal reclaims a
   few GB; the cost when it bites is a dead job and days of misdirected
   debugging. No other option has that asymmetry.
2. **The evidence is unusually strong for this backlog.** A named user with a
   reproduced case, a diagnosis, a proposed fix, a proposed fallback, and 11 👍
   across three years. Nothing else in the backlog is this well-evidenced.
3. **A default is a claim about the common case**, and the common case for this
   action is a space-constrained CI job — which, as @zaikunzhang put it, is *"not
   uncommon"* to be memory-hungry as well. So the option most likely to be harmful
   is one many users will have switched on without considering it.
4. **The same judgment was already made once, in this repository.** `tool-cache`
   was defaulted `false` in 2022 for precisely this reason (ADR-0001 §4a). This
   ruling is the second application of an existing standard, not a new one.

### Consequences

- **This is a behaviour change**, and the only one in the reconstruction push's
  vicinity. It also makes the action's defaults *uniform under the stated rule for
  the first time*: after the flip, every option whose failure is non-attributable
  defaults `false`, and every option whose failure names itself defaults `true`.
  That was never true before, and nobody could have checked it, because the rule
  was not written anywhere. It ships with a release, not with the documents. Workflows pinned to
  `@main` reclaim less space after the release; tag-pinned workflows are
  unaffected until they move.
- **Anyone who actually wanted swap removed must now say so** — one line,
  `swap-storage: true`. That is the correct direction for the cost asymmetry: the
  explicit choice belongs with the destructive option.
- **A FAQ is a new surface for this repository**, and the first documentation
  whose organising question is *side effects* rather than *features*. Draft:
  `FAQ-DRAFT-side-effects.md`.
- **Issue #12 closes as fixed**, not as wontfix, and the commit references it —
  per his instruction, *"link to their issue in our fix, and make sure they are
  credited."*
- **The record now contains a decision with no reconstruction cost.** For every
  other decision in this repository, someone spent hours recovering the reasoning
  or failed to. This one took the length of a sentence, because it was written
  while it was still true.

---

## Questions

### QST-FLIP-GENERALISES: Does the attributability boundary change any other default?
- Status: **answered 2026-09-17 — no, and the reason is better than the question**
- Why asking: `QST-DEFAULTS-INHERITED` (ADR-0006) asked whether the options that inherited `true` by pattern-matching deserve their own ruling. This ADR supplies the criterion that question was missing — *is the failure attributable?* — so it can be applied rather than re-argued.
- Need: the criterion applied to the remaining four options
- Options:
  - **A — No further flips.** Apply the criterion; find that the remaining defaults already satisfy it.
  - **B — Flip `tool-cache` too.** (This was the recommendation until 2026-09-17. It rested on a false premise; see the answer.)
  - **C — Defer** until the `swap-storage` flip has been in the wild.

**ANS:** (by Sherd 5 and Parallax 6, from the artifact — 2026-09-17)

**A — no further flips, and the criterion turns out to have a precedent.**

Applying it to the four remaining options:

| option | failure if wrongly removed | attributable? | default |
|---|---|---|---|
| `android` / `dotnet` / `haskell` | build cannot find the toolchain | yes, names itself | `true` — correct |
| `large-packages` | a command is not found | yes | `true` — correct |
| `docker-images` | images re-pull; slower, not broken | yes, and non-fatal | `true` — correct |
| `tool-cache` | `actions/setup-*` silently does more work | **no** | **already `false`** |

> **This block previously recommended flipping `tool-cache`. That was wrong:
> `tool-cache` has defaulted to `false` since `45c205b` (2022-06-27) and has never
> been `true`.** Caught by Parallax 6 against the upstream checkout. The premise
> came from this corpus's own uncorrected claim that every option defaulted to
> `true` — an error which had already propagated into three records before anyone
> read the table. See ADR-0001 `## Iterations`, Iteration 3.

**And the correction strengthens this ADR rather than weakening it.** The one
option whose failure is non-attributable was *already* defaulted off — decided in
2022, five days after an expert described the failure mode, and never written
down. So the criterion in this record is not a rule invented in 2026 to justify a
flip. **It is a rule that correctly predicts a choice the author made four years
earlier, before he could state it.** A criterion that only explains the decision
that prompted it is a rationalisation; one that retrodicts an independent decision
is a finding.

What four years of silence cost was therefore not the judgment — he had it — but
its **reuse**. `swap-storage` needed the same judgment in 2023 and did not get it,
because an unwritten rule has to be re-derived for every case.

---

### QST-FAQ-SCOPE: Should the FAQ cover side effects only, or become the repository's general FAQ?
- Status: unanswered
- Why asking: He specified the question — *"what are possible side-effects of these settings?"* — which is narrower than a FAQ usually is. The backlog contains several items that are really FAQ entries (issue #33 "specifying `dotnet: false` still deletes dotnet"; issue #40 and PR #26 on slowness; issue #21 on containers). Deciding the scope now avoids a second document later.
- Need: a scope; "just the side effects, keep it tight" is complete
- Options:
  - **A — Side effects only.** Exactly his question, one table, nothing else. Smallest surface, hardest to let rot.
  - **B — Side effects plus the recurring confusions** (#33, #40, #21) — each already has an ADR behind it, so answers are short and consistent.
  - **C — A general FAQ** that grows with the backlog.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**A — side effects only, for now.** Because a document that answers one question
well survives, and the three recurring confusions are better answered *in their
threads* first, where the people asking will actually see them. **Confidence:
medium-high** — this is a judgment about documentation decay, and the repository
already demonstrates what happens to prose nobody owns. **If wrong**: the same
three questions keep arriving and the FAQ has to be reopened anyway, which costs
one commit.

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

## Evidence

| claim | source |
|---|---|
| The ruling, verbatim | Author, 2026-09-16, in a recorded review session |
| The defaults policy it bounds | ADR-0001 §4 — author's testimony, 2026-09-16 |
| @zaikunzhang's argument, diagnosis, and proposed fallback | issue #12, 2023-09-10 (GitHub API, retrieved 2026-09-16); 11 👍 |
| The two-day diagnosis | issue #12, his own words |
| `large-packages` slowness (a separate question, not folded in) | issue #12 comment, @StevePotter, 2024-12-29 |
| `tool-cache`'s existing caveat | `README.md`, current |
| Severity convention cited in option D | ADR-0004 §the two exceptions |

*Recorded by Sherd 5 (Claude Opus 5), 2026-09-16 — the same day the decision was
made. Method: the campaign's protocol log. **Implementation is not this
seat's lane** (`swap-storage`'s default is product code); this record exists so
that whoever makes the change does not have to reconstruct why.*
