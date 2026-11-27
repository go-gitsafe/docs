---
title: "Merge gates"
weight: 30
---

# Merge gates {#merge-gates}

Every merge in the fleet goes through `ghmerge` (and `ghrelease`, which calls
it). It merges on evidence, not on the absence of a failure.

## Checks

A check must have run, every check that ran must have passed, and GitHub must
say the pull request is mergeable.

## The release gate

A Go change in a tagged module must not be merged without a tag. Merged alone,
the code is on `main` and invisible to `go get`: the module proxy resolves
`@latest` to the highest release, and an untagged commit is only a
pseudo-version ([Go modules reference, "Versions"](https://go.dev/ref/mod#versions)
and ["Mapping versions to commits"](https://go.dev/ref/mod#vcs-version)). It
happened eight times, the last two on 2026-11-27.

The gate asks the question **per module**. Each changed file belongs to the
module of the nearest `go.mod` above it, and a module's releases are the tags
under its own prefix:

| Module | Its tags |
| --- | --- |
| the repository root | `vX.Y.Z` |
| a module in a subdirectory `sub` | `sub/vX.Y.Z` |
| a module in a major version subdirectory `sub/v2` | `sub/v2.X.Y` |

Tags of one module say nothing about another: a repository whose only tags are
`sub/v1.2.0` is a tagged module for `sub` and not for its root.

`ghmerge` refuses, before merging, a pull request that changes a non-test `.go`
file, a `go.mod` or a `go.sum` (a rename out of one counts) in a module that has
at least one stable release. It names the exact command, with the next PATCH over
every tag of that module (GitHub's tag listing is not in semver order):

```text
ghmerge: merge AND tag it in one command: ghrelease owner/repo 42 sub/v0.10.1
```

`ghrelease` tags a prefixed module, one tag per merge, and checks the tag against
the `go.mod` at the merge commit. A pull request that changes several tagged
modules gets a command for each; the others run after the merge, and `ghrelease`
tags the merge commit of an already merged pull request.

What the gate cannot read is not "no Go change": a failed tag, file or tree read,
a file list shorter than the pull request says, or a truncated tree (which cannot
say which `go.mod` owns a file) refuses, naming the cause. `--no-release
"<why>"` merges anyway, for a series whose later pull request carries the tag;
there is no default and an empty reason is refused. `ghrelease` passes it itself,
since it is the release.

A module and its major version subdirectory that are both released (`sub` and
`sub/v2`) share the prefix `sub/`: the gate names a version of the highest line,
`ghrelease` rejects a wrong one against the `go.mod`, and `--no-release` is the
way through.

The full rules are in the
[`gitsafe` README](https://github.com/go-gitsafe/gitsafe#the-release-gate-a-go-change-in-a-tagged-module-is-merged-with-a-tag-or-not-at-all).

## The issues gate

A pull request must not land while an issue it mentions is left silently open.
Before any write, `ghmerge` reads the pull request body and resolves each `#N` or
`owner/repo#N` with one REST call. Each open issue must be preceded, on the same
line, by:

- a closing keyword (`Closes`, `Fixes`, `Resolves` and their other forms): the
  issue is closed by this pull request; or
- `Refs`: the issue stays open on purpose.

A bare mention is refused, naming the issue and both forms. `--issues-waiver
"<reason>"` merges anyway and records the reason as a comment on the pull
request; there is no default and an empty reason is refused.

### Quoting a reference, so it decides nothing

A reference inside a code span, a fenced block, an HTML comment or a plain URL
is not a reference: that is how a body discusses an issue number without
disposing of it, and it is the only honest way to name an issue that is already
closed, since neither form fits one — `Closes` says this pull request finishes
it, and `Refs` says it stays open on purpose.

Since **v0.59.0** a code span may wrap a line break, as CommonMark allows. The
earlier reading required a span to fit on one line, with two consequences that
reflowing a paragraph to 80 columns could introduce invisibly: a quoted
identifier became a real reference the author was asked to dispose of, and — the
dangerous direction — a merely quoted `Closes` counted as a real disposition, so
the gate could allow a merge on a decision GitHub itself never carries out.

A backtick run opens a span that only a run of the same length closes, and an
unclosed run is literal text. The line breaks a span covered are preserved,
because a keyword binds only within its own line.

⚠ A caller pinning an older `gitsafe` keeps the older reading: a body is judged
by whichever version the tool that reads it was built with.

After the merge, every issue closed by keyword is read back. If GitHub did not
close it (another repository, a base branch that is not the default, a missing
permission) `ghmerge` closes it with the comment `Closed by <owner/repo>#<PR>
(merge <sha12>)`. If it cannot, it exits 3 naming the issue.

The full rules are in the
[`gitsafe` README](https://github.com/go-gitsafe/gitsafe#the-issues-gate-no-pull-request-lands-over-a-silently-open-issue).
