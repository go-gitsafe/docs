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

After the merge, every issue closed by keyword is read back. If GitHub did not
close it (another repository, a base branch that is not the default, a missing
permission) `ghmerge` closes it with the comment `Closed by <owner/repo>#<PR>
(merge <sha12>)`. If it cannot, it exits 3 naming the issue.

The full rules are in the
[`gitsafe` README](https://github.com/go-gitsafe/gitsafe#the-issues-gate-no-pull-request-lands-over-a-silently-open-issue).
