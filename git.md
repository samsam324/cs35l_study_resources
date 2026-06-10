Want to practice? Try the Interactive Git Tutorial and the Advanced Git Tutorial — hands-on exercises in a real Linux system right in the browser!

In modern software construction, version control is not just a convenience — it is a foundational practice that solves several major challenges of managing code: collaboration, change tracking, traceability, safe rollback, and parallel development. Git is by far the most common tool for version control.

By the end of this chapter, you’ll be able to:

Explain in your own words what a commit, branch, HEAD, and the commit DAG are — and why Git treats commits as immutable.
Go through the everyday local workflow fluently: stage, commit, inspect, branch, switch, and merge.
Collaborate through a remote: push, fetch, pull, resolve a merge conflict, and open a pull request.
Diagnose and recover from the common failure modes — merge conflicts, detached HEAD, “lost” commits, accidental commits on the wrong branch.
Decide between merge, rebase, cherry-pick, revert, and reset for a given situation.
Recognise at a glance which commands rewrite history and which are additive — and why that distinction matters on shared branches.
Assumed background: comfort with a Unix shell (running commands, cd, ls, chaining with &&); the idea that a hash is a fixed-length fingerprint of content; familiarity with text editors. No prior Git experience is required — every command you meet here is introduced with a before/after graph before you’re expected to use it.

How to read this chapter. On a first pass, read it linearly — the sections build on each other. After that, use the Choosing the Right Tool table at the end as your lookup index. At the end of each major section you’ll find short retrieval prompts with collapsible answers — pause and try to answer them before revealing. They feel slow on purpose; that’s the effort that makes the material stick.

This page is organized by workflow phase — the same sequence you move through on a real project:

Core Concepts — the mental model everything else builds on.
Setup — create or clone a repository and configure it.
Author — write code, craft commits, manage your working tree.
Share — branch, merge, push, pull, collaborate via pull requests and tags.
Maintain — polish history, organize the team’s branching strategy, manage submodules.
Debug — investigate when things go wrong, and recover safely.
A final section — Choosing the Right Tool — is the decision table to come back to when you know what you want to do but can’t remember which command does it.

Throughout the page you will find interactive command cards — click the button to animate the graph transformation a command performs, and click again to undo. This is the fastest way to build an intuition for what each Git command actually does to your commit graph.

Core Concepts
Before the commands, the mental model. Each section below opens with the question it answers — if you think you already know the answer, try to articulate it in your own words before reading on. That tiny act of retrieval is more valuable than a careful re-read.

What is Version Control?
Why do we need version control?
Imagine four teammates editing the same 500-line program. You finish a function and email your copy around. Alice has already changed three of the files you touched; Bob is working on a fourth that you haven’t seen; Carol fixed a bug last week that somehow didn’t make it into your copy. When it’s time to combine the work, whose version wins? Which edits are new? If the merged result crashes, how do you tell which change broke it?

Manual version control — saving files with names like homework_final_v2_really_final.txt — collapses under this kind of pressure within hours. A Version Control System (VCS) is a tool that automates the job. It records every change with who/when/why metadata, lets many people work concurrently without clobbering each other, and makes it possible to undo a change that turned out to be wrong — days, weeks, or years later.

The five concrete problems a VCS solves:

Collaboration — multiple developers can work concurrently without overwriting each other’s changes.
Change tracking — see exactly what has changed since you last worked on a file.
Traceability — every modification records who made it, when, and why.
Reversion — if a bug is introduced, return to a known-good state.
Parallel development — branches let you work on features or fixes in isolation.
The most common version control systems:

Git (most common for open source, also used by Microsoft, Apple, and most other companies)
Mercurial (used by Meta, Jane Street, and others (Goode and Rain 2014))
Piper (Google’s internal tool (Potvin and Levenberg 2016))
Subversion (some older projects)
Centralized vs. Distributed
Why is Git “distributed”?
Because requiring a network connection for every Git operation is a terrible user experience — and older centralised systems like Subversion suffered from exactly that. Want to see what changed last week? Talk to the server. Want to commit? Talk to the server. Server is down? You can’t work.

A distributed VCS inverts this: every developer’s machine holds a full copy of the entire history. Commit, branch, and inspect history offline on a train; sync with teammates when you have a network. The three concrete wins:

Speed. Local operations touch a local disk, no round-trip. git log on a 20-year-old repo is instant.
Resilience. Every clone is a complete backup. The central server can die and the project survives.
Flexibility. You can experiment on branches locally without permissions or policies getting in the way.
The trade-off is that “the truth” has to be reconciled when people sync — which is what most of the “merge” machinery in this chapter is about.

Core Concepts table
Feature	Centralized (e.g., Subversion, Piper)	Distributed (e.g., Git, Mercurial)
Data Storage	Single central repository	Every developer has a full copy of history
Offline Work	Needs server connection to commit	Work and commit fully offline
Best For	Small teams with strict central control	Large teams, open-source, distributed workflows
Commits
What is a commit, and why do we need them?
A commit is a named snapshot of your entire project at one moment, with a short message explaining why you took that snapshot. It’s the fundamental unit Git reasons about: every branch, merge, rebase, and undo operation is expressed in terms of commits.

Why not just auto-save continuously?
Three reasons we commit in discrete, meaningful units instead of letting the OS or editor save every keystroke:

Meaningful units. “Yesterday at 3:47 PM” is a useless coordinate when hunting a bug. “The commit where we added rate limiting” is something you can find, read, revert, or cherry-pick. Commits let you slice history into intention-sized pieces.
Explanatory metadata. Each commit records who made it, when, and — crucially — why, through its message. The diff shows what changed; the message tells future-you or your teammate the reasoning. A trail of good messages is project memory.
Shared vocabulary. Because every commit has a unique identity (a SHA — we’ll meet hashes later), you and a teammate on another continent can refer to the exact same state of the project with a single string. “The bug reproduces on a3f2d9c but not on b7e1c4d.” Commits are the atoms that reviews, releases, and deployments are built out of.
🔧 Under the Hood: what a commit actually is (content addressing, snapshots vs. diffs) (optional — skip on first pass)
The Three States
Why do we need a staging area?
You might reasonably expect a simpler design: you edit files, you commit, done. Two states — working directory and history. Why does Git insert a middle layer?

The answer is that what you edited and what you want in the next commit are not always the same thing. Common situations:

You’ve edited five files in one session — two for a feature, three for an unrelated cleanup. You want two commits, not one messy one. The staging area lets you add the feature files, commit, then add the cleanup files and commit separately.
You’ve edited a file that mixes a real change with a debug print you forgot to remove. You want to commit the real change without the print. Staging individual hunks of a file (git add -p) lets you take half of a file now and leave the other half for later.
You want to review what you’re about to commit before committing. git diff --staged shows you exactly that — the staging area is the preview.
So Git operates across three areas that every file passes through:

Working directory — files as they exist on your disk right now.
Staging area (a.k.a. the index) — a preview of the next commit. Think of it as a commit editor: you can add files here, remove them, tweak which version goes in, and only commit when it reads the way you want.
Local repository — the permanent history, where committed snapshots live forever.
git add moves changes from the working directory into the staging area. git commit turns everything in staging into a new, immutable snapshot in the repository. git status tells you what’s currently in each area.

HEAD, Branches, and the Commit Graph
What are branches, and why do we need them?
A branch is a named line of history you can work on in parallel with other lines. In practice: one branch per feature, bug fix, or experiment.

Why bother? Because real projects always have multiple streams of work happening at once. Without branches, you’d have exactly two bad options:

Queue everything. Alice’s feature blocks Bob’s bug fix blocks Carol’s refactor. Nobody ships until everything is ready.
Mix everything on one timeline. Half-finished features, debug prints, and WIP experiments all live together on main. Every commit is a gamble about what’s actually production-ready.
Branches solve this by letting each stream of work live on its own timeline. When a feature is done, you combine it back (“merge”) into main. An experiment that doesn’t pan out can be discarded without polluting the shared history. And critically, all the branches are the same project — the same files, the same history up to the point they diverged — so switching between them is instant.

How do branches, HEAD, and the commit graph fit together?
Conceptually: a branch is a pointer to a commit, plus the chain of parent commits you can reach by walking backwards. HEAD is a pointer to “where you are right now” — usually at a branch, so that new commits extend that branch. All the Git graphs on this page are visualisations of branches as pointers into a Directed Acyclic Graph (DAG) of commits — each commit records one or more parent commit SHAs (zero for the root, one for a normal commit, two for a merge commit), and following the parent links walks you backwards through history.

🔧 Under the Hood: what branches, HEAD, and the `.git/` directory look like on disk (optional — skip on first pass)
The One Big Idea: Additive or Rewrite
Git stores your project as an append-only history of snapshots. Branches and HEAD are just pointers into that history.

Once you hold that picture, every Git command fits in one of two buckets:

Every Git command either (a) creates new snapshots and moves a pointer to them, or (b) only moves pointers. It never edits an existing snapshot in place.

The (a) bucket is additive — safe on shared branches, because nothing anyone already has changes. The (b) bucket is more interesting: moving pointers backward (e.g. git reset --hard) effectively discards work, and some commands in bucket (a) create new snapshots that replace older ones (e.g. git commit --amend, git rebase). Collectively these are the commands that rewrite history — safe locally, dangerous after you’ve pushed. Throughout this page every such command carries an ⚠️ rewrites history callout at first mention.

Why Git can work this way — the content-addressed hash machinery that makes snapshots cheap and tamper-evident — is covered in the optional 🔧 Under the Hood callouts scattered throughout this page. For now, the pointer-and-snapshot picture is enough.

Quick Check — Core Concepts. Before moving on, try these without looking back:

In your own words: what’s the difference between a branch and HEAD? Where does each point?
You run git branch feature and then make a commit. On which branch does the new commit land, and why?
Which of these are additive (safe on shared branches) and which rewrite history? git commit, git merge, git reset --hard, git commit --amend, git revert.
Why does Git keep commits instead of editing them in place when you change something?
Click to view answers
Setting Up a Repository
Before you can commit anything, you need a repository and an identity. This is a one-time setup per project or machine — fast once, rarely revisited.

Creating a New Repository (git init)
git init turns an existing directory into a Git repository by creating a hidden .git/ folder. Everything Git tracks lives inside .git/: objects, refs, branches, config. Delete .git/ and you have an ordinary folder again.

git init myproject
cd myproject
The command is instantaneous because it only creates directory scaffolding — no network, no files copied. You now have an empty repository with one branch (main by default, since Git 2.28 if configured, or master on older setups) and no commits.

Cloning an Existing Repository (git clone)
If the project already exists elsewhere (GitHub, GitLab, a teammate’s server), use git clone instead of git init. It downloads the full repository — every commit, every branch, every tag — and creates a local copy with the remote already configured as origin:

git clone https://github.com/example/myproject.git
cd myproject
A cloned repo is fully functional offline — because Git is distributed, every local clone contains the entire history.

Configuring Your Identity
Every commit records who made it. Before your first commit, tell Git who you are:

git config --global user.name "Your Name"
git config --global user.email "you@example.com"
These settings live in ~/.gitconfig and apply to every repo on your machine. Override per-repo with git config user.name "..." (omit --global) when you need a different identity for one project — common when mixing work and personal accounts.

Ignoring Files (.gitignore)
Why do we need .gitignore?
Not every file in your project directory is source code that belongs in version control. Your working tree also accumulates files that are generated from the source, personal to your machine, or downright dangerous to commit:

Build artefacts — compiled binaries, *.pyc bytecode, node_modules/, dist/, target/. These are reproducible from the source and re-generated on every build. Committing them wastes repo space, creates merge conflicts on every build, and pollutes diffs.
Editor / OS debris — .DS_Store, Thumbs.db, .idea/, .vscode/settings.json (sometimes). These reflect your machine’s setup, not the project.
Local config and secrets — .env, *.pem, database passwords, API keys. These must never enter history (see the security warning below).
Huge binary files — videos, datasets, model checkpoints. Git is optimized for text; large opaque binaries bloat the repo and can’t be diffed meaningfully. Use Git LFS for those.
Without a .gitignore, Git constantly reports these files as “untracked” in git status, and eventually someone stages git add -A and commits the wrong thing. The file tells Git to pretend these paths don’t exist — they won’t show up in git status, won’t be staged by accident, and won’t be tracked.

What goes in a .gitignore, and why?
A typical Python project’s .gitignore, annotated:

# Compiled Python — regenerated from .py sources, never need to share
*.pyc
__pycache__/

# Virtual environments — machine-local, contains thousands of installed packages
venv/
.venv/

# Secrets — never commit (rotate immediately if you do)
.env
*.pem

# OS clutter — only relevant to macOS / Windows file browsers
.DS_Store
Thumbs.db

# Editor metadata — reflects your personal editor, not the project
.vscode/
.idea/
The shape generalizes: for each entry, ask “is this reproducible from source?” or “is this personal to my machine?” or “is this a secret?” If yes to any of those, it belongs in .gitignore. If it’s hand-authored content that’s part of the project, it does not.

A few defaults worth knowing for common ecosystems:

Setting Up a Repository table
Ecosystem	Typical ignores
Python	__pycache__/, *.pyc, .venv/, venv/, .pytest_cache/, *.egg-info/, dist/, build/
Node.js	node_modules/, dist/, build/, .next/, coverage/, *.log
Java / JVM	target/, build/, *.class, *.jar (unless vendored), .gradle/
C / C++	*.o, *.obj, build/, cmake-build-*/, *.exe
Rust	target/, Cargo.lock (only ignore for libraries, commit it for apps)
OS / editor	.DS_Store, Thumbs.db, .idea/, .vscode/
GitHub publishes a curated gitignore template collection — pick your language’s file and copy it as a starting point.

Pattern syntax
Pattern syntax table
Pattern	Matches
*.pyc	Any file with a .pyc extension in any directory
__pycache__/	Trailing / restricts the match to directories named __pycache__
.env	A specific filename at any depth
/build/	Leading / anchors to the repo root only (not nested build/ folders)
docs/*.html	A path-prefix glob
!important.log	Leading ! negates a prior match — “include this even though *.log would exclude it”
Why do I need to set .gitignore up before my first commit?
.gitignore has no retroactive effect on files that are already tracked. If you commit node_modules/ first and add node_modules/ to .gitignore second, the directory stays tracked — Git keeps following every change inside it. You have to explicitly untrack it:

git rm --cached node_modules -r
git commit -m "Stop tracking node_modules"
(The --cached flag removes the files from Git’s index only, not from your working directory.) Adding the pattern before the first commit avoids this step entirely — which is why every language guide tells you to create .gitignore first.

Why commit .gitignore itself?
Because the rules are a project-level concern, not a personal one. Sharing the file means every teammate and every future clone automatically gets the same ignore rules. Without this, each developer independently re-discovers which files to ignore — and someone eventually commits .env.

⚠️ .gitignore is not a security tool. If a secret was ever committed — even in a commit that was later removed — it remains in history and in the reflog, visible to anyone who clones the repository. The correct response to a leaked credential is to rotate it immediately and scrub history with tools like git filter-repo or BFG Repo Cleaner.

🔧 Under the Hood: other places ignore rules can live (optional — skip on first pass)
Quick Check — Setting Up. Try these before peeking:

When would you reach for git init versus git clone?
Your first commit on a new project has node_modules/ in it. You add node_modules/ to .gitignore and commit. Is it still tracked? Why?
Your teammate accidentally committed .env (containing an API key) last week and the commit is on main. Someone suggests “just add .env to .gitignore and we’re fine.” Why is that advice wrong, and what should happen instead?
Click to view answers
Making Commits
The canonical local workflow is the same every day:

Initialise the repo with git init (or clone it) — see Setting Up a Repository.
Edit files in your working directory.
Stage the exact changes you want in the next snapshot with git add <filename>.
Commit the snapshot with git commit -m "message".
Check state with git status at any time; review history with git log.
Git tracks files through the three trees you met in Core Concepts: the working directory (files on disk), the index/staging area (what your next commit will contain), and the repository (committed history). The strip above each graph below mirrors what git status prints — Untracked, Not staged, and Staged. git add moves files into Staged; git commit turns Staged into the next node in the graph.

Typing a new file doesn't involve Git yet — it lives only in your working directory. git add copies the current contents into the staging area (the index), marking them for the next commit. git commit then turns whatever is staged into a new, immutable node in the graph.

Click through to see each step: a fresh login.js appears untracked, moves to staged with git add, then folds into commit C when you commit.

Step 0 of 2 (initial state).

←
Back

▶
git add login.js
Untracked:
login.js
Not staged:
modified:
README.md
Staged:
main
HEAD
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. working tree has 1 unstaged file, 1 untracked file. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit. Commits in newest-first order: 1. Commit B: Initial commit. Labels: HEAD -> main. Parents: A: Repository init. Children: none. 2. Commit A: Repository init. Parents: none. Children: B: Initial commit. Changes not staged for commit: modified README.md. Untracked files: login.js.Git graph updated: HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. working tree has 1 unstaged file, 1 untracked file.
Inspecting Before You Commit
Before turning staged changes into a permanent snapshot, look at them. git diff compares different versions of your code:

git diff — working directory vs. staging area.
git diff --staged (or --cached) — staging area vs. the latest commit. Useful to review exactly what you are about to commit.
git diff HEAD — working directory vs. the latest commit.
git diff HEAD^ HEAD — parent vs. latest commit (shows what the latest commit changed).
git diff main..feature — file-level differences between the tips of main and feature (the .. is treated as a separator; equivalent to git diff main feature). To list the commits unique to feature, use git log main..feature instead.
git status is the dashboard; git diff --staged is the review step. Run both before every commit — it’s the single best habit for keeping commits clean.

Staging Shortcuts: git add -A vs. git commit -am
Typing git add <file> for every modified file gets tedious. Two shortcuts stage multiple files at once, but they differ in one critical way: whether they touch untracked files.

git add -A (or --all) stages every change in the working tree — modifications to tracked files, deletions, and brand-new untracked files. Watch notes.txt (untracked) and src/utils.js (modified) both slide into Staged.

This is the "just stage everything" command, and the reason some teams discourage it: it's easy to accidentally stage generated files, logs, or secrets you didn't mean to commit.

The real undo for this is git restore --staged . — it unstages every file, moving rows back from Staged to their previous zones.


▶
git add -A
Untracked:
notes.txt
Not staged:
modified:
src/utils.js
Staged:
main
HEAD
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. working tree has 1 unstaged file, 1 untracked file. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit. Commits in newest-first order: 1. Commit B: Initial commit. Labels: HEAD -> main. Parents: A: Repository init. Children: none. 2. Commit A: Repository init. Parents: none. Children: B: Initial commit. Changes not staged for commit: modified src/utils.js. Untracked files: notes.txt.Git graph updated: HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. working tree has 1 unstaged file, 1 untracked file.
git commit -a (or -am to add a message in one go) auto-stages only tracked files that have been modified or deleted, then commits in a single step — untracked files are ignored entirely.

Watch the difference vs. git add -A above: src/utils.js (tracked, modified) flies into the new commit C, but notes.txt (untracked) stays put in the Untracked zone. This is -am's safety feature — you won't accidentally commit files Git has never seen before.

The real undo is git reset --soft HEAD~1 — it removes commit C while keeping the changes staged (so you can edit and re-commit).


▶
git commit -am "Update utils"
Untracked:
notes.txt
Not staged:
modified:
src/utils.js
Staged:
main
HEAD
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. working tree has 1 unstaged file, 1 untracked file. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit. Commits in newest-first order: 1. Commit B: Initial commit. Labels: HEAD -> main. Parents: A: Repository init. Children: none. 2. Commit A: Repository init. Parents: none. Children: B: Initial commit. Changes not staged for commit: modified src/utils.js. Untracked files: notes.txt.Git graph updated: HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. working tree has 1 unstaged file, 1 untracked file.
Rule of thumb: git add -A stages everything new (dangerous); git commit -am is a safe shortcut for tracked-only commits. When in doubt, run git status first to see what each will affect.

Writing Good Commit Messages
A commit message is a note to your future self and your teammates. Professional projects follow a small set of conventions that compound across thousands of commits.

The 50/72 rule:

Subject line: ≤50 characters. A short imperative summary, no trailing period.
Blank line.
Body: wrap at 72 characters. Explain the why, not just the what — the diff already shows what.
Imperative mood. Write the subject as a command describing what the commit does, not a past-tense description of what you did:

Making Commits table
✅ Imperative	❌ Past tense / gerund
Add login endpoint	Added login endpoint
Fix off-by-one in pagination	Fixing off-by-one in pagination
Refactor user-service for clarity	Refactored user service
Mnemonic: a good subject line completes the sentence “If applied, this commit will __“. “Add login endpoint” — yes. “Added login endpoint” — grammatically awkward.

Conventional Commits (optional, team-level). Many teams adopt the Conventional Commits convention — a structured prefix that enables automated changelog generation and semantic-version bumping:

<type>(<optional scope>): <subject>

<optional body>

<optional footer(s)>
Common types: feat (new feature), fix (bug fix), docs, refactor, test, chore, ci, build. Example:

feat(auth): add rate limiting to login endpoint

Requests from a single IP are capped at 5 per minute.
Exceeding the limit returns HTTP 429 with a Retry-After
header. Protects against credential-stuffing attacks.

Closes #342
Whether to adopt Conventional Commits is a team decision — but writing imperative, ≤50-character subjects is universal.

Fixing Your Last Commit (git commit --amend)
⚠️ This command rewrites history. Safe for commits you have not yet pushed. Never amend a commit that has been pushed to a shared branch — see the Golden Rule of Shared History.

Why do we need --amend?
Because the most common “oops” in Git is noticing a typo in the commit message, or realizing you forgot to git add a file, seconds after committing. Without --amend you’d have two bad options: leave the broken commit in history and create a follow-up (“fix typo in previous message”), or reset the branch and rebuild the commit manually. Neither is great. --amend gives you a dedicated “I meant this, not that” operation that replaces the tip commit with a corrected version.

What it does
git commit --amend combines the staging area with the current tip commit and rewrites it — new hash, same branch position.

Typical uses:

Fix the message: git commit --amend -m "Correct subject line".
Include a forgotten file: git add forgotten.py && git commit --amend --no-edit (keeps the original message).
⚠️ Rewrites history. Replaces the most recent commit with a new one — typically to fix a typo in the message or include a file you forgot to stage.

The commit is not actually edited in place (commits are immutable). Git creates a brand-new commit C′ with the amended content and moves the branch pointer to it; the original C becomes unreferenced and is eventually garbage-collected.

Safe for local work, but never amend a commit you've already pushed — collaborators' clones still reference the old hash and will see a diverged branch on next pull.


▶
git commit --amend
main
HEAD
C
Fix bug (typo)
B
Add feature
A
Initial commit
HEAD is on branch main at commit C: Fix bug (typo). 3 commits. 1 branch. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit C: Fix bug (typo). Branch tips: main at C: Fix bug (typo). Commits in newest-first order: 1. Commit C: Fix bug (typo). Labels: HEAD -> main. Parents: B: Add feature. Children: none. 2. Commit B: Add feature. Parents: A: Initial commit. Children: C: Fix bug (typo). 3. Commit A: Initial commit. Parents: none. Children: B: Add feature.Git graph updated: HEAD is on branch main at commit C: Fix bug (typo). 3 commits. 1 branch.
Amend is the simplest of Git’s rewrite operations — and therefore the gateway drug to the rest of Reshaping History.

Quick Check — Making Commits. Try these before peeking:

Name the three areas a file passes through on its way into history. Which Git command moves it between each?
You have src/utils.js (modified) and notes.txt (untracked). You run git commit -am "Update utils". What ends up in the new commit, and why?
You commit, then notice a typo in the message two seconds later. Which command fixes it, and why must you only use it on local commits?
Rewrite this commit subject in imperative mood: “Fixed the pagination off-by-one error that broke the dashboard”.
Click to view answers
Managing Uncommitted Changes
Your working tree is often in a state you don’t want to commit yet — half-finished edits, debug prints, generated files. Three commands manage this space.

Discarding Changes (git restore)
git restore <file> replaces the file in your working directory with its committed version, discarding any unsaved edits:

git restore src/app.py               # discard working-tree edits
git restore --staged src/app.py      # unstage, but keep the edits
git restore --source=HEAD~3 src/app.py  # restore from 3 commits ago
Without --staged, restore overwrites your working tree — uncommitted edits are lost with no undo.
With --staged, restore only touches the index (moves the file out of “staged”), leaving your working-tree edits intact.
git restore and its sibling git switch (for branch navigation) were introduced in Git 2.23 as cleaner replacements for the overloaded git checkout. git checkout still works, but the split is clearer — navigate branches with switch, discard file changes with restore.

Shelving Work in Progress (git stash)
git stash saves your uncommitted changes (staged and unstaged) to a private stack, then cleans the working tree — letting you switch contexts without making a messy commit:

git stash                   # save; working tree becomes clean
git switch hotfix           # do something urgent
# …commit and merge the hotfix…
git switch original-branch  # return
git stash pop               # restore and drop the stash
Flags worth knowing:

git stash -u also stashes untracked files (otherwise ignored — a common surprise).
git stash pop restores and drops the stash; git stash apply restores but keeps the stash in the stack (useful when you want to apply the same shelf to multiple branches).
git stash list shows the stack; entries are named stash@{0} (most recent), stash@{1}, etc.
git stash drop stash@{n} deletes an entry without applying it.
🔧 Under the Hood: how stash actually works (optional — skip on first pass)
You're mid-change on main, but need to jump to another branch for a quick fix. Committing half-finished work is ugly; git stash saves the state aside so you can come back to it with pop later.

Stash is not a separate storage area — it's regular commit objects (i for the index, w for the working tree) on a dangling branch refs/stash. Watch the graph: the new commits pop into a sibling lane during git stash and vanish during git stash pop. The stash shelf is just a friendlier view of the same data.

Step 0 of 2 (initial state).

←
Back

▶
git stash
Untracked:
Not staged:
modified:
checkout.js
Staged:
modified:
cart.js
main
HEAD
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. working tree has 1 staged file, 1 unstaged file. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit. Commits in newest-first order: 1. Commit B: Initial commit. Labels: HEAD -> main. Parents: A: Repository init. Children: none. 2. Commit A: Repository init. Parents: none. Children: B: Initial commit. Staged files: modified cart.js. Changes not staged for commit: modified checkout.js.Git graph updated: HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. working tree has 1 staged file, 1 unstaged file.
Cleaning Untracked Files (git clean)
git clean is git restore’s cousin for files Git doesn’t track. git restore can only touch files Git already knows about; git clean removes entire untracked files and directories:

git clean -n          # dry run — list what would be removed
git clean -f          # force — actually delete untracked files
git clean -fd         # also remove untracked directories
git clean -fdx        # also remove ignored files (!!!)
Like git restore without --staged, this is permanent — git clean -fd cannot be undone by Git. Always dry-run first. -fdx removes files that .gitignore excludes (build artefacts, node_modules/, caches) — useful for a full reset before diagnosing a build issue, but dangerous if .gitignore covers anything you don’t want to lose.

Quick Check — Managing Uncommitted Changes. Try these before peeking:

Three files are all uncommitted but in different states: a.js is staged, b.js is modified-but-unstaged, c.js is brand-new-and-untracked. You run git stash. What happens to each?
What’s the functional difference between git restore file.js and git restore --staged file.js?
You run git clean -fd in your project and realize too late that you had some untracked scratch notes in there. Can Git recover them? Why or why not?
Click to view answers
Branching
A branch is Git’s way of supporting parallel lines of development — you can experiment on a feature branch without touching main, and combine the work back only when it’s ready.

What a Branch Physically Is
Recall from Core Concepts: a branch is a 41-byte pointer file in .git/refs/heads/ containing one commit’s SHA. That’s it — no per-branch copy of your files, no hidden metadata. Creating a branch is one fwrite(); it costs milliseconds even on a 10 GB repo.

This lightweight pointer is why Git encourages branching liberally. If branches were expensive copies, you’d avoid creating them. Because they’re nearly free, best practice is to branch often — one branch per feature, bug fix, or experiment.

Creating, Switching, and Deleting Branches
git branch                   # list local branches (* marks current)
git branch feature           # create a branch at HEAD (do NOT switch)
git switch feature           # switch HEAD to an existing branch
git switch -c feature        # create AND switch in one step (most common)
git branch -d feature        # delete (refuses if unmerged; safe)
git branch -D feature        # force-delete (no safety check)
Creates a new branch pointer at the current commit and moves HEAD onto it in a single step — shorthand for git branch feature followed by git switch feature.

Both pointers start at the same commit, so no files change; future commits you make will land on feature rather than main.


▶
git switch -c feature
main
HEAD
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit. Commits in newest-first order: 1. Commit B: Initial commit. Labels: HEAD -> main. Parents: A: Repository init. Children: none. 2. Commit A: Repository init. Parents: none. Children: B: Initial commit.Git graph updated: HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch.
Repoints HEAD from one branch to another — pure pointer movement.

No commit objects are created or modified; Git just updates your working directory to match the target branch's tip.

Use this to navigate between parallel lines of development.


▶
git switch feature
main
feature
HEAD
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 2 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit; feature at B: Initial commit. Commits in newest-first order: 1. Commit B: Initial commit. Labels: HEAD -> main; feature. Parents: A: Repository init. Children: none. 2. Commit A: Repository init. Parents: none. Children: B: Initial commit.Git graph updated: HEAD is on branch main at commit B: Initial commit. 2 commits. 2 branchs.
Common Mistake: git branch Without Switching
Where a commit lands depends entirely on where HEAD is pointing when you run git commit. A very common beginner mistake is running git branch <name> and then immediately starting work — git branch creates the pointer but leaves HEAD on the current branch, so all new commits continue landing there. The two labs below show this side-by-side.

Common mistake: git branch without switching. git branch feature creates the pointer but does not move HEAD — commits keep landing on main.

Watch HEAD carefully: it never leaves main.

Step 0 of 2 (initial state).

←
Back

▶
git branch feature
main
HEAD
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit. Commits in newest-first order: 1. Commit B: Initial commit. Labels: HEAD -> main. Parents: A: Repository init. Children: none. 2. Commit A: Repository init. Parents: none. Children: B: Initial commit.Git graph updated: HEAD is on branch main at commit B: Initial commit. 2 commits. 1 branch.
Correct approach: switch first, then commit. git switch feature moves HEAD onto the branch before you start working. Commits then land exactly where you intended.

Step 0 of 2 (initial state).

←
Back

▶
git switch feature
main
feature
HEAD
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 2 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit; feature at B: Initial commit. Commits in newest-first order: 1. Commit B: Initial commit. Labels: HEAD -> main; feature. Parents: A: Repository init. Children: none. 2. Commit A: Repository init. Parents: none. Children: B: Initial commit.Git graph updated: HEAD is on branch main at commit B: Initial commit. 2 commits. 2 branchs.
Detached HEAD, the third common HEAD state, is covered under Undoing Committed Work — it’s most useful when investigating and recovering, not during normal branching.

Quick Check — Branching. Try these before peeking:

Your repo has 10 GB of code. How long does git branch feature take, and why?
You run git branch feature. Without moving from main, you stage and commit a new file. Sketch the graph (or describe it in one sentence). Where did the commit actually land?
What do git switch feature and git switch -c feature each do? When would you pick one over the other?
Click to view answers
Merging
Once work has happened in parallel on two branches, you eventually want to bring it back together. Git has three modes of git merge, each with a distinct graph shape.

Fast-Forward Merge
Fast-forward merge. Because main hasn't advanced since feature branched, there's nothing to reconcile.

Git simply slides main's pointer forward to feature's tip — no merge commit is created, history stays perfectly linear, and both pointers now reference the same commit.


▶
git merge feature
main
feature
HEAD
D
Add tests
C
Add login
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 4 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit; feature at D: Add tests. Commits in newest-first order: 1. Commit D: Add tests. Labels: feature. Parents: C: Add login. Children: none. 2. Commit C: Add login. Parents: B: Initial commit. Children: D: Add tests. 3. Commit B: Initial commit. Labels: HEAD -> main. Parents: A: Repository init. Children: C: Add login. 4. Commit A: Repository init. Parents: none. Children: B: Initial commit.Git graph updated: HEAD is on branch main at commit B: Initial commit. 4 commits. 2 branchs.
Three-Way Merge
Three-way merge. Both branches have commits the other doesn't: main added E; feature added C and D.

Git compares both tips against their common ancestor (B) and creates a new merge commit M with two parents — one per branch.

The resulting diamond shape in the graph is the hallmark of a three-way merge.

The strategy name shown in the output is ort (default since Git 2.34, Nov 2021; introduced as opt-in in Git 2.33, Aug 2021); older versions and Pro Git show recursive — the algorithm is equivalent for this case.


▶
git merge feature
main
feature
HEAD
E
Hotfix
D
Feature B
C
Feature A
B
Initial commit
A
Repository init
HEAD is on branch main at commit E: Hotfix. 5 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit E: Hotfix. Branch tips: main at E: Hotfix; feature at D: Feature B. Commits in newest-first order: 1. Commit E: Hotfix. Labels: HEAD -> main. Parents: B: Initial commit. Children: none. 2. Commit D: Feature B. Labels: feature. Parents: C: Feature A. Children: none. 3. Commit C: Feature A. Parents: B: Initial commit. Children: D: Feature B. 4. Commit B: Initial commit. Parents: A: Repository init. Children: E: Hotfix; C: Feature A. 5. Commit A: Repository init. Parents: none. Children: B: Initial commit.Git graph updated: HEAD is on branch main at commit E: Hotfix. 5 commits. 2 branchs.
Forcing a Merge Commit: --no-ff
Forces a merge commit even though a fast-forward would have worked. The result M has two parents: the previous main tip (B) and the feature tip (D).

The extra commit looks redundant but preserves a visible trace that a feature branch was integrated — handy when reviewing history to see which commits belonged to which feature.


▶
git merge --no-ff feature
main
feature
HEAD
D
Add tests
C
Add login
B
Initial commit
A
Repository init
HEAD is on branch main at commit B: Initial commit. 4 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Initial commit. Branch tips: main at B: Initial commit; feature at D: Add tests. Commits in newest-first order: 1. Commit D: Add tests. Labels: feature. Parents: C: Add login. Children: none. 2. Commit C: Add login. Parents: B: Initial commit. Children: D: Add tests. 3. Commit B: Initial commit. Labels: HEAD -> main. Parents: A: Repository init. Children: C: Add login. 4. Commit A: Repository init. Parents: none. Children: B: Initial commit.Git graph updated: HEAD is on branch main at commit B: Initial commit. 4 commits. 2 branchs.
Squash Merge
⚠️ This variant rewrites history in the sense that it produces one new commit whose parent is main’s previous tip — not feature’s tip. The feature branch’s individual commits are not recorded on main.

Collapses every commit on feature into a single new commit on main (C+D).

Two-step command. git merge --squash feature only stages the combined changes and writes a draft message into .git/SQUASH_MSG — it does not create a commit on its own. You then run git commit to materialise the squashed snapshot.

Crucially, the resulting commit has only one parent — the previous main tip (E) — not the feature branch's tip (D). That's because --squash produces a regular commit, not a merge commit: Git records the squashed work as if you had written those changes directly on main yourself, with no structural link back to feature.

The feature branch stays at D, unreferenced from main's history; commands like git log main won't show it as an ancestor.

Useful when you want main to read as a series of clean features, not every intermediate "fix typo" commit — but the trade-off is that you lose the ability to trace the original commits from main.


▶
git merge --squash feature && git commit -m "Squashed C+D"
main
feature
HEAD
E
Hotfix
D
Feature D
C
Feature C
B
Initial commit
A
Repository init
HEAD is on branch main at commit E: Hotfix. 5 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit E: Hotfix. Branch tips: main at E: Hotfix; feature at D: Feature D. Commits in newest-first order: 1. Commit E: Hotfix. Labels: HEAD -> main. Parents: B: Initial commit. Children: none. 2. Commit D: Feature D. Labels: feature. Parents: C: Feature C. Children: none. 3. Commit C: Feature C. Parents: B: Initial commit. Children: D: Feature D. 4. Commit B: Initial commit. Parents: A: Repository init. Children: E: Hotfix; C: Feature C. 5. Commit A: Repository init. Parents: none. Children: B: Initial commit.Git graph updated: HEAD is on branch main at commit E: Hotfix. 5 commits. 2 branchs.
Trade-off. Squash merge makes main’s log read as one commit per feature (clean), but you lose the intermediate commits — which hurts git bisect precision if a regression later narrows to “the whole squashed feature”. The internal commits still exist on the feature branch (if you don’t delete it) and in reflog.

Handling Merge Conflicts
When Git cannot automatically reconcile differences (usually because the same lines were changed in both branches), it marks the conflicting sections in the file with conflict markers:

<<<<<<< HEAD
your version of the code
=======
incoming branch version
>>>>>>> feature-branch
The full resolution sequence is: edit the conflicting file to remove all markers and keep the correct content, stage it with git add, then finalise with git commit. Use git merge --abort to cancel a merge in progress and return to the pre-merge state.

Your editor probably has a nicer UI for this. VS Code, JetBrains IDEs, and most other editors surface conflicts inline with “Accept Current” / “Accept Incoming” / “Accept Both” buttons above each conflict block — you click rather than hand-edit the markers. The underlying command sequence is identical (git add then git commit to finalise); the buttons are just a friendlier way to produce the same resolved file.

Resolving a merge conflict step by step. When git merge cannot automatically reconcile changes it pauses and leaves conflict markers in the affected files. The graph does not change until you complete or abort the merge.

Step 0 of 4 (initial state).

←
Back

▶
git merge feature
Untracked:
Not staged:
Staged:
main
feature
HEAD
E
Hotfix on main
D
Edit greeting on feature
B
Initial commit
A
Repository init
HEAD is on branch main at commit E: Hotfix on main. 4 commits. 2 branchs. working tree clean. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit E: Hotfix on main. Branch tips: main at E: Hotfix on main; feature at D: Edit greeting on feature. Commits in newest-first order: 1. Commit E: Hotfix on main. Labels: HEAD -> main. Parents: B: Initial commit. Children: none. 2. Commit D: Edit greeting on feature. Labels: feature. Parents: B: Initial commit. Children: none. 3. Commit B: Initial commit. Parents: A: Repository init. Children: E: Hotfix on main; D: Edit greeting on feature. 4. Commit A: Repository init. Parents: none. Children: B: Initial commit. Working tree: clean.Git graph updated: HEAD is on branch main at commit E: Hotfix on main. 4 commits. 2 branchs. working tree clean.
Merge Strategies (ort, -X ours, -X theirs)
Since Git 2.34 (November 2021), the default merge strategy is ort (Ostensibly Recursive’s Twin) — a reimplementation of the older recursive strategy that’s faster and handles renames better. (ort was introduced as opt-in in Git 2.33, August 2021, and promoted to the default in 2.34.) For typical two-branch merges the output is identical; you rarely need to pick a strategy explicitly.

When the default auto-resolution doesn’t do what you want, strategy options (-X) tune the behavior:

git merge feature -X ours              # on conflict, keep OUR version (current branch)
git merge feature -X theirs            # on conflict, keep THEIR version (incoming)
git merge feature -X ignore-all-space  # ignore whitespace differences
Important: -X ours/-X theirs only affect conflicting lines — non-conflicting changes from both branches are still combined normally. Don’t confuse them with the whole-branch strategies -s ours (discard the other branch’s changes entirely) or -s subtree — far rarer and more dangerous operations.

Use -X theirs when integrating generated or vendored files where the incoming version is authoritative. Use -X ours sparingly — it’s easy to silently lose incoming fixes.

Quick Check — Merging. Try these before peeking:

main is at commit B. feature branched from B and added commits C and D. main has not moved. You run git merge feature from main. What shape does history take — fast-forward or merge commit? Why?
Same setup, but now main has also added a commit E since feature branched. You run git merge feature. What’s the shape now? How many parents does the new commit have?
git merge --squash feature produces a commit whose parent is main’s previous tip — not feature’s tip. What does this mean for git log --graph after the squash? Can you still tell from main’s history that feature existed?
Mid-merge, you open a conflicted file and edit it. You run git status and the file is still marked unmerged. What command officially marks it resolved?
Click to view answers
Remotes
Git really shines once you’re sharing work with other people. This section opens with the two questions that trip up most newcomers.

What’s the difference between a local and a remote repository?
A local repository is the one on your laptop — the .git/ folder inside your project directory. It’s where your commits actually live while you work, and everything in this chapter up to now has only touched it.

A remote repository is another copy of the same project, living somewhere else — typically on GitHub, GitLab, or a self-hosted server. The remote is how your work becomes visible to anyone else: teammates, CI systems, deployment scripts, the open-source world.

Why have both? Three reasons:

Collaboration. Your teammates need access to your work. A single shared remote is the source of truth that everybody pushes to and pulls from.
Backup. Your laptop could die, be stolen, or get dropped in a lake. The remote is insurance — if your local repo vanishes, a fresh clone from the remote reconstructs it.
Distribution. In open-source projects, you don’t have permission to write directly to the main repository. You clone your own copy, push commits to your remote (a “fork”), and open a pull request asking the maintainers to pull your changes into theirs.
The local↔remote split is also why Git feels different from older, centralised systems like SVN. In SVN, you need a network to commit at all — the server is the repo. In Git, your local repo is fully featured: you commit, branch, and inspect history offline, then sync with a remote when you’re ready. Every Git command in this chapter up to now works without network access.

A remote — in the narrow Git sense — is a named URL pointing to another copy of the repository. origin is the conventional name for the primary remote (the one you cloned from). A single repo can have multiple remotes with different names (common in open-source: origin for your fork, upstream for the maintainer’s repo).

🔧 Under the Hood: what a server-side remote actually stores (optional — skip on first pass)
What’s the difference between git clone and git pull?
They sound similar and both “get code from a remote”, which causes endless confusion. They do fundamentally different jobs:

Remotes table
Question	git clone <url>	git pull
When you run it	Once per project, to get started	Repeatedly, to catch up with teammates’ commits
Needs an existing local repo?	No — you run it outside of any repo	Yes — you run it inside the repo
What it does	Creates a new local repo from a remote: downloads every commit, branch, and tag; checks out the default branch; configures origin to point at <url>	Downloads new commits from the remote (git fetch) and integrates them into your current branch (git merge or git rebase)
Directory it produces	Creates a new folder named after the repo	Doesn’t create anything — updates the existing working tree in place
How often you run it	Effectively once (per machine, per project)	Many times a day on an active team
The tidy way to think about it: clone is how a local repo is born; pull is how it stays current.

A worked example:

# Day 1 — you join a project. You have no copy of it yet.
git clone https://github.com/acme/myproject.git     # creates myproject/ and downloads everything
cd myproject

# Days 2..N — you work on the project. Each day, teammates push new commits.
git pull                                             # brings those new commits into your branch
# ...do your work...
git push                                             # ship your commits back
git pull                                             # tomorrow morning: catch up again
If you ever find yourself running git clone twice for the same project, you probably wanted git pull. If you ever find yourself running git pull and getting “not a git repository”, you probably wanted git clone.

The five remote commands
The five commands that define remote collaboration:

git clone <url> — creates a local copy of a remote repository (Setup).
git remote — lists configured remotes. git remote add origin <url> registers a remote named origin (the conventional primary remote name); git remote -v lists existing remotes with their URLs.
git fetch — downloads new commits and branches from a remote without modifying your working directory or current branch. Useful for reviewing before deciding how to integrate.
git pull — shorthand for git fetch followed by git merge. Fetches and immediately merges into your current branch.
git push — uploads your local commits to a remote. git push -u origin <branch> pushes and sets up upstream tracking, so future git push and git pull on this branch can omit the remote name.
The diagram below shows how each command moves data between the four areas Git works with:

git clone / git fetch
git checkout
git add
git commit
git commit -a
git merge
git pull
git push
WorkingTree
StagingArea
LocalRepo
RemoteRepo
Detailed description

UML sequence diagram with 4 participants (WorkingTree, StagingArea, LocalRepo, RemoteRepo). Messages: RemoteRepo asynchronously messages LocalRepo with "git clone / git fetch"; LocalRepo asynchronously messages WorkingTree with "git checkout"; WorkingTree asynchronously messages StagingArea with "git add"; StagingArea asynchronously messages LocalRepo with "git commit"; WorkingTree asynchronously messages LocalRepo with "git commit -a"; LocalRepo asynchronously messages WorkingTree with "git merge"; RemoteRepo asynchronously messages WorkingTree with "git pull"; LocalRepo asynchronously messages RemoteRepo with "git push".

Participants

WorkingTree
StagingArea
LocalRepo
RemoteRepo
Messages

1. RemoteRepo asynchronously messages LocalRepo with "git clone / git fetch"
2. LocalRepo asynchronously messages WorkingTree with "git checkout"
3. WorkingTree asynchronously messages StagingArea with "git add"
4. StagingArea asynchronously messages LocalRepo with "git commit"
5. WorkingTree asynchronously messages LocalRepo with "git commit -a"
6. LocalRepo asynchronously messages WorkingTree with "git merge"
7. RemoteRepo asynchronously messages WorkingTree with "git pull"
8. LocalRepo asynchronously messages RemoteRepo with "git push"
Remote-Tracking Branches: origin/main vs. main
This is one of Git’s most persistent sources of confusion. There are actually three different pointers for any shared branch:

Your local branch (main) — the tip of your own work.
Your remote-tracking branch (origin/main) — your snapshot of where the remote was the last time you communicated with it. A read-only local reference stored in .git/refs/remotes/origin/.
The actual remote branch — what GitHub/GitLab/your server shows right now. You can only see its current state by running git fetch (or git ls-remote).
These three can be out of sync in different ways:

After you commit locally: main is ahead of both origin/main and the actual remote. A git push synchronises them by uploading your commits.
After a teammate pushes: the actual remote is ahead of both origin/main and your main. A git fetch updates origin/main. A git pull does both fetch and merge, bringing your main in sync.
After both you and teammates pushed: you’ve diverged. Neither simple push nor simple pull works — you must integrate (merge or rebase) and then push. See Diverged Pull below.
Useful inspection commands that rely on this distinction:

git log origin/main                    # what's on the (last-fetched) remote
git log main..origin/main              # commits on remote not yet on local (incoming)
git log origin/main..main              # commits on local not yet on remote (unpushed)
git diff main origin/main              # content differences between the two
Rule of thumb: origin/main is a read-only local cache of the remote. You never commit to it; it only moves when you fetch, pull, or push. In the graphs below it appears with a dashed label and gray color to distinguish it from your local branch pointer.

Fetching vs. Pulling — Why You Have Two Commands
git fetch and git pull both “download” from the remote, but they differ in how invasive they are:

git fetch — downloads new commits and updates remote-tracking branches only. Your local branches and working tree are untouched. Safe to run any time.
git pull — shorthand for git fetch followed by git merge (or git rebase if configured). Downloads and integrates into your current branch.
Downloads new commits from the remote into the remote-tracking branch (origin/main) without touching your local branch or working directory.

After a fetch you can review what changed (git log main..origin/main for the new commits, git diff main...origin/main for the file changes the merge would introduce) before deciding to merge.


▶
git fetch
main
origin/main
HEAD
B
Latest commit
A
Initial commit
HEAD is on branch main at commit B: Latest commit. 2 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Latest commit. Branch tips: main at B: Latest commit; origin/main at B: Latest commit. Commits in newest-first order: 1. Commit B: Latest commit. Labels: HEAD -> main; origin/main. Parents: A: Initial commit. Children: none. 2. Commit A: Initial commit. Parents: none. Children: B: Latest commit.Git graph updated: HEAD is on branch main at commit B: Latest commit. 2 commits. 2 branchs.
Shorthand for git fetch + git merge — downloads the remote commits and immediately fast-forwards the local branch to match origin/main.

Both pointers land on the same commit. If the local branch had diverged, a merge commit would be created instead.


▶
git pull
main
origin/main
HEAD
B
Latest commit
A
Initial commit
HEAD is on branch main at commit B: Latest commit. 2 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Latest commit. Branch tips: main at B: Latest commit; origin/main at B: Latest commit. Commits in newest-first order: 1. Commit B: Latest commit. Labels: HEAD -> main; origin/main. Parents: A: Initial commit. Children: none. 2. Commit A: Initial commit. Parents: none. Children: B: Latest commit.Git graph updated: HEAD is on branch main at commit B: Latest commit. 2 commits. 2 branchs.
The case for running them separately — the fetch → inspect → merge pattern:

git fetch                          # update origin/main
git log main..origin/main          # what's new? any dangerous changes?
git diff main origin/main          # what content would come in?
git merge origin/main              # integrate only after you've inspected
This pattern is especially valuable for branches you share with many people, where you want to see what’s coming before you commit to integrating. Use plain pull for your own feature branch where you already know what’s incoming (your CI, your own work on another machine), or during trivial fast-forward syncs.

Diverged Pull: Merge vs. Rebase
The fast-forward case above is the lucky path — your local branch had no new commits of its own, so Git could simply slide main forward. The interesting case is when both you and the remote have moved on since your last sync. Suppose you committed B locally, and while you were working, a teammate pushed C to the remote. Now main and origin/main have diverged, both descending from the common ancestor A.

git pull handles this by creating a merge commit that ties the two tips together — preserving the full DAG but littering history with auto-generated “Merge remote-tracking branch ‘origin/main’” commits:

After git fetch, Git sees that main (at B) and origin/main (at C) have diverged from their common ancestor A. git pull's default strategy is merge: it creates a new merge commit M with two parents — your local B and the remote's C — and advances main to M.

History is preserved exactly (no hashes change), but the graph gains a diamond and an auto-generated message Merge remote-tracking branch 'origin/main' (or, on older Git, Merge branch 'main' of ). On a busy team branch, these pile up and clutter the log.


▶
git pull
main
origin/main
HEAD
C
Teammate's change
B
Your local work
A
Shared base
HEAD is on branch main at commit B: Your local work. 3 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Your local work. Branch tips: main at B: Your local work; origin/main at C: Teammate's change. Commits in newest-first order: 1. Commit C: Teammate's change. Labels: origin/main. Parents: A: Shared base. Children: none. 2. Commit B: Your local work. Labels: HEAD -> main. Parents: A: Shared base. Children: none. 3. Commit A: Shared base. Parents: none. Children: C: Teammate's change; B: Your local work.Git graph updated: HEAD is on branch main at commit B: Your local work. 3 commits. 2 branchs.
git pull --rebase is the antidote. Instead of merging, it replays your local commits on top of the fetched remote tip, producing a linear history with no merge commit. Your local B becomes B′ with a new hash, parented on the remote’s C instead of the shared ancestor A:

Same diverged situation, different integration strategy. git pull --rebase fetches origin/main (bringing in C), then rebases your local commits onto it — each of your commits is replayed as a brand-new commit with a new hash.

Here your single local commit B is replayed on top of C, becoming B′. The old B is discarded. History reads as a straight line (A → C → B′), and no merge commit appears.

Rule of thumb: prefer git pull --rebase on your own feature branch to keep the log clean; stick with the default git pull (merge) on shared long-lived branches where rewriting any history — even your own — is risky.


▶
git pull --rebase
main
origin/main
HEAD
C
Teammate's change
B
Your local work
A
Shared base
HEAD is on branch main at commit B: Your local work. 3 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Your local work. Branch tips: main at B: Your local work; origin/main at C: Teammate's change. Commits in newest-first order: 1. Commit C: Teammate's change. Labels: origin/main. Parents: A: Shared base. Children: none. 2. Commit B: Your local work. Labels: HEAD -> main. Parents: A: Shared base. Children: none. 3. Commit A: Shared base. Parents: none. Children: C: Teammate's change; B: Your local work.Git graph updated: HEAD is on branch main at commit B: Your local work. 3 commits. 2 branchs.
You can make --rebase the default for a branch (git config branch.main.rebase true) or globally (git config --global pull.rebase true) so you don’t have to type the flag every time.

Pushing
git push is the mirror image of git fetch: it uploads your local commits to the remote and then advances the remote-tracking branch origin/main to match. The commits themselves do not change (no new hashes) — only the gray dashed label slides forward to catch up with your local main:

You've made two local commits (C and D) on top of the last shared state (B). git push sends them to the remote and, once the remote accepts them, Git advances origin/main to match your local main.

Notice what doesn't happen: no new commit is created, and no existing commit gets a new hash. The only visible change is that the dashed origin/main label hops from B up to D — a pointer move, nothing more. This is why push is a structural no-op on the graph: it only updates where the remote-tracking label sits.

git push fails if the remote has commits you don't have locally (someone else pushed since your last fetch). Pull first to reconcile, then push.


▶
git push
main
origin/main
HEAD
D
Add tests
C
Add feature
B
Fix bug
A
Initial commit
HEAD is on branch main at commit D: Add tests. 4 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit D: Add tests. Branch tips: main at D: Add tests; origin/main at B: Fix bug. Commits in newest-first order: 1. Commit D: Add tests. Labels: HEAD -> main. Parents: C: Add feature. Children: none. 2. Commit C: Add feature. Parents: B: Fix bug. Children: D: Add tests. 3. Commit B: Fix bug. Labels: origin/main. Parents: A: Initial commit. Children: C: Add feature. 4. Commit A: Initial commit. Parents: none. Children: B: Fix bug.Git graph updated: HEAD is on branch main at commit D: Add tests. 4 commits. 2 branchs.
The Force-Push Warning
git push -f (force-push) overwrites remote history to match your local copy. On a shared branch this permanently deletes commits your collaborators have already pushed. Never force-push to main or any shared integration branch. If you’ve rebased or amended commits that are already remote, push to a new branch instead — or use --force-with-lease, which at least refuses to overwrite if the remote has moved since your last fetch.

Pull Requests and Code Review
On every real-world team, code doesn’t go straight from your laptop to main. It goes through a pull request (PR, on GitHub or Bitbucket) or merge request (MR, on GitLab) — a proposal asking teammates to review the change before it lands.

The daily loop:

Branch. git switch -c feat-login — one branch per feature or bug fix.
Commit. Make your changes as a series of focused commits.
Push. git push -u origin feat-login — uploads your branch and sets upstream tracking.
Open a PR. On the hosting platform, request that feat-login be merged into main. Write a description explaining what changed and why. Link related issues.
Review. Teammates read the diff, leave inline comments, request changes or approve.
Iterate. Commit fixes locally, push again — the PR updates automatically.
Merge. After approval (and green CI), someone clicks “Merge” on the platform. Most platforms offer three merge strategies — regular merge, squash-and-merge, or rebase-and-merge — as a team-wide setting or per-PR choice.
Clean up. Delete the feature branch locally and on the remote.
Why teams use PRs:

Isolation. Broken work never touches main; CI runs on the PR branch.
Review. Every change is read by at least one other human before it ships.
Audit trail. The PR is a durable record of the design discussion and approvals — valuable long after the commits themselves.
CI gate. The platform can block merging until tests pass and reviewers approve.
Forks vs. direct branches. In internal team repositories, everyone pushes branches directly to the same origin and opens PRs there. In open-source projects (and some strict security contexts), you don’t have push access to the main repo — you fork it into your own account, push branches to your fork, and open a PR from yourfork:branch → upstream:main. The mechanics are the same; only the where you pushed the branch differs.

Quick Check — Remotes. Try these before peeking:

There are three pointers that all sit on what feels like “the main branch”: main, origin/main, and the actual branch on the remote server. Which one moves when you run each of these? git commit, git fetch, git push.
What’s the practical difference between git fetch and git pull — and why have two commands?
You and a teammate both pushed to main since your last pull. A plain git pull succeeds but adds a Merge remote-tracking branch 'origin/main' commit. What would git pull --rebase have done instead, and why might you prefer it on a feature branch?
Why is git push -f to main considered dangerous even if you’ve only “cleaned up” your own commits?
Click to view answers
Tagging Releases
A tag is a permanent, human-meaningful name for a specific commit — typically used to mark a release (v1.0.0, v2.3.1-beta, release-2024-01-15). Unlike branches, tags don’t move. Once v1.0.0 is created, it points to that commit forever.

Lightweight vs. Annotated Tags
Git has two kinds of tags:

Lightweight tag — just a pointer to a commit, like a branch that never moves. Created with git tag <name>.
Annotated tag — a full Git object that carries a tagger name, email, timestamp, and message (and can be GPG-signed). Created with git tag -a <name> -m "message".
For releases, always use annotated tags. They record who released what and when, and they’re required for signed-release verification.

git tag -a v1.0.0 -m "Release v1.0.0: initial public release"
Use lightweight tags only for quick, personal markers you don’t share.

Listing, Pushing, and Checking Out Tags
git tag                           # list all tags
git tag -l "v1.*"                 # list tags matching a glob
git show v1.0.0                   # inspect the tag and its commit
git push origin v1.0.0            # push ONE tag to the remote
git push --tags                   # push ALL local tags
git switch --detach v1.0.0        # check out the tagged commit (detached HEAD)
git tag -d v1.0.0                 # delete the tag locally
git push origin :refs/tags/v1.0.0 # delete the tag on the remote
Tags are not pushed by default with git push. You must explicitly push them, either individually or with --tags. This is a common source of confusion — “I tagged the release but my teammate can’t see it.”

Semantic Versioning and git describe
Teams often follow Semantic Versioning (SemVer): MAJOR.MINOR.PATCH. Each component signals a different level of change:

Tagging Releases table
Bump	When	Example
PATCH (1.2.3 → 1.2.4)	Backwards-compatible bug fix	Fix crash when input is empty
MINOR (1.2.4 → 1.3.0)	Backwards-compatible new feature	Add optional --verbose flag
MAJOR (1.3.0 → 2.0.0)	Breaking change that existing callers can’t use unchanged	Remove deprecated function; change default argument
Conventional Commits plug directly into this: tools like semantic-release and standard-version read the feat: / fix: / BREAKING CHANGE: prefixes in your commit history and automatically decide the next version number. For example, given these three commits since the last release (v1.2.3):

fix(parser): handle empty input
feat(cli): add --verbose flag
fix(logger): correct timestamp format
semantic-release sees one feat (MINOR bump wins over fix) and releases v1.3.0 — generating a CHANGELOG.md entry that groups the commits by type. A single commit with BREAKING CHANGE: in its footer would instead bump the MAJOR. The convention is a machine-readable protocol, not just a naming style.

git describe produces a human-readable version string from the nearest tag:

$ git describe
v1.2.0-15-ga3f2d9c
Read this as “15 commits past the v1.2.0 tag, at commit a3f2d9c“. Build systems use this to stamp binaries with their exact source version.

Quick Check — Tagging Releases. Try these before peeking:

What’s the practical difference between git tag v1.0.0 (lightweight) and git tag -a v1.0.0 -m "…" (annotated)? Which one should you use for a public release?
You’ve tagged v1.0.0 locally and pushed your branch. Your teammate pulls — can they see v1.0.0? What do you need to do?
Your project uses SemVer. A commit introduces a change to a public API that old callers can no longer use unchanged. Should the next version bump the MAJOR, MINOR, or PATCH number?
Click to view answers
Rewriting History
The commands in this section either create new commit objects with new hashes or move branch pointers backward — operations that rewrite or rearrange history. They are powerful, but the rule below is non-negotiable.

The Golden Rule: Never Rewrite Pushed Commits
⚠️ Never rewrite a branch that has been pushed to a shared remote. The new commits look the same to you but have different hashes, so collaborators’ clones still reference the old hashes — a recipe for conflicts, duplicate patches, and lost work.

All of the operations below create new commit objects or move pointers backward. They are safe on local, unpushed commits and dangerous on anything that has been pushed. When in doubt, use git revert (additive — see Undoing Committed Work) instead.

Rebasing a Branch
Why would I ever rebase instead of merging?
Because merge and rebase produce different shapes of history, and sometimes you want the shape rebase gives you. A git merge feature into main preserves the fact that feature was a parallel line of work — you get a diamond in the graph. A git rebase main on feature replays your feature commits on top of the latest main, producing a straight line of history with no fork.

Three concrete situations where people reach for rebase:

Cleaning up before a PR. Your feature branch has been open for a week; main has moved; you want the diff in the PR to be exactly your changes, not “your changes plus everything else that happened”. A git rebase main replays your commits on top of the current main so the PR is clean.
Keeping a linear log. Some teams prefer git log --oneline on main to read as a single chain of features rather than a braided mess of merges. Rebasing feature branches before merging keeps the line straight.
Squashing WIP commits. Interactive rebase (-i) lets you combine, reorder, reword, or drop commits — handy when you have “fix typo” and “oops forgot semicolon” commits you don’t want in the permanent record.
The cost: because replayed commits have different hashes from the originals, rebasing a branch you’ve already pushed breaks everyone else’s clone of it. That’s why rebase is safe locally and dangerous after pushing — the same rule that governs every other “rewrites history” operation.

Takes the commits unique to feature (C and D) and replays them on top of main's current tip (E).

Because each replayed commit has a different parent than before, it gets a new hash C′, D′ — the old C and D are gone. feature now looks like it was branched from the latest main.

Clean linear history — but because hashes changed, never rebase a branch you've already pushed.


▶
git rebase main
main
feature
HEAD
E
Hotfix
D
Feature B
C
Feature A
B
Initial commit
A
Repository init
HEAD is on branch feature at commit D: Feature B. 5 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch feature at commit D: Feature B. Branch tips: main at E: Hotfix; feature at D: Feature B. Commits in newest-first order: 1. Commit E: Hotfix. Labels: main. Parents: B: Initial commit. Children: none. 2. Commit D: Feature B. Labels: HEAD -> feature. Parents: C: Feature A. Children: none. 3. Commit C: Feature A. Parents: B: Initial commit. Children: D: Feature B. 4. Commit B: Initial commit. Parents: A: Repository init. Children: E: Hotfix; C: Feature A. 5. Commit A: Repository init. Parents: none. Children: B: Initial commit.Git graph updated: HEAD is on branch feature at commit D: Feature B. 5 commits. 2 branchs.
Divergence and Time-Travel
The single-step card above shows rebase as a finished magic trick — two commits appear on top of main with new hashes. The multi-step walkthrough below pulls the trick apart: you build up the divergence yourself, pause to see the fork, and only then ask Git to replay history. Watch the graph, not the commands — the whole point is to replace “commands I memorised” with “pointer moves I can picture”.

Start from an empty repo. We'll build up a two-branch divergence commit by commit, pause to observe the fork, and then use git rebase to flatten it — watching commit C vanish and a brand-new C′ appear on top of D.

Three ideas to hold in your head as you click through:

1. Commands are pointer moves. Branches are lightweight labels; checkout and commit just slide those labels along a DAG. 2. Parallel universes are real. Once main and feature both have commits the other lacks, history is not a single timeline anymore. 3. Commits are immutable. Rebase never moves C — it copies its changes onto a new parent, producing a different commit object (C′) with a different hash.

Step 0 of 6 (initial state).

←
Back

▶
git commit -m "A" && git commit -m "B"
🌳
No commits yet.
Run git commit to see the graph.
HEAD position is not available. 0 commits. 0 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD position is not available. No branches are shown. No commits are shown.Git graph updated: HEAD position is not available. 0 commits. 0 branchs.
Interactive Rebase
git rebase -i <base> opens an editor with a todo file listing each commit between <base> and HEAD. You change the action in front of each line to rewrite history exactly how you like:

Rewriting History table
Action	Effect
pick	Keep the commit as-is
reword	Keep, but edit the message
edit	Stop at this commit to amend it
squash	Fold into the previous commit (combine messages)
fixup	Like squash, but discard this commit’s message
drop	Remove the commit entirely
Interactive rebase lets you rewrite a range of commits commit-by-commit.

Here we open the todo file, keep B as pick, and mark C and D as squash — meaning Git melds them into the previous commit.

The three commits collapse into one new commit B+C+D with a unified message. Perfect for tidying up a series of "fix typo" / "oops" commits before sharing.

~/.git/rebase-merge/git-rebase-todo
pick   B Add login feature
squash C Fix login bug
squash D Fix typo again

# Rebase A..D onto A (3 commands)
#
# Commands:
# p, pick   = use commit
# r, reword = use commit, edit message
# e, edit   = use commit, stop to amend
# s, squash = meld into previous commit
# f, fixup  = like squash, discard this commit's message
# d, drop   = remove commit

▶
git rebase -i HEAD~3 (squash)
main
HEAD
D
Fix typo again
C
Fix login bug
B
Add login feature
A
Repository init
HEAD is on branch main at commit D: Fix typo again. 4 commits. 1 branch. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit D: Fix typo again. Branch tips: main at D: Fix typo again. Commits in newest-first order: 1. Commit D: Fix typo again. Labels: HEAD -> main. Parents: C: Fix login bug. Children: none. 2. Commit C: Fix login bug. Parents: B: Add login feature. Children: D: Fix typo again. 3. Commit B: Add login feature. Parents: A: Repository init. Children: C: Fix login bug. 4. Commit A: Repository init. Parents: none. Children: B: Add login feature.Git graph updated: HEAD is on branch main at commit D: Fix typo again. 4 commits. 1 branch.
Same interactive-rebase mechanism, different action: marking C as drop removes it from history entirely.

The later commit D is replayed on top of B (skipping C), producing a new commit D′ with a new hash.

Useful for removing an embarrassing commit or a stray debug change that shouldn't have been committed.

~/.git/rebase-merge/git-rebase-todo
pick B Add login
drop C Oops debug print
pick D Add tests

# Rebase A..D onto A (3 commands)
#
# Commands:
# p, pick   = use commit
# r, reword = use commit, edit message
# e, edit   = use commit, stop to amend
# s, squash = meld into previous commit
# f, fixup  = like squash, discard this commit's message
# d, drop   = remove commit

▶
git rebase -i HEAD~3 (drop)
main
HEAD
D
Add tests
C
Oops debug print
B
Add login
A
Repository init
HEAD is on branch main at commit D: Add tests. 4 commits. 1 branch. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit D: Add tests. Branch tips: main at D: Add tests. Commits in newest-first order: 1. Commit D: Add tests. Labels: HEAD -> main. Parents: C: Oops debug print. Children: none. 2. Commit C: Oops debug print. Parents: B: Add login. Children: D: Add tests. 3. Commit B: Add login. Parents: A: Repository init. Children: C: Oops debug print. 4. Commit A: Repository init. Parents: none. Children: B: Add login.Git graph updated: HEAD is on branch main at commit D: Add tests. 4 commits. 1 branch.
Cherry-Picking a Commit
git cherry-pick <hash> copies a single commit from another branch onto the current branch as a new commit (new hash, same changes). Useful to grab a specific fix without merging an entire branch:

Copies the changes from a single commit onto the current branch as a brand-new commit.

Here we grab a specific fix (H) from a short hotfix branch without merging the whole branch.

The copy gets a new hash H′ because its parent is different, but it contains the same changes. Useful for backporting a bug fix to a release branch.


▶
git cherry-pick H
main
hotfix
HEAD
D
Feature D
C
Feature C
H
Security patch
B
Feature B
A
Initial commit
HEAD is on branch main at commit D: Feature D. 5 commits. 2 branchs. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit D: Feature D. Branch tips: main at D: Feature D; hotfix at H: Security patch. Commits in newest-first order: 1. Commit D: Feature D. Labels: HEAD -> main. Parents: C: Feature C. Children: none. 2. Commit C: Feature C. Parents: B: Feature B. Children: D: Feature D. 3. Commit H: Security patch. Labels: hotfix. Parents: A: Initial commit. Children: none. 4. Commit B: Feature B. Parents: A: Initial commit. Children: C: Feature C. 5. Commit A: Initial commit. Parents: none. Children: H: Security patch; B: Feature B.Git graph updated: HEAD is on branch main at commit D: Feature D. 5 commits. 2 branchs.
Deciding Between Rebase, Cherry-Pick, and Squash Merge
All three create new commits with new hashes. Their difference is scope and intent:

Rewriting History table
Command	Scope	Intent
git rebase <base>	All commits unique to the current branch	“Put my work on top of the latest base.” Produces linear history before a PR.
git cherry-pick <sha>	One commit (or a small range)	“I need this one fix on a different branch.” Backports, selective pickups.
git merge --squash <branch>	All commits on a branch, collapsed into one	“Land this whole feature as a single commit on main.” Clean feature-log.
All three obey the Golden Rule — never rewrite pushed history.

Quick Check — Rewriting History. Try these before peeking:

State the Golden Rule in your own words and explain why it exists (what actually breaks if you ignore it?).
Your branch has three commits on top of main: Add login, Oops debug print, Add tests. You want to land this as clean work on main. Which rewrite tool removes the middle commit without touching the other two, and what happens to the hashes?
A hotfix went in as commit a3f2d9c on the release-2.x branch. You need the same fix on main. You have two choices: git merge release-2.x or git cherry-pick a3f2d9c. Which do you pick, and why?
git rebase and git merge --squash both “clean up” history. Name one concrete situation where each is the right tool.
Click to view answers
Branching Strategies
Once you can branch, merge, and open pull requests, the next question is: how should the team organize branches? Different answers emerge based on release cadence, team size, and tolerance for complexity. Three strategies cover most industry practice.

Gitflow
Gitflow uses long-lived main and develop branches plus short-lived feature/*, release/*, and hotfix/* branches.

Branching Strategies table
Branch	Purpose	Lifetime
main	Production-ready code; tagged with release versions	Permanent
develop	Integration branch for unreleased work	Permanent
feature/X	New feature	Days–weeks
release/X	Stabilisation before a release	Days
hotfix/X	Urgent fix to production	Hours
Pros: Clear roles; supports parallel releases and post-release hotfixes. Cons: Heavy for small teams and fast-moving projects; long-lived branches invite merge-hell. Best for: Versioned, shipped-to-customer software with slow release cadences.

Trunk-Based Development
Trunk-based development keeps a single long-lived branch (main or trunk) and insists that feature branches live for hours, not days. Developers integrate multiple times a day. Unfinished work hides behind feature flags rather than on separate branches.

Pros: Minimal integration pain; small PRs; fast CI feedback. Cons: Requires CI discipline; feature flags add complexity; riskier for regulated environments. Best for: Continuous-deployment SaaS, high-velocity teams, modern web applications.

Feature Branches with Pull Requests (GitHub Flow)
The middle ground, popular on GitHub: one long-lived main branch plus short-lived feature branches, each merged via a pull request after review and CI. No develop, no release/*.

Pros: Simple model; aligns with the platform UX; supports PR review. Cons: No built-in place for release stabilisation. Best for: Most modern teams — this is the default for open-source and many internal projects.

Choosing a Strategy
A rough decision tree:

Ship continuously to production, one version? → Trunk-based or GitHub Flow.
Ship multiple versions in parallel to customers on different schedules? → Gitflow.
Small team, no strong preference? → GitHub Flow (least ceremony).
The single most important choice is keeping feature branches short. Regardless of strategy, branches that live for weeks accumulate merge conflicts and hide unfinished work from CI. Aim for days, not weeks.

Quick Check — Branching Strategies. Try these before peeking:

A startup ships a SaaS product to production several times a day from a single live version. Which strategy fits best, and what mechanism lets unfinished features live in main without shipping?
An enterprise product ships quarterly releases and simultaneously maintains v1.x, v2.x, and v3.x lines for different customers. Which strategy fits best, and why?
Regardless of strategy, one discipline matters more than the strategy choice itself. What is it, and why?
Click to view answers
Submodules
For very large projects, Git submodules let you include another Git repository as a subdirectory while keeping its history independent. The superproject records two things for each submodule: a pinned commit SHA of the external repo, and a URL in a top-level .gitmodules file. Pulling always brings in the pinned revision, which makes submodule updates explicit rather than automatic.

🔧 Under the Hood: where the submodule's .git directory lives (optional — skip on first pass)
The walk-through below covers the commands you’ll meet most: adding submodules, cloning a parent repo that uses them, and updating submodules to new commits. Each step mutates the directory tree; the changed rows are announced in the lab status and also flash briefly so you can see exactly what the command touched.

A superproject myproject/ starts with just its own source. We'll add two submodules, commit the result, then see what happens on a fresh clone.

Step 0 of 8 (initial state).

←
Back

▶
git submodule add https://github.com/acme/libfoo libs/foo
myproject/
.git/
src/
app.js
Quick Check — Submodules. Try these before peeking:

A submodule pins one specific thing about the external repo. What is it, and what does that mean for teammates who pull?
You clone a repo that uses submodules with plain git clone. The submodule directories exist but are empty. What one-command alternative would have populated them, and which two commands would you run after a plain clone to fix it?
Why use submodules over just copy-pasting the dependency’s files into your repo?
Click to view answers
Investigating History
Once a project has accumulated history, reading it — and searching it — becomes its own skill. Four commands cover almost all investigation work.

Viewing Commits (git log, git show)
git log shows the sequence of past commits. Useful flags:

-p — show each commit’s full patch (diff).
--oneline — one commit per line (hash + subject).
--graph --all — ASCII art graph across all branches and merges.
--stat — per-file change summary (no full diff).
--grep="<pattern>" — search commit messages.
-S"<string>" — “pickaxe”: find commits whose diff adds or removes <string>.
-- <path> — limit to commits that touched <path>.
git log --oneline --graph --all   # the most useful overview
git log -p -- src/auth.py         # every change to one file, with diffs
git log --grep="rate limit"       # find "rate limit" in commit messages
git log -S"RateLimiter"           # find commits that added/removed the string "RateLimiter"
git show <commit> displays detailed information about a specific commit — the message, the author, the full diff. Pair it with git blame (below) to go from a suspicious line to the commit that wrote it:

git blame -L 42,42 src/auth.py   # who last touched line 42?
# copy the SHA, then:
git show <sha>                    # read the full context
Tracing a Line’s Origin (git blame)
git blame <file> annotates each line with the author, commit hash, and timestamp of the last person to modify it. Essential for understanding why a line exists before changing it:

git blame src/auth.py             # annotate every line
git blame -L 42,50 src/auth.py    # narrow to lines 42–50
git blame -w src/auth.py          # ignore whitespace-only changes (skip reformat commits)
What blame doesn’t see: lines that used to exist but were deleted. For those — or for any behavioural regression where you don’t yet know which line is at fault — use git bisect.

Binary-Searching for Regressions (git bisect)
git bisect binary-searches through commit history to find the exact commit that introduced a bug. You mark known-good and known-bad commits, then Git checks out the midpoint repeatedly. With 1,000 commits in the range, it finds the culprit in at most 10 tests.

The workflow for git bisect is always the same six-step ritual — start a session, mark bad, mark good, then let Git drive. Click through the demo below to see each command and its effect on the graph.

Our repository has 6 commits on main. A bug appeared recently — we know commit B (Initial setup) was clean. We'll use git bisect to binary-search the history and pinpoint the exact bad commit.

git bisect eliminates half the remaining candidates on each step, so even a repo with 1,000 commits needs at most 10 tests.

Step 0 of 6 (initial state).

←
Back

▶
git bisect start
main
HEAD
F
Add analytics
E
Fix layout
D
Add auth
C
Add login
B
Initial setup
A
Repository init
HEAD is on branch main at commit F: Add analytics. 6 commits. 1 branch. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit F: Add analytics. Branch tips: main at F: Add analytics. Commits in newest-first order: 1. Commit F: Add analytics. Labels: HEAD -> main. Parents: E: Fix layout. Children: none. 2. Commit E: Fix layout. Parents: D: Add auth. Children: F: Add analytics. 3. Commit D: Add auth. Parents: C: Add login. Children: E: Fix layout. 4. Commit C: Add login. Parents: B: Initial setup. Children: D: Add auth. 5. Commit B: Initial setup. Parents: A: Repository init. Children: C: Add login. 6. Commit A: Repository init. Parents: none. Children: B: Initial setup.Git graph updated: HEAD is on branch main at commit F: Add analytics. 6 commits. 1 branch.
Automating bisect. If your test script exits 0 on success and non-zero on failure, git bisect run <script> automates the whole search — Git runs the script at each candidate and uses the exit code to decide. Always end with git bisect reset — without it, HEAD stays on the last-checked historical commit, which is a confusing state to leave behind.

Quick Check — Investigating History. Try these before peeking:

You want to find every commit that mentions “rate limit” in its message, and — separately — every commit whose diff added or removed the string RateLimiter. Which git log flags?
A line in src/auth.py looks wrong. Which command tells you who last touched it, and which command do you then run to see the full context of that change?
A regression slipped in between release v1.2.0 (known good) and HEAD (known bad). The range covers 256 commits. At most how many tests does git bisect need to find the culprit, and why?
Your bug is caused by a line that used to exist and was deleted. Why won’t git blame find it, and what tool would you use instead?
Click to view answers
Undoing Committed Work
Mistakes reach your history eventually — a buggy commit, an accidental merge, an embarrassing message. Git provides two opposing tools for undoing committed work, plus a safety net that makes both survivable.

Why do we need two ways to “undo” a commit?
Because there are two genuinely different situations, and they call for opposite strategies:

The commit is only in your local repo (you haven’t pushed). You can just rewind the branch pointer — the commit becomes unreachable, garbage-collected later, and nobody else ever saw it. This is what git reset does.
The commit has been pushed and teammates have it. You can’t safely erase it — their clones still reference it, and trying to rewrite shared history makes every pull a conflict. The only safe undo is to add another commit that inverts the change. This is what git revert does.
The rule of thumb: reset for private mistakes, revert for public mistakes. The rest of this section unpacks both.

Reverting a Commit (git revert)
✅ Additive. Safe on shared branches — preserves history exactly.

git revert <sha> creates a new commit whose changes are the exact inverse of the target commit. The original commit stays in history; the revert commit cancels its effect. Because no existing commits are modified, revert is safe even on branches that teammates have already pulled.

Additive operation — safe on shared branches. Instead of rewriting history, revert appends a new commit R whose changes exactly undo those of the target commit.

The original commit (C) still exists and is still reachable — history now records both the bug being introduced and it being reverted.

Because no existing hashes change, teammates who already pulled the buggy commit see the revert commit as a normal follow-up. This is the only safe way to undo a commit that has already been pushed.


▶
git revert HEAD
main
HEAD
C
Buggy commit
B
Add feature
A
Repository init
HEAD is on branch main at commit C: Buggy commit. 3 commits. 1 branch. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit C: Buggy commit. Branch tips: main at C: Buggy commit. Commits in newest-first order: 1. Commit C: Buggy commit. Labels: HEAD -> main. Parents: B: Add feature. Children: none. 2. Commit B: Add feature. Parents: A: Repository init. Children: C: Buggy commit. 3. Commit A: Repository init. Parents: none. Children: B: Add feature.Git graph updated: HEAD is on branch main at commit C: Buggy commit. 3 commits. 1 branch.
Resetting a Branch (git reset)
⚠️ Rewrites history. Only safe on local, unpushed commits.

git reset <sha> moves the current branch pointer to <sha>, effectively discarding every commit between the old tip and <sha>. Those commits become unreachable from any branch and are eventually garbage-collected (though reflog can recover them within the retention window).

Three modes determine what happens to the working tree and staging area:

Undoing Committed Work table
Mode	Branch pointer	Staging area	Working tree	Use this when…
--soft	moves to target	preserved	preserved	You want to un-commit but keep everything staged — to re-commit with a better message, or to split the commit into smaller pieces.
--mixed (default)	moves to target	reset to target	preserved	You want to un-commit and un-stage, keeping your edits as plain working-tree changes to re-organize.
--hard	moves to target	reset to target	overwritten	You want the commit and its changes gone — a full wipe back to the target. Your uncommitted work is destroyed.
Most common uses:

git reset --soft HEAD~1 — “un-commit” the last commit while keeping the changes staged (perfect for re-committing with a better message or splitting into smaller commits).
git reset HEAD~1 — un-commit and un-stage (changes stay as unstaged edits).
git reset --hard HEAD~1 — discard the commit and the changes entirely.
Rewinds the main branch pointer by one commit, dropping everything that was on top.

The tip commit (C) is no longer reachable from any branch and will be garbage-collected eventually.

--hard also overwrites your working directory and staging area to match the target commit, so any uncommitted changes are lost — watch the staged notes.txt vanish along with commit C.

Only safe for local, unpushed work.


▶
git reset --hard HEAD~1
Untracked:
Not staged:
modified:
src/bug.js
Staged:
new file:
notes.txt
main
HEAD
C
Buggy commit
B
Add feature
A
Repository init
HEAD is on branch main at commit C: Buggy commit. 3 commits. 1 branch. working tree has 1 staged file, 1 unstaged file. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit C: Buggy commit. Branch tips: main at C: Buggy commit. Commits in newest-first order: 1. Commit C: Buggy commit. Labels: HEAD -> main. Parents: B: Add feature. Children: none. 2. Commit B: Add feature. Parents: A: Repository init. Children: C: Buggy commit. 3. Commit A: Repository init. Parents: none. Children: B: Add feature. Staged files: new file notes.txt. Changes not staged for commit: modified src/bug.js.Git graph updated: HEAD is on branch main at commit C: Buggy commit. 3 commits. 1 branch. working tree has 1 staged file, 1 unstaged file.
Choosing: reset vs. revert
Choosing: reset vs. revert table
Situation	Use
Mistake is on a local, unpushed branch	git reset (any mode)
Mistake has been pushed to a shared branch	git revert — always
You want to preserve history as an audit trail	git revert
You want to erase an embarrassing experiment (local only)	git reset --hard
Force-pushing a rewritten shared branch after git reset is how teams accidentally destroy each other’s work. See the Force-Push Warning.

Detached HEAD
HEAD normally points at a branch (e.g. ref: refs/heads/main). If you point HEAD directly at a commit — git switch --detach <sha>, checking out a tag, or mid-bisect — you are in detached HEAD state. No branch is “following” your commits.

Points HEAD directly at a specific commit (here, one step before the current tip) instead of at a branch. This is called detached HEAD state.

Useful for inspecting old code — but commits made in this state aren't anchored to any branch and are easy to lose when switching away.

Recover them with git reflog, or stay safe by running git switch -c to bookmark your work before leaving.


▶
git switch --detach HEAD~1
main
HEAD
B
Add login
A
Initial commit
HEAD is on branch main at commit B: Add login. 2 commits. 1 branch. A generated detailed text alternative follows in the reading order.Detailed Git graph text alternative. HEAD is on branch main at commit B: Add login. Branch tips: main at B: Add login. Commits in newest-first order: 1. Commit B: Add login. Labels: HEAD -> main. Parents: A: Initial commit. Children: none. 2. Commit A: Initial commit. Parents: none. Children: B: Add login.Git graph updated: HEAD is on branch main at commit B: Add login. 2 commits. 1 branch.
Why it matters: any commits you make while detached are only reachable through HEAD. The moment you git switch to another branch, your new commits have no branch pointer anchoring them — they are orphaned. Git will garbage-collect them after the reflog retention window expires.

The fix is always the same: before leaving detached HEAD, create a branch to anchor any new work:

git switch -c my-experiment
The Safety Net: git reflog
🔧 Under the Hood: why "deleted" commits are recoverable (optional — skip on first pass)
Every time HEAD moves — commit, checkout, reset, rebase, merge, cherry-pick, stash — Git records the movement in the reflog, a per-repository diary of HEAD’s positions. The reflog is local, never pushed, and kept for a generous retention window by default (configurable via gc.reflogExpire and gc.reflogExpireUnreachable).

$ git reflog
a3f2d9c HEAD@{0}: reset: moving to HEAD~2
b7e1c4d HEAD@{1}: commit: Add login validation
c9a2f3e HEAD@{2}: checkout: moving from main to feat-login
...
Each entry is <sha> HEAD@{n}: <operation>: <description>. The @{n} syntax is reflog-relative — HEAD@{1} means “where HEAD was one move ago”, HEAD@{2} two moves ago, and so on.

The universal recovery recipe — for any destructive operation (rebase drop, hard reset, detached-HEAD orphan, merge gone wrong):

Run git reflog and find the SHA of the state you want to return to.
Create a branch anchoring that SHA:
git branch rescued-work <sha>
# or, if you want to reset your current branch instead:
git reset --hard <sha>
That’s the whole pattern. Every “oh no, I lost my commits” question on Stack Overflow resolves to these two steps, as long as the reflog still has the entry and git gc hasn’t pruned the unreachable objects.

Why this works. Commits are immutable and SHAs are content-addressed. A “deleted” commit isn’t deleted — it’s unreferenced. As long as some reference (a branch, a tag, or the reflog) still mentions its SHA, the object is safe. The reflog is therefore the universal bookmark, surviving even when every branch pointer has moved away.

The reflog is one of the deepest reasons Git is forgiving: destructive commands look scary, but they are almost always recoverable for weeks after the fact.

Quick Check — Undoing Committed Work. Try these before peeking:

A buggy commit has been pushed to main and several teammates have already pulled it. Should you git reset --hard or git revert? Why?
For git reset, rank the three modes by how much state they destroy (least to most): --soft, --mixed, --hard.
You do git switch --detach <sha>, make two commits, then git switch main without creating a branch. Your new commits appear to be “gone”. Are they really deleted? What’s the recovery recipe?
State the universal recovery recipe for “I lost my commit” in two steps.
Click to view answers
Choosing the Right Tool
Return-readers come to this page with a specific intent: “I want to do X, which Git command?” This table is that index.

Choosing the Right Tool table
You want to…	Reach for…	Section
Make your changes part of the project’s history	git add then git commit	Making Commits
Discard your uncommitted edits to one file	git restore <file>	Managing Uncommitted Changes
Un-stage a file you accidentally added	git restore --staged <file>	Managing Uncommitted Changes
Temporarily save your work for later	git stash / git stash pop	Managing Uncommitted Changes
Fix a typo in your most recent commit (local only)	git commit --amend ⚠️	Making Commits
Start a new line of work	git switch -c <branch>	Branching
Bring a feature branch into main	git merge <branch>	Merging Branches
Land a feature as a single clean commit on main	git merge --squash <branch> ⚠️	Merging Branches
Preview what an incoming merge would change	git fetch then git diff main...origin/main (triple-dot)	Collaborating with Remotes
Copy one specific commit from another branch	git cherry-pick <sha>	Reshaping History
Clean up messy WIP commits before opening a PR	git rebase -i <base> ⚠️	Reshaping History
Rebase your feature branch onto the latest main	git rebase main ⚠️	Reshaping History
Mark a commit as release v1.0.0	git tag -a v1.0.0 -m "..." then git push --tags	Tagging Releases
Undo a commit that’s already been pushed	git revert <sha>	Undoing Committed Work
Delete commits on your local (unpushed) branch	git reset --hard <sha> ⚠️	Undoing Committed Work
Find which commit introduced a bug	git bisect start + git bisect run <test>	Investigating History
Find who last changed line 42 of a file	git blame -L 42,42 <file> then git show <sha>	Investigating History
Recover a commit that looks “lost”	git reflog + git branch <name> <sha>	Undoing Committed Work
See the history graph across all branches	git log --oneline --graph --all	Investigating History
Upload your branch for a PR	git push -u origin <branch>	Collaborating with Remotes
Get teammates’ changes without merging yet	git fetch	Collaborating with Remotes
Get and integrate teammates’ changes	git pull (or git pull --rebase)	Collaborating with Remotes
Include another repo as a pinned dependency	git submodule add <url> <path>	Submodules
Legend: ⚠️ = rewrites history; never run on commits that have been pushed to a shared branch.

Best Practices
A condensed checklist. Each item links back to its full section.

Write meaningful commit messages. Imperative mood, ≤50-character subject, blank line, wrapped body explaining why.
Commit small and often. Prefer many coherent commits over one giant “everything” update.
Create .gitignore before your first commit. It has no retroactive effect on tracked files. Commit .gitignore itself so the team shares the rules.
Never commit secrets. .gitignore is not a security tool — if a secret is ever committed, rotate it immediately and scrub history.
Never force-push on shared branches. git push -f can permanently delete your collaborators’ work. Use --force-with-lease only on branches only you work on.
Prefer revert over reset for shared history. reset --hard destroys commits; revert preserves history.
Follow the golden rule of shared history. Never rewrite pushed commits — use revert instead.
Pull frequently. Regularly pull the latest changes from main to catch merge conflicts while they are small.
Prefer git switch and git restore over git checkout. The checkout command is overloaded — it does both branch navigation and file restoration. The split replacements (introduced in Git 2.23) make intent clearer. git checkout is still fully supported for backward compatibility.
Review branching strategy with your team. Short-lived branches beat long-lived ones every time, regardless of which strategy you pick.
Let git reflog be your safety net. Destructive operations are almost always recoverable within Git’s retention window (configured via gc.reflogExpire / gc.reflogExpireUnreachable). Don’t panic, reflog first.
Practice
Basic Git
Basic Git Flashcards
Which Git command would you use for the following scenarios?

Difficulty:
Intermediate
You have a specific commit hash and want to see detailed information about it, including the commit message, author, and the exact code diff it introduced.

Show Answer
Basic Git Quiz
Test your knowledge of core version control concepts, Git architecture, branching, merging, and collaboration.

Difficulty:
Advanced
Arrange the Git commands into the correct order to: create a feature branch, make changes, and integrate them back into main via a merge.

Drag fragments into the answer area in the correct order (some items are distractors that should not be used). Keyboard: focus a line and press Space or Enter to move it between the bank and the answer area. Use Arrow Up or Arrow Down to reorder within the answer area.
→ Drop here →
Check Order
Reset
Advanced Git
Advanced Git Flashcards
Which Git command would you use for the following advanced scenarios?

Difficulty:
Advanced
You want to integrate a feature branch into main, but instead of bringing over all 15 tiny incremental commits, you want them combined into one clean commit on the main branch.

Show Answer
Advanced Git Quiz
Test your knowledge of advanced Git commands, debugging tools, and integration strategies.

Difficulty:
Intermediate
You have some uncommitted, incomplete changes in your working directory, but you need to switch to another branch to urgently fix a bug. Which command is best suited to temporarily save your current work without making a messy commit?


A
git revert


B
git bisect


C
git stash


D
git cherry-pick

References
(Goode and Rain 2014): Durham Goode and Rain (2014) “Scaling Mercurial at Facebook.” Engineering at Meta.
(Potvin and Levenberg 2016): Rachel Potvin and Josh Levenberg (2016) “Why Google Stores Billions of Lines of Code in a Single Repository,” Communications of the ACM, 59(7), pp. 78–87.