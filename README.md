# Deprecation notice
This project has been archived and isn't used anymore.

# Workman
Workman is a working directory management utility, it is designed for use with large projects where
you typically have multiple clones/worktrees of the upstream repository to avoid rebuilding when
switching between ongoing tasks.

Workman automates the creation of new working directories and
making sure that currently unused working directories are kept up-to-date so that you don't need to
build when you start using them.

# Usage
Workman requires a configuration file placed at `.workman_config` in the directory where working
directories should be managed. An example configuration file for working on
[rustc](https://github.com/rust-lang/rust) is shown below:

```sh
# Directory where working directories are kept.
WORKDIR_PATH="."
# Name of stamp file placed in active working directories.
ACTIVE_STAMP_NAME=".workman_active_working_directory"
# Name of stamp file placed in working directories that are updated but not assigned.
DO_NOT_ASSIGN_STAMP_NAME=".workman_do_not_assign"

# Name of project, used in working directory names.
PROJECT_NAME="rust"
# Default branch to check out and update.
DEFAULT_BRANCH="master"

# Name of upstream remote to create and update from.
UPSTREAM_NAME="upstream"
# Url of upstream remote to create and update from.
UPSTREAM_URL="git@github.com:rust-lang/rust.git"

# Name of origin remote to clone from.
ORIGIN_NAME="origin"
# Url of origin remote to clone from.
ORIGIN_URL="git@github.com:davidtwco/rust.git"

# Command to run before creating a new working directory.
BEFORE_COMMAND="
ln -s ../config.toml ./config.toml &&
ln -s ../rgignore ./.rgignore &&
ln -s ../lvimrc ./.lvimrc &&
ln -s ../envrc ./.envrc &&
ln -s ../shell.nix ./shell.nix &&
echo \".envrc\" >> .git/info/exclude &&
echo \"shell.nix\" >> .git/info/exclude
"
# Command to run to clean a working directory.
CLEAN_COMMAND="python2 x.py clean -j6"
# Command to run to build in a working directory.
BUILD_COMMAND="python2 x.py build -j6"
# Command to run after creating a new working directory.
AFTER_COMMAND="
rustup toolchain link $(basename $(pwd))-stage1 ./build/x86_64-unknown-linux-gnu/stage1 &&
rustup toolchain link $(basename $(pwd))-stage2 ./build/x86_64-unknown-linux-gnu/stage2
"

# vim:ft=sh
```

This configuration manages sequential working directories, `rust0` through `rustN`.

## Creating new working directories
New working directories can be created by running `workman new`. This subcommand will create a new
working directory with the next index, clone from `ORIGIN_URL`, add the upstream remote, pull from
the upstream remote, run `BEFORE_COMMAND`, run `CLEAN_COMMAND` and then `BUILD_COMMAND` and
`AFTER_COMMAND`.

## Assigning a working directory
Working directories can be assigned to a task using `workman assign`. This will check out a new
branch in the next working directory that isn't assigned and start a tmux session.

## Unassigning a working directory
Once work on a task is complete, working directories can be unassigned using `workman unassign`.
This will find the directory assigned to that task, checks out the `DEFAULT_BRANCH`, deletes the
task's branch, pulls from upstream, pushes `DEFAULT_BRANCH` to `ORIGIN_URL`, runs `CLEAN_COMMAND`
and kills any tmux sessions.

Unassigned working directories are marked as needing refreshed so that they won't be immediately
reassigned before having been re-built.

## Listing working directories and tasks
Working directories can be listed with `workman list` and current tasks with `workman tasks`.

## Updating working directories
Unassigned working directories can be updated using `workman update`. This will visit each
unassigned directory and pull from upstream, push to `ORIGIN_URL`, run `CLEAN_COMMAND` then
`BUILD_COMMAND` - ensuring that directories are ready to use. This can be scheduled in a cron job
or systemd timer.

# Author
This project is developed by [David Wood](https://davidtw.co), with contributions from:

- [Oliver Scherer](https://github.com/oli-obk)

# License
Workman is distributed under the terms of both the MIT license and the Apache License (Version 2.0).

See [LICENSE-APACHE](LICENSE-APACHE) and [LICENSE-MIT](LICENSE-MIT) for details.
