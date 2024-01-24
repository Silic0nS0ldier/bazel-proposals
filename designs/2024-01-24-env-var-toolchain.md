---
created: 2024-01-24
last updated: 2024-01-24
status: To be reviewed
reviewers:
  -
title: Toolchain for Environment Variables
authors:
  - Silic0nS0ldier
---


# Abstract

Bazel provides flags to control environment variables however they are too coarse, have many counterinutitive behaviours, and are hard to consume in rules.


# Background

(add usage notes from the "lore comment" from the Canva internal `.bazelrc` file here)

(add OS specific challenges here)
- POSIX
  - When `PATH` is not set, some members of the `exec` syscall family will apply a default. (expand with info from investigation)
  - When `PATH` is not set, shells like `bash` provide a hard-coded default.
    However this is not an environment variable (think `export FOO=""` vs. `FOO=""`).
- Windows
  - `PATHEXT` influences what is resolved by `PATH`.
  - (does Windows also have a `PATH` default?)

(add UX notes from development of Canva internal wrapper around NodeJS's `child_process` API)

# Proposal

This proposal is simple in principle, but is complicated by UX requirements. 

- A dedicated toolchain type for environment variables.
  - Allows a bare bones POSIX only default. More specialised environments can be brought in via constraints.
- Env vars toolchain automatically included in every rule for every resolved execution platform.
- `ctx.actions.run(env = { ... })` merges with toolchain when enabled (matching Bazle 7 behaviour with `use_default_shell_env`).
- `ctx.actions.run(use_default_shell_env = False)` to disable auto-usage of toolchain.
  - `use_default_shell_env` could be renamed to `use_default_env`, or something clearer.
    But likely not worth the migration effort. _Maybe_ in conjunction with other changes (read on).
- On POSIX, default env has `PATH=""` which is _actually_ hermetic (OS and shell defaults do not kick in).
  Actually this will be very disruptive and slow down adoption. It should be gated behind its own flag.
- Remove `ctx.configuration.default_shell_env`, toolchain provides the API.

(do we keep `--action_env`?)
(`--host_action_env` will almost definitely be removed)
(what about repo rules? can't use the toolchain. perhaps mandate behaviour in this proposal for completeness?)
(what about tests? actually... these _can_ just use the toolchain)
(do we keep `--test_env`?)

# Backward-compatibility

Describe here how this proposal impacts backward compatibility. If the proposal
is implemented, can it possibly break any user?
