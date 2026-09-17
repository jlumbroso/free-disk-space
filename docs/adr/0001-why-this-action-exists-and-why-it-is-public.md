# ADR-0001: Why this action exists, and why it is public

- **Date of decision**: 2022-06-22 (repository created)  |  **Date of record**: 2026-08-28  |  **Iteration**: 3 (2026-09-17)
- **Status**: Accepted — implemented and shipped
- **Deciders**: Jérémie Lumbroso

*Dual-date header, deliberately: reconstructed ADRs have two dates that matter —
when the decision was made and when the record was written — and collapsing them
into the template's single `Date` field would hide the four-year gap that is this
corpus's whole subject. The frozen fields are otherwise canonical.*

> ### 📜 This is a **partial reconstruction**, written four years after the fact
>
> This decision was not recorded when it was made. It was reassembled in 2026
> from the repository, its README history, and — for everything that matters —
> **the author's own testimony**, given in answer to questions the artifacts
> could not settle.
>
> Each section below is marked **Record** (the reasoning is in the evidence, or
> stated by the author) or **Reconstruction** (inferred; confidence and
> falsifier stated). Nothing is asserted that is neither.
>
> One question this ADR set out to answer was **abandoned as unrecoverable** and
> is marked as such rather than filled in. A reconstruction that shows no seams
> has hidden them.
>
> **Iteration 2 (2026-09-16)** folds in a second round of author testimony that
> **corrected this ADR on one point and was itself corrected by the artifact on
> another.** Both moves are recorded in place and in `## Iterations` rather than
> edited in silently — see the section for what changed and why it matters.

**TL;DR**: This action exists because a musicology project needed to process
scanned manuscript facsimiles in CI and kept running out of disk; it is *public*
because the author judged that a niche need was probably a widespread one, and
because the sources he had built on were themselves shared. The design — opt-out
settings, explained choices, visible endpoints — was deliberate, not incidental,
**and so were the aggressive defaults those opt-outs exist to make safe.**

---

## Context

**Record.** The author's account, 2026-08-28, in response to a direct question:

> "The Domenico Scarlatti project was the project of a lifetime: When I was a kid
> I was introduced to Domenico Scarlatti through Scott Ross' complete works by my
> grandfather, and I instantly fell in love. But in the beginning, access to sheet
> music was extremely limited […] But in the early 2020s, it was released in the
> public domain, and many sources of Scarlatti's original manuscript came online.
> I was able, during the pandemic, and with the support of Princeton University's
> Center for Digital Humanities, to commission digitizations of manuscripts that
> had been offline. And with the support of GitHub's academic program, I was able
> to host it all online."

The technical requirement followed from a scholarly one:

> "I wanted to have both perfect traceability ('these are the files I received'),
> but also traceability of process. So I designed the pipeline to be run on
> GitHub Actions. But that's when I realized that when you are manipulating large
> files, every available GB of space matters."

**The constraint was real and measurable.** By March 2024 the source repository
had reached 19 GB, exceeding the 17 GB original. A GitHub-hosted Ubuntu runner
does not have room for that without first reclaiming space.

**Record — what made it hard, in 2022 rather than now:**

> "I looked for solutions, but I was not an expert in GitHub Actions or continuous
> integration, and it was a struggle. […] these things would need to be scheduled
> or timed to pushes, would be hard to retrigger (this was in the beginning of
> GitHub Actions, the interface has much improved since) and also would take a
> very long time to run until they would hit the problems we needed to resolve.
> The existing Actions were insufficient."

---

## Decision

### 1. Reclaim disk space as a reusable action rather than an inline script

**Record.** The README has said so since the first day — *"This GitHub Actions
came around because I kept rewriting the same few lines of `rm -rf` code."* The
sentence entered at `6111681`, **2022-06-22 11:41 EDT** — eight hours and forty
minutes after the initial commit, and **six minutes before v1.0.0 was published**
(2022-06-22 15:47 UTC). The motive was written down while it was still true, and
that is the only reason it survived.

### 2. Publish it, rather than keep it private

**Record**, and this is the part no artifact in this repository explains. The
author, 2026-08-28:

> "I decided to fork something for my needs, to take the time to solve this
> problem better than the existing had, and **to reshare it with the community as
> the sources I cited had themselves.** […] **This design was intentional** — I
> figured, if I had this need in my niche Scarlatti project, it was probably very
> widespread, and taking a bit of time for the TLC would probably benefit a lot of
> people."

Two reasons, and they are different in kind: **reciprocity** (the manuscript
sources this project depended on were themselves shared, and the tooling should
be too) and **a generalisation bet** (a niche need is usually not niche).

> **Note on the reconstruction, recorded because it is instructive.** Before
> asking, this ADR's author inferred that publication was closer to *reflex than
> deliberation* — reasoning from the README sentence above, at stated confidence
> 0.35. **That was wrong**, and wrong in a specific way worth naming: the README
> sentence explains why the code became *reusable*, not why it became *public*.
> An answer read apart from the question it actually answered is not weak
> evidence — it is evidence about something else, wearing the right costume. The
> low confidence was doing its job; the inference was not.

#### 2a. …and "public" turned out to mean more than the author remembers — **Record, from the artifact**

Asked again on **2026-09-16**, the author volunteered: *"I don't really think I
published it. I don't think it's published in the marketplace. I just — it was
just on my repo because I needed it."*

**The artifact does not agree, and here the artifact wins.** The action is listed
on the GitHub Marketplace as **"Free Disk Space (Ubuntu)"**
([`marketplace/actions/free-disk-space-ubuntu`](https://github.com/marketplace/actions/free-disk-space-ubuntu)),
carrying all five releases (v1.0.0 → v1.3.1) and the install snippet `uses:
jlumbroso/free-disk-space@main`. Listing is not automatic: it requires a
`branding:` block in `action.yml` (present since v1.0) *and* an explicit opt-in
checkbox at release time. So it was done, deliberately, and then forgotten —
which is precisely what a four-year-old low-cost decision looks like from the
inside.

**This is the mirror image of §Iteration 2's other correction, and the pair is
the point.** On the defaults, testimony corrected the reconstruction. Here, the
artifact corrects the testimony. Neither source outranks the other in general;
each is strong exactly where the other is weak — memory holds *reasons*, artifacts
hold *acts* — and a method that privileged either one categorically would have
shipped one of these two errors.

### 3. Design for agency rather than for automation

**Record.** The author's stated intent, 2026-08-28:

> "I tried to put the fragment of my philosophy, of **explaining decisions, of
> connecting choices to settings that are opt-out, and allow for agency by the
> end-user, but also clearly show endpoints in the code.**"

This is visible in the artifact and predates any vocabulary for it: every
reclamation category is a named, separately-toggleable input; the README
documents what each removes and roughly how much it saves; the script prints
what it freed, per category, as it goes.

> **And it had a cost, recorded here rather than left to the reader to find.**
> Making an inherited *unconditional* script toggleable requires auditing every
> path that performs the action each new switch claims to control. That audit was
> not done for the `large-packages` block, which arrived as a lifted unit — so
> `dotnet: false` does not actually stop dotnet from being removed
> ([issue #33](../../issues/33); mechanism in
> [ADR-0003 §3](0003-inherited-foundations-what-came-from-where.md)). Neither
> parent could have this bug, because neither made anything optional. **The design
> intent in this section is what produced it**, and an ADR that recorded only the
> intent and not its cost would be flattering its subject.

### 4. Delete aggressively by default — **Record, added at Iteration 2; scope corrected at Iteration 3**

**Until 2026-09-16 this ADR recorded the all-`true` defaults as never
deliberated. That was wrong, and the author corrected it unprompted.** His
account, dictated, verbatim:

> "One thing — the odd true defaults, never deliberated at all. And that's not
> true. […] I figured that by default, you wanted to get as much disk space left
> as you could. And what I reason is that most people would discover this. […]
> it's sort of like, I was mirroring *ask for forgiveness, not permission* —
> which is delete everything you can. And then as you find out that things are
> missing, you can switch them back on."

So there is a policy, and it has a name the author gave it: **ask forgiveness,
not permission.** Delete aggressively by default; let the user discover what they
needed and toggle it back. The design premise is that a user who reaches for this
action is already out of disk space, so the expensive error is reclaiming too
little, and the cheap error is reclaiming something restorable by flipping one
input to `false`.

**This is not in tension with §3 — it completes it.** Aggressive defaults are
only defensible *because* every category is separately named, separately
toggleable, and separately reported. The opt-out design is what makes
forgiveness cheap enough to ask for. The two decisions are one decision, and the
ADR had half of it.

**What remains empirical is the option *set*, not the default *policy*.** The
author, in the same breath: *"So I know why they were all set to true by
default, but the specific options — empirical."* That distinction is the whole
correction: **which things to offer** was found by looking at runners; **what to
do with them unless told otherwise** was reasoned.

> **Correction at Iteration 3 (2026-09-17), and it makes the section stronger
> rather than weaker.** They were **not** all set to `true`. `tool-cache` has
> defaulted to **`false`** since the commit that introduced it — `45c205b`,
> 2022-06-27, *"Added `tool-cache` opt thx to @miketimofeev 🙏🏻"* — and has never
> been `true` in this repository's history. Six of seven options default `true`;
> that one never did. Caught by **Parallax 6**, on an independent read of the
> upstream checkout; this reconstruction had asserted the uniform claim without
> checking the defaults table. See `## Iterations` for why that failure is worse
> than it looks.
>
> **The exception is not a counter-example to the policy. It is the policy's
> boundary, practised four years before it was articulated** — see §4a.

> **The README never says any of this.** Four years of users met a tool that
> deleted six of seven categories by default and were left to infer the
> philosophy or resent it. Issue #12 (11 👍) is that inference failing in public.
> The policy existed; the *record* of it did not — which is this campaign's
> thesis stated in one artifact.

#### 4a. The boundary was practised in 2022 — **Record, from the artifact**

`tool-cache` is the option that arrived by ADR-0006's **mode 2**: the author
asked the runner-images maintainers a question and shipped their answer,
credited. @miketimofeev's reply, 2022-06-22, told him what the path actually did:

> "The second one **removes all the pre-cached tools (Node, Go, Python, Ruby)
> that are used by the correspondent actions like `actions/setup-python`**…"

That is a description of a **non-attributable failure**: `actions/setup-python`
does not error saying "the tool cache was deleted"; it quietly does more work, or
fails for a reason that names itself rather than the cause. **Five days later he
shipped the option with `default: "false"` and the README caveat** — *"might
remove tools that are actually needed"*.

**He was told a failure would be hard to trace, and he defaulted it off.** That is
exactly the criterion he articulated in 2026 as *"the defaults shouldn't break
something so elementary as OOM"* — applied, in 2022, on the first occasion it
came up, and then never written down.

> **So ADR-0007 does not invent the boundary. It recognises one already in the
> artifact and extends it to a case that had been missed.** This matters for how
> much weight the boundary can carry: a criterion stated once, in 2026, in
> response to a complaint, is a rationalisation risk. **A criterion stated in
> 2026 that correctly predicts a choice made in 2022 is a finding.**

And there is a sharper observation inside it. His 2026 testimony got the *reason*
right and the *scope* wrong — he said "they were all set to true," and one was
not. **His reason predicts his own past behaviour better than his memory of that
behaviour does.** Memory is strong on reasons and weak on acts; here both halves
of that are visible in a single dictated paragraph.

#### 4b. …and it was finally articulated in 2026 — **see ADR-0007**

Within hours of this section existing, the policy was applied to a second case
and the limit §4a had practised was finally said out loud.
**`swap-storage` now defaults to `false`** ([ADR-0007](0007-swap-storage-defaults-to-false-the-policy-acquires-a-boundary.md)).

The policy's premise is a working feedback loop: delete aggressively *because
discovery is cheap*. @zaikunzhang's issue #12 reported that diagnosing a removed
swap took **two days** — because the failure is not "something is missing" but
"the job died," with nothing connecting the death to this action. The author's
ruling: *"the defaults shouldn't break something so elementary as OOM."*

> **Delete aggressively by default — where the failure is attributable.** An
> option is not a candidate for an aggressive default if its failure mode is both
> a *fundamental resource failure* rather than a missing artifact, **and**
> *non-attributable*.

**The policy is applied here, not overridden.** Its scope is unchanged; it simply
does not reach a case where discovery is not cheap.

**And the sequence is the point.** This policy spent four years unwritten and
un-examined. It was stated on 2026-09-16 and **bounded the same day** — not
because anyone learned anything new about swap, but because a written rule can be
held next to a counter-example, and an unwritten one cannot. Issue #12 had carried
the counter-example since 2023. Nothing was missing except somewhere to put it.

**And §4a is why that is not a lucky coincidence.** The boundary was already in
the artifact, applied correctly to `tool-cache` in 2022. What four years of
silence cost was not the judgment — he had it — but the ability to **apply it a
second time**. An unwritten rule has to be re-derived from scratch for every new
case, which is precisely the condition he named as *"epistemic paralysis."*
Writing it down did not make him smarter about swap. It made a judgment he had
already exercised **reusable**.

---

## Consequences

- **★708, 112 forks, and no promotion** — *with one qualification found at
  Iteration 2.* The author, 2026-08-28: *"I don't know how the action got 700
  stars, as I never advertised, or mentioned its existence to anybody, I just
  created, used it myself, and that's it."* He is right that he never advertised.
  But the action **is** Marketplace-listed (§2a), and the Marketplace is where
  people go to find Actions. **The mystery is smaller than it looked**: the
  distribution channel was switched on once, at release time, and then forgotten.
  What remains genuinely unexplained is the *rate* — Marketplace listing is table
  stakes for an Action, and the overwhelming majority of listed actions do not
  reach 700 stars. Listing explains discoverability, not selection.
- **His own conclusion, worth preserving as stated**: *"note the importance of:
  (a) good functional names, and (b) putting things out there!"* — and note that
  §2a makes (b) more literally true than he remembered.
- The action outlived the project it was built for. The Scarlatti pipeline
  ([`scarlatti`](https://github.com/scarlatti) — *"Preservation, diffusion and
  promotion of Domenico Scarlatti's work, particularly the keyboard sonatas"*;
  the consuming repository `scarlatti/sources-ext` is **private**, so the
  workflow file cited below is real but not publicly checkable) ran from 2023-11
  to 2023-12; this action is still in use in thousands of workflows.
- **The design intent in §3 is why this repository is worth converting at all.**
  The practice of explaining decisions and preserving user agency was present in
  a 2022 shell script, four years before it had a name. This ADR is less a
  translation than a completion.

---

## What is NOT recorded here

**Absent — the reasoning is not recoverable, and no ADR should pretend otherwise.**

- **Why the option set is exactly these seven categories, in this order.** The
  author confirms the categories were *empirical* — the largest things actually
  found on a runner, not a designed taxonomy. There is no decision to record, and
  inventing a rationale would be the failure mode this banner exists to prevent.
  ~~He also recalls the all-`true` defaults as **never deliberated at all**.~~
  **Struck at Iteration 2, 2026-09-16 — this was wrong.** The defaults *were*
  deliberated, under a stated policy, and are now recorded as Decision §4. The
  strike is left visible rather than deleted: this ADR's credibility rests on
  showing where it was corrected, and an absence that turned out to be a presence
  is the most important kind of correction a reconstruction can make.
- **The precise moment of the publication decision.** No commit, issue, or
  message marks it. The reasoning survives (§2); the timing does not.

---

## Questions

> **Iteration 2 note — this block used to be one question and was really three,
> and that is exactly how the defaults error happened.** The original
> `QST-TAXONOMY-ORIGIN` asked why these categories, *and* in this order, *and*
> (in its rationale) why they default to `true` — three claims carried at three
> different confidences under one heading. It drew one undifferentiated "Yes,
> perfect read, all throughout," which ratified the one conjunct that was false.
> Split below. The handle `QST-TAXONOMY-ORIGIN` is retained on the question it
> originally named, because it has been cited elsewhere.

### QST-TAXONOMY-ORIGIN: Why these option categories?
- Status: **answered** (2026-08-28, reconfirmed 2026-09-16)
- Why asking: The option set is this action's entire public API.
- Need: explanation; "no reason, I just found them" is a complete answer
- Options: none (elicitation — no choice is being put to the decider)

**ANS:** (by Jérémie Lumbroso)
Empirical, twice stated. 2026-08-28: assent to "the biggest things you actually
found on a runner, not a designed taxonomy." 2026-09-16, unprompted and in his
own frame: *"the specific options — empirical."* The second statement is the
load-bearing one; see `## Iterations` for why an unprompted restatement counts
for more than the assent that preceded it.

---

### QST-TAXONOMY-ORDER: Why are the options declared in *this* order?
- Status: unanswered
- Why asking: Declaration order in `action.yml` is the order users read the README table and the order the script executes. If it tracks expected savings it is a design; if it tracks the order they were discovered it is sediment. ADR-0006 shows the last three arrived on datable occasions, so order is at least partly chronology — but the *original four* were declared together and their internal order is unexplained.
- Need: one sentence; "no idea, that's just how I typed them" is complete and useful
- Options: none (elicitation)

**Recommendation**: (by Sherd 5, Claude Opus 5)

**Sediment, not design — but ask rather than record it.** Because the three later
options each append at the end in arrival order (ADR-0006's table), so the file's
growth rule is demonstrably "add to the bottom," and the simplest explanation of
the original four is the same rule applied at once. If wrong: the ADR records
accretion where there was a size-ordering judgment — the precise error Iteration 2
just had to correct one section above, which is reason to hold this loosely
rather than confidently.

**ANS:** (by Jérémie Lumbroso)
[Fill this in]   <!-- literal placeholder — parser-significant, do not paraphrase -->

---

## Evidence

| claim | source |
|---|---|
| Origin, motive, publication reasoning, design intent | Author's testimony, 2026-08-28, given in answer to written questions. Quoted here at length precisely because the source is not public — see the note below |
| "kept rewriting the same few lines of `rm -rf`" | `README.md`, Acknowledgement section, entered at `6111681`, 2022-06-22 11:41 EDT |
| **Defaults policy ("ask forgiveness, not permission")** | **Author's testimony, 2026-09-16, dictated and unprompted while reviewing this record** |
| **Marketplace listing, all five releases** | **`github.com/marketplace/actions/free-disk-space-ubuntu`, retrieved 2026-09-16; `branding:` block in `action.yml` @ `3a49c22`** |
| 19 GB vs 17 GB repository sizes | Author, 2024-03-05, contemporaneous |
| Consumer was `scarlatti/sources-ext`, pinned at `v1.3.1` | `.github/workflows/compile-sonatas.yaml` — **repository is private**; the only consumer found across the author's entire local estate |
| ★708 / 112 forks | GitHub API, 2026-08-25 |

> **A note on the testimony cited above, because a citation you cannot follow is
> a weak one and should say so.** The author's answers were given in writing and
> in recorded review sessions that are not published. That is a real limit on what
> a reader can verify, and it is why the quotations here are long and literal
> rather than summarised: **you cannot check the source, so you should at least be
> able to see exactly what was said and judge the inference yourself.** Where the
> quoted reasoning is the only evidence for a claim, the claim says so.

---

## Iterations

### Iteration 2 — 2026-09-16: the round trip that corrected both directions

The author reviewed all six ADRs in a single sitting and accepted each one. Two
of his notes changed this record, and they point opposite ways.

**What testimony corrected.** The all-`true` defaults, recorded here since
2026-08-28 as never deliberated, **were** deliberated — under a policy he named
himself (§4). He raised it unprompted, having accepted the ADR a moment earlier.

**How the error survived nineteen days, stated plainly because the method is the
subject.** The original question bundled three claims at three confidences under
one heading, and offered them as one compound guess ending *"I'd bet the defaults
were never deliberated at all."* He answered: *"Yes, perfect read, all
throughout."* A blanket assent to a compound proposition cannot say which
conjunct it endorses. The ADR then recorded that assent as **Record** — the
strongest evidence label in the system — and the seam closed over.

Two rules follow, and both are cheap:

1. **One claim per question when the claims carry different confidences.** My own
   stated confidences (0.60 on "empirical", 0.50 on "defaults undeliberated")
   were already telling me these were two questions. I asked them as one.
2. **Never summarise testimony into a claim; show the question beside the
   answer.** "The author confirms the defaults were never deliberated" is
   unfalsifiable by a reader. *"Asked whether X, he said 'Yes, perfect read'"*
   lets the reader grade the evidence themselves — and would have exposed this.

This is the same mechanism as the publication-reflex error preserved in §2 above:
in both cases an answer was separated from its question and then read as though
it addressed something it did not. That the same corpus now carries two
independent instances is not an embarrassment; it is the closest thing to a
measured failure rate this method has.

**What the artifact corrected.** He believed the action was never published to
the Marketplace. It was (§2a). Memory is strong on reasons and weak on acts;
artifacts are the reverse. Neither source is authoritative in general, and the
practical consequence is that **every testimony-derived claim about an act should
be checked against the record before it is labelled Record** — the reverse
direction of the discipline this corpus already applies to inference.

**Ruled by the author, on the honest mechanics** (2026-09-16): *"Willing to have
my remarks be factored as Iteration 2 (rather than pretend that's how it was
always)."* Nothing above was edited silently; the struck claim remains visible in
`## What is NOT recorded here`.

### Iteration 3 — 2026-09-17: the rule from Iteration 2, broken within a day

**`tool-cache` has always defaulted to `false`.** This ADR asserted that every
option defaulted to `true`, and did so while *correcting* a different error about
those same defaults. Caught by **Parallax 6** — an external reviewer from a
different model family, recruited for exactly this — reading the upstream
checkout rather than this corpus.

**The failure is not the wrong fact. It is that Iteration 2 named the rule that
would have prevented it, one section above, hours earlier:**

> *Memory is strong on reasons and weak on acts. Artifacts are the reverse.*
> **Every testimony-derived claim about an act should be checked against the
> record before it is labelled Record.**

*"They were all set to true by default"* is a claim about an **act**. It came from
testimony. It went in as **Record**, unchecked, in the same edit that introduced
the rule requiring it to be checked. Reading one table in `action.yml` — thirty
seconds — would have caught it.

**What this says about corrective rules, and it is not comfortable.** Writing a
rule down is not the same as having applied it; the two feel identical from the
inside, and the feeling of having *just learned something* is actively misleading,
because it supplies the sense of a debt discharged. The Iteration-2 entry read as
finished work. It was a note about work.

The operational consequence is mechanical rather than attitudinal, because
attitude is what failed: **the defaults table is now a preflight check, not a
memory.** Any claim in this corpus about what the artifact *does* — a default, a
count, a date, a package list — gets verified against the artifact at the moment
it is written, by a command, in the same sitting.

**And the scoreboard moves again, in the same direction as before.** Iteration 2
established that a wrong reconstruction which collected an assent leaves the
denominator. This is a second leak: a wrong claim that entered as **Record**
without ever being a reconstruction at all — it was never marked as a guess, so it
was never counted as one. *Two of eight* counts neither kind. The honest statement
is that this corpus has no reliable denominator, and the reason to say so is that
a methodology which reports its own accuracy optimistically is the exact machine
the campaign exists to warn about.

---

*Reconstructed by Sherd 5 (Claude Opus 5) for the estate conversion campaign,
2026-08-28; Iterations 2 and 3, 2026-09-16/17. Method: the campaign's protocol log.
Questions the record could not answer were put to the author before anything was
written, and his answers are quoted rather than summarised where they carry the
reasoning — a discipline this ADR has now twice had to learn the hard way.*
