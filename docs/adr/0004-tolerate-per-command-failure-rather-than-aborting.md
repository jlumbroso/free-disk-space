# ADR-0004: Tolerate per-command failure rather than aborting the cleanup

- **Date of decision**: 2023-09-29 (merged); proposed 2023-06-15
- **Date of record**: 2026-09-01  |  **Iteration**: 2 (2026-09-16)
- **Status**: Accepted — implemented; **incomplete**, see Consequences
- **Deciders**: Jérémie Lumbroso, with @kfir4444 (proposer)

> ### 📜 Mostly **Record**, not reconstruction
>
> Unusually for this repository, the deliberation survives — in the pull request
> thread. The questions the author asked, the proposer's answers, and the
> objection considered-and-declined are all on the record. Quoted rather than
> inferred.

**TL;DR**: One `apt-get remove` carrying nine packages aborts the whole cleanup
if any single package is absent. Failure was made **non-fatal per command**
rather than fatal for the step — after explicitly considering, and declining, an
option to make it configurable.

---

## Context

**Record.** ADR-0003 establishes that the `large-packages` block is
`apache/flink`'s line, inherited verbatim — written against one known Azure
image. On a runner image where any listed package is missing or renamed,
`apt-get` exits non-zero and **every subsequent removal in that step is skipped.**

The symptom is not a visible error. It is *silently reclaiming less space than
reported*, which is worse.

**Record.** @kfir4444 opened PR #8 on **2023-06-15**:

> "Added `|| true` at the end of large file removal, so that if one step of it
> fails, the clean-up job would continue."

---

## Decision

**Make each removal non-fatal, and surface the failure rather than swallowing it.**

The shipped form is not the proposed `|| true`. It is:

```bash
sudo apt-get remove -y '<pkg>' --fix-missing || echo "::warning::The command [...] failed to complete successfully. Proceeding..."
```

`|| true` hides; `|| echo "::warning::"` proceeds *and reports*. The current
`action.yml` carries **nine `::warning::`** — seven of the nine `apt-get remove`
invocations, plus `autoremove` and `clean`.

### The two exceptions, and the second decision nobody wrote down — **found at Iteration 2**

The other two `apt-get remove` lines report **`::debug::`**, not `::warning::`:

```bash
sudo apt-get remove -y google-cloud-sdk --fix-missing || echo "::debug::…"
sudo apt-get remove -y google-cloud-cli --fix-missing || echo "::debug::…"
```

**This is deliberate, and the reasoning is recoverable from the timestamps.**
`google-cloud-sdk` was *renamed* to `google-cloud-cli`, which means **exactly one
of these two commands is guaranteed to fail on every single run, on every runner
image.** A warning that always fires is not a warning; it is noise that trains
users to ignore the channel. So the severity was matched to the surprise.

The five-minute sequence, on the night of 2023-09-28 (EDT):

| time | commit | what |
|---|---|---|
| 23:46:52 | `f099676` | PR #8 merged — the decision in this ADR ships, with `::warning::` |
| 23:51:02 | `a5a437d` | main merged into @andreped's PR #18 branch; **the `::debug::` lines are composed here, in the conflict resolution** |
| 23:52:46 | — | PR #18 merged (`merged_at` 2023-09-29T03:52:46Z) |
| next day 13:00 | `d5af243` | *"Fixed `google-cloud-sdk` warning"* — the package is pulled out of the batch line, completing the isolation |

Neither side of that merge contained the `::debug::` text. @andreped's branch had
the rename but not the `|| echo` pattern; main had the pattern but not the rename.
**The resolution is the author's own composition**, written six minutes after
shipping the principle it refines.

**So the decision in this ADR is really two decisions, and the second one is
sharper:** report rather than swallow — *and report at a severity that matches
how surprising the failure is.* Expected absence is `::debug::`; unexpected
absence is `::warning::`. Nothing in the repository says this. It survived only
as a difference between two string literals.

> **Method note.** This was found only because the history was read with
> `--full-history`. Git's default pathspec simplification omits merge commits, so
> `git log -- action.yml` shows 24 commits and **hides `a5a437d` entirely** —
> along with the decision it carries. This is the third time in this campaign
> that merge commits turned out to hold what the linear history did not (see
> `PROTOCOL.md`: *"right for solo repos, wrong for any repo maintained by
> integration"*). It is the first time the lesson paid rather than cost.

### The deliberation, on the record

**Record.** The author's reply, 2023-06-28 — and it is a direct instance of the
side-effect caution ADR-0002 records as his method:

> "I wanted to ask your opinion:
> - **Do you think there are any scenarios where the user may want this to fail?**
>   (Should this be a flag.)
> - **Do you think that it would be useful to print something when the command
>   fails**, or do you think th[at]…"

**Record.** @kfir4444's answer, 2023-06-29:

> "I can think of some scenarios where this action could be used as a test, so a
> failure here could be important, but I think that the general use case only
> benefits from not stopping a CI due to failed deletion, which hinders
> develop[ment]"

**Both of the author's questions were answered in the shipped code**: no flag
(the general case dominates), and yes to reporting (`::warning::` rather than
`true`). *The second question is the one he acted on most — the warning form is
his, not the proposal's.*

### Rejected: make it configurable

A `fail-on-error` flag was considered and not added. Rationale, from the
exchange: the "use it as a test" case is real but rare, and the general case —
a CI job that should not die because a package was already absent — dominates.
Consistent with ADR-0002: fewer knobs, more predictable side effects — *reconstructor's gloss; the thread does not cite ADR-0002's principle, which did not exist as a written thing.*

---

## Consequences

### It worked, and it is incomplete

The same failure recurred on a platform that did not exist in 2023.
**Issue #41** (@jim60105, 2025-10): `google-chrome-stable` is absent from the
Ubuntu 24.04 **ARM** image, so the line containing it is skipped — taking the
other eight packages with it. **PR #42** isolates that package the way
`google-cloud-cli` was already isolated.

**This ADR is why #42 is cheap to judge.** It is not a new proposal; it is *the
continuation of a decision already made*, applied to a package the 2023 fix did
not separate. The architecture was right; the coverage was partial.

Same for **PR #24** (@opsiff — snapd, microsoft-edge) and the fork fixes by
@dbouget, @sebrandon1, @emedgene, @jerryliang122: all are instances of one
established decision, not seven separate judgment calls.

### The delay is the most instructive part of this record

| date | event |
|---|---|
| 2023-06-15 | PR #8 opened |
| 2023-06-28 | author asks two careful design questions |
| 2023-06-29 | proposer answers |
| 2023-07-14 | *"many user facing issues"* — @pratik-techholding |
| 2023-07-14 | proposer offers his own fork as a workaround for other users |
| 2023-07-19 | *"Maybe this could fix #9"* — @acgxv |
| 2023-08-17 | *"this issue is breaking my CI too"* — @yinweisu |
| **2023-09-29** | **merged — 3 months and 14 days after opening** |

**The author was not ignoring it.** He was doing the thing ADR-0002 records as
his method — asking whether the change had unexamined side effects, and whether
it should be optional. He got an answer within a day. Then it sat for three
months while users routed around it.

This is the clearest documented instance in the repository of what the author
later named:

> "epistemic paralysis to prevent epistemic degradation, that just creates
> calcification" — 2026-08-31

**And it is why this ADR exists.** The deliberation happened; it simply was not
written anywhere a future maintainer — including its author — could find it. The
next person to meet a missing package does not have to re-derive whether failure
should be fatal, whether it should be a flag, or whether warnings should be
printed. It is decided, and here is who decided it and why.

---

## Questions

### QST-ISOLATION-SYSTEMATIC: Should package isolation be systematic rather than case-by-case?
- Status: unanswered
- Why asking: This decision was implemented per-line in 2023 and has since needed extending twice — `google-cloud-cli`, then `google-chrome-stable` on ARM (issue #41, PR #42). Each recurrence costs a report, a PR, and a wait. A systematic form — one package per command, generated rather than hand-listed — would close the class instead of the instance.
- Need: a ruling; a contributor could implement either shape
- Options:
  - **A — Case-by-case, as now.** Isolate a package when it breaks. Minimal change, minimal runtime cost, and each new runner image buys another bug report and another three-month wait.
  - **B — Systematic.** One `apt-get remove` invocation per package, generated from the list rather than hand-batched. Closes the class; costs one process spawn per package on every run.
  - **C — Systematic with a batched fast path.** Try the batch; on failure, retry the batch's members individually. Keeps the common case fast and degrades to B only when something is missing.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**C — systematic, with a batched fast path.** Because the failure mode is
structural (ADR-0003: an inherited list written against one known image) and will
recur on every new runner image — issue #41 is the second instance, not the last
— but the cost that made "isolate everything" unattractive is paid only in the
common case, where nothing is missing and the batch succeeds. Try the batch;
retry member-by-member only on failure. **Confidence: medium-high** on the shape,
**low** on whether the added branch is worth it in a shell script whose virtue is
being readable — that is a maintainer's taste call, not an evidential one. If
wrong: a script that was legible becomes a script with a retry path, and the next
person to read it has one more thing to hold.

> **Method note, recorded because it is the lint rule earning its keep.** This
> recommendation was **B** until 2026-09-16, when Shipwright 5's review caught
> that the block had no Options list. Writing the options out produced C — which
> resolves the exact tension my own falsifier had named (reliability against the
> performance work in issue #40 and PR #26) and which I had not seen while the
> alternatives lived only in prose. The Options block is not bookkeeping: **the
> discipline of naming the alternatives is what produced a better answer.**

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

## Evidence

| claim | source |
|---|---|
| The proposal and its text | PR #8, @kfir4444, 2023-06-15 |
| The author's two design questions | PR #8 comment, @jlumbroso, 2023-06-28 |
| The proposer's answer | PR #8 comment, @kfir4444, 2023-06-29 |
| User pressure during the gap | PR #8 comments, 2023-07-14 → 2023-08-17 |
| Merge date | `f099676` ("Merge pull request #8"), 2023-09-28 23:46 EDT; GitHub API `merged_at` 2023-09-29 03:46 UTC — same moment, two clocks |
| Shipped form (9 × `::warning::`, 2 × `::debug::`) | `action.yml` @ `54081f1` — which is still `main`'s head; the default branch has not moved since 2023-10-18 (GitHub API, 2026-09-16) |
| The `::debug::` severity refinement | `a5a437d` (2023-09-28 23:51 EDT) — composed in the merge resolution; absent from **both** parents (`5d6ed7b`, `f099676`). Visible only with `git log --full-history` |
| @andreped's rename fix | PR #18, opened and merged 2023-09-29T03:52:46Z; branch `google-cloud-cli-renamed`, commit `af55c94` |
| Recurrence on ARM | issue #41, PR #42 (@jim60105, 2025-10) |
| Inherited origin of the batched line | ADR-0003 |

*Reconstructed by Sherd 5 (Claude Opus 5), 2026-09-01. This ADR required almost
no inference: the pull request thread preserved the deliberation. That is worth
noting — **the reasoning existed and was public the whole time**, and was still
unavailable to its own author three years later, because a PR comment is not a
place anyone looks.*
