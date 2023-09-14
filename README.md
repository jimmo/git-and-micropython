# Git and MicroPython

[![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

A guide for using git with the [MicroPython project](https://github.com/micropython/micropython).
The advice here is tailored specifically to the workflow used by the
MicroPython project, but should apply generally to most projects that use git
and GitHub.

The advice here has been collected from personal experience of several years
of working as a maintainer on the MicroPython project, and aims to address the
common scenarios that contributors might run into.

## Status

DRAFT. Currently this is a work in progress, many of the snippets have not
been tested.

Copyright © 2023 [Jim Mussared](https://github.com/jimmo)

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg

## Contributing

Please [raise an
issue](https://github.com/jimmo/git-and-micropython/issues/new) if you have a
scenario that isn't covered here, or if the advice here doesn't work for you.
Please include as much information as possible, including commands that you
have run and corresponding output.

PRs are also most welcome!

If you found this guide useful, please consider
[sponsoring me](https://github.com/sponsors/jimmo/) or the
[MicroPython project](https://github.com/sponsors/micropython/) on GitHub Sponsors.

## Quick notes

* Don't use the GitHub Desktop tool, and prefer to use `git` directly rather
  than GitHub's `gh` client. These are good tools, but designed with a
  different workflow in mind.
* It can however be very helpful to use a graphical git client for some
  operations. For example, I highly recommend
  [https://www.sublimemerge.com/](https://www.sublimemerge.com/) especially
  for working with merge/rebase conflicts, browsing the repo, creating
  commits, and in particular selectively stating files, hunks, or lines. It
  works very well in conjunction with the `git` command line tool.
* Force push (i.e. `git push -f`) is totally normal and a required part of the
  PR workflow (despite what some guides will tell you).
* You almost never need to close and create a new PR to resolve an issue with
  your PR / commits. If you think you need to close your PR to update it, please
  raise an issue against this repo to describe your scenario and I can help you.
* Never use `git merge`. MicroPython maintains a completely linear history.
* You almost never need to use `git pull`. The one exception is if you want to
  update the master branch with the latest changes. I prefer to work with
  "detached head" for this, i.e. `git fetch origin; git switch --detach origin/master`.
* Consider adding [git aliases](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases)
  to simplify common tasks. e.g. I have `git rmi` for `git rebase -i origin/master`.

## Assumptions

This guide assumes you've already set up your fork and local clone, and have
the "upstream" `micropython/micropython` repository as a remote named
`origin`, and your fork as a second remote (e.g. named the same as your GitHub
username).

For example, I have:

```bash
$ git remote -v
jimmo   git@github.com:jimmo/micropython.git (fetch)
jimmo   git@github.com:jimmo/micropython.git (push)
origin  git@github.com:micropython/micropython.git (fetch)
origin  git@github.com:micropython/micropython.git (push)
```

**These remotes can be named whatever you like**, but the rest of the guide
will use `origin` to mean "the remote that is `micropython/micropython`, and
`jimmo` to mean the remote that points to my fork. Substitute `jimmo` in this
guide with your username.

### Other remote naming schemes

It is also common to use `origin` as the remote pointing to your fork, and
`upstream` to mean `micropython/micropython`. I discourage this because

a) It feels "natural" that `origin/master` should be the latest upstream code.
   Your local `master` branch should track `origin/master`, and it's a hassle
   to keep your fork's `master` branch updated.
b) "upstream" can be confused with the concept of a branch's "upstream branch"
   (i.e. the `--set-upstream` flag in `git push`).
c) It is common for a contributor to first start as a MicroPython user, i.e.
   the first repo they clone will be `micropython/micropython`, hence that
   will be their `origin`, then later they create and add their fork as a
   remote.
d) On every project I work with, `origin` is always the "source" (i.e. the
   main project). `origin/master` (or ideally, `origin/main`) is _always_ the
   latest code for the project. This is true regardless of whether it's one of
   my tiny repos or a big project (e.g. Zephyr or MicroPython).
e) Sometimes you want to add other remotes (e.g. other contributors, or
   downstream forks, and this allows you to keep with the pattern of using the
   username, e.g. `adafruit`).

But as above, you are welcome to set it up this way too. For example:

```bash
$ git remote -v
origin    git@github.com:jimmo/micropython.git (fetch)
origin    git@github.com:jimmo/micropython.git (push)
upstream  git@github.com:micropython/micropython.git (fetch)
upstream  git@github.com:micropython/micropython.git (push)
```

And then you would use `git fetch upstream` anywhere I have written `git fetch
origin` below, and `git push -u origin` where I have written `git push -u
jimmo` below.

### Authentication

Note these remotes all use `git@`, i.e. they rely on having set up [SSH key
authentication to GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh). You can also [set up a passkey](https://docs.github.com/en/authentication/authenticating-with-a-passkey/about-passkeys).

### Starting from scratch

#### Fork

This can be done from the GitHub web interface, by [clicking the fork button](https://github.com/micropython/micropython/fork). You should leave the "Copy the master branch only" checkbox checked.

#### Clone & add remotes

Now you can clone the MicroPython repository locally:

```bash
cd src
git clone git@github.com:micropython/micropython.git
cd micropython
git remote add jimmo git@github.com:jimmo/micropython.git
git fetch jimmo
```

# Scenarios

### Creating a new PR

The typical workflow is:

1. Get the latest code from upstream:

```bash
git fetch origin
```

2. Create a new branch to work on your feature / fix, based on the latest
   code, and switch to it:

```
git switch -c <new-branch> origin/master
```

3. Implement your feature:

Use `git add` and `git commit -s` to build up your set of commits.

Use `git rebase -i origin/master` to restructure commits as necessary.

4. Create the PR:

```
git push -u jimmo
```

Change `jimmo` to the name of your fork's remote (your GitHub username if
you're using the instructions above). The `-u` flag will make this the
"upstream" for this branch, so that subsequent pushes will automatically go
there.

### I forgot to sign off a commit

You can use the `-s` flag to `git commit` to add the `Signed-Off-By` line, but
it's easy to forget. The automated checks will flag this when you create the
PR.

#### My PR only has one commit

Run

```bash
git switch <branchname>
git commit --amend -s
git push -f
```

This will amend the commit, and `-s` will automatically add the
`Signed-Off-By` line.

#### My PR has multiple commits

Run

```bash
git fetch origin
git switch <branchname>
git rebase -i origin/master
```

Then for each commit you need to update, change `pick` to `edit`, then run

```bash
git commit --amend -s
git rebase --continue
```

for each commit, and then run

```bash
git push -f
```

to update the PR.

### I need to rebase my PR / branch

You will have created your branch at some point by branching off the `master`
branch. But periodically you will want to rebase in order to get the latest
changes.

Run

```bash
git fetch origin
git switch <branchname>
git rebase -i origin/master
git push -f
```

You don't strictly need the `-i` (interactive) on the rebase, but it's a good
check that it's going to do what you expect (i.e. it should list just your
commits).

### I want to make changes to the commits my PR / branch

For example, to address review comments, or just to keep working on it after
creating the PR.

#### My PR only has one commit

Make your changes, stage them (with `git add`) and then use `git commit --amend`
to add them to the existing commit. Then `git push -f` to update the PR. You can
also use `git commit --amend -a` to avoid needing to use `git add`.

#### My PR has multiple commits

It is common that you need to make changes that need to go into different
commits. There are lots of ways to do this, but I find the simplest workflow
is to make new (temporary) fixup commits with changes for each of your
existing commits, and then "fix" them into those commits.

_This is a workflow that Sublime Merge makes especially easy -- instead of
using `git add -p` you can graphically chose the files/hunks/lines to add to
the fixup commits._

For example, if your PR branch had the following three commits:

```
- py/x.c: Add support for X.
- py/y.c: Add support for Y.
- py/z.c: Use X and Y to implement Z.
```

and furthermore, all three commits made changes to `py/runtime.h`.

And now you have unstaged changes (e.g. address review comments) to
`py/runtime.h`, `py/x.c`, `py/y.c` and `py/z.c`.

Now create a temporary commit with just the changes relevant to the first
commit.

```bash
git add -p py/runtime.h
git add py/x.c
git commit -m 'fixup! fixes for X.'
```

`git add -p` will let you selectively choose the changes to `py/runtime.h`
that you want.

Now same for the second and third commits:

```bash
git add -p py/runtime.h
git add py/y.c
git commit -m 'fixup! fixes for Y.'
git add -p py/runtime.h
git add py/z.c
git commit -m 'fixup! fixes for Z.'
```

Now "fix" them into the original commit (and rebase while you're at it)

```bash
git fetch origin
git rebase -i origin/master
```

Re-order the lines so that they are grouped, and change `pick` to `fix` for
the fixup commits.

```
pick <hash> py/x.c: Add support for X.
fix <hash> fixup! fixes for X.
pick <hash> py/y.c: Add support for Y.
fix <hash> fixup! fixes for Y.
pick <hash> py/z.c: Use X and Y to implement Z.
fix <hash> fixup! fixes for Z.
```

Then `git push -f` to update the PR.

This also works for adding/removing/moving files.

### A maintainer made changes to my branch

We generally only do this immediately before merging your PR. If you use
`git pull` (and especially if you do not have `[pull] rebase = true` in your
`.gitconfig`), it will likely get into a mess.

Instead what you should do is fetch your fork's remote, then reset the branch to this:

```
git fetch jimmo
git switch <branchname>
git reset --hard jimmo/<branchname>
```

This is essentially saying: "just make my local branch equal to the updated remote".

### I want to add IDE-specific rules to `.gitignore`

There are lots of reasons to add rules to `.gitignore`, but we typically
prefer that you set up local rules instead.

You can do this by adding rules (using the same format as `.gitignore`) to `.git/info/exclude`.

### I need to split my PR into smaller commits

One option here is to soft reset your branch, which has the effect of
unstaging all your changes, then re-create the commits. See the next scenario
for how to do this.

But usually you want to split just a single commit without re-creating the
commit. For example, if you had the following

```
- py/x.c: Add support for X.
- py/z.c: Add Y and use Y and X to implement Z.
- tests/basics/z.py: Add test for Z.
```

and wanted to split the second commit, first to implement Y, then to implement
Z.

```
git fetch origin
git rebase -i origin/master
```

There are two ways to do this:

#### Repeat the commit

In the rebase TODO list, duplicate the line for the commit you want to split,
and change `pick` to `edit` for both.

Now you can remove the things from the first commit you don't want to be
there. Stage these changes and use `git commit --amend` to update the commit
message. Then `git rebase --continue` to move to the second commit, which will
add back just the removed changes. Use `git commit --amend` to update the
commit message, then `git rebase --continue` to finish.

In more detail, what happens here is when the rebase attempts to apply the
second copy of the original commit, it does so on the partial set of changes
that you made in the modified first commit.

It's possible you may see a rebase/merge conflict on the second commit. This
is expected if you want the intermediate state to be something other than
either "no change" or the final change.

_Using a tool like Sublime Merge makes removing the unwanted changes from the
first commit easy, because you can interactively remove the diffs._

#### Edit the commit

Instead of the duplicate-commit approach, you can just switch the commit to
`edit`, then use TODO... (stage the removal, with the un-removal unstaged).
Then `git commit --amend`, then `git commit -a` to add a second commit.

### My branch is a mess, I want to recreate the commits from scratch

This might be the case if your PR shows the expected diff, but the commit tab
shows lots of unexpected commits (or merge commits).

Run the following commands on your branch:

```bash
git fetch origin
git reset --soft origin/master
```

This will
* Ensure that the `origin` remote is up to date, so that `origin/master`
  reflects the current latest state.
* Reset your branch such that it points to the same commit as `origin/master`,
  but (thanks to `--soft`) preserve your changes. All your changes will now be
  unstaged, ready to be staged (via `git add`) and committed.

For example, you might now do the following to create a commit

```bash
git add -p
git commit -s
```

This will allow you to interactively chose which differences you want to use,
and then commit them (with sign-off). Repeat this until you have no more
unstaged changes.

Then you can use

```bash
git push -f
```

to update the PR.

### I want to test someone's PR

If you know the PR number, you can make a local branch with the contents of
that PR.

```bash
git fetch origin pull/<PR>/head:<username>_<branchname>
git switch <username>_<branchname>
```

If you plan to make changes, you may want to create your own branch from this
point.

If they update the PR, you can use:

```bash
git fetch -f origin pull/<PR>/head:<username>_<branchname>
```

to force update it from the remote. If you are currently on this branch, you
must temporarily switch to another branch before doing this, e.g.

```
git switch --detach
git fetch -f origin pull/<PR>/head:<username>_<branchname>
git switch -
```

which will detach HEAD, fetch the branch, then switch back to the branch.

_Note: You can use anything you like in place of `<username>_<branchname>` as
the local branch name, this is just my convention._

# Concepts

## Remotes & forks

TODO

## Rebasing

TODO

## Sign off

This is MicroPython's equivalent of a [Developer Certificate of Origin](https://en.wikipedia.org/wiki/Developer_Certificate_of_Origin). See also
[The Developer Certificate of Origin is a great alternative to a CLA](https://drewdevault.com/2021/04/12/DCO.html).

See the [MicroPython Code Conventions](https://github.com/micropython/micropython/blob/master/CODECONVENTIONS.md#git-commit-conventions)
for more details and the background about why the MicroPython project requires
this.

It is not the same thing as "commit signing", i.e. anything about PGP keys or cryptography is not relevant to this.

In most cases, if you always remember to use `git commit -s` you'll never need
to worry about this. You can use `git commit --amend -s` to add the sign-off
you a commit you just created.
