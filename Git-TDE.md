# Where am I?
- `git log` -- a more detailed point of view is useful sometimes!
- `git log -N` -- show the last N commits
- `git log --decorate` -- shows where the branch pointers are pointing
- `git log --oneline -N` -- oneline is really useful, with helpful colorization at least in Git Bash
- `git log --oneline --decorate --all --graph` -- the kitchen sink!
For general daily use I like `git log --oneline -N` the best...

# Mirror mirror...
- `git branch <new branch name>` -- creates the new branch but you stay where you are
- `git checkout <the-branch>` -- moves you to the branch you requested
- `git checkout -b <new-branch-name>` -- creates the new branch and moves you there
Note that `git branch` and `git checkout -b` will create an identical clone from _where you are now_ in the commit history of the current branch. This is useful knowledge when you are trying to debug something!

# Who am I?
- `git branch`
- `git branch -vv` -- so great, shows remote/upstream/tracking branch
- `git branch -avv` -- overwhelming, but informative
The shortest/easiest way to grab a branch you see on remote (with `-a`) but don't have locally:
- `git checkout <the-name-of-the-remote-branch-minus-origin-slash>`
The easiest way to push a branch that exists only locally to your remote and ensure it's tracked there is:
- `git push -u origin <the-name-of-my-local-branch-not-yet-tracked-on-remote>`
From that point forward you can just `git push` and don't have to worry about `-u` (`--set-upstream`) nonsense.

# What have I changed?
- `git diff` -- what hasn't been staged or committed?
- `git diff --cached` or `git diff --staged` -- diff between staging area and HEAD

# MDE-specific tips and tricks

## Rolling back to a prior state
Bitbucket has custom hooks (required by MDE) which prevent the rewriting of history. This is annoying at times, but you always have options for working around it...

### Work around it locally, then push
One of the quickest and simplest ways to "roll back" to an arbitrary prior commit or tag is to use any variation of of `git reset` that you prefer. For a good explanation of git reset, see https://www.gitkraken.com/learn/git/git-reset.

If you just go ahead and do this on the remote-tracked branch, when you go to commit, you'll be prevented from doing so by the MDE commit hook that is in place.

Instead, just do the "local workaround" and push a fresh branch to your remote:
- `git checkout <branch-to-be-rolled-back> && git pull` -- make sure you're where you want to be
- `git checkout -b my-new-local-copy` -- make a local copy, do _not_ push to remote!
- run your `git reset` command of your choice -- roll back the state, locally!
- _now_ push to remote: `git push -u origin my-new-local-copy`

You have just successfully rewritten history. The same process can work for any other kind of "rewriting history" type of command you might want to use, such as `rebase -i` for instance...

### Use the "magical Git incantation" -- `git read-tree -um <roll-back-ref>`
This is one of my favorte "magical Git incantations" of all time. I learned it from Eli Zukowski. It works. It basically "rolls back" your state but does so by moving forwards in time (with a new commit), not backwards like with the `git reset` variations. No commits are removed. Running the command will stage (but not commit) all the changes involved in the rollback.

The way I use it is to roll back by state (on my current branch) to the roll-back-ref which is a prior commit (on my current branch). Scanning through the Git manual on this command indicates it is generally used for much more complex use-cases than this. But here we're keeping it super simple, and just using its "magical powers" for rolling back while actually moving forward...

### More detail on a few `git reset` varieties
- `git reset --soft`
    - say you have some changes in your working tree
    - and you run git reset --soft
    - the 'rollback' changes will drop into the staged area (e.g. appear green in your git status)
    - your working tree changes will stay in your working tree (e.g. appear red in your git status)
    - this is super handy! and neat!
- `git reset --hard`
    - everything between you and the ref you're rolling back is now gone
    - nothing is staged
    - nothing in the working tree
    - just an empty working tree and staging area, and HEAD rolled back to the ref you asked for
    - poof, everything is gone
- `git reset --mixed`
    - similar to `git reset --soft` except that everything drops into the working tree, nothing is staged
    - say you alreayd had changes in your working tree
    - then you run `git reset --mixed`
    - everything drops into the working tree (e.g. appears red in the git status)
    - any local changes you had going are now have the rollback changes mixed in

Mostly, I use `git reset --hard` for rollback scenarios such as the one described above. In those cases I know exactly what I need and I'm not worried about "losing" stuff b/c everything is on remote, and/or I just created a new mirror branch locally, so nothing is ever actually 'lost'. However, if you don't have your branch mirrored locally or pushed to remote, then be careful with `git reset --hard`.

On the other hand, `git reset --soft` is a great way to do some rebasing locally, without having to dive into `git rebase -i`! I really liked this Stack Overflow answer which gave me that insight -- [Use Case - Combine a series of local commits](https://stackoverflow.com/a/26172014)

# Rebasing -- `git rebase -i`
I really can't say it better than this:
> And then there’s `git rebase --interactive`, which is a bit like `git commit --amend` hopped up on acid and holding a chainsaw - completely insane and quite dangerous but capable of exposing entirely new states of mind. Here you can edit, squash, reorder, tease apart, and annotate existing commits in a way that’s easier and more intuitive than it ought to be.
> source: https://tomayko.com/blog/2008/the-thing-about-git

However, that said, `git rebase -i` can actually be lots of fun and quite instructive. See the [documentation](https://git-scm.com/docs/git-rebase) but really, the best way to learn it is by doing: just run `git rebase -i` and follow the on-screen commands and explanations.