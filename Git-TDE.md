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
