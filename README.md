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
 * `A`, `B`, `B1`, `B2` etc. - commits
 * `A'`, `A''`, `A1'` etc. - rebased commits
 * _dev_ - branch name

## Initial state

There is some story of project development and _master_ represents
released product, with latest commits `A` and `B`. At `B` new development
branch _dev_ has started.

Other changes (e.g. documentation) were introduced to _master_ (`D`).

We are on _dev_.

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

`git push` will require to merge in remote changes before any push.

`git pull` will most likely cause a merge conflict (which you may 
want to abort with `git merge --abort`).

The proper way would be `git pull --rebase` or `git fetch && git rebase origin/dev`.
`git push` to update the remote immediately.

Anyway if you have forgotten to rebase on pull, do the following 
 1. `git merge --abort` (tree on the left)
 1. `git rebase origin/dev` (tree on the right)

```
*   65dfd56 F (HEAD -> dev)              *   b7200ef F'(HEAD -> dev)
| * 0b160f5 E (origin/dev)               *   0b160f5 E (origin/dev)
|/                                       |
| * e58c3c0 D (origin/master, master)    | * e58c3c0 D (origin/master, master)
* | 801f263 C                            * | 801f263 C
|/                                       |/
*   bdef04a B                            *   bdef04a B
*   9fa8598 A                            *   9fa8598 A
*   6b2487d Init                         *   6b2487d Init
```

(_actual `git log` output above and further down is slightly amended for the sake of readability_)

Should you anyway run into merger conflict:
 1. Resolve conflicts
 1. `git add <changed-files>`
 1. `git rebase --continue` to complete
 
Note that `F'` is assigned a different SHA.

## Case 2. Tiding up

You work on a feature on its own branch _fea01_. Every addition 
and amendment has its own commit. Finally you feel code is good
enough (tested, etc) to be merged.

You find out that commits `G1`..`G4` refer to class **Golf**, 
`H1`..`H3` refer to class **Hockey**, and `I1`..`I4` refer to
some core function.

```
*    c051b13 I4 (HEAD -> fea01)
*    0324a66 H3
*    b23329b I3
*    b91896e G4
*    0b2fd60 I2
*    f9f844d H2
*    fbff5ee I1
*    5f382e6 G3
*    01664cf G2
*    13ee55b H1
*    ede1d37 G1
*    acc1c91 F' (origin/dev, dev)
*    0b160f5 E
| *  e58c3c0 D (origin/master, master)
* |  801f263 C
|/
*    bdef04a B
*    9fa8598 A
*    6b2487d Init
```

However commits related to an entity are scattered across the branch.
It is sensible to group commits per entity and to probably even squash
commits related to the same entity for it doesn't make sense to keep 
track for every tiny change anymore (especially if some part of code
has evolved and its early versions are obsolete and useless).

### Targets

Bring some order, squash commits. Have a back up to start over.

### Rebase script

Rebasing commits on a branch has a side-effect:
```
 ...-G-H-I-J-K
             ^ feature-branch
			 
 git rebase -i HEAD~4              # (J-K-I-H)
 
 ...-G-J'-K'-I'-H'
                ^ feature-branch
```

Where are H, I, J, K? They are gone (almost), there is no 
reference (branch or tag) to track them.

Just create a branch pointer at current position to be able
to undo things easily.

```
git checkout fea01
# Create a back-up pointer
git branch fea01-backup
# Rebase last 11 commits; 
# alternatively refer to a particular SHA, e.g. acc1c91 (refers to F')
git rebase -i HEAD~11

# here is what git offers as a command list
pick becaf4d G1
pick b74e607 H1
pick 44db105 G2
pick 6a5edee G3
pick 2ec4aa5 I1
pick 91c528f H2
pick 427403a I2
pick 25c5f5a G4
pick 09def4c I3
pick 6406b2e H3
pick 9d4219f I4
# you see commits listed in chronological order

# bring some order
pick becaf4d G1
pick 44db105 G2
pick 6a5edee G3
pick 25c5f5a G4
pick b74e607 H1
pick 91c528f H2
pick 6406b2e H3
pick 2ec4aa5 I1
pick 427403a I2
pick 09def4c I3
pick 9d4219f I4

# squash (retaining only first commit in a group message)
pick becaf4d G1
fixup 44db105 G2
fixup 6a5edee G3
fixup 25c5f5a G4
pick b74e607 H1
fixup 91c528f H2
fixup 6406b2e H3
pick 2ec4aa5 I1
fixup 427403a I2
fixup 09def4c I3
fixup 9d4219f I4
# Save and quit the editor

# Resolve conflicts if any
# You may run into commit --allow-empty warning if you were undoing
# changes by means of committing deletions of what was added before
git commit --allow-empty -m "abc..."

# Or you may be required to resolve conflicts manually.
# When done stage files
git add README.md

# Whenever git rebase prompts for an additional action
# when done do:
git rebase --continue

# Redo the above until success reported

# At any stage until success reported you may revert to the initial state
git rebase --abort
```

Notes:
 * `pick` : keep commit as is
 * `reword` : make a pause,  edit message
 * `edit` : make a pause, make any changes
 * `squash` : commit squahed onto upper, message added up
 * `fixup` : commit squashed onto upper, message ignored
 * `drop` : do not add this commit

Merger conflicts appear often when you are trying to reorder changes to the
same file. Good reason to keep classes in their own files.

### Result

Now your tree looks like
```
*    d034fee I' (HEAD -> fea01)
*    20f43d4 H'
*    35fcd3b G'
| *  c051b13 I4 (HEAD -> fea01-backup)
| *  0324a66 H3
| *  b23329b I3
| *  b91896e G4
| *  0b2fd60 I2
| *  f9f844d H2
| *  fbff5ee I1
| *  5f382e6 G3
| *  01664cf G2
| *  13ee55b H1
| *  ede1d37 G1
|/
*    acc1c91 F' (origin/dev, dev)
*    0b160f5 E
| *  e58c3c0 D (origin/master, master)
* |  801f263 C
|/
*    bdef04a B
*    9fa8598 A
*    6b2487d Init
```

You've got 3 squashed and ordered commits `G'`, `H'`, and `I'`.
_fea01-backup_ points at rebase source commits.

If anything goes wrong just get rid of rebased commits:
 1. `git checkout fea01-backup` - switch to backup branch
 1. `git branch -f fea01` - forcely re-assign _fea01_ to the "old" location

Now your're totally back. Try again :)

If you're happy with the rebase result (e.g. tests run OK) 
just get rid of _fea01-backup_.

```
# forcely remove branch
git branch -D fea01-backup

# results in
*    d034fee I' (HEAD -> fea01)
*    20f43d4 H'
*    35fcd3b G'
*    acc1c91 F' (origin/dev, dev)
*    0b160f5 E
| *  e58c3c0 D (origin/master, master)
* |  801f263 C
|/
*    bdef04a B
*    9fa8598 A
*    6b2487d Init


```

### Workflow

You may keep developing on _fea01_ further, rebasing onto _fea0-release_
and further onto _dev_ from time to time whenever you have your changes 
consistent and wish to test it against codebase of a higher level.

So general workflow recurring feature development is:

1. Branch off _feaXX_ from _dev_
2. Commit to _feaXX_
3. Make a backup branch pointer _feaXX-backup_
4. Rebase and test until you're happy
5. Kill _feaXX-backup_

### Bonus

```
* 1b9180 Fix query further
| ...
* d4f67a Fix filtered query
```

`1b9180` is a catch up of `d4f67a` where we've forgotten to add a semi-colon (e.g.)

Normally, you would squash it during rebase.

Just tell git to suggest squashing at interactive rebase when committing:

```
* 1b9180 fixup! Fix filtered query
| ...
* d4f67a Fix filtered query
```

At `git rebase -i` `1b9180` will be suggested right after `d4f67a` with a fixup command.
Also `git rebase -i --autosquash <targetBase>` will try its best.
