# archon-candidates

Compiled `archon` binaries, published as releases, for one machine to fetch.

**This repository holds binaries, the workflow that builds them, and nothing else.** No
notes about the system that consumes them, no configuration of it, no issue history.

The rule was written as *binaries and nothing else* and is amended here rather than
quietly stretched. The build workflow lives in this repository so that it can publish
with the automatic, repository-scoped `GITHUB_TOKEN`; the alternative was a personal
access token held elsewhere, and **not needing a credential is better than scoping one**.

The amendment is narrow, and the test it has to pass is *not* "is it small" but **"is it
worth reading?"** — the workflow describes how to build a public upstream and nothing
about what consumes the result. That was checked, not assumed: it names no host, no
scheduler, no consumer, and no design decision of the system that fetches from here.

## Why it is public, and why that is safe

The machine that consumes these binaries holds **no GitHub API credential** — one
read-only SSH deploy key, and nothing else. A private repository's release assets and
Actions artifacts both require an authenticated API call, so a private store is
unreachable to it. A public store is reachable anonymously, over the same code path that
already fetches upstream releases.

The binaries here compile [coleam00/Archon](https://github.com/coleam00/Archon), which is
public. Nothing secret is published by publishing them.

## The rule, and it is a rule rather than a preference

**Do not add a third kind of thing here, and do not let the workflow grow a second job.**

The reason this repository is safe to be public is that there is nothing in it worth
reading. A README explaining the system that consumes it, a workflow step that "just"
does one more thing, a note about a host — each is individually harmless and collectively
turns an artifact store into a description of an architecture.

The build workflow is the one exception, and it earns it by being **about the upstream
project only**. A change to it that mentions the consumer — a host, a schedule, a path it
writes to, why a particular version is wanted — is the thing this rule exists to stop,
even though the workflow itself is allowed.

Nothing enforces any of this but the person reading it, which is why it is written down.

## What a release contains

| file | what it is |
|---|---|
| `archon-linux-x64` | the binary |
| `checksums.txt` | its `sha256`, in `sha256sum -c` format |
| `provenance.json` | what it was built from, precisely enough to build it again |

`provenance.json` distinguishes the two kinds of entry:

- **`kind: "built-artifact"`** — built from an upstream commit by CI. `ref` is the
  **resolved commit SHA**, never a branch name, because a branch names a different commit
  tomorrow.
- **`kind: "release-mirror"`** — an upstream release asset, copied here byte-for-byte and
  unmodified, so that consuming this repository does not change which bytes run.

## Tags

A mirrored upstream release keeps the vendor's own tag.

A CI build is tagged `v<version the binary reports>-dev.<committer date>.<short sha>` —
for example `v0.10.1-dev.20260830T1200.3148914c`. Each part is load-bearing, and the
scheme is built around **measured** `sort -V` behaviour rather than around semver:

- `sort -V` orders `0.10.2-dev.x` **above** `0.10.2`, the opposite of semver pre-release
  precedence. Tagging a dev build against the *next* version would make it outrank the
  real release forever.
- A dev build reports the **last** release's version, so `v0.10.1-dev.…` is both correctly
  ordered and honest: 0.10.1 plus commits.
- The committer date is there because a bare SHA sorts **lexically, not chronologically**.

## Reproducibility, measured rather than claimed

**The web dist is byte-reproducible. The binary is not.** Both were measured on 2026-09-01
by building commit `3148914c` three times, twice in this repository and once elsewhere:

| | result |
|---|---|
| `archon-web.tar.gz` | `7bf777e1…` in **every** build, including across repositories |
| `archon-linux-x64` | identical between two builds **in this repository**; **different** from a build of the same commit made elsewhere |

So the deterministic `tar` flags do their job, and something in the binary compile does not.
The likely cause is the build path — a runner checks out to
`/home/runner/work/<repo>/<repo>/…`, so the repository name is part of the path, and
`bun build --compile` is known to embed paths. **That is a plausible explanation, not a
confirmed one**, and it is written as such.

Two consequences worth stating plainly:

- **A checksum here cannot be independently reproduced** by rebuilding the same commit
  somewhere else. `provenance.json` identifies the source precisely enough to *rebuild*,
  which is not the same as precisely enough to *get the same bytes*. Verify a download
  against `checksums.txt`; do not expect your own rebuild to match it.
- Builds should stay comparable as long as they keep happening here, which is now the
  only place they happen.
