---
layout: post
title:  "Managing Remotes"
date:   2025-07-23 14:00:00 +0100
tags: git remotes
categories: git
---

Git allows collaboration using remotes. After you clone a repository, you get your first remote: `origin`. In the OpenJDK, you likely added a remote called `upstream`. Yet the remotes aren't limited to these two, you can have as many remotes as you like. Use [`git remote`](https://git-scm.com/docs/git-remote) command to manage remotes.

## Adding Others' Remotes

Let's add some remotes to ease looking at changes:

```bash
git remote add renjith https://github.com/Renjithkannath/jdk.git
git remote add rajat   https://github.com/rajamah/jdk.git
```

The output of `git remote` could look like this:

```text
git remote
origin
rajat
renjith
upstream
```

## Collaborating

### Example 1

To review changes in a branch Rajat's working on, fetch it and checkout it:

```bash
git fetch rajat 8321151:rajat/8321151-theme-fallback
git checkout rajat/8321151-theme-fallback
```

Rajat's branch is `8321151`, it exists in his remote, and now it's also a remote branch in my local repository. After the colon, I specified the name of the *local* branch that I'll use to refer to his branch, its name is `rajat/8321151-theme-fallback`.

If Rajat pushes additional changes to his branch, use [`git pull`](https://git-scm.com/docs/git-pull) to pull his changes into your local branch.

```bash
git pull rajat 8321151
```

It's possible to add your own changes and share them:

```bash
git push origin rajat/8321151-theme-fallback
```

(It could be possible to push directly to another remote if you agreed on that and allowed doing so. It's how [Skara](https://wiki.openjdk.org/display/SKARA) backports work: after the bot creates a backport for you, you can add additional changes into the branch that the bot created by pushing directly into that branch.)

At this stage, Rajat needs to pull the changes into his local branch:

```bash
git pull https://github.com/aivanov-jdk/jdk.git rajat/8321151-theme-fallback
```

Yes, you can use a URL instead of a name, yet a name is more convenient when you share code frequently.


### Example 2

Get Renjith's branch to review it:

```bash
git fetch renjith 8320343-v1:renjith/8320343-generate-GIFs
git checkout renjith/8320343-generate-GIFs
```

Pull further changes Renjith made:

```bash
git pull renjith 8320343-v1
```

Review the latest changes in your favourite tool.

## Remote Branches

### Listing Branches

To list remote branches, use `-r` or [`--remotes`](https://git-scm.com/docs/git-branch#Documentation/git-branch.txt---remotes) option to [`git branch`](https://git-scm.com/docs/git-branch). Let's explore different variants:

```bash
$ git branch
  rajat/8321151-theme-fallback
* renjith/8320343-generate-GIFs
$ git branch -r
  origin/rajat/8321151-theme-fallback
  rajat/8321151
  renjith/8320343-v1
$ git branch -a
  rajat/8321151-theme-fallback
* renjith/8320343-generate-GIFs
  remotes/origin/rajat/8321151-theme-fallback
  remotes/rajat/8321151
  remotes/renjith/8320343-v1
```

By default, `git branch` lists *local* branches; with `-r`/`--remotes`, it lists *remote* branches; with `-a`/[`--all`](https://git-scm.com/docs/git-branch#Documentation/git-branch.txt---all), it lists *all* the branches.

### Deleting Branches

To delete remote branches, add the `-r`/`--remotes` option to the `-d`/[`--delete`](https://git-scm.com/docs/git-branch#Documentation/git-branch.txt---delete) option.

If you fetch from a remote without specifying a particular branch, you'll fetch all the branches that exist in the remote at that moment. You probably don't want to do it.

To keep the list of branches manageable, delete old branches. When you delete a remote branch, you delete its record from your local repository, the branch in the remote is not affected.

#### Cleanup Examples

As you can see, fetching remote branches creates local records, pushing a branch to your `origin` remote creates another branch. Once the work on the branches is finished, remove the local records:

```bash
git branch --delete --remotes rajat/8321151 renjith/8320343-v1
```

To remove local branches, use `--delete --force` options or [`-D`](https://git-scm.com/docs/git-branch#Documentation/git-branch.txt--D) as its shortcut:

```bash
git branch --delete --force rajat/8321151-theme-fallback renjith/8320343-generate-GIFs
```

You need the [`--force`](https://git-scm.com/docs/git-branch#Documentation/git-branch.txt---force) option because the changes from the branch aren't merged; even after the PR is integrated, it's still needed because Skara squashes all the commits. **Note:** ensure there's nothing in the branch that you'd like to preserve.

To remove a branch in the `origin` remote, push an empty branch:

```bash
git push origin :rajat/8321151-theme-fallback
```

Alternatively, you can use GitHub interface to remove branches. If you remove branches in GitHub interface, use `git fetch --prune origin` to fetch all the changes from your `origin` and *prune* (remove) all `origin`'s remote branches locally that were removed in `origin`.
