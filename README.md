# archon-candidates

Compiled `archon` binaries, published as releases, for one machine to fetch.

**This repository holds binaries and nothing else.** No source, no configuration, no
notes about the system that consumes it, no issue history. Its entire content is release
entries, each carrying `archon-linux-x64`, `checksums.txt` and `provenance.json`.

## Why it is public, and why that is safe

The machine that consumes these binaries holds **no GitHub API credential** — one
read-only SSH deploy key, and nothing else. A private repository's release assets and
Actions artifacts both require an authenticated API call, so a private store is
unreachable to it. A public store is reachable anonymously, over the same code path that
already fetches upstream releases.

The binaries here compile [coleam00/Archon](https://github.com/coleam00/Archon), which is
public. Nothing secret is published by publishing them.

## The rule, and it is a rule rather than a preference

**Do not add a second kind of thing here.**

The reason this repository is safe to be public is that there is nothing in it worth
reading. A README explaining the system that consumes it, a workflow that "just" does one
more thing, a note about a host — each is individually harmless and collectively turns an
artifact store into a description of an architecture. Nothing enforces this but the person
reading it, which is why it is written down.

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
