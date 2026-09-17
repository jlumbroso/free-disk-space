# ADR-0005: Report what each option actually saved

- **Date of decision**: 2022-06-22 (v1.0, and refined the same day)
- **Date of record**: 2026-09-01  |  **Iteration**: 2 (2026-09-16)
- **Status**: Accepted — the repository's original contribution
- **Deciders**: Jérémie Lumbroso

> ### 📜 Mostly **Record** — the artifact and its commits carry the reasoning
>
> Two same-day commits (`3a49c22`, `76d6ce2`) show this being designed *and
> corrected* within hours. Where intent is inferred it is marked, and three
> questions are raised back to the author rather than answered here.

**TL;DR**: Neither parent measured anything. This action reports, per option, in
human units, how much space it actually freed — and that reporting was the first
thing written and the first thing debugged. **This reconstruction reads it as the
repository's actual contribution; the author does not, and the disagreement is
preserved in Consequences rather than resolved.**

---

## Context

**Record.** ADR-0003 establishes the inheritance. Neither parent reports:

- `apache/flink`'s script prints `df -h` three times — before, middle, after.
- `ShubhamTatvamasi`'s action prints `df -h /` twice, labelled `Before` / `After`.

Both answer *"how much is free now?"* Neither answers **"what did each thing I
turned on actually buy me?"** — which is the question a user with seven toggles
needs, because it is the question that tells them which toggles to keep.

---

## Decision

**Measure and report each reclamation category separately, in human-readable
units, at the point it happens.**

v1.0 (`3a49c22`) opens with a macro block present in neither parent:

```bash
printSeparationLine()   # framing
getAvailableSpace()     # df -a | awk 'NR > 1 {avail+=$4} END {print avail}'
formatByteCount()       # numfmt --to=iec-i --suffix=B --padding=7
printSavedSpace()       # "=> ${title}: Saved $(formatByteCount $saved)"
printDH()               # captioned df output
```

and each option is wrapped:

```bash
BEFORE=$(getAvailableSpace)
… do the removal …
AFTER=$(getAvailableSpace)
printSavedSpace $((AFTER-BEFORE)) "Android library"
```

### This is the design intent, stated four years later and visible in the artifact

**Record**, the author, 2026-08-28:

> "I tried to put the fragment of my philosophy, of **explaining decisions, of
> connecting choices to settings that are opt-out, and allow for agency by the
> end-user, but also clearly show endpoints in the code.**"

Per-option measurement *is* that sentence implemented. A toggle whose effect is
unmeasured is not really a choice; the user cannot evaluate it. Measurement is
what makes the opt-out meaningful rather than decorative.

---

## Refinement, same day (`76d6ce2`, "Enriched output")

**One hour of work** (`3a49c22` 03:58:45 → `76d6ce2` 04:59:12 — 1h00m27s), worth
recording individually because it shows the measurement being **corrected**, not
merely written. The tightness of the window is the point: the units bug was
introduced, found, fixed, and documented inside a single hour.

### 1. A units bug, found and fixed immediately

```diff
- # macro to make bytecounts human readable
- formatByteCount() { echo $(numfmt --to=iec-i --suffix=B --padding=7 $1) }
+ # macro to make Kb human readable (assume the input is Kb)
+ formatByteCount() { echo $(numfmt --to=iec-i --suffix=B --padding=7 $1'000'); }
```

`df` reports in **kilobytes**; `numfmt` was being handed those numbers as if they
were **bytes**. Every saving was being reported roughly 1000× too small.

**The fix is not the interesting part. The comment is.** He did not just multiply;
he rewrote the docstring to state the assumption — *"assume the input is Kb"* —
so the next reader knows why the `'000'` is there. That is the documented-decision
habit operating at the level of a single macro, in 2022.

### 2. Measurement became scoped

```diff
- getAvailableSpace() { echo $(df -a | awk …) }
+ getAvailableSpace() { echo $(df -a $1 | awk …); }
```

An optional path argument — so a category can be measured against a specific
mount rather than the whole machine.

### 3. Borrowed code replaced with his own, and marked honestly

```diff
- # REF: https://stackoverflow.com/a/5799353/408734
- str=${1:'*'}; v=$(printf "%-${num}s" "$str"); echo "${v// /*}"
+ # (silly but works)
+ counter=1; output=""; while [ $counter -le $num ] …
```

The StackOverflow `printf` trick was swapped for an explicit loop, and the
citation replaced with **`# (silly but works)`**. Self-deprecating, accurate,
and — notably — he *removed a citation when the code stopped being borrowed.*
The attribution habit runs in both directions.

### 4. Attribution added *in the code*, not only the README

```diff
+ # option inspired after:
+ # https://github.com/apache/flink/blob/master/tools/azure-pipelines/free_disk_space.sh
  large-packages:
```

The `large-packages` option arrived on day one carrying its source, inline.
ADR-0003 credited him with citing sources in the README and in macro comments;
this is stronger — **the inherited option is credited at its definition.**

### 5. The branding stopped being inherited

`icon: "archive"` → `icon: "trash-2"`, with a link to GitHub's branding docs.
Shubham's icon was replaced with a chosen one, and the choice was cited.

---

## Consequences

- **This is the feature the repository is actually for** — *this reconstruction's
  reading, marked as such because the author's own account differs: he attributes
  the uptake to "(a) good functional names, and (b) putting things out there"
  (ADR-0001), not to the reporting.* Space reclamation is a commodity — Flink's
  script and Shubham's action both do it. *Telling you what each option bought* is
  what distinguishes this action technically, and it was there from the first
  version. Whether it is what *drew users* is a separate claim, and the evidence
  does not settle it.
- **It is why the backlog contains the requests it contains.** Issue #25
  (@ffMathy) asks for **timings** per option — *"so that I can opt out of some of
  them that are taking long"* — which is this ADR's principle extended from space
  to time. @ChrisCarini's fork extends it a different way, wrapping each removal in
  a Sentry tracing span *(space in the log, durations to telemetry — not what #25
  asks for)*. Both are continuations of this decision, not new proposals.
- **Issue #50** (negative savings printing `=> Docker images: Saved` with no
  value) is a defect *in this feature* — `formatByteCount` passing a negative to
  `numfmt`. The units fix above handled scale, not sign.

---

## Open — raised with the author rather than assumed

### The `dh` question

`printDH()` is named that in **v1.0**, and `76d6ce2` then adds the output strings
`echo "$ dh -a /"`. So the helper name came first and the echo followed it.

**If "DH" is shorthand for `df -h`, then `dh` is deliberate compression, not a
typo** — and six or seven people have independently "corrected" an abbreviation
(PRs #28, #37, #46; forks by @linka-cloud, @zhang-brook, @gruve-p). It does not
change whether to standardise on `df` — clarity wins — but it changes what this
ADR says, and whether the PRs are fixes or preferences.

*Confidence at first writing: genuinely split — the naming order is suggestive;
nothing else was. **Updated 2026-09-16**: the author now reads `DH` as shorthand
for `df -h` too, converging with this reconstruction — but explicitly as his own
inference ("I think", "whether or not there was a justification"), not as a
recalled intent. So the question is **converged, not settled**, and the practical
consequence holds: the open PRs are naming preferences to judge on the merits,
not corrections of an error. Full treatment in ADR-0003 `QST-PRINTDH`; his
follow-on want — better helper names, chosen by decomposition — is
`QST-HELPER-NAMES` there.*

### Two smaller ones

The first is a real open question and is raised as `QST-KB-BUG` below. The second
is deliberately *not* raised, and says so:

**NOT: `--padding=7` is recorded as colour, not raised as a question.** A
column-alignment choice for the output block. It is unrecoverable, it is
consequence-free, and asking it would spend the author's attention — the
project's scarcest resource — on a number that changes nothing whichever way it
is answered. *Recorded here so that its absence is a decision rather than an
oversight; an exemplar must not teach that open questions may live outside the
grammar, and the counterpart rule is that not every unknown deserves a QST.*

---

## Questions

### QST-KB-BUG: Was the kilobyte units bug caught by noticing implausible output, or by reading `numfmt`'s docs?
- Status: unanswered — *promoted from a prose bullet at Iteration 2 (Shipwright 5's catch: an open question living outside the grammar is invisible to tooling and teaches a bad habit). Originally raised in the onboarding seed under this same handle.*
- Why asking: `76d6ce2` changes `formatByteCount` from `$1` to `$1'000'` and rewrites its comment to *"assume the input is Kb"* — a ~1000× reporting error, fixed within the hour. **If it was caught by noticing implausible output, the measurement was being checked against reality on day one** — a materially stronger claim about this ADR's subject than the diff alone supports.
- Need: which one; "don't recall" is complete and expected, and would itself settle that the record cannot carry the stronger claim
- Options:
  - **A — Noticed the output.** You ran it, saw a saving in KiB where you expected gigabytes, and went looking. The measurement was being validated against reality from the first hour.
  - **B — Read the documentation.** You checked what `numfmt` expects and found the mismatch by audit rather than by observation. Careful, but a smaller story.
  - **C — Don't recall.** Then ADR-0005 keeps the weaker, diff-supported claim and says why.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**A — noticed the output.** Because the fix landed 61 minutes after v1.0 and
carries a rewritten *comment* documenting the assumption, which is the shape of
someone who was surprised and then wrote down why, rather than someone auditing
an API. **Confidence: medium, and resting on timing alone** — an hour is fast for
either route, and the comment is equally consistent with careful reading. If
wrong: this ADR overstates the empirical discipline it is built to celebrate,
which is the flattering direction of error and therefore the one to distrust.

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

### QST-MEASURE-TIME: Should measurement extend from space to time?
- Status: unanswered
- Why asking: Issue #25 (@ffMathy) asks for per-option timings — *"so that I can opt out of some of them that are taking long"* — which is this ADR's own principle applied to a second axis. The decision is whether that is in scope. *(An earlier version of this block cited @ChrisCarini's fork as already implementing it. **Corrected 2026-09-17 on Parallax 6's catch**: that fork wraps removals in Sentry tracing spans and prints saved *space* — the durations go to a third-party service, not to the workflow log. It is not the feature #25 asks for, which is seeing the numbers in the log.)*
- Need: a ruling
- Options:
  - **A — Yes, per-option timings**, printed in the workflow log beside the space figures. Same principle, second axis.
  - **B — No.** Space is the product; timing is observability, and the run already draws complaints for slowness (issue #40). Adding measurement to a slow step makes it slower.
  - **C — Yes, behind an input.** `report-timings: false` by default, so users who want it pay for it and nobody else does — consistent with §4 of ADR-0001 only in the opposite direction, since here the cheap error is *not* measuring.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**A — yes; it is the same decision, not a new feature.** Because this ADR's
rationale is that an unmeasured toggle is not a real choice, and the run takes
minutes: a user choosing between options needs both costs, not one. If wrong:
output grows noisier for users who only care about space, and the timing itself
adds overhead to a step already criticised for slowness (issue #40).

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

### QST-NEGATIVE-SAVINGS: How should the report handle a negative or zero saving?
- Status: unanswered
- Why asking: Issue #50 (@Ravna) — `formatByteCount` passes a negative to `numfmt`, which errors and prints `=> Docker images: Saved` with no value. A defect in this ADR's own feature, and the reporter notes it is intermittent and that something *consumed* space during the step.
- Need: a decision on the desired behaviour, then a fix
- Options:
  - **A — Report the negative honestly.** `=> Docker images: -1.2GiB` — the run consumed space, and the report says so.
  - **B — Clamp to zero.** `=> Docker images: 0B`. Tidy output; hides the finding that something grew during the step.
  - **C — Report zero, warn on negative.** Clean number in the normal channel, `::warning::` when the delta is negative — the severity-matching pattern ADR-0004 already established in this codebase.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**A — report the negative honestly rather than clamping to zero.** Because the
reporter's actual surprise was that space was consumed, and hiding the sign hides
the finding — the measurement exists to tell the truth about what happened. If
wrong: users see confusing negative numbers in normal runs and read them as a bug
in the action rather than a fact about the runner.

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

## Evidence

| claim | source |
|---|---|
| Neither parent measures per-category | `cited-sources/flink-2022-06-AS-FORKED.sh`, `shubham-2022-06-AS-FORKED.yml` |
| Macro block original to v1.0 | `action.yml` @ `3a49c22` |
| Units fix, scoping, loop rewrite, inline Apache citation, icon change | `git diff 3a49c22 76d6ce2 -- action.yml` |
| Design intent | Author's testimony, 2026-08-28 |
| Downstream continuations | issue #25; `FORK-DIGEST.md` (@ChrisCarini) |
| Defect in this feature | issue #50 (@Ravna, 2026-01-12) |

*Reconstructed by Sherd 5 (Claude Opus 5), 2026-09-01.*
