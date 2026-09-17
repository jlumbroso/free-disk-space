# ADR-0006: The option set grew; it was not designed

- **Date of decisions**: 2022-06-22 → 2023-06-28 (four accretions)
- **Date of record**: 2026-09-05  |  **Iteration**: 2 (2026-09-16)
- **Status**: Accepted — descriptive of how the API came to be
- **Deciders**: Jérémie Lumbroso, with @miketimofeev and @rake5k (Christian Harke)

> ### 📜 Mostly **Record** — every accretion is dated and self-attributing
>
> This ADR needed almost no inference. **Each option's arrival is credited in its
> own commit message or inline comment, by the author, at the time.** The
> reconstruction is little more than reading them in order.

**TL;DR**: The action's seven-option API was not designed; it accreted in four
steps over twelve months. **Four of the seven came from outside** — and they
arrived by three different mechanisms: lifted from a cited source, solicited by
asking a maintainer, and contributed as a pull request. Every one is credited at
the point of arrival.

---

## The four accretions

| when | option(s) | origin | credited where |
|---|---|---|---|
| 2022-06-22 `3a49c22` | `android`, `dotnet`, `haskell`, `swap-storage` | v1.0 — inherited paths from `ShubhamTatvamasi` (ADR-0003) | README acknowledgements |
| 2022-06-22 `f814dfc` | `large-packages` | `apache/flink` | **inline in `action.yml`**: `# option inspired after: <flink url>` |
| 2022-06-27 `45c205b` | `tool-cache` | **asked a maintainer and got an answer** | commit subject: *"Added `tool-cache` opt thx to @miketimofeev 🙏🏻"* |
| 2023-06-28 (PR #5) | `docker-images` | contributed | commit `0add001` authored by **Christian Harke** (@rake5k), authorship preserved through merge |

### The `tool-cache` story is the one worth telling

**Record.** @miketimofeev's comment on `actions/virtual-environments#2875` is
dated **2022-06-22** — *the same day v1.0 shipped* — and it is a **reply to
Jérémie**:

> "@jlumbroso the first one doesn't exist anymore on the image. The second one
> removes all the pre-cached tools (Node, Go, Python, Ruby) that are used by the
> correspondent actions like `actions/setup-python`…"

So the sequence is: publish v1.0 → **go and ask the runner-images maintainers a
question** → be told one of your two proposed paths is stale and what the other
actually does → ship it five days later as an option, crediting the answer.

Two things follow. **The option carries its expert's caveat**: `tool-cache` is
the only option whose README entry warns that it *"might remove tools that are
actually needed"* — because the person who explained it said so. And **this is
active research on day one**, not passive accretion. The taxonomy did not merely
grow; part of it was *solicited*.

---

## Decision

There was no single decision. What is recorded here is a **pattern**, and it is
consistent across all four accretions:

**An option is added when a specific, externally-evidenced reclamation
opportunity is identified, and it is credited to whoever identified it.**

No option in this action was invented at a whiteboard. Each traces to a source, a
maintainer's answer, or a contributor's PR.

### On the defaults — **rewritten at Iteration 2, 2026-09-16**

> **What this section used to say, and why it was wrong.** It read: *"ADR-0001
> marks the taxonomy's rationale absent on the author's own account that the
> original four were empirical and their `true` defaults never deliberated. That
> stands — for the original four."* The author corrected the underlying claim on
> 2026-09-16 (ADR-0001 §4 and its `## Iterations`), and asked for this ADR to be
> adjusted with it: *"you need to adjust based on what I told you re defaults at
> true in ADR 0001's notes."*

The corrected picture separates two things this ADR had run together:

| | the option **set** | the `true` **default** |
|---|---|---|
| **original four** | empirical — the biggest things found on a runner (Record, stated twice) | **deliberate** — *"ask forgiveness, not permission"* (Record, ADR-0001 §4) |
| **three later** | each arrived on a datable occasion, from a source, an answer, or a PR (the table above) | **two inherited `true` by convention** (`large-packages`, `docker-images`) — no commit or thread argues for it. **`tool-cache` did not**: it shipped `default: "false"` at `45c205b` and has never been `true`, because the maintainer who explained it had just described a failure nobody could trace (ADR-0001 §4a) |

**So "it grew, it was not designed" remains true of the API's *shape* and is now
false of its *default policy*.** The policy was designed, once, at the start, and
then applied by pattern-matching for a year — which is a different and more
common thing than never having been decided. This ADR's title survives; its
claim about defaults does not.

**Why this matters for issue #12** (`swap-storage` default, 11 👍, the
repository's most-supported item). The change strengthens the reply rather than
weakening it, and in a way that must not be got backwards:

- The old reading offered #12 an easy concession — *nobody thought about it,
  so changing it costs nothing.* **That concession is no longer available**, and
  offering it would now be false.
- The true position is more interesting: **there was a philosophy, it was
  coherent, and it was never written down.** #12's reporter was arguing against a
  reasoned design he had no way of knowing existed. The failure was the record's,
  not his and not the design's.
- And the split still holds: `swap-storage` is an *original* option, covered by
  the stated policy. `large-packages` and `docker-images` inherited `true`
  without anyone applying the policy to them. **`tool-cache` is the one case
  where the policy WAS applied** — correctly, in 2022, producing the only
  non-`true` default in the action. **Ruling on #12
  alone would silently ratify three defaults the policy was never tested
  against** — see `QST-DEFAULTS-INHERITED` below, which this correction makes
  *more* pressing, not less.

> **Ruled, 2026-09-16 — and the two-rulings recommendation was vindicated by the
> outcome rather than by the argument.** `swap-storage` flips to `false`
> ([ADR-0007](0007-swap-storage-defaults-to-false-the-policy-acquires-a-boundary.md)),
> on a boundary the author stated as *"the defaults shouldn't break something so
> elementary as OOM."* That ruling supplies the **criterion**
> `QST-DEFAULTS-INHERITED` was missing — *is the failure attributable?* — so the
> three inherited defaults can now be judged rather than re-argued. Applying it:
> `large-packages` and `docker-images` remove artifacts whose absence names
> itself, so `true` stands. **`tool-cache` needs no ruling at all: it has
> defaulted to `false` since 2022.** *(This reconstruction recommended flipping
> it before checking — ADR-0001 `## Iterations`, Iteration 3.)* Applied to all
> four, the criterion changes nothing further: the action's defaults are
> **correct as of the `swap-storage` flip**.

---

## Consequences

**The contribution rails already exist, and already worked — three different ways.**

This bears directly on issue #43 (*"identify a path to allow community
support"*). The record shows three functioning modes, before any governance
document existed:

1. **Lift with citation** — take a solution from a cited source (`large-packages`).
2. **Ask and credit** — put a question to the people who know, ship their answer
   attributed (`tool-cache`).
3. **Accept a PR with authorship intact** — `docker-images` still carries
   Christian Harke's name in the commit, three years later.

**The question #43 asks is not how to create a path. It is which of these three
to formalise** — and mode 2 is the one nobody has proposed, despite being the
only one that produced a correction *before* code was written.

**Attribution is the invariant.** Across four accretions, three mechanisms, and
twelve months, every externally-originated option is credited at its point of
arrival — in a commit subject, an inline comment, or preserved authorship. This
is the same habit ADR-0005 records at the level of a single macro and ADR-0003
records for the inherited package list. **It is not a policy anyone wrote down;
it is a practice visible in every layer of the artifact.**

---

## Questions

### QST-DEFAULTS-INHERITED: Do the later options' `true` defaults deserve their own ruling, separately from #12?
- Status: **answered 2026-09-16 — yes, two rulings; #12 is settled and the other three are not**
- Why asking: Issue #12 argues `swap-storage` should default `false`. `swap-storage` is an *original* option, covered by the stated forgiveness-not-permission policy (ADR-0001 §4). `large-packages` and `docker-images` inherited `true` by pattern-matching, and the policy was never applied to them. **`tool-cache` is the exception that turned out to matter**: it was defaulted `false` in 2022, at the moment an expert described its failure mode — the same judgment #12 asks for. Ruling on #12 alone would leave two defaults untested, and would miss that the reasoning already had a precedent in the artifact.
- Need: whether this is one ruling or two
- Options:
  - **A — One ruling.** Whatever #12 settles for `swap-storage` applies to every option; the defaults are treated as a single policy question.
  - **B — Two rulings.** #12 settles `swap-storage` under the stated forgiveness-not-permission policy; the three inherited defaults get examined separately, because the policy was never actually applied to them.
  - **C — Two rulings, and re-examine the policy itself.** As B, plus an explicit decision about whether the 2022 policy still fits a tool now used in thousands of workflows by people who did not choose it.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**B — two rulings, and `tool-cache` is the one to examine first.** Because it is the
only option carrying an explicit warning in its own README entry — *"might remove
tools that are actually needed"* — sourced from the maintainer who explained the
mechanism. An option that ships with a caveat and a `true` default is the
strongest candidate in the set for a default nobody deliberated. If wrong:
splitting the question doubles the adjudication for a decision that may be
uniform anyway, and #12's reporter waits longer for an answer that was already
three years overdue.

**ANS:** (by Jérémie Lumbroso) — 2026-09-16, by ruling rather than by reply

He ruled #12 on its own merits and on a criterion that does **not** automatically
extend: *"the defaults shouldn't break something so elementary as OOM."* So this
question is answered in the affirmative — **two rulings, not one** — and the
second one is now tractable rather than open-ended, because ADR-0007 supplies the
test. The remaining work moved to `QST-FLIP-GENERALISES` (ADR-0007), where the
criterion is applied; the recommendation there is `tool-cache` only, and not in
the same release.

---

### QST-NEW-OPTIONS: Should the option set keep growing, and by which of the three modes?
- Status: unanswered
- Why asking: PR #24 (@opsiff) proposes snapd and microsoft-edge and asks about `temurin-*-jdk`; issue #23 offers a merged script freeing 39 GB. Both are mode-1 proposals. Without a stated position, each arrives as an open-ended judgment call — which is the condition ADR-0004's record shows costs three months per item.
- Need: a direction; a contributor could then argue within it
- Options:
  - **A — Closed.** The option set is final; new reclamation targets belong in forks. Cheapest to maintain, and the fork network already shows what that produces.
  - **B — Open by all three modes.** Lift-with-citation, ask-and-credit, accept-a-PR — whichever arrives, judged case by case as now.
  - **C — Open by modes 2 and 3; mode 1 only with a measured saving** on a named runner image, because a lifted package list carries assumptions invisibly (ADR-0003).
  - **D — Open, but behind a size threshold.** Any option must demonstrate a minimum saving to justify its place in the API surface, whatever mode it arrives by.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**C — growth by mode 2 and mode 3; mode 1 only with measurement.** Because a
contributed PR (mode 3) and a solicited answer (mode 2) both arrive with someone
accountable for the claim, whereas lifting a package list from another script
(mode 1) is exactly how the inherited `apt-get` line entered and became a
four-year defect (ADR-0003) — the assumption travels invisibly with the code. A
mode-1 addition should carry a measured saving on a named runner image. If wrong:
a useful package goes unremoved because nobody wanted to run a measurement, and
the action reclaims less than it could.

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

## Evidence

| claim | source |
|---|---|
| v1.0's four options | `action.yml` @ `3a49c22`, 2022-06-22 |
| `large-packages` + inline Flink citation | `f814dfc`, 2022-06-22 |
| `tool-cache` + attribution | `45c205b`, 2022-06-27 — *"thx to @miketimofeev 🙏🏻"* |
| @miketimofeev's reply to Jérémie | `actions/virtual-environments#2875`, comment 1163392159, 2022-06-22 |
| `docker-images` authored by Christian Harke | `0add001`; PR #5 @rake5k, opened 2023-06-01, merged 2023-06-28 |
| Original four empirical | Author's testimony, 2026-08-28 (seed QST-OPTIONS: *"Yes, perfect read, all throughout"*), restated unprompted 2026-09-16: *"the specific options — empirical"* |
| **Defaults deliberate, not inherited-by-accident** | **Author's testimony, 2026-09-16 — ADR-0001 §4. Supersedes this ADR's Iteration-1 claim; see the note in §On the defaults** |
| `tool-cache`'s caveat | `README.md`, current |

*Reconstructed by Sherd 5 (Claude Opus 5), 2026-09-05. This is the least
inferential ADR in the set: the author credited every external contribution at
the moment it arrived, so the provenance was already written — it had simply
never been read in one place.*
