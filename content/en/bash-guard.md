---
title: "The Bash guard"
weight: 40
---

# The Bash guard {#the-bash-guard}

`guard-bash` reads a command line before it runs and answers `allow` or `deny`.
A harness that runs shell commands on an operator's behalf calls it first, which
makes it the most exercised command in the module: on a machine that has it
wired in, every Bash call goes through it.

It is a guard on what a command *does*, not on who runs it. The rules exist
because a written instruction is not a control — someone has to remember it at
the moment they are busy doing something else.

## A refusal names what to do instead

A rule here is not finished when it can say no. Every refusal names the program
that does the same job properly: a GitHub write that must go through its own
fleet command is refused *and told which one*, and a command line that records a
status in a file is refused with the program that reports that status honestly.

A guard that only forbids teaches nothing and gets worked around. One that
names the replacement moves the work onto the tool that cannot get it wrong.

## An escape where the form is sometimes right

Most rules carry one, as a single variable in front of one command:

```
GITSAFE_ALLOW_<RULE>=1 <the command>
```

It covers that command and nothing after it. The variable exists where the
refused shape is sometimes legitimate — a throwaway repository under a volatile
path, a backport deliberately started from an older base.

Some rules have none, deliberately. The rule added in **v0.61.0** for a
background command line that records its status in its output and exits with the
status of the trailing `echo` refuses with no variable at all: the correct form
already exists as a program, so an escape would only reopen the defect it was
written for. Where the right way is already one command away, "allow it anyway"
is not a missing feature.

## Judged against labelled command lines, not against opinion

Two commands keep the guard honest, and both work on real command lines rather
than on examples invented to pass:

- `hookcheck` replays a corpus of labelled command lines — each one marked
  `allow` or `deny` — through the guard and reports how many it answers wrongly,
  in both directions. A false refusal counts exactly as much as a miss, because
  a guard that refuses correct work is one people route around.
- `guardscan --rule <name>` replays a single rule over a body of command lines,
  which is how a new rule's over-refusals are measured **before** it lands
  rather than discovered afterwards.

A rule that reads shell text is also asked of the shells themselves: the oracle
runs `bash`, `dash` and `zsh`, so what a grammar permits and what a production
shell actually runs cannot quietly diverge. A runner missing one of those shells
fails the oracle by name instead of reading as a pass.

## Why this page does not list the rules

For the same reason the API is not copied here: a hand-kept list of a moving
thing becomes a second copy to go stale, and the first thing anyone would notice
is that it disagrees with the tool.

The [`gitsafe` README](https://github.com/go-gitsafe/gitsafe#guard-bash-and-why-a-written-rule-was-not-enough)
has one section per rule, each with the measurement that caused it — the command
line that was really typed, what it really did, and what now refuses it.
