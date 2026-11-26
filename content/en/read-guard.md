---
title: "The read guard"
weight: 50
---

# The read guard {#the-read-guard}

`guard-read` is a PreToolUse hook for the **Read** and **Bash** tools. It refuses
a call that would display a file holding a credential, and it judges the file by
its **content**, not by its name. `secretpeek` is the safe way to look at such a
file instead. Both ship in `gitsafe` since **v0.68.0**.

## Why a name is not enough

`guard-bash` has a name-based rule: it fires on a command substitution over a file
called token, secret or `.pem`. A file called `cli-config.toml` does not look like
a secret, and a plain `cat` on it can print an admin token straight into a
transcript. The read guard opens the file and looks at what is in it.

## What it scans

It opens regular files only, reads the first 1 MiB, and looks for:

- JWTs and PEM private keys;
- GitHub, GitLab, AWS, Slack and Anthropic tokens;
- credentials embedded in a URL;
- `password`, `secret` or `token` keys with a value, in TOML, INI, YAML and JSON.

Known credential directories are refused by path whatever the content. For the
**Bash** tool it reads the file operands of programs that display content (`cat`,
`head`, `tail`, `grep`, `sed`, `awk`, `jq`, `strings`, `xxd`, `diff`, and others)
and `< file` redirects; globs are expanded, 64 files at most.

## A refusal never echoes the value

The refusal names the file, the kind of secret and the line, and points to the
safe alternative. The value itself is never printed.

## `secretpeek`

It prints a file's keys with each value replaced by its class, for example
`<secret jwt, 66 bytes>`, so the structure is readable and the secret is not.
With `--key` it prints one value, and only if that value is not a secret.

## Install

```
go install github.com/go-gitsafe/gitsafe/cmd/guard-read@v0.68.0
go install github.com/go-gitsafe/gitsafe/cmd/secretpeek@v0.68.0
```

Add the hook to `~/.claude/settings.json`:

```json
{"matcher":"Read|Bash","hooks":[{"type":"command","command":"~/.local/bin/guard-read","timeout":10}]}
```

## It fails open, and says so

A file it cannot read is allowed, with a line on stderr saying the content was not
checked: a refusal is not a verdict about a file that was never seen. It does not
cover `ssh host cat file`, a path built at run time, an interactive pager, or the
Grep tool.

The [`gitsafe` README](https://github.com/go-gitsafe/gitsafe#guard-read-and-secretpeek-a-files-name-says-nothing-about-its-content)
has the full section.
