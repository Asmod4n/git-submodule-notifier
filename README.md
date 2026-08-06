# git-submodule-notifier

A github notifier when a tracked submodule gets new tag.

A reusable GitHub Actions workflow that watches every **tag-pinned git
submodule** in a repository and tells you when upstream cuts a release.

Written once, called from everywhere. It never changes a pin — it opens
an issue and a human decides.

## Why

Pinning a submodule to a tag is right: it never moves on its own. That is
also the whole risk. Nobody notices a release, and for anything that
parses untrusted bytes — a compression library, a TLS stack, a parser —
"nobody noticed" is how a fixed CVE stays unfixed.

Dependabot can bump submodules, but it moves to the newest *commit*
rather than the newest tag, and it will happily keep several PRs open.
This does neither.

## Use it

Add this to any repository, as `.github/workflows/deps-upstream.yml`:

```yaml
name: deps upstream

on:
  schedule:
    - cron: '17 6 * * 1'   # Mondays 06:17 UTC
  workflow_dispatch:

permissions:
  contents: read
  issues: write

jobs:
  check:
    uses: Asmod4n/git-submodule-notifier/.github/workflows/deps-upstream.yml@main
    with:
      notify: '@your-github-handle'
```

That is the whole per-repository cost. The schedule has to live in the
caller because GitHub does not let a reusable workflow bring its own
trigger; nothing else does.

Nothing in it is repository-specific: it reads `.gitmodules`, so it
covers whatever is pinned today, anything added later without either file
changing, and does nothing at all in a repository with no submodules.

### Inputs

| input | default | meaning |
|---|---|---|
| `notify` | `''` | who to `@`-mention in the issue body, so GitHub mails you |
| `label_prefix` | `deps` | label namespace for the tracking issues |

## This repository must be public

On a personal account a reusable workflow in a **private** repository
cannot be called from another repository at all — cross-repository reuse
is an organization feature. Private, and callers silently fail to resolve
it. There are no secrets in this workflow, so public costs nothing.

## What it does, per submodule

1. Resolves the pinned tag with `git describe --tags --exact-match`.
   A submodule tracking a **branch** is skipped: it is already moving on
   its own and has nothing to report.
2. `git ls-remote` against that submodule's own URL, filtered to stable
   tags (`^v?N.N.N$`) — `-RC` and `-beta` are not what a pinned
   dependency should be nudged toward. A leading `v` is tolerated.
3. Compares with `sort -V`.

## Exactly one issue per submodule

Found by **label** (`deps: <name>`), never by title. The title carries the
version and changes every release, so matching on it would open a second
issue for 2.3.5 while the one about 2.3.4 was still open.

The issue is edited in place as further releases appear, and **closed
automatically** once the pin is moved past it — so an open issue always
describes the repository now, not the day it was filed.

A `concurrency` group prevents two runs both finding no issue and both
creating one, which is the single failure this design exists to avoid.

You are `@`-mentioned in the body so GitHub sends mail. Assignment is
attempted as well, but the mention is the load-bearing part: assignment
fails silently when the account cannot be assigned on that repository,
and a mention does not.

## What it does not cover

**Dependencies that are not submodules.** A package manifest pinning
`:github => ..., :branch => ...` floats by definition, and this never
sees it. If that is how most of your dependencies are pinned, this
watches the smaller half of your exposure.
