# Git Cheatsheet (candidate items)

## Setup
- `git init` create repo
- `git clone <url>` clone repo
- `git clone --recursive <url>` include submodules
- `git config --global user.name "..."`
- `git config --global user.email "..."`

## Daily basics
- `git status` see state
- `git add <file>` stage
- `git add .` stage all in cwd
- `git restore <file>` unstage / revert working changes
- `git restore --staged <file>` unstage only
- `git commit -m "msg"` commit
- `git commit --amend` replace last commit (rewrites history!)
- `git diff` working-tree vs staged
- `git diff --staged` staged vs HEAD
- `git diff <a> <b>` between two commits
- `git log` history
- `git log --oneline --graph --all` compact view
- `git log -p <file>` per-file changes
- `git show <commit>` commit details

## Branches
- `git branch` list
- `git branch <name>` create
- `git switch <name>` switch (newer)
- `git switch -c <name>` create and switch
- `git checkout <name>` switch (older)
- `git branch -d <name>` delete (safe)
- `git branch -D <name>` force delete

## Merging
- `git merge <branch>` merge branch into current
- `git merge --squash <branch>` combine prereqs into one commit (then `git commit`)
- `git merge --abort` abort during conflict

## Rebasing
- `git rebase <base>` replay current branch onto base
- `git rebase -i HEAD~5` interactive (last 5 commits)
- Interactive verbs: `pick reword edit squash fixup drop`
- Rewrites history — never on shared commits!

## Remote
- `git remote -v` list remotes
- `git remote add origin <url>`
- `git fetch` download remote commits, don't merge
- `git pull` fetch + merge
- `git pull --rebase` fetch + rebase (linear)
- `git push` upload
- `git push -u origin <branch>` set upstream
- `git push --force` ⚠️ rewrites remote history

## Inspecting
- `git log --grep "msg"` search commit messages
- `git log -S "string"` find commits adding/removing string
- `git log <branch>..<branch>` commits in one but not other
- `git blame <file>` who last touched each line
- `git blame -L 42,50 <file>` lines 42-50
- `git blame -w` ignore whitespace

## Bisect
- `git bisect start`
- `git bisect bad`
- `git bisect good <commit>`
- `git bisect run <test_script>` auto
- `git bisect reset` — ALWAYS DO when done

## Cherry-pick
- `git cherry-pick <sha>` copy commit onto current branch

## Revert
- `git revert <sha>` create NEW commit that undoes the given commit
- Safe for shared history (additive, no rewrite)

## Stash
- `git stash` save uncommitted changes
- `git stash pop` reapply + remove from stack
- `git stash list`
- `git stash apply` reapply, keep in stack

## Reset (⚠️ destructive)
- `git reset --soft <sha>` move HEAD, keep index + working tree
- `git reset --mixed <sha>` (default) keep working tree, reset index
- `git reset --hard <sha>` reset everything — destroys uncommitted work

## Reflog (safety net)
- `git reflog` shows every HEAD move
- Recover lost commits: `git checkout <reflog-sha>` or `git reset --hard <reflog-sha>`

## Detached HEAD
- HEAD points at a commit (not a branch)
- Caused by `git checkout <sha>`, tags, mid-bisect
- Commits made in detached HEAD aren't on any branch — switch back with `git switch <branch>`

## Relative commit addresses
- `HEAD~1` = parent
- `HEAD~2` = grandparent (via first parent)
- `HEAD^` = first parent (same as ~1)
- `HEAD^2` = SECOND parent (only valid on merge commits)
- `BRANCH~n` works the same

## Submodules
- `git submodule add <url> [path]` add new submodule
- `git submodule update --init --recursive` initialize on fresh clone
- `git submodule update` sync to pinned commit
- `git submodule update --remote` advance to latest

## Tags
- `git tag -a v1.0.0 -m "Release"`
- `git push --tags`

## .gitignore
```
*.o
build/
.env
node_modules/
```

## Conflict markers
```
<<<<<<< HEAD
your code
=======
their code
>>>>>>> branch
```
Edit the file, remove markers, `git add`, then `git commit`.

## Golden rule
**Never rewrite history (rebase, amend, reset, push --force) on commits that have been pushed to a shared branch.** Use `revert` instead.

## Quick decision table
| Scenario | Command |
|---|---|
| Just made a bad commit (local) | `git reset --soft HEAD~1` |
| Already pushed bad commit | `git revert <sha>` |
| Fix commit message | `git commit --amend` |
| Move work to a new branch | `git switch -c <name>` |
| Save work to switch tasks | `git stash` |
| Find who broke a line | `git blame -L` |
| Find which commit broke it | `git bisect` |
| Copy one commit elsewhere | `git cherry-pick` |
| Linearize history | `git pull --rebase` |
| Combine feature commits | `git merge --squash` |

## Quick concepts
- Fast-forward merge = no merge commit possible
- Non-FF merge = merge commit with 2 parents
- Squash merge = single new commit, NO merge link
- HEAD = pointer to current commit/branch
- Staging area / index = what goes in next commit
- Bus factor = how many people can leave before project stalls

## Additional items (potentially missing)

### git log filtering
- `git log --author="Alice"`
- `git log --since="2 weeks ago"`
- `git log --until="2024-12-31"`
- `git log <file>` — only commits touching file
- `git log -p <file>` — with diffs
- `git log <branch1>..<branch2>` — in branch2 not branch1
- `git log --merges` / `--no-merges`
- `git log --first-parent` — main line only (ignore merged branches)
- `git log -n 5` — last 5 commits

### git diff variants
- `git diff` — working tree vs index
- `git diff --staged` (or `--cached`) — index vs HEAD
- `git diff HEAD` — working tree vs HEAD
- `git diff <c1> <c2>` — between two commits
- `git diff <c1>..<c2>` — same
- `git diff <c1>...<c2>` — from common ancestor
- `git diff --stat` — summary
- `git diff --name-only` — just filenames

### git show
- `git show <commit>` — message + diff
- `git show <commit>:<file>` — file content AT that commit
- `git show <commit> --stat`

### Working tree cleanup
- `git clean -n` — dry run, see what would be deleted
- `git clean -fd` — actually remove untracked files + dirs ⚠️
- `git clean -fdx` — also remove ignored files

### Config aliases (nice quality of life)
```
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --all"
```

### Tags (versioning)
- `git tag` — list
- `git tag v1.0` — lightweight tag
- `git tag -a v1.0 -m "Release"` — annotated tag
- `git push --tags` — push tags
- `git show v1.0` — show tag

### Remote URLs
- `git remote -v` — list with URLs
- `git remote add upstream <url>`
- `git remote remove <name>`
- `git remote set-url origin <new-url>`

### Branch from a specific commit
- `git switch -c new-branch <commit>`
- `git checkout -b new-branch <commit>` (older syntax)

### Apply patches
- `git format-patch HEAD~3` — create patch files for last 3 commits
- `git am < patch.patch` — apply mailbox-format patch
- `git apply patch.diff` — apply a unified diff

### Cherry-pick range / multiple
- `git cherry-pick A B C` — pick three commits
- `git cherry-pick A..B` — pick A's child through B
- `git cherry-pick -x <sha>` — record original SHA in message

### Worktrees (advanced)
- `git worktree add ../proj-feature feature` — check out branch in second working dir
- Lets you have multiple branches open simultaneously

### git push tricks
- `git push origin --delete <branch>` — delete remote branch
- `git push origin :<branch>` — old syntax for delete
- `git push --force-with-lease` — safer than `--force` (won't overwrite teammate's work)

### Commit message conventions (Conventional Commits)
```
feat: add user signup endpoint
fix: handle null in auth middleware
docs: update README
refactor: extract validation logic
test: add edge cases for divider
chore: bump deps
```

### .gitattributes (line-ending, diff strategies)
```
*.txt text eol=lf
*.png binary
```

### Git hooks (briefly)
- `.git/hooks/` directory
- `pre-commit`, `commit-msg`, `pre-push`, etc.
- Run automatically; can reject commits/pushes

### Multiple remotes pattern
- `origin` — your fork
- `upstream` — original repo
- Fetch from upstream: `git fetch upstream && git rebase upstream/main`
