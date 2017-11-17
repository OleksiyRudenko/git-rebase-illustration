# git rebase techniques

This repo is designated for experiments with rebasing.

The only document changed is this README.md.

At this stage it is not so important to follow
the history of commits, therefore early commit messages
are used to easily identify commits (e.g. `A`, `B` etc.) 
and their rebased variations (e.g. `A'`, `B'`, `B''` etc.)

Decorated one-line logs of this repo are used to illustrate
the effect of rebase operations.

**Legend**
 * `A`, `B` etc. - commits
 * `A'`, `A''` etc. - rebased commits
 * _dev_ - branch name

## Initial state

There is some story of project development and _master_ represents
released product, with latest commits `A` and `B`. At `B` new development
branch _dev_ has started.

Other changes (e.g. documentation) were introduced to __master__ (`D`).

We are on __dev__.

```
*        e9378e5 C (HEAD -> dev)
| *      e58c3c0 D (origin/master, master)
|/  
*        bdef04a B
*        9fa8598 A
*        6b2487d Init
```

## Case 1. Sync with remote

We've done some changes `F` that we want to push, but it appears that
meanwhile someone has already updated remote with `E`.

`git pull` will most likely cause a merge conflict (which you may want to abort with `git merge --abort`).

The proper way would be `git pull --rebase`.

Anyway if you have forgotten to rebase on pull, do the following 
 1. `git merge --abort` (tree on the left)
 1. `git rebase origin/dev` (tree on the right)

```
*        65dfd56 F (HEAD -> dev)              *        3678a93 F' (HEAD -> dev)
| *        0b160f5 E (origin/dev)             *        0b160f5 E (origin/dev)
|/                                            |
*        801f263 C                            *        801f263 C
| *      e58c3c0 D (origin/master, master)    | *      e58c3c0 D (origin/master, master)
|/                                            |/  
*        bdef04a B                            *        bdef04a B
*        9fa8598 A                            *        9fa8598 A
*        6b2487d Init                         *        6b2487d Init
```

Should you anyway run into merger conflict:
 1. Resolve conflicts
 1. `git add <changed-files>`
 1. `git rebase --continue` to complete
 
Note that your commits are assigned with different hashes. `F => F'`


G1 G2
G3
G4

H1
H2
