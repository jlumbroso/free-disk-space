# Contributing

Thanks for helping keep this action useful. Contributions of every size are
welcome — most of the fixes in this repository's history came from users who hit
a problem in their own CI.

## The one rule: changes carry their reasoning

Every proposed change should include its reasoning: what problem it addresses,
why this approach, and any relevant trade-offs. **The detail should match the
change — for a small fix, a sentence can be enough.** This approach is called
[ORRCF](https://adrs.systems/orrcf), and this file is where it becomes concrete
for this repository:

- **Small fix** (typo, doc correction, an obviously-absent package breaking a
  removal): one or two sentences in the PR description. *"`google-chrome-stable`
  is absent on `ubuntu-24.04-arm`, so the batched line skips everything after it;
  this isolates it the way `google-cloud-cli` already is."* That is a complete
  rationale.
- **Behaviour change or new option**: the fuller form, still just in the PR
  description — the options you considered, the one you recommend and why, how
  confident you are, and **what would change your mind** (the part most reviews
  never state, and the part that makes review fast).
- **A change that crosses a recorded boundary**: some designs here are deliberate
  and documented — deletion-not-repartitioning
  ([ADR-0002](docs/adr/0002-reclaim-space-by-deletion-not-by-repartitioning.md))
  is the big one. If your change crosses one, argue it *in the record*: the
  relevant ADR's Questions section is the venue, and several questions there are
  open precisely because contributors could reasonably win them. You are not
  asking permission to disagree; the record exists so that disagreement lands
  somewhere durable.

## Why

The reasoning behind this repository's design used to live only in its
maintainer's head, and the maintenance gap this repo went through is what that
costs (the story is in [issue #55](../../issues/55) and [docs/adr/](docs/adr/)).
Recording the *why* next to the code is what lets a PR be judged against a shared
record instead of a recollection — that is the whole trick, and your PR
description becomes part of it.

## Practical notes

- **Where reasoning lives**: PR descriptions for changes; `docs/adr/` for
  decisions worth a permanent record. If a discussion in your PR turns out to be
  decision-sized, the maintainer may ask to capture it as an ADR — or you can
  propose one yourself.
- **Answering an open question**: the ADRs carry `### QST:` headings with open
  questions, each with a recommendation already staked. If you have evidence or a
  view, comment on it in a PR or issue and quote the question's handle (e.g.
  `QST-ISOLATION-SYSTEMATIC`) so it is findable. Any shape of answer helps.
- **Formatting**: ADR files follow a small grammar so they stay machine-readable
  (questions are `### QST:` headings, answers live in `**ANS:**` blocks). If you
  edit them, keep the shapes.
- **Credit**: externally-contributed options and fixes in this repository are
  credited at their point of arrival — in the commit, the inline comment, or
  preserved authorship
  ([ADR-0006](docs/adr/0006-the-option-set-grew-it-was-not-designed.md) records
  the pattern). That continues.
- **Testing**: `action.yml` is a composite action; test changes by pointing a
  workflow in your fork at your branch (`uses: <you>/free-disk-space@<branch>`)
  and include the run link in the PR when the change affects behaviour.

## Scope, briefly

This action **deletes** preinstalled things to free space; it does not repartition
disks or restructure the runner ([ADR-0002](docs/adr/0002-reclaim-space-by-deletion-not-by-repartitioning.md)
records why). New removal options are welcome when they carry a measured saving on
a named runner image ([ADR-0006](docs/adr/0006-the-option-set-grew-it-was-not-designed.md)
records how options have arrived so far).

### If you are proposing a new option, there is a rule for its default

Aggressive defaults are the norm here — the premise is that anyone reaching for
this action is already out of disk, so the expensive error is reclaiming too
little. **But that only holds while the failure is traceable.** An option should
default to `false` if removing the thing produces a failure that is *both* a
fundamental resource failure rather than a missing artifact, *and* gives the user
no visible connection back to this action.

That is why `swap-storage` and `preinstalled-runtimes` default to `false` and
everything else defaults to `true`. The reasoning, and the report that forced it
into words, are in [ADR-0007](docs/adr/0007-swap-storage-defaults-to-false-the-policy-acquires-a-boundary.md).

Stating your option's failure mode — *how would a user who did not want this find
out?* — is the most useful single line you can put in the PR.
