# ADR-0003: Inherited foundations — what came from where

- **Date of decision**: 2022-06-22 (composition of two prior works)
- **Date of record**: 2026-08-31  |  **Iteration**: 2 (2026-09-16)
- **Status**: Accepted — descriptive of the codebase as it stands
- **Deciders**: Jérémie Lumbroso

> ### 📜 Reconstruction — but an unusually well-evidenced one
>
> This ADR is derived from **byte comparison against dated references**: the
> cited sources as they existed *on the day this action was written*, retrieved
> from their own git histories, not their current state. It asserts provenance,
> not intent. Where intent is claimed it is marked.
>
> Sources archived at `evidence/free-disk-space/cited-sources/`.

**TL;DR**: This action is a composition of two prior works plus one original
contribution. **The architecture is `ShubhamTatvamasi/free-disk-space-action`'s.
The `large-packages` list is `apache/flink`'s, verbatim. Per-category
measurement is the author's own.** Knowing which is which is not trivia: of three
long-standing defects, **two are inherited rather than authored, and the third
exists only in the composition** — created by the act of making an inherited
unconditional script optional. That changes how all three should be triaged.

---

## Context

The README has always credited its sources. Nothing recorded **what was taken
from each**, and four years later the author described the consequence:

> "since this is not maintained so often, I don't remember all my choices, and so
> I'm just filled with dread of breaking something I don't understand."
> — 2026-08-31

**This ADR exists to make that fear precise, and thereby smaller.** Some of the
code was never his; it can be evaluated on its origin's terms rather than
re-derived from scratch.

---

## The composition

### From `ShubhamTatvamasi/free-disk-space-action` — the architecture

Compared at `559b72118` (2022-04-24), the last commit before this action existed.
That file is byte-identical today, so the comparison is stable.

| element | inherited |
|---|---|
| `runs: using: "composite"` + `- shell: bash` | the action's whole shape |
| `icon: "archive"`, `color: "green"` | both branding fields, unchanged |
| `sudo rm -rf /usr/local/lib/android` → `/usr/share/dotnet` → `/opt/ghc` | **all three paths, same order** |
| `remove-swap` → `swapoff -a` + `rm -f /mnt/swapfile` | renamed `swap-storage`, same body |
| `df -h /` before and after | the before/after reporting shape |
| **`if [[ ${{ inputs.x }} == 'true' ]]`** | **the input-interpolation pattern** |

The author's own account — *"I decided to fork something for my needs"* — refers
to this. It is not a GitHub fork; the repository is not marked as one.

### From `apache/flink`, `tools/azure-pipelines/free_disk_space.sh` — the package list

Compared at `db6baf471` (2022-05-16). The `apt-get` block is **unchanged between
that ref and today**, so what the author read is what is quoted here:

```bash
sudo apt-get remove -y '^dotnet-.*'
sudo apt-get remove -y '^llvm-.*'
sudo apt-get remove -y 'php.*'
sudo apt-get remove -y '^mongodb-.*'
sudo apt-get remove -y '^mysql-.*'
sudo apt-get remove -y azure-cli google-cloud-sdk hhvm google-chrome-stable firefox powershell mono-devel libgl1-mesa-dri
sudo apt-get autoremove -y
sudo apt-get clean
```

**Only the packages came from Flink.** In 2022 Flink's script removed exactly one
directory (`rm -rf /usr/share/dotnet/`, no `sudo`, commented `# deleting 15GB`).
Its directory list grew to seven *after* this action shipped — so today's Flink
resembles this action more than the Flink the author actually read did.

### The author's own contribution

Neither parent measures anything. Flink prints `df -h` three times; Shubham twice.
v1.0 of this action opens with a `# MACROS` block present in neither:

```bash
printSeparationLine()   getAvailableSpace()   formatByteCount()
printSavedSpace()       printDH()
```

**Per-category measurement and human-readable reporting is original**, and
present from the first working version. The macros also carry four
`# REF:` citations to StackOverflow and Unix.SE — a line-level citation habit
that predates any methodology by four years.

---

## Consequences — three defects have a provenance, and they are three different kinds

### 1. The batched `apt-get remove` is Flink's line, and its failure mode is inherited

One command carrying nine packages **fails entirely if any one is absent or
renamed.** That is:

- **issue #41** — `google-chrome-stable` absent on Ubuntu 24.04 ARM, skipping everything after it
- **PR #42** — @jim60105's isolation fix
- **PR #8** — @kfir4444's per-line failure isolation, merged 2023-09-29
- fork fixes by @dbouget, @sebrandon1, @emedgene, @jerryliang122, independently

**This is not a design error by the author.** It is an Apache CI script's line,
written against *one known Azure image*, inherited into a general-purpose action
that would later run on images nobody had seen. The real defect is structural:
**a context-specific assumption carried into a context-free tool.**

Corollary, now explained: **PR #3 (@gruve-p, 2022-12-03) — the first PR ever
merged here — was "remove hhvm."** `hhvm` appears in Flink's list and had ceased
to exist on GitHub runners. The first act of maintenance was removing an
inherited artifact from a parent's environment.

**Triage consequence**: any PR that further isolates package removals is
*restoring the tool to its own context*, not modifying the author's design. That
is a much cheaper judgment.

### 2. The template-injection pattern is Shubham's, present at v1.0

`${{ inputs.x }}` interpolated directly into `bash` is the documented GitHub
Actions template-injection shape — input values substituted before the shell sees
them. It arrived with the architecture.

**PR #51** (@nbuckwalt, also in the Contrast-Security-OSS fork) moves the seven
inputs into an `env:` block and quotes them.

**Triage consequence**: this is not a change to anything the author decided. It
corrects an inherited pattern, and the fix is mechanical. *This ADR takes no
position on severity — see the seed's open items and the maintainer seat.*

### 3. A third defect, of a different class: `dotnet: false` does not stop dotnet being removed

**Record, and the users got here long before this reconstruction did.**

- **@gmij, [#6](../../issues/6), 2023-08-08** — the first report, and a complete
  one: *"when dotnet set false, and Large is true, shell will be clean all dotnet
  runtime. please fix."*
- **@ashleney, [#33](../../issues/33), 2024-11-02** — reported again as its own
  issue: *"Specifying `dotnet: false` still deletes dotnet."*
- **@ax3l, [#33](../../issues/33), 2025-06-23** — identified the mechanism and
  supplied the workaround, citing the exact lines: *"dotnet is also removed in the
  `large-packages` list of things, so you will need to set that to `false`, too."*

> **An earlier version of this section said this defect was "found 2026-09-16"
> and that it took "four years to diagnose." Both were false, and the falsity was
> the same kind this campaign exists to document: the record existed — in three
> issue comments, in public, on this repository — and I had not read it before
> claiming novelty.** Caught by Parallax 6 against the primary threads. What this
> section actually contributes is the *provenance* — why the overlap exists at all
> — not the discovery of it.

The reports are correct, and the mechanism is visible in `action.yml`:

| option | what it does to dotnet |
|---|---|
| `dotnet` | `sudo rm -rf /usr/share/dotnet` — the SDK directory |
| **`large-packages`** | `sudo apt-get remove -y '^dotnet-.*'` **and** `'^aspnetcore-.*'` — the apt packages |

So `dotnet: false` with `large-packages: true` — **the default configuration** —
removes dotnet anyway, by the other path. The toggle is not wired to everything
that does the thing it names.

**This defect is neither inherited nor authored. It is emergent from the join**,
and that makes it a third class worth separating from the two above:

- **`apache/flink` had both removals** — `apt-get remove -y '^dotnet-.*'` at line
  36 and `rm -rf /usr/share/dotnet/` at line 47 of the script as forked.
  Removing dotnet twice is **harmless in Flink's script, because nothing there is
  optional.** It is a cleanup script; everything always runs.
- **`ShubhamTatvamasi`'s action had only the directory removal**, and also no
  per-category toggles for it.
- **Neither parent could exhibit this bug.** It exists only in this repository,
  and only because of the thing ADR-0001 §3 records as the author's design
  intent: *making the categories separately toggleable.*

> **The agency design is what created the bug, and that is worth stating plainly
> rather than softening.** Adding a user-facing switch to an inherited
> unconditional script requires auditing every path that performs the action the
> switch claims to control. The `large-packages` block arrived as an opaque unit
> — a cited line lifted whole (defect 1 above, same cause) — so it was never read
> for overlap with the options being built around it. **The cost of the good
> decision was a silent one — and then it was reported, twice, and diagnosed by a
> third user, and still sat for three years.** The failure was never that nobody
> noticed; it was that noticing had nowhere to land.

This is the clearest instance in the repository of something the code cannot tell
you: **the bug is not in either component, and not in either parent's design. It
is in the relationship between an inherited unit and a new abstraction placed over
it** — and that relationship is exactly what no diff records and no comment
mentions.

**Fixed in `9881e8b`** (2026-09-17), by option (b) — the `large-packages` block
now gates its `dotnet-*`/`aspnetcore-*` removals on `inputs.dotnet`, and logs a
`::notice::` when the exemption fires. The governing rule, ruled by Mint 5: *a
specific subject outranks a general category.*

*Scope of that claim, stated precisely because an earlier draft overstated it:
**the policy is now explicit, and this release applies it to .NET.** Future
subject options must add and test the corresponding exemption themselves — there
is no general mechanism, and saying "the class is fixed" would claim an
implementation that does not exist. A policy is valuable without that claim. The
same overlap audit is still worth running against the rest of the
`large-packages` list — `llvm`, `php`, `mongodb`, `mysql` have no toggles today,
so nothing collides, but the first one that gets a toggle inherits this exact
trap unless someone checks.*

### 4. The general consequence

**Knowing the provenance converts several open items from judgment calls into
bookkeeping.** The author's dread — *"breaking something I don't understand"* —
was well-founded precisely because a substantial part of the code was never his
to understand from memory. **Naming the inheritance is the direct remedy for the
specific fear that stopped maintenance.**

---

## What this ADR does not settle

- **Attribution obligations.** Both parents are open-source (Flink is Apache-2.0;
  Shubham's action carries its own terms). This action is MIT. Whether the
  composition creates any notice obligation is a licensing question, routed to
  the licensing lane, and **not** adjudicated here.
- **Whether `printDH` is a typo.** The helper is named `printDH` in v1.0, before
  any output string containing "dh" existed. If "DH" is shorthand for `df -h`,
  then six or seven people have been correcting an abbreviation rather than a
  slip. Open as a question for the author; see the onboarding seed.
- **What `github.community/…/17267/11` contained.** Investigated and
  unrecoverable: one Wayback capture of the thread (2020-09, predating any
  script), every capture of post #11 a redirect or uncaptured, the forum retired.
  A named absence, not an unexamined source.

---

## Questions

### QST-NOTICE-OBLIGATION: Does composing these two sources create any attribution or notice obligation?
- Status: unanswered
- Why asking: This action is MIT; `apache/flink` is Apache-2.0. The `apt-get` block is inherited close to verbatim. Attribution is currently by prose citation (README + inline comments) rather than by any formal notice. This ADR establishes the facts; it does not adjudicate the licensing.
- Need: a ruling from someone qualified; explicitly **not** a question this ADR answers
- Options:
  - **A — Prose citation is sufficient.** The README acknowledgement and the inline `# option inspired after:` comment already name both sources; Apache-2.0 §4 is satisfied in substance.
  - **B — Add a `NOTICE`/`THIRD-PARTY` file** recording the Apache-2.0 provenance of the inherited `apt-get` block explicitly, leaving the MIT licence of the whole unchanged.
  - **C — Get a qualified ruling first** and implement whatever it says; treat A and B as unverified guesses until then.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**C — routed, not guessed; out of my competence.** The
facts are established above and archived under `cited-sources/`; the judgment
belongs to whoever owns licensing. If wrong to leave open: the repository ships
with an unexamined obligation, which is a worse failure than an inelegant notice.

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

### QST-PRINTDH: Is `dh` shorthand for `df -h`, or a typo?
- Status: **answered 2026-09-16 — but read the answer's register before relying on it**
- Why asking: The most-rediscovered thing in this repository — three open PRs (#28, #37, #46) and three fork fixes. Whether those are corrections or preferences depends entirely on this answer, and contributors keep spending effort on it because nothing records the intent.
- Need: one sentence; "no idea, I don't remember" is complete and useful — and would have been a real finding
- Options:
  - **A — Deliberate shorthand.** `DH` abbreviates `df -h` (disk free, human-readable); the helper prints that. The open PRs are preference changes, not bug fixes.
  - **B — A typo for `printDF`**, propagated into the output strings and never noticed. The open PRs are correct and have been for years.
  - **C — Unrecoverable.** Nobody, including the author, can now distinguish A from B, and the record should say so.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**A — deliberate shorthand.** Because the helper is named `printDH()` in v1.0
(`3a49c22`), *before* any output string containing "dh" existed — those arrive in
`76d6ce2` the same day, made to match the name. A typo usually propagates the
other way. If wrong: this ADR credits an abbreviation where there was a slip, and
six contributors were right all along.

**ANS:** (by Jérémie Lumbroso) — 2026-09-16

> "I think DH was like Delta Human in the same way that `df -h` is (I believe)
> 'diskfree --human' essentially. At any rate, whether or not there was a
> justification, I think this proves the limitations of cryptic names. We should
> determine better names, maybe using architecture of complexity to name and
> determine which helpers to keep."

**This converges with option A, and is recorded as convergence rather than as
settlement — the distinction is load-bearing.** Read the hedges: *"I think"*,
*"(I believe)"*, *"whether or not there was a justification"*. The author is not
recalling a decision; he is **reasoning about his own artifact from the artifact**,
four years later, exactly as this reconstruction did. Two independent readings
agreeing is real evidence — but it is not the same kind of evidence as a
remembered intent, and labelling it **Record** would manufacture precisely the
false trace this corpus exists to avoid.

So: **the question stays marked as converged-not-settled**, and option C remains
live. What this *does* settle is the practical matter — the author does not
experience the open PRs as bug reports, and #28/#37/#46 can be answered on the
merits of naming rather than as corrections of an error.

*(His `df -h` gloss is "disk free, human-readable"; "Delta Human" is his
recollection of his own coinage, not a claim about the Unix tool.)*

---

### QST-HELPER-NAMES: Should the helpers be renamed, and by what principle?
- Status: unanswered — **raised by the author, 2026-09-16**
- Why asking: He drew a general conclusion from the `printDH` excavation — *"this proves the limitations of cryptic names"* — and named a want in the same breath: *"We should determine better names, maybe using architecture of complexity to name and determine which helpers to keep."* That is a design intent stated in 2026 about code written in 2022, and it is the first forward-looking decision this corpus has surfaced rather than recovered.
- Need: a direction; this is a want, not yet a specification
- Options:
  - **A — Rename nothing.** The helpers are internal; renaming churns every fork and open PR against them for a readability gain that only maintainers see.
  - **B — Rename, descriptively.** `printDH` → `printDiskUsage`, etc. Cheap, obvious, breaks any fork that calls them by name.
  - **C — Rename *and* prune, using decomposition as the criterion** (his "architecture of complexity" reference — Simon's near-decomposability: a helper earns its name and its existence by being a stable intermediate the rest of the script can depend on without knowing its internals). The naming question and the which-helpers-to-keep question get answered together, because they are the same question.
  - **D — Defer to the rewrite.** Fold into whatever larger refactor the maintenance restart produces, rather than as a standalone change.

**Recommendation**: (by Sherd 5, Claude Opus 5)

**C, but not now — and the reason is timing, not merit.** Because he asked for
naming and pruning together, and that pairing is the substantive part: "which
helpers to keep" is a decomposition question, and a name is just the answer to it
made visible. If wrong: this defers a cheap readability win behind a larger
refactor that may never happen, and the next contributor reads `printDH` and
opens a fourth PR. **Falsifier**: a fourth `printDH` PR arriving before the
rename ships would show the deferral cost more than it saved.

*Scope note: this is a change to product code and therefore outside this seat's
lane (`PROTOCOL.md` §division of labour). Recorded here so the reasoning is not
lost; execution belongs to whoever takes the maintainer role.*

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

## Evidence

| claim | source |
|---|---|
| Shubham's action as of 2022-04-24 | `cited-sources/shubham-2022-06-AS-FORKED.yml` @ `559b72118` |
| Flink's script as of 2022-05-16 | `cited-sources/flink-2022-06-AS-FORKED.sh` @ `db6baf471` |
| Flink's `apt-get` block unchanged since | `diff` of the two retrieved versions, 2026-08-30 |
| This action's v1.0 | `action.yml` @ `3a49c22`, 2022-06-22 |
| Downstream defect instances | [#41](../../issues/41), [#42](../../pull/42), [#8](../../pull/8), [#33](../../issues/33), [#51](../../pull/51), and the fork commits named above |
| The archived sources themselves | `evidence/cited-sources/` — both parents at the refs in force when this action was written, plus `easimon`'s action and the Wayback capture of the discussion thread |

*Reconstructed by Sherd 5 (Claude Opus 5), 2026-08-31. Method note: the first
pass compared against the sources' present state and was wrong about which parent
contributed the directory list; corrected by retrieving both sources at refs
predating this action; the archived `*-AS-FORKED` files in `evidence/cited-sources/` are the corrected basis.*
