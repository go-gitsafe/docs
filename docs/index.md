# go-gitsafe

Guards on what leaves a machine: a global git hook that refuses a credential in a remote URL and a write to the branch pull requests land on, a credential helper that never lets a token reach a command line, and a scope checker that reads a token's powers without printing it.

Part of the **Packaging & supply chain** family of the
[pure-Go ecosystem](https://go-desktop.github.io/) — 1 module,
all `CGO_ENABLED=0`.

## What is here

This site is the organisation's reference index: what each module is, and where its
API documentation lives. The API itself is generated from the source and served by
[pkg.go.dev](https://pkg.go.dev/), which is always current with the released tags —
duplicating it here would only create a second copy to go stale.

- **[Modules](modules.md)** — every module in go-gitsafe, with its source and its reference.

## What every module here is held to

- `CGO_ENABLED=0`: no cgo, and no shelling out to a command-line tool in place of a
  library.
- Built and tested on amd64, arm64, riscv64, loong64, ppc64le and s390x — the last
  being big-endian, which keeps every on-disk and on-wire encoding honest.
- 100% statement coverage as a CI gate, error branches included.
- BSD-3-Clause.

The standard is described in full on the
[ecosystem map](https://go-desktop.github.io/docs/latest/standards/).
