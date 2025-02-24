# Mirror mirror...
- `git branch <new branch name>` -- creates the new branch but you stay where you are
- `git checkout <the-branch>` -- moves you to the branch you requested
- `git checkout -b <new-branch-name>` -- creates the new branch and moves you there
Note that `git branch` and `git checkout -b` will create an identical clone from _where you are now_ in the commit history of the current branch. This is useful knowledge when you are trying to debug something!

# Where am I?
- `git log` -- a more detailed point of view is useful sometimes!
- `git log -N` -- show the last N commits
- `git log --decorate` -- shows where the branch pointers are pointing
- `git log --oneline -N` -- oneline is really useful, with helpful colorization at least in Git Bash
- `git log --oneline --decorate --all --graph` -- the kitchen sink!
For general daily use I like `git log --oneline -N` the best...

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
- `git status` -- shows stuff git does / doesn't know about -- reminds you to add new files!
- `git diff` -- what's stuff that git already knows about, but hasn't been staged or committed?
- `git diff --cached` or `git diff --staged` -- diff between staging area and HEAD

# Making it official...
- `git add <file-path>`
- `git add .`
- `git commit -m "your-commit-message"`
- `git commit -am "your commit message"` -- the `-a` automatically stages and commits, but this only works if the file in question has already been initially committed, this doesn't pick up new files. The way you can tell is by looking at `git diff` versus `git status`. But please note that the `-a` option does pick up deleted files.

## Untangling a working tree into atomic commits
Suppose you've been working away on your local branch and when it comes time to commit, you realize you have several unrelated changes all living together in your currnet working copy. How to untangle things into atomic commits? The patch option is super helpful for stashing or staging just part of your WIP working tree!
- `git stash -p -m "your stash message"` (then, later, you unstash the rest and commit it, too)
- `git add -p`

## Oops! I forgot something!...

### Just for the prior commit
When you need to add something to the commit you just made... _If that commit hasn't yet been pushed to remote_, then you're in luck. Just stage the stuff you want to add to your just prior commit, and do:
- `git commit --amend --no-edit` -- if you want to leave the commit message the same
- `git commit --amend -m "your new commit message"` -- if you want to change the commit message
- optionally include `-a` if it's just modifications to existing files, but remember new files need their own explicit `git add`, first!

### For something longer ago than that...
Lots of options exist, but the question becomes: Is it worth it? How important is it that the thing you forgot get melded in with that earlier commit you have in mind? This is because "rewriting history" isn't allowed by the MDE BitBucket pre-commit hooks, so you'd have to engage in some workaround, described below...
If you decide it's not worth the extra trouble of getting around the "you can't rewrite history" rule, then just make a new commit with the change you forgot... It's okay... We all end up with messy commit histories, unless we use `git rebase -i` on the regular...

# Some MDE-specific tips and tricks

## I just pushed put the commit isn't showing up in my PR!
I've seen this happen more than once with Bitbucket. You push a commit. You know you did. It shows up under the commits for the branch. But it's not showing up in the PR. You can't "re-push" an already pushed commit, so how to get the PR to update, especially if, say, it was the last commit in the workflow, like putting in a tag, or a versioning commit, and you have nothing else to "say" on that branch. In that case, you can just push an empty commit. Here's the "magical incancation" for that: 
- `git commit --allow-empty -m "Pushing an empty commit so that the prior commits will show up in the PR"`
As a courtesy, it's nice to write a descriptive commit message explaining why you are pushing this empty commit...

## Rolling back to a prior state
Bitbucket has custom pre-commit hooks (required by MDE) which prevent the rewriting of history. This is annoying at times, but you always have options for working around it...

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

# A few extra details

## A few `git reset` varieties
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

## Rebasing -- `git rebase -i`
I really can't say it better than this:
> And then there’s `git rebase --interactive`, which is a bit like `git commit --amend` hopped up on acid and holding a chainsaw - completely insane and quite dangerous but capable of exposing entirely new states of mind. Here you can edit, squash, reorder, tease apart, and annotate existing commits in a way that’s easier and more intuitive than it ought to be.
> source: https://tomayko.com/blog/2008/the-thing-about-git

However, that said, `git rebase -i` can actually be lots of fun and quite instructive. See the [documentation](https://git-scm.com/docs/git-rebase) but really, the best way to learn it is by doing: just run `git rebase -i` and follow the on-screen commands and explanations.

## Stashing
- `git stash` is a handy tool especially when used with `--patch` (`-p`) for stashing away just part of your working tree, and untangling a messy WIP situation into discrete, atomic commits.
- This is still one of my favorite `git stash` primers -- https://www.freecodecamp.org/news/git-stash-commands/ -- it tells you all the basic stuff you need to know to start using `git stash` with confidence