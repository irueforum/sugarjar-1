# SugarJar

[![Lint](https://github.com/jaymzh/sugarjar/workflows/Lint/badge.svg)](https://github.com/jaymzh/sugarjar/actions?query=workflow%3ALint)
[![Unittest](https://github.com/jaymzh/sugarjar/workflows/Unittests/badge.svg)](https://github.com/jaymzh/sugarjar/actions?query=workflow%3AUnittests)
[![DCO](https://github.com/jaymzh/sugarjar/workflows/DCO%20Check/badge.svg)](https://github.com/jaymzh/sugarjar/actions?query=workflow%3A%22DCO+Check%22)

> [!IMPORTANT]
> If you are a _new_ user, I suggest looking
> at [Sapling](https://sapling-scm.com/) instead of this. This was written as
> a stop-gap to approximate some features of the Facebook/Meta internal
> development tools before they were released.  Now that those tools have been
> open-sourced and work with GitHub, this is likely to be deprecated at some
> point. Sapling is far more feature rich, faster, and have more development
> resources behind them. SugarJar is a smaller transition, and for the time
> being I'm maintaining it for all existing users.

Welcome to SugarJar - a git/github helper. It needs one of the GitHub CLI's:
either [gh](https://cli.github.com/) or the older [hub](https://hub.github.com/).

SugarJar is inspired by [arcanist](https://github.com/phacility/arcanist), and
its replacement at Facebook, JellyFish. Many of the features they provide for
the Phabricator workflow this aims to bring to the GitHub workflow.

In particular there are a lot of helpers for using a squash-merge workflow that
is poorly handled by the standard toolsets.

If you miss Mondrian or Phabricator - this is the tool for you!

If you don't, there's a ton of useful stuff for everyone!

## Installation

Sugarjar is packaged in a variety of Linux distributions - see if it's on the
list here, and if so, use your package manager (or `gem`) to install it:

[![Packaging status](https://repology.org/badge/vertical-allrepos/sugarjar.svg)](https://repology.org/project/sugarjar/versions)

For Ubuntu users, Sugarjar will likely be available starting in the upcoming
23.04 release. Until then, for supported LTS releases starting 20.04 (focal)
and the latest non-LTS release, you can use [this PPA](
https://launchpad.net/~michel-slm/+archive/ubuntu/sugarjar) maintained by the
Debian package maintainer.

Another option is to use our [omnibus
packages](https://github.com/jaymzh/sugarjar/releases). We keep Omnibus
packages for most distros that do not package Sugarjar. Omnibus packages are
distro packages (deb, rpm, etc.) that have all dependencies bundled up together
(including Ruby), and install in `/opt/sugarjar` allowing you to manage the
installation with your package manager, but with as a hermetically sealed app
that doesn't depend on the rest of your distro.

Finally, if none of those work for you, you can clone this repo and run it
directly from there.

## Auto cleanup squash-merged branches

It is common for a PR to go back and forth with a variety of nits, lint fixes,
typos, etc. that can muddy history. So many projects will "squash and merge"
when they accept a pull request. However, that means `git branch -d <branch>`
doesn't work. Git will tell you the branch isn't fully merged. You can, of
course `git branch -D <branch>`, but that does no safety checks at all, it
forces the deletion.

Enter `sj bclean` - it determines if the contents of your branch has been merge
and safely deletes if so.

``` shell
sj bclean
```

Will delete a branch, if it has been merged, **even if it was squash-merged**.

You can pass it a branch if you'd like (it defaults to the branch you're on):
`sj bclean <branch>`.

But it gets better! You can use `sj bcleanall` to remove all branches that have
been merged:

```shell
$ git branch
* argparse
  master
  feature
  hubhost
$ git bcleanall
Skipping branch argparse - there are unmerged commits
Reaped branch feature
Reaped branch hubhost
```

## Smarter clones and remotes

There's a pattern to every new repo we want to contribute to. First we fork,
then we clone the fork, then we add a remote of the upstream repo. It's
monotonous. SugarJar does this for you:

```shell
sj smartclone jaymzh/sugarjar
```

(also `sj sclone`)

This will:

* Make a fork of the repo, if you don't already have one
* Clone your fork
* Add the original as an 'upstream' remote

Note that it takes short names for repos. No need to specify a full URL,
just a $org/$repo.

Like `git clone`, `sj sclone` will accept an additional argument as the
destination directory to clone to. It will also pass any other unknown options
to `git clone` under the hood.

## Work with stacked branches more easily

It's important to break changes into reviewable chunks, but working with
stacked branches can be confusing. Enter `binfo` - it gives you a view of your
current branch all the way up to master. In this example imagine we have a
branch structure like:

```text
                      +- test2.1
                     /
master --- test --- test2 --- test3
```

This is what `binfo` on test3 looks like:

```shell
$ sj binfo
* e451865 (HEAD -> test3) test3
* e545b41 (test2) test2
* c808eae (test1) test1
o 44cf9e2 (origin/master, origin/HEAD, master) Lint/gemspec cleanups
```

while `binfo` on test2.1 looks like:

```shell
$ sj binfo
* 36d0136 (HEAD -> test2.1) test2.1
* e545b41 (test2) test2
* c808eae (test1) test1
o 44cf9e2 (origin/master, origin/HEAD, master) Lint/gemspec cleanups
```

## Have a better lint/unittest experience!

Ever made a PR, only to find out later that it failed tests because of some
small lint issue? Not anymore! SJ can be configured to run things before
pushing. For example,in the SugarJar repo, we have it run Rubocop (ruby lint)
and Markdownlint "on_push". If those fail, it lets you know and doesn't push.

You can configure SugarJar to tell it how to run both lints and unittests for
a given repo and if one or both should be run prior to pushing.

The details on the config file format is below, but we provide three commands:

```shell
git lint
```

Run all linters.

```shell
git unit
```

Run all unittests.

```shell
git smartpush # or spush
```

Run configured push-time actions (nothing, lint, unit, both), and do not
push if any of them fail.

## Better push defaults

In addition to running pre-push tests for you `smartpush` also picks smart
defaults for push. So if you `sj spush` with no arguments, it uses the
`origin` remote and the same branch name you're on as the remote branch.

## Cleaning up your own history

Perhaps you contribute to a project that prefers to use merge commits, so you
like to clean up your own history. This is often difficult to get right - a
combination of rebases, amends and force pushes. We provide two commands here
to help.

The first is pretty straight forward and is basically just an alias: `sj
amend`. It will amend whatever you want to the most recent commit (just an
alias for `git commit --amend`). It has a partner `qamend` (or `amendq` if you
prefer) that will do so without prompting to update your commit message.

So now you've rebased or amended, pushing becomes challenging. You can `git push
--force`, but everyone knows that's incredibly dangerous. Is there a better
way? There is! Git provides `git push --force-with-lease` - it checks to make
sure you're up-to-date with the remote before forcing the push. But man that
command is a mouthful! Enter `sj fpush`. It has all the smarts of `sj
smartpush` (runs configured pre-push actions), but adds `--force-with-lease` to
the command!

## Better feature branches

When you want to start a new feature, you want to start developing against
latest. That's why `sj feature` defaults to creating a branch against what we
call "most master". That is, `upstream/master` if it exists, otherwise
`origin/master` if that exists, otherwise `master`. You can pass in an
additional argument to base it off of something else.

```shell
$ git branch
  master
  test1
  test2
* test2.1
  test3
$ sj feature test-branch
Created feature branch test-branch based on origin/master
$ sj feature dependent-feature test-branch
Created feature branch dependent-feature based on test-branch
```

## Smartlog

Smartlog will show you a tree diagram of your branches! Simply run `sj
smartlog` or `sj sl` for short.

![smartlog screenshot](https://github.com/jaymzh/sugarjar/blob/main/smartlog.png)

## Pulling in suggestions from the web

When someone 'suggests' a change in the GitHub WebUI, once you choose to commit
them, your origin and local branches are no longer in-sync. The
`pullsuggestions` command will attempt to merge in any remote commits to your
local branch. This command will show a diff and ask for confirmation before
attempting the merge and  - if allowed to continue - will use a fast-forward
merge.

## And more!

See `sj help` for more commands!

## Using SugarJar as a git wrapper

SugarJar, by default, will pass any command it doesn't know straight to `hub`
(which passes commands **it** doesn't know to `git`). If you have configured
SugarJar to use `gh` instead of `hub`, then it will pass commands straight to
`git` since `gh` doesn't act as a `git` wrapper.

As such you can alias it to `git` and just have a super-git.

```shell
$ alias git=sj
$ git config -l | grep color
color.diff=auto
color.status=auto
color.branch=auto
color.branch.current=yellow reverse
color.branch.local=yellow
color.branch.remote=green
$ git br
* dependent-feature 44cf9e2 Lint/gemspec cleanups
  master            44cf9e2 Lint/gemspec cleanups
  test-branch       44cf9e2 Lint/gemspec cleanups
  test1             c808eae [ahead 1] test1
  test2             e545b41 test2
  test2.1           c1831b3 test2.1
  test3             e451865 test3
```

It's for this reason that SugarJar doesn't have conflicting command names. You
can turn off fallthru by setting `fallthru: false` in your config.

The only command we "override" is `version`, in which case we not only print
our version, but also call `hub version` which prints its version and calls
`git version` too!

## Configuration

Sugarjar will read in both a system-level config file
(`/etc/sugarjar/config.yaml`) and a user-level config file
`~/.config/sugarjar/config.yaml`, if they exist. Anything in the user config
will override the system config, and command-line options override both. The
yaml file is a straight key-value pair of options without their '--'. For
example:

```yaml
log_level: debug
github_user: jaymzh
```

In addition, the environment variable `SUGARJAR_LOGLEVEL` can be defined to set
a log level. This is primarily used as a way to turn debug on earlier in order to
troubleshoot configuration parsing.

## Repository Configuration

Sugarjar looks for a `.sugarjar.yaml` in the root of the repository to tell it
how to handle repo-specific things. Currently there options are:

* `lint` - A list of scripts to run on `sj lint`. These should be linters like
  rubocop or pyflake. Linters will be run from the root of the repo.
* `lint_list_cmd` - A command to run which will print out linters to run, one
  per line. Takes precedence over `lint`. The command (and the resulting
  linters) will be run from the root of the repo.
* `unit` - A list of scripts to run on `sj unit`. These should be unittest
  runners like rspec or pyunit. Test will be run from the root of the repo.
* `unit_list_cmd` - A command to run which will print out the unit tests to
  run, one more line. Takes precedence over `unit`. The command (and the
  resulting unit tests) will be run from the root of the repo.
* `on_push` - A list of types (`lint`, `unit`) of checks to run before pushing.
  It is highly recommended this is only `lint`. The goal here is to allow for
  the user to get quick stylistic feedback before pushing their branch to avoid
  the push-fix-push-fix loop.
* `commit_template` - A path to a commit template to set in the `commit.template`
  git config for this repo. Should be either a fully-qualified path, or a path
  relative to the repo root.

Example configuration:

```yaml
lint:
  - scripts/lint
unit:
  - scripts/unit
on_push:
  - lint
commit_template: .commit-template.txt
```

### Commit Templates

While GitHub provides a way to specify a pull-request template by putting the
right file into a repo, there is no way to tell git to automatically pick up a
commit template by dropping a file in the repo. Users must do something like:
`git config commit.template <file>`. Making each developer do this is error
prone, so this setting will automatically set this up for each developer.

## Enterprise GitHub

Like `hub`, SugarJar supports GitHub Enterprise. In fact, we provide extra
features just for it.

We recommend the global or user config specify the `github_host`. However, most
users will also have a few repos from upstream so always specifying a
`github_host` is sub-optimal.

So, when you overwrite the `github_host` on the command line, we go ahead and
set the `hub.host` git config in that single repo so that it'll "just work"
from there on out.

In other words, assuming your global SJ config has `github_host:
github.sample.com`, and the you clone sugarjar with:

```shell
sj clone jaymzh/sugarjar --github-host githuh.com
```

We will add the `hub.host` to the `sugarjar` clone so that future `hub` or `sj`
commands work without needing to specify..

## Choosing a GitHub CLI

SugarJar will use `gh` if it is available or otherwise fall back to `hub`. You
can override this by specifying `--github-cli` on the command line or setting
`github_cli` to either `gh` or `hub` (it defaults to `auto`) in your
configuration.

## FAQ

**Why the name SugarJar?**

It's mostly a backronym. Like jellyfish, I wanted two letters that were on home
row on different sides of the keyboard to make it easy to type. I looked at the
possible options that where there and not taken and tried to find one I could
make an appropriate name out of. Since this utility adds lots of sugar to git
and github, it seemed appropriate.

**Why did you originally use `hub` instead of the newer `gh` CLI?**

When I originally wrote SugarJar, `gh` was in early development, and `hub` had
many more features. In addition, I wrote SugarJar to be a wrapper for git/hub,
and `hub` allows this but `gh` does not.

When `gh` matured, we added experimental `gh` support in 0.0.11, and switched the
default to prefer `gh` in 1.0.0.

**I'd like to package SugarJar for my favorite distro/OS, is that OK?**

Of course! But I'd appreciate you emailing me to give me a heads up. Doing so
will allow me to make sure it shows up in the Repology badge above as well as
stop building Omnibus packages for whatever distro.

**What platforms does it work on?**

Since it's Ruby, it should work across all platforms, however, it's developed
and primarily tested on Linux as well as regularly used on Mac. I've not tested
it on Windows, but I'll happily accept patches for Windows compatibility.
