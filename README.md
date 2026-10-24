# git-submodule-notifier

Opens a GitHub issue when a submodule is behind upstream: a newer
stable tag for a submodule pinned to a tag, a newer head of the default
branch for a submodule pinned to a commit. It never moves a pin.

## Use it

`.github/workflows/deps-upstream.yml` in the calling repository:

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
    uses: Asmod4n/git-submodule-notifier/.github/workflows/deps-upstream.yml@<sha>
    with:
      notify: '@your-github-handle'
```

The schedule has to live in the caller: a reusable workflow cannot bring
its own trigger.

Pin a SHA, not `@main` — this runs with `issues: write` in *your*
repository.

| input | default | meaning |
|---|---|---|
| `notify` | `''` | who to `@`-mention, so GitHub mails you |
| `label_prefix` | `deps` | label namespace for the tracking issues |

## Behaviour

- Reads `.gitmodules`; nothing is repository-specific.
- A submodule pinned to a **branch** is skipped — it already moves.
- Stable tags only (`^v?N.N.N$`), compared with `sort -V`.
- A submodule not at a tag is compared with upstream `HEAD`; the issue
  lists the commits it is behind.
- One issue per submodule, found by label, edited as further releases
  appear, closed once the pin moves past it.

This repository must stay **public**: on a personal account, a reusable
workflow in a private repository cannot be called from elsewhere.
