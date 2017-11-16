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
 * __dev__ - branch name

## Initial state

There is some story of project development and __master__ represents
released product, with latest commits `A` and `B`. At `B` new development
branch __dev__ has started.