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
