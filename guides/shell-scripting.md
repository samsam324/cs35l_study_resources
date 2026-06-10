Start here: If you are new to shell scripting, begin with the Interactive Shell Scripting Tutorial — hands-on exercises in a real Linux system. This article is a reference to deepen your understanding afterward.

If you have ever found yourself performing the same repetitive tasks on your computer—renaming batches of files, searching through massive text logs, or configuring system environments—then shell scripting is the magic wand you need. Shell scripting is the bedrock of system administration, software development workflows, and server management.

In this detailed educational article, we will explore the concepts, syntax, and power of shell scripting, specifically focusing on the most ubiquitous UNIX shell: Bash.

Basics
What is the Shell?
To understand shell scripting, you first need to understand the “shell”.

An operating system (like Linux, macOS, or Windows) acts as a middleman between the physical hardware of your computer and the software applications you want to run. It abstracts away the complex details of the hardware so developers can write functional software.

The kernel is the core of the operating system that interacts directly with the hardware. The shell, on the other hand, is a command-line interface (CLI) that serves as the primary gateway for users to interact with a computer’s operating system. While many modern users are accustomed to graphical user interfaces (GUIs), the shell is a program that specifically takes text-based user commands and passes them to the operating system to execute.

Motivation: Why the Shell is Essential
As a software engineer, you need to be familiar with the ecosystem of tools that help you build software efficiently. The Linux ecosystem offers a vast array of specialized tools that allow you to write programs faster and debug log files by combining small, powerful commands. Understanding the shell increases your productivity in a professional environment and provides a foundation for learning other domain-specific scripting languages. Furthermore, the shell allows you to program directly on the operating system without the overhead of additional interpreters or heavy libraries.

The Unix Philosophy
The shell’s power is rooted in the Unix philosophy, which dictates:

Write programs that do one thing and do it well.
Write programs to work together.
Write programs to handle text streams, because that is a universal interface.
By treating data as a sequence of characters or bytes—similar to a conveyor belt rather than a truck—the shell allows parallel processing and the composition of complex behaviors from simple parts.

Essential UNIX Commands
Before writing scripts, you need to know the fundamental commands that you will be stringing together. These are the building blocks of any UNIX environment.

1. File Handling
These are the foundational tools for interacting with the POSIX filesystem:

ls: List directory contents (files and other directories).
cd: Change the current working directory (e.g., use .. to move to a parent folder).
pwd: Print the name of the current/working directory so you don’t get lost.
mkdir: Create a new directory.
cp: Copy files. Use -r (recursive) to copy a directory and its contents.
mv: Move or rename files and directories.
rm: Remove (delete) files. Use -r to remove a directory and its contents recursively.
rmdir: Remove empty directories (only works on empty ones).
touch: Create an empty file or update timestamps.
Play each card to see the command’s effect; click again to undo. The descriptions call out the flags you’ll reach for most often.

ls — list directory contents
Lists directory contents. -l switches to the long format (permissions, owner, size, mtime). -a includes hidden entries — anything starting with .. -h prints sizes as 1.2K / 4.3M instead of raw bytes. Plain ls gives you just the visible names.


▶
ls -la
project/
← (you are here)
.env
README.md
src/
app.js
Folder tree rooted at project/ with 2 folders and 3 files. Top-level entries: .env, README.md, src/. Entries: project/ (folder)  .env (file)  README.md (file)  src/ (folder)   app.js (file)
cd — change working directory
Changes the current working directory. A relative name like src is resolved from the cwd. .. goes up to the parent. ~ jumps to your home directory. - (a single dash) returns to the previous cwd. The change only affects the current shell process — a cd done inside a subshell (or a script run as ./script.sh) disappears when that shell exits, so it can't change the parent shell's cwd.


▶
cd src
project/
← (you are here)
README.md
src/
app.js
utils.js
Folder tree rooted at project/ with 2 folders and 3 files. Top-level entries: README.md, src/. Entries: project/ (folder)  README.md (file)  src/ (folder)   app.js (file)   utils.js (file)
pwd — print current path
Prints the absolute path of the current working directory. -L (the default) keeps symlink names in the path as-is. -P resolves symlinks to the real, physical path. Useful to sanity-check where you are before running destructive commands.


▶
pwd
project/
README.md
src/
← (you are here)
app.js
Folder tree rooted at project/ with 2 folders and 2 files. Top-level entries: README.md, src/. Entries: project/ (folder)  README.md (file)  src/ (folder)   app.js (file)
mkdir — create a directory
Creates a new directory. -p creates any missing parents and stays silent if the directory already exists — the safe, idempotent form used in scripts. -m 755 sets the permission mode at creation time. Without -p, every parent must already exist or mkdir errors out.


▶
mkdir -p docs/api
project/
← (you are here)
README.md
src/
app.js
Folder tree rooted at project/ with 2 folders and 2 files. Top-level entries: README.md, src/. Entries: project/ (folder)  README.md (file)  src/ (folder)   app.js (file)
mkdir without -p — missing parent
Without -p, every parent must already exist. Here docs/ has not been created yet, so mkdir refuses to create docs/api — it errors out and the filesystem is unchanged. The fix is mkdir -p docs/api or first running mkdir docs.


▶
mkdir docs/api
project/
← (you are here)
README.md
src/
app.js
Folder tree rooted at project/ with 2 folders and 2 files. Top-level entries: README.md, src/. Entries: project/ (folder)  README.md (file)  src/ (folder)   app.js (file)
cp — copy files and directories
Copies a file or directory. -r (or -R) is mandatory for directories — it recursively copies everything inside. -i prompts before overwriting an existing destination. -n never overwrites. -v prints every copied path. The source is preserved; this is duplication, not a move.


▶
cp -r src/ backup/
project/
← (you are here)
README.md
src/
app.js
utils.js
Folder tree rooted at project/ with 2 folders and 3 files. Top-level entries: README.md, src/. Entries: project/ (folder)  README.md (file)  src/ (folder)   app.js (file)   utils.js (file)
cp without -r — directory requires the flag
-r is mandatory when the source is a directory. Without it, cp refuses to copy src/ and prints an error — the filesystem is left unchanged. Add -r (or -R) to copy the directory and all its contents.


▶
cp src/ backup/
project/
← (you are here)
README.md
src/
app.js
utils.js
Folder tree rooted at project/ with 2 folders and 3 files. Top-level entries: README.md, src/. Entries: project/ (folder)  README.md (file)  src/ (folder)   app.js (file)   utils.js (file)
mv — move or rename
Moves or renames. If the destination is an existing directory (like archive/), the source is moved into it. If the destination is a new name, the source is renamed. -i prompts before overwriting; -n refuses to overwrite. Unlike cp, no -r is needed — mv handles directories natively.


▶
mv notes.txt archive/
project/
← (you are here)
README.md
notes.txt
archive/
Folder tree rooted at project/ with 2 folders and 2 files. Top-level entries: README.md, notes.txt, archive/. Entries: project/ (folder)  README.md (file)  notes.txt (file)  archive/ (folder)
rm — remove files and directories
Removes files. -r recurses into directories and deletes their contents. -f forces removal: no prompts, no errors for missing files. -i is the opposite — prompts for every file (safest). There is no trash can: once rm finishes, the files are gone. Double-check your path before pressing Enter.


▶
rm -rf tmp/
project/
← (you are here)
README.md
src/
app.js
tmp/
cache.db
logs/
build.log
Folder tree rooted at project/ with 4 folders and 4 files. Top-level entries: README.md, src/, tmp/. Entries: project/ (folder)  README.md (file)  src/ (folder)   app.js (file)  tmp/ (folder)   cache.db (file)   logs/ (folder)    build.log (file)
rmdir — remove an empty directory
Removes only empty directories. If build/ still contains files or subdirectories, rmdir refuses and errors out — far safer than rm -r when you just want to clean up a leftover empty folder. -p also removes parent directories that become empty as a side-effect.


▶
rmdir build/
project/
← (you are here)
README.md
build/
src/
app.js
Folder tree rooted at project/ with 3 folders and 2 files. Top-level entries: README.md, build/, src/. Entries: project/ (folder)  README.md (file)  build/ (folder)  src/ (folder)   app.js (file)
rmdir on a non-empty directory
rmdir only removes empty directories. src/ still contains app.js, so the command refuses and prints an error — the filesystem is left unchanged. Use rm -r src/ to remove a directory and all its contents.


▶
rmdir src/
project/
← (you are here)
README.md
src/
app.js
Folder tree rooted at project/ with 2 folders and 2 files. Top-level entries: README.md, src/. Entries: project/ (folder)  README.md (file)  src/ (folder)   app.js (file)
touch — create an empty file / bump timestamps
Creates an empty file if it doesn't exist, or updates the access/modification timestamps of an existing file. -a updates only the access time; -m only the modification time. -d "2024-01-01" (or -t) sets a specific timestamp — useful for reproducible builds and tricking make into re-running a target.


▶
touch .env
project/
← (you are here)
README.md
src/
app.js
Folder tree rooted at project/ with 2 folders and 2 files. Top-level entries: README.md, src/. Entries: project/ (folder)  README.md (file)  src/ (folder)   app.js (file)
Walkthrough: file handling in action
Step through a realistic session to see each command’s effect on the directory tree. New or changed rows are announced in the lab status and also flash briefly; the (you are here) marker tracks the current working directory.

Start in an empty project/ directory. Each step runs one command; the tree and ls output update to match what you would see in a real shell.

Step 0 of 10 (initial state).

←
Back

▶
pwd
project/
← (you are here)
README.md
src/
app.js
utils.js
2. Text Processing and Data Manipulation
Unix treats text streams as a universal interface, and these tools allow you to transform that data:

cat: Concatenate and print files to standard output.
grep: Search for patterns using regular expressions.
sed: Stream editor for filtering and transforming text (commonly search-and-replace).
tr: Translate or delete characters (e.g., changing case or removing digits).
sort: Sort lines of text files alphabetically; add -n for numeric order, -r to reverse.
uniq: Filter adjacent duplicate lines; the -c flag prefixes each line with its occurrence count. Because it only compares consecutive lines, you almost always pipe sort first so that duplicates are adjacent.
wc: Word count (lines, words, characters).
cut: Extract specific sections/fields from lines.
comm: Compare two sorted files line by line.
head / tail: Output the first or last part of files.
awk: Advanced pattern scanning and processing language.
These commands do not modify the filesystem tree — they transform streams of text. The lab cards below make that visible: inputs flow in from the left (stdin + any referenced files), the command transforms them, and outputs emerge on the right (stdout + stderr + exit status). For a few cards you will be asked to predict the output before running it — that one small act of committing a guess is worth far more than reading the answer cold.

cat — print a single file
cat reads one or more files and prints them to stdout. Given a single file argument it just dumps the contents. With no arguments at all, cat reads from stdin — which is why cat on its own looks like it "hangs" waiting for input.

📄
file · notes.txt
milk
eggs
bread

▶
cat notes.txt
Press Run to see what the command produces.
cat — what the name actually means: concatenate
Given multiple file arguments, cat prints them back-to-back in the order you listed them — literal concatenation. Combined with > redirection, this is a common way to stitch fragments together into a single file. The source files are never modified.

Three fragments are listed in order. What will book.txt contain after running this? Write out your expected output by copying lines from the input. Then run the command to check your answer.
Type your guess here, then run the command to compare…
📄
file · intro.txt
== Preface ==
Welcome.
📄
file · chapter1.txt
== Chapter 1 ==
It was a dark and stormy night.
📄
file · outro.txt
== The End ==

▶
cat intro.txt chapter1.txt outro.txt > book.txt
Write your prediction above, then press Run to reveal the output.
Common mistake — useless use of cat
This works, but there's no reason to spawn cat just to feed a single file into grep. grep can open the file itself: grep ERROR log.txt. Beginners reach for cat | grep because it feels like "the shell way" — but it hides what's really feeding the pipeline. The idiomatic form is below.

📄
file · log.txt
08:00 INFO start
08:01 ERROR disk full
08:02 INFO retry
08:03 ERROR timeout

▶
cat log.txt | grep ERROR
Press Run to see what the command produces.
grep — search for lines matching a pattern
grep prints each line of the file (or stdin) that matches the given pattern. Lines that don't match are silently dropped. The exit code is 0 if at least one match was found, 1 if nothing matched, and 2 on error.

Before running: which lines from log.txt do you expect to see on stdout? Write them out by copying lines from the input. Then run the command to check your answer.
Type your guess here, then run the command to compare…
📄
file · log.txt
08:00 INFO  start
08:01 ERROR disk full
08:02 INFO  retry
08:03 ERROR timeout
08:04 INFO  done

▶
grep ERROR log.txt
Write your prediction above, then press Run to reveal the output.
Common mistake — regex metacharacters in an unquoted pattern
The student wants lines containing the literal string a.b. But . is a regex metacharacter meaning "any single character" — so the pattern matches aab, axb, a3b, etc. Use grep -F 'a.b' for a fixed (literal) string, or escape the dot with grep 'a\.b'.

Which lines of names.txt will grep a.b print? Remember . matches exactly one character — not a dot, and not a run.
Type your guess here, then run the command to compare…
📄
file · names.txt
alice
aab
axb
a3b
aab-extra
aleph.bet
foobar

▶
grep a.b names.txt
Write your prediction above, then press Run to reveal the output.
grep — no match is not the same as error (exit code 1)
Sometimes students see a command "produce nothing" and assume it broke. grep deliberately exits with status 1 when it doesn't find any matches — that's not an error, it's a usable signal. Scripts branch on it: if grep -q ERROR log.txt; then alert; fi. Watch the exit badge carefully: the command succeeded in doing its job, it just found nothing.

📄
file · log.txt
08:00 INFO  start
08:01 INFO  retry
08:02 INFO  done

▶
grep WARN log.txt
Press Run to see what the command produces.
sed — stream editor (search and replace)
sed applies an editing script to each line of its input. s/ERROR/FAIL/ replaces the first occurrence of ERROR on each line with FAIL. Add a trailing g (s/ERROR/FAIL/g) to replace all occurrences on each line. The original file is not modified — sed writes the transformed stream to stdout. Use -i for in-place editing.

What does stdout look like after s/ERROR/FAIL/ runs on every line of log.txt?
Type your guess here, then run the command to compare…
📄
file · log.txt
08:01 ERROR disk full
08:03 ERROR timeout

▶
sed 's/ERROR/FAIL/' log.txt
Write your prediction above, then press Run to reveal the output.
Common mistake — single quotes block variable expansion in sed
The student has user="alice" in their shell and expects sed to substitute alice for guest. But single quotes prevent the shell from expanding $user — sed literally searches for the five characters $user. Since no line contains $user, nothing is replaced. Use double quotes (sed "s/$user/guest/") when you need variable expansion.

$user is alice in the shell environment. What do you think stdout will show — and how many lines get replaced?
Type your guess here, then run the command to compare…
⚙
environment
user
=
alice
📄
file · log.txt
alice logged in
bob logged in
alice logged out

▶
sed 's/$user/guest/' log.txt
Write your prediction above, then press Run to reveal the output.
tr — translate or delete characters
tr reads from stdin only (never a file — pipe or redirect into it) and translates characters one-for-one. Here every lowercase letter becomes uppercase. tr -d 'aeiou' would delete the listed characters instead of translating them. tr -s ' ' squeezes runs of spaces into one.

←
stdin
Hello, Ada Lovelace!

▶
tr 'a-z' 'A-Z'
Press Run to see what the command produces.
sort — sort lines
Sorts its input alphabetically, line by line. Add -n for numeric order (otherwise 10 sorts before 2), -r to reverse, -u to also drop duplicates. Works on stdin if no file argument is given.

Write the lines of names.txt in the order sort will print them.
Type your guess here, then run the command to compare…
📄
file · names.txt
charlie
alice
bob
alice

▶
sort names.txt
Write your prediction above, then press Run to reveal the output.
uniq — filter adjacent duplicate lines
uniq collapses duplicate lines — but it's a streaming tool that keeps exactly one line of memory. That means it only notices duplicates when they appear next to each other in the input.

names.txt contains four names, with alice appearing twice (but not in consecutive lines). How many lines will uniq print, and which ones? Write the exact output you expect here. Then run the command.
Type your guess here, then run the command to compare…
📄
file · names.txt
charlie
alice
bob
alice

▶
uniq names.txt
Write your prediction above, then press Run to reveal the output.
The fix — sort | uniq puts duplicates next to each other
Pipe through sort first so identical lines become adjacent — now uniq can see them and collapse them. The sort | uniq idiom is so common that sort -u names.txt is shorthand for exactly this pipeline. With -c, uniq also prefixes each line with its occurrence count (great for frequency tables).

📄
file · names.txt
charlie
alice
bob
alice

▶
sort names.txt | uniq
Press Run to see what the command produces.
wc — word / line / character count
Counts lines, words, and characters by default. -l shows just the line count, -w the word count, -c the byte count. Pedantic detail: wc -l counts newline characters — a file whose last line has no trailing newline reports one fewer line than you might expect.

How many lines does log.txt have? (Including the format — wc -l prints the number and the filename.)
Type your guess here, then run the command to compare…
📄
file · log.txt
08:00 INFO  start
08:01 ERROR disk full
08:02 INFO  retry
08:03 ERROR timeout
08:04 INFO  done

▶
wc -l log.txt
Write your prediction above, then press Run to reveal the output.
cut — extract columns from each line
Splits each line on the delimiter given by -d and prints the field(s) chosen by -f. Here -d: splits on colons and -f1 prints the first field — the usernames from the password file. -f1,3 would print fields 1 and 3; -f1-3 prints fields 1 through 3.

Given the file below, what will stdout contain? Write your prediction. Then run the command to check your answer.
Type your guess here, then run the command to compare…
📄
file · /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
alice:x:1000:1000:Alice:/home/alice:/bin/bash

▶
cut -d: -f1 /etc/passwd
Write your prediction above, then press Run to reveal the output.
Common mistake — cut -d ' ' on whitespace-separated data
The student wants the second "column" of a space-separated log file. But cut treats every single space as a delimiter. When fields are separated by runs of spaces, there are empty fields between them — so -f 2 is the empty string for most lines. Use awk '{print $2}' instead: awk collapses runs of whitespace by default.

The student expects INFO, ERROR, INFO — one level per line. Do you agree, or will something else happen? Write your prediction.
Type your guess here, then run the command to compare…
📄
file · log.txt
08:00  INFO  start
08:01  ERROR disk
08:02  INFO  retry

▶
cut -d ' ' -f 2 log.txt
Write your prediction above, then press Run to reveal the output.
comm — compare two sorted files
Compares two sorted files line by line and prints three columns: lines unique to the first file, lines unique to the second file, lines in both. Use flags to suppress columns: -1 hides column 1, -12 hides both unique columns (leaving only common lines). The files must be sorted; otherwise the output is meaningless.

📄
file · a.txt
alice
bob
charlie
📄
file · b.txt
alice
charlie
dave

▶
comm a.txt b.txt
Press Run to see what the command produces.
head — print the first N lines
Prints the first N lines of each input (default 10). -n 3 gives just the first three. Pair it with tail to grab a slice out of the middle of a file: head -n 20 file | tail -n 5 yields lines 16–20.

📄
file · log.txt
08:00 INFO  start
08:01 ERROR disk full
08:02 INFO  retry
08:03 ERROR timeout
08:04 INFO  done

▶
head -n 3 log.txt
Press Run to see what the command produces.
tail — print the last N lines
Prints the last N lines (default 10). -f follows the file as new lines are appended (the canonical way to watch a live log). tail -n +K prints from line K onward, which is the other way to stream a file-tail.

📄
file · log.txt
08:00 INFO  start
08:01 ERROR disk full
08:02 INFO  retry
08:03 ERROR timeout
08:04 INFO  done

▶
tail -n 2 log.txt
Press Run to see what the command produces.
awk — field-aware text processing
awk runs its program once per input line. $1, $2, … are the whitespace-separated fields (collapsing runs of whitespace by default, unlike cut -d ' '). This program swaps field order and prints them separated by a space. -F: changes the field separator.

{print $2, $1} prints the second field, then the first, on every line. What does stdout look like?
Type your guess here, then run the command to compare…
📄
file · names.txt
alice Lovelace
ada King
grace Hopper

▶
awk '{print $2, $1}' names.txt
Write your prediction above, then press Run to reveal the output.
3. Permissions, Environment, and Documentation
These tools manage how your shell operates and how you access information:

man: Access the manual pages for other commands. This is arguably the most useful command, providing built-in documentation for every other command in the system.
chmod: Change file mode bits (permissions). Files in a Unix-like system have three primary types of permissions: read (r), write (w), and execute (x). For security reasons, the system requires an explicit execute permission because you do not want to accidentally run a file from an unknown source. Permissions are often read in “bits” for the owner (u), group (g), and others (o).
which / type: Locate the binary or type for a command.
export: Set environment variables. The PATH variable is especially important; it tells the shell which directories to search for executable programs. You can temporarily update it using export or make it permanent by adding the command to your ~/.bashrc or ~/.profile file.
source / .: Execute commands from a file in the current shell environment.
chmod — add execute permission
chmod +x grants execute permission so the shell will actually run the file when you type ./deploy.sh. With a default umask 0022 (typical on most Linux systems), +x adds the bit for owner, group, and others; on a stricter umask such as 0077, only the categories the umask permits get the bit. Use chmod a+x to explicitly target everyone, or chmod u+x to target only the owner. Without an execute bit, the shell rejects the file with Permission denied.

📄
file · deploy.sh
#!/bin/bash
echo "deploying…"
mode before: -rw-r--r-- (644)

▶
chmod +x deploy.sh
Press Run to see what the command produces.
Common mistake — running a script without chmod +x (exit code 126)
A classic "it should work" moment: the script exists, the contents are correct, but you never ran chmod +x on it. The shell finds the file but refuses to execute it, returning exit code 126 — a POSIX-specific code meaning "command found but not executable". Distinct from 127 (command not found entirely).

📄
file · deploy.sh
#!/bin/bash
echo "deploying…"
mode: -rw-r--r-- (644 — no execute bit)

▶
./deploy.sh
Press Run to see what the command produces.
Common mistake — chmod 777 as a security shortcut
777 grants read, write, and execute to everyone on the system. Students reach for it when they see "Permission denied" and want to make the problem go away — but on a shared or networked machine this is an open invitation for any other user or process to read and modify the file. The right answer is almost never 777. Use 644 for regular files, 755 for executables and directories, 600 for secrets.

📄
file · secrets.txt
API_TOKEN=sk-abc123
mode before: -rw------- (600)

▶
chmod 777 secrets.txt
Press Run to see what the command produces.
which — locate a command’s binary
which searches $PATH for the named command and prints the full path of the first match. Use it to confirm which copy of a tool you're actually running when multiple versions exist. For shell builtins and aliases use type -a instead — which only finds on-disk binaries.

⚙
environment
PATH
=
/home/alice/.local/bin:/usr/local/bin:/usr/bin:/bin

▶
which python3
Press Run to see what the command produces.
Common mistake — command not found (exit code 127)
Every developer has fat-fingered this one. The shell scans every directory in $PATH for gti, doesn't find it (the student meant git), and bails with exit code 127 — the universal "command not found" signal. Scripts use the same exit code to detect "an expected tool isn't installed" and fall back or fail loudly.

⚙
environment
PATH
=
/usr/local/bin:/usr/bin:/bin

▶
gti status
Press Run to see what the command produces.
export — set an environment variable for child processes
export makes a variable visible to child processes — scripts and programs you launch. Setting VAR=value without export only makes the variable visible to the current shell. Here we append /opt/tools/bin to $PATH so the shell finds binaries in that directory. The change lasts until you close this shell; put it in ~/.bashrc to make it permanent.

⚙
environment
PATH
=
/usr/local/bin:/usr/bin:/bin

▶
export PATH=$PATH:/opt/tools/bin
Press Run to see what the command produces.
source — run a script in the current shell
source env.sh (or . env.sh) runs the script's commands in the current shell, so any variable assignments it makes persist. If you had instead run ./env.sh, the script would run in a child shell and its assignments would disappear the moment it exits. This is how tools like nvm, pyenv, and virtualenvs activate themselves.

⚙
environment
API_URL
=
(unset)
📄
file · env.sh
export API_URL=https://api.example.com
export DEBUG=1

▶
source env.sh
Press Run to see what the command produces.
4. System, Networking, and Build Tools
Tools used for remote work, debugging, and automating the construction process:

ssh: Secure shell to connect to remote machines like SEASnet.
scp: Securely copy files between hosts.
wget / curl: Download files or data from the internet.
make: Build automation tool that uses shell-like syntax to manage the incremental build process of complex software, ensuring that only changed files are recompiled.
gcc / clang: C/C++ compilers.
tar: Manipulate tape archives (compressing/decompressing).
The Power of I/O Redirection and Piping
The true power of the shell comes from connecting commands. Every shell program typically has three standard stream ports:

Standard Input (stdin / 0): Usually the keyboard.
Standard Output (stdout / 1): Usually the terminal screen.
Standard Error (stderr / 2): Where error messages go, also usually the terminal.
Redirection
You can redirect these streams using special operators:

>: Redirects stdout to a file, overwriting it. (e.g., echo "Hello" > file.txt)
>>: Redirects stdout to a file, appending to it without overwriting.
<: Redirects stdin from a file. (e.g., cat < input.txt)
2>: Redirects stderr to a specific file to specifically log errors.
2>&1: Redirects stderr to the standard output stream. Note: order matters — command > file.txt 2>&1 sends both streams to the file, whereas command 2>&1 > file.txt only redirects stdout to the file while stderr still goes to the terminal.
> — redirect stdout to a file (overwrite)
> captures the command's stdout and writes it to greeting.txt, overwriting whatever was there. The shell opens the destination file before the command runs, which is why > always truncates the target. Note: nothing appears on stdout in the terminal anymore — it all went to the file.

📄
file · greeting.txt
old contents

▶
echo hello > greeting.txt
Press Run to see what the command produces.
Common mistake — > silently clobbers existing data
The student wants to sort a file in place. But the shell opens data.txt for writing (truncating it) before sort runs — so sort reads an empty file, produces no output, and data.txt is now empty. This is one of the most painful shell beginners-traps; many hours of work have been lost this way. Use an intermediate file: sort data.txt > sorted.tmp && mv sorted.tmp data.txt — or sort -o data.txt data.txt (sort has a safe in-place mode).

Before running: what do you think data.txt will contain afterwards — the sorted content, or something else?
Type your guess here, then run the command to compare…
📄
file · data.txt
charlie
alice
bob

▶
sort data.txt > data.txt
Write your prediction above, then press Run to reveal the output.
>> — redirect stdout and append
>> appends stdout to the file rather than truncating it. Always the right choice for log files or any destination whose existing content you care about. > would delete everything previously in log.txt.

📄
file · log.txt
08:00 start
08:01 ready

▶
echo "new entry" >> log.txt
Press Run to see what the command produces.
2> — redirect stderr to a separate file
Stdout and stderr are two separate streams — sending one to a file does nothing to the other. Here stdout still shows the matches; any error message (e.g. missing file) would have been captured in errors.log instead. The 2 is the file descriptor for stderr; 1 is stdout.

📄
file · log.txt
08:01 ERROR disk full
08:02 INFO retry
08:03 ERROR timeout

▶
grep ERROR log.txt 2> errors.log
Press Run to see what the command produces.
Common mistake — redirection order: 2>&1 > file vs > file 2>&1
The student wants both stdout and stderr captured into build.log. But redirections are applied left-to-right: 2>&1 first makes stderr a duplicate of the current stdout (which is still the terminal), and only then does > build.log redirect stdout to the file. Result: stdout goes to the file, but stderr still goes to the terminal. The correct form is build.sh > build.log 2>&1 (set stdout first, then dup stderr from stdout).

📄
file · build.sh
#!/bin/bash
echo "compiling main.c"
>&2 echo "warning: unused variable"
exit 0

▶
build.sh 2>&1 > build.log
Press Run to see what the command produces.
Piping
The pipe operator | is the most powerful composition tool. It takes the stdout of the command on the left and sends it directly into the stdin for the command on the right.

Example: cat access.log | grep "ERROR" | wc -l This pipeline reads a log file, filters only the lines containing “ERROR”, and then counts how many lines there are.

Pipe | — composing commands
A pipe takes the stdout of the left command and wires it directly into the stdin of the right command. No temporary files, no shared memory — just a stream of bytes. This is the single most powerful idea in the UNIX philosophy: small tools compose into pipelines. Here we filter the log for error lines and then count them.

Two commands, one pipe. What number appears on stdout?
Type your guess here, then run the command to compare…
📄
file · log.txt
08:00 INFO  start
08:01 ERROR disk full
08:02 INFO  retry
08:03 ERROR timeout
08:04 INFO  done

▶
grep ERROR log.txt | wc -l
Write your prediction above, then press Run to reveal the output.
Here Documents and Here Strings
Sometimes you need to feed a block of text directly into a command without creating a temporary file. A here document (<<) lets you embed multi-line input inline, up to a chosen delimiter:

cat <<EOF
Server: production
Version: 1.4.2
Status: running
EOF
The shell expands variables inside the block (just like double quotes). To suppress expansion, quote the delimiter: <<'EOF'.

A here string (<<<) feeds a single expanded string to a command’s standard input — a concise alternative to echo "text" | command:

grep "ERROR" <<< "08:15:45 ERROR failed to connect"
Process Substitution
Advanced shell users often utilize process substitution to treat the output of a command as a file. The syntax looks like <(command). For example, H < <(G) >> I allows you to refer to the standard output of command G as a file, redirect it into the standard input of H, and append the output to I.

Writing Your First Shell Script
When you find yourself typing the same commands repeatedly, you should create a shell script. A shell script is written in a plain text file (often ending in .sh) and contains a sequence of commands that the shell executes as a program.

Interpreted Nature
Unlike a compiled language like C++, which is compiled into machine code before execution, shell scripts are interpreted at runtime rather than ahead of time. This allows for rapid prototyping. Bash always reads at least one complete line of input, and reads all lines that make up a compound command (such as an if block or for loop) before executing any of them. This means a syntax error on a later line inside a multi-line compound block is caught before the block starts executing — but an error in a branch that is never reached at runtime may go unnoticed. Use bash -n script.sh to check for syntax errors without running the script.

The Shebang
Every script should start with a “shebang” (#!). This tells the operating system which interpreter should be used to run the script. For Bash scripts, the first line should be:

#!/bin/bash
Execution Permissions
By default, text files are not executable for security reasons. Execute permission is required only if you want to run the script directly as a command:

chmod +x myscript.sh
./myscript.sh
Alternatively, you can bypass the execute-permission requirement entirely by passing the file as an argument to the Bash interpreter directly — no chmod needed:

bash myscript.sh
You can also run a script’s commands within the current shell (inheriting and potentially modifying its environment) using source or the . builtin: source myscript.sh.

Debugging Scripts
When a script behaves unexpectedly, Bash has built-in tracing modes that let you see exactly what the shell is doing:

bash -n script.sh: Reads the script and checks for syntax errors without executing any commands. Always run this first when a script refuses to start.
bash -x script.sh (or set -x inside the script): Prints a trace of each command and its expanded arguments to stderr before executing it — indispensable for logic bugs. Each traced line is prefixed with +.
bash -v script.sh (or set -v): Prints each line of input exactly as read, before expansion — useful for seeing the raw source being interpreted.
You can combine flags: bash -xv script.sh. To turn tracing on for only a section of a script, use set -x before that section and set +x after it.

Error Handling (set -e and Exit Status)
By default, a Bash script will continue executing even if a command fails. Every command returns a numerical code known as an Exit Status; 0 generally indicates success, while any non-zero value indicates an error or failure. Continuing after a failure can be dangerous and lead to unexpected behavior. To prevent this, you should typically include set -e at the top of your scripts:

#!/bin/bash
set -e
This tells the shell to exit immediately if any simple command fails, making your scripts safer and more predictable.

Work through each script in your head first — predict what reaches stdout before pressing Run. Each echo call below prints on its own line, so the number of lines on stdout tells you exactly how many echo statements ran. The output literally stops where execution stopped. The comparison panel will tell you if you got it; if not, the Notice below will explain why.

Lab 1 — set -e before vs. after
This script runs false twice — once before set -e is enabled, and once after. false is a tiny program whose only job is to exit with status 1, so it's a convenient stand-in for "a command that failed". Trace the script line by line and commit to a prediction.

Which echo statements reach stdout, and in what order? (Each one prints on its own line.)
Type your guess here, then run the command to compare…
📄
file · demo.sh
#!/bin/bash
echo "A"
false
echo "B"
set -e
echo "C"
false
echo "D"

▶
bash demo.sh
Write your prediction above, then press Run to reveal the output.
Lab 2 — set -e is suppressed inside && and ||
A critical subtlety: Bash does not let set -e fire when a failing command is part of an && or || chain, because its exit status is being tested. Only a bare failing command kills the script. Contrast the two patterns below — one false && … chain and one bare false — and predict exactly what reaches stdout.

set -e is active throughout. Which echo statements reach stdout, and in what order?
Type your guess here, then run the command to compare…
📄
file · chain.sh
#!/bin/bash
set -e
echo "A"
false && echo "skip"
echo "B"
false || echo "rescue"
echo "C"
false
echo "D"

▶
bash chain.sh
Write your prediction above, then press Run to reveal the output.
Lab 3 — Synthesis: functions, set -e, ||, && — all at once
A synthesis of everything from the first two labs — a function with its own return 1, set -e, and a mix of || rescues, && chains, and bare commands. Trace line by line. Somewhere in the script set -e fires and the rest is unreachable — finding where is the whole puzzle.

What appears on stdout, line by line? (The function prints Never gonna every time it's called, not each time its return value is used.)
Type your guess here, then run the command to compare…
📄
file · song.sh
#!/bin/bash
never_gonna() {
    echo "Never gonna"
    return 1
}
never_gonna
false
set -e
false || true
true || echo "give you"
false || echo "up."
never_gonna || true
echo "let you" && false
echo "down."

▶
bash song.sh
Write your prediction above, then press Run to reveal the output.
Syntax and Programming Constructs
Bash is a full-fledged programming language, but because it is an interpreted scripting language rather than a compiled language (like C++ or Java), its syntax and scoping rules are quite different.

5. Scripting Constructs
In our scripts, we also treat these keywords as “commands” for building logic:

#! (Shebang): An OS-level interpreter directive on the first line of a script file — not a Bash keyword or command. When the OS executes the file, it reads #! and uses the rest of that line as the interpreter path. Within Bash itself, any line starting with # is simply a comment and is ignored.
read: Read a line from standard input into a variable. Common flags: -p "prompt" displays a prompt on the same line, -s silently hides typed input (useful for passwords), and -n 1 returns after exactly one character instead of waiting for Enter.
if / then / elif / else / fi: Conditional execution.
for / do / done / while: Looping constructs.
case / in / esac: Multi-way branching on a single value.
local: Declare a variable scoped to the current function.
return: Exit a function with a numeric status code.
exit: Terminate the script with a specific status code.
read — read a line of stdin into a variable
read consumes one line from stdin (up to the next newline) and assigns it to the named variable. -p prints a prompt (to stderr, not stdout) before reading. Here the user types Ada and presses Enter; the second command prints a greeting. Always pair read with -r when reading arbitrary text — without it, backslashes get interpreted as line-continuation.

←
stdin
Ada

▶
read -p "Name: " name; echo "Hi, $name"
Press Run to see what the command produces.
Variables
You can assign values to variables without declaring a type. Note that there are no spaces around the equals sign in Bash.

NAME="Ada"
echo "Hello, $NAME"
Parameter Expansion — Default Values and String Manipulation
Beyond simple $VAR substitution, Bash supports a powerful set of parameter expansion operators that let you handle missing values and manipulate strings entirely within the shell, without spawning external tools.

Default values:

# Use "server_log.txt" if $1 is unset or empty
file="${1:-server_log.txt}"

# Use "anonymous" if $NAME is unset or empty, AND assign it
NAME="${NAME:=anonymous}"
String trimming — remove a pattern from the start (#) or end (%) of a value:

path="/home/user/project/main.sh"
filename="${path##*/}"    # removes longest prefix up to last /  → "main.sh"
noext="${filename%.*}"    # removes shortest suffix from last .  → "main"
The double form (## / %%) removes the longest match; the single form (# / %) removes the shortest.

Search and replace:

msg="Hello World World"
echo "${msg/World/Earth}"    # replaces first match  → "Hello Earth World"
echo "${msg//World/Earth}"   # replaces all matches  → "Hello Earth Earth"
Scope Differences
Unlike C++ or Java, Bash lacks strict block-level scoping (like {} blocks). Variables assigned anywhere in a script — including inside if statements and loops — remain accessible throughout the entire script’s global scope. There are, however, several important isolation boundaries:

Function-level scoping: variables declared with the local builtin inside a Bash function are visible only to that function and its callees.
Subshells: commands grouped with ( list ), command substitutions $(...), and background jobs run in a subshell — a copy of the shell environment. Any variable assignments made inside a subshell do not propagate back to the parent shell.
Per-command environment: a variable assignment placed immediately before a simple command (e.g., VAR=value command) is only visible to that command for its duration, leaving the surrounding scope untouched.
Arithmetic
Math in Bash is slightly idiosyncratic. While a language like C++ operates directly on integers with + or /, arithmetic in Bash needs to be enclosed within $(( ... )) or evaluated using the let command.

x=5
y=10
sum=$((x + y))
echo "The sum is $sum"
Control Structures: If-Statements and Loops
Bash supports standard control flow constructs.

If-Statements:

if [ "$sum" -gt 10 ]; then
    echo "Sum is greater than 10"
elif [ "$sum" -eq 10 ]; then
    echo "Sum is exactly 10"
else
    echo "Sum is less than 10"
fi
[ is a shell builtin command: The single bracket [ is not special syntax — it is a builtin command, a synonym for test. Because Bash implements it internally, its arguments must be separated by spaces just like any other command: [ -f "$file" ] is correct, but [-f "$file"] tries to run a command named [-f, which fails. This is why the spaces inside brackets are mandatory, not just stylistic. (An external binary /usr/bin/[ also exists on most systems, but Bash uses its builtin by default — you can verify with type -a [.)

The following table covers the most important tests available inside [ ]:

Basics table
Test	Meaning
-f path	Path exists and is a regular file
-d path	Path exists and is a directory
-z "$var"	String is empty (zero length)
"$a" = "$b"	Strings are equal
"$a" != "$b"	Strings are not equal
$x -eq $y	Integers are equal
$x -gt $y	Integer greater than
$x -lt $y	Integer less than
! condition	Logical NOT (negates the test)
Important: use -eq, -lt, -gt for numbers and = / != for strings. Mixing them produces wrong results silently.

[ vs [[: The double bracket [[ ... ]] is a Bash keyword with additional power: it does not perform word splitting on variables, allows && and || inside the condition, and supports regex matching with =~. Prefer [[ ]] in new Bash scripts.

Loops:

for i in 1 2 3 4 5; do
    echo "Iteration $i"
done
For numeric ranges, the C-style for loop (the arithmetic for command) is often cleaner:

for (( i=1; i<=5; i++ )); do
    echo "Iteration $i"
done
This is a distinct looping construct from the standalone (( )) arithmetic compound command. In this form, expr1 is evaluated once at start, expr2 is tested before each iteration (loop runs while non-zero), and expr3 is evaluated after each iteration — the same semantics as C’s for loop.

Loop control keywords:

break: Exit the loop immediately, regardless of the remaining iterations.
continue: Skip the rest of the current iteration and jump to the next one.
for f in *.log; do
    [ -s "$f" ] || continue    # skip empty files
    grep -q "ERROR" "$f" || continue
    echo "Errors found in: $f"
done
Quoting and Word Splitting
How you quote text profoundly changes how Bash interprets it — this is one of the most common sources of bugs in shell scripts.

Single quotes ('...'): All characters are literal. No variable or command substitution occurs. echo 'Cost: $5' prints exactly Cost: $5.
Double quotes ("..."): Spaces are preserved, but $VARIABLE and $(command) are still expanded. echo "Hello $USER" prints Hello Ada.
A critical pitfall is word splitting: when you reference an unquoted variable, the shell splits its value on whitespace and treats each word as a separate argument. Consider:

FILE="my report.pdf"
rm $FILE      # WRONG: shell splits into two args: "my" and "report.pdf"
rm "$FILE"    # CORRECT: the entire value is passed as one argument
Always quote variable references with double quotes to protect against word splitting.

Command Substitution
Command substitution captures the standard output of a command and uses it as a value in-place. The modern syntax is $(command):

TODAY=$(date +%Y-%m-%d)
echo "Backup started on: $TODAY"
The shell runs the inner command in a subshell, then replaces the entire $(...) expression with its output. This is the standard way to assign the results of commands to variables.

Positional Parameters and Special Variables
Scripts receive command-line arguments via positional parameters. If you run ./backup.sh /src /dest, then inside the script:

Basics table
Variable	Value	Description
$0	./backup.sh	Name of the script itself
$1	/src	First argument
$2	/dest	Second argument
$#	2	Total number of arguments passed
$@	/src /dest	All arguments — when written as "$@", expands to one separately-quoted word per argument (preserving spaces inside arguments)
$?	(exit code)	Exit status of the most recent command
When iterating over all arguments, always use "$@" (quoted). Without quotes, $@ is subject to word splitting and arguments containing spaces are silently broken into multiple words:

for f in "$@"; do
    echo "Processing: $f"
done
Command Chaining with && and ||
Because every command returns an exit status, you can chain commands conditionally without writing a full if/then/fi block:

&& (AND): The right-hand command runs only if the left-hand command succeeds (exit code 0). mkdir output && echo "Directory created" — only prints if mkdir succeeded.
|| (OR): The right-hand command runs only if the left-hand command fails (non-zero exit code). cd /target || exit 1 — exits the script immediately if the directory cannot be entered.
This compact chaining idiom is widely used in professional scripts for concise, readable error handling.

Background Jobs
Appending & to a command runs it asynchronously — the shell launches it in the background and immediately returns to the prompt without waiting for it to finish:

./long_running_build.sh &
echo "Build started, continuing with other work..."
Two special variables are useful when managing background processes:

$$: The process ID (PID) of the current shell process. Bash deliberately does not update $$ inside subshells (( … ), $(…), pipelines), so it remains a stable identifier — useful for unique temporary file names: tmp_file="/tmp/myscript.$$". The actual PID of a subshell is exposed in $BASHPID.
$!: The PID of the most recently backgrounded job. Use it to wait for or kill a specific background process.
The jobs command lists all active background jobs; fg brings the most recent one back to the foreground, and bg resumes a stopped job in the background.

Functions — Reusable Building Blocks
When the same logic appears in multiple places, extract it into a function. Functions in Bash work like small scripts-within-a-script: they accept positional arguments via $1, $2, etc. — independently of the outer script’s own arguments — and can be called just like any other command.

greet() {
    local name="$1"
    echo "Hello, ${name}!"
}

greet "engineer"   # → Hello, engineer!
The local Keyword
Without local, any variable set inside a function leaks into and overwrites the global script scope. Always declare function-internal variables with local to prevent subtle bugs:

process() {
    local result="$1"   # visible only inside this function
    echo "$result"
}
Returning Values from Functions
The return statement only carries a numeric exit code (0–255), not data. To pass a string back to the caller, have the function echo the value and capture it with command substitution:

to_upper() {
    echo "$1" | tr '[:lower:]' '[:upper:]'
}

loud=$(to_upper "hello")   # loud="HELLO"
You can also use functions directly in if statements, because a function’s exit code is treated as its truth value: return 0 is success (true), return 1 is failure (false).

Case Statements — Readable Multi-Way Branching
When you need to check one variable against many possible values, a case statement is far cleaner than a chain of if/elif:

case "$command" in
    start)   echo "Starting service..."  ;;
    stop)    echo "Stopping service..."  ;;
    status)  echo "Checking status..."   ;;
    *)       echo "Unknown command: $command" >&2; exit 2 ;;
esac
Each branch ends with ;;. The * pattern is the catch-all default, matching any value not handled by earlier branches. The block closes with esac (case backwards).

Exit Codes — The Language of Success and Failure
Every command — including your own scripts — exits with a number. 0 always means success; any non-zero value means failure. This is the opposite of most programming languages where 0 is falsy. Conventional exit codes are:

Basics table
Code	Meaning
0	Success
1	General error
2	Misuse — wrong arguments or invalid input
Meaningful exit codes make scripts composable: other scripts, CI pipelines, and tools like make can call your script and take action based on the result. For example, ./monitor.sh || alert_team only triggers the alert when your monitor exits non-zero.

Shell Expansions — Brace Expansion and Globbing
The shell performs several rounds of expansion on a command line before executing it. Understanding the order helps you predict and control what the shell does.

Brace Expansion
First comes brace expansion, which generates arbitrary lists of strings. It is a purely textual operation — no files need to exist:

mkdir project/{src,tests,docs}      # creates three directories at once
cp config.yml config.yml.{bak,old}  # copies to two names simultaneously
echo {1..5}                          # → 1 2 3 4 5  (sequence expression)
Brace expansion happens before all other expansions. Because of this, you cannot use a variable to drive the range ({$a..$b} does not work), but you can freely combine the result of brace expansion with variables and globbing in the surrounding text (e.g., cp $f.{bak,old}).

Supercharging Scripts with Regular Expressions
Because the UNIX philosophy is heavily centered around text streams, text processing is a massive part of shell scripting. Regular Expressions (RegEx) is a vital tool used within shell commands like grep, sed, and awk to find, validate, or transform text patterns quickly.

Globbing vs. Regular Expressions: These look similar but are entirely different systems. Globbing (filename expansion) uses *, ?, and [...] to match filenames — the shell expands these before the command runs (e.g., rm *.log deletes all .log files). The three special pattern characters are: * matches any string (including empty), ? matches any single character, and [ opens a bracket expression [...] that matches any one of the enclosed characters — e.g., [a-z] matches any lowercase letter, and [!a-z] matches any character that is not a lowercase letter. Regular Expressions use ^, $, .*, [0-9]+, and similar constructs — they are pattern languages used by tools like grep, sed, and awk, and also natively by Bash itself via the =~ operator inside [[ ]] conditionals (which evaluates POSIX extended regular expressions directly without spawning an external tool). Critically, * means “match anything” in globbing, but “zero or more of the preceding character” in RegEx.

RegEx allows you to match sub-strings in a longer sequence. Critical to this are anchors, which constrain matches based on their location:

^ : Start of string. (Does not allow any other characters to come before).
$ : End of string.
Example: ^[a-zA-Z0-9]{8,}$ validates a password that is strictly alphanumeric and at least 8 characters long, from the exact beginning of the string to the exact end.

Conclusion
Shell scripting is an indispensable skill for anyone working in tech. By viewing the shell as a set of modular tools (the “Infinity Stones” of your development environment), you can combine simple operations to perform massive, complex tasks with minimal effort. Start small by automating a daily chore on your machine, and before you know it, you will be weaving complex UNIX tools together with ease!

Practice
Shell Commands — What Does It Do?
Match each shell command to its purpose

Difficulty:
Basic
What does cat do?

Show Answer
Shell Commands Flashcards
Which Shell command would you use for the following scenarios?

Difficulty:
Basic
You want to quickly view the entire contents of a small text file named ‘config.txt’ printed directly to your terminal screen.

Show Answer
Shell Pipelines
Practice connecting UNIX commands together with pipes to solve real tasks.

Difficulty:
Intermediate
List all running processes and show only those belonging to user tobias.

Show Answer
Shell Scripting & UNIX Philosophy Quiz
Test your conceptual understanding of shell environments, data streams, and scripting paradigms beyond basic command memorization.

Difficulty:
Advanced
A C++ developer writes a Bash script with a for loop. Inside the loop, they declare a variable temp_val. After the loop finishes, they try to print temp_val expecting it to be undefined or empty, but it prints the last value assigned in the loop. Why did this happen?


A
The script was executed with sudo privileges, which ignores standard scoping rules.


B
The developer forgot to use the let keyword when declaring the variable.


C
Bash automatically exports all variables to the global environment by default.


D
Bash has no block-level scoping; loop and if variables leak afterward.

Shell Script Parsons Problems
Arrange shell-pipeline fragments to filter, sort, count, and combine log and config files.

Difficulty:
Intermediate
Arrange the fragments to combine two log files and display every unique line in sorted order.

Drag fragments into the answer area in the correct order (some items are distractors that should not be used). Keyboard: focus a line and press Space or Enter to move it between the bank and the answer area. Use Arrow Up or Arrow Down to reorder within the answer area.
→ Drop here →
Check Order
Reset



---

# Appendix: Lecture 2 Slides (raw extracted text)

The following is the full extracted text of Lecture 2 (L2-ShellScripting.pdf). Preserved verbatim so no information is lost. Some formatting artifacts from PDF column extraction remain.

```text
CS 35L Software
Construction
L2 � Files & Shell
Scripting
Assistant Teaching Professor
Computer Science Department
Course Logistics
� Homework 1 will be released today. Due in one week
   � Writing User Stories & Justifying INVEST
   � Creating a shell script on SEASNet
       � If you have not requested an account yet, do so today
� Discussions on Friday
   � Provide Hands-on Experience
   � Give you a head start on the Homework
Discussion Worksheet  No big deal to
Bonus Points            miss a few
                         Only regular
                      submission earns
                         many points
Different Course Resources are Designed for
Different Students. Use them If these Apply to You
� If a topic is unclear to you after the lecture
   � Read the SE Book page on the topic
   � Ask on Piazza if you believe others might have the same question
   � Come to TA/LA/Prof office hours We're eager to support you
� If you want additional practice
   � Go to discussions
   � Talk to the digital tutors . They will give you practice problems & feedback
� If you already know everything and you're bored by the class
   � Come to Prof office hours . Read SE Book pages and further readings
How to use Course Resources?
� Always walk through the tutorials we link!
   � The discussions will give you a head start
   � Then you should finish them at home
� If you are in this class to learn and/or get a good grade (hopefully everyone)
   � Regularly Practice the quiz & flashcard questions in the SE Gym
A Good Software Engineer has a Great
Understanding of Existing Tools
 Today's lectures teaches you
how to use common UNIX tools
Operating Systems offer Useful Functionality
Avoid this       Your Software       abstracts away details of
if possible                                  the hardware
              Operating System
             (e.g., Linux, Windows,                   Legend
                MacOS, Android)
                  Hardware                                         Component
                                                                          Call
UNIX offers you Many Useful Tools
� UNIX is a family of multitasking, multi-user computer operating systems.
� Arguably the most influential operating system of all time, serving as the
  foundation or inspiration for systems like Linux, macOS, and Android
� Unix design philosophy:
   � "Write programs that do one thing and do it well."
   � "Write programs to work together."
   � "Write programs to handle text streams, because that is a universal interface."
UNIX Commands for File Handling
� cd <dir> (change directory)             Please familiarize yourself
� ls (list files)                            with these commands
� mkdir <dir> (make directory)
� cp <source> <destination>: (copy file)
� mv <source> <destination>: (move/rename file)
� rm <file>: delete file
� less <file>: view file contents one screen at a time
� cat <file1> <file2>: Concatenate files and print their content to standard
  output. Often used to quickly view a small file
UNIX Commands for Text Processing
� sed (stream editor) used for filtering and  Please familiarize yourself
  transforming text, most commonly for           with these commands
  search-and-replace operations
� grep (global regular expressions) find and replace
� head/tail <file>: prints the first / last lines of a file
� wc <file>: word count (counts characters, words, lines)
� sort: sorts lines in a file
� cut: Extracts characters / fields / bytes at specific positions / delimiters / offsets
Other Frequently Used UNIX Commands
� ssh <remote>: secure shell.            Please familiarize yourself
  Used connect to a remote computer         with these commands
� htop: shows system monitoring tools
� man <command>: manual for a command
� pwd (print working directory) checks your current path
� chmod: (change mode). Sets the permissions for a file
� Files have three types of permission:
� READ (r)
� WRITE (w)
� EXECUTE (x)
"Write programs to handle text streams, because
that is a universal interface." � UNIX Philosophy
� To handle inputs and output, UNIX programs have:                                       Legend
   � a standard input stream port (stdin)
                                                                                        Program
   � a standard output stream port (stdout) These are
   � a standard error stream port (stderr) text streams                                     Port
                                                                                           Pipe
� Additional inputs can be added via command line arguments
                                                                                                           12
� Composition: By connecting ports of multiple commands,
  you can create a larger program
cat stdout stdin stdout stdin
file.txt  sort  head
cat file.txt | sort | head
                     CS 35L Software Construction: Lecture 2 � Files & Shell Scripting
                     Tobias D�rschmid
Streams are sequences of characters / bytes
                  cmd1 | cmd2
cmd1                                                               cmd2
      stdout stdin
      Streams are like conveyor belts, not like trucks
Streams transport portions of data individually rather than all at once
+ Allows for parallel processing  - Can result in indefinite waiting
                                                     for stream to close
I/O (Input & Output) Streams
� Reading from a file: cmd < file
   � Reads the file and forwards it to the standard input (stdin) of the cmd
� Writing to file: cmd > file or cmd 1> file
   � Writes the content of the standard output (stdout) stream to the file
� Logging errors to a file: cmd 2> file
   � Writes the content of the standard error (stderr) stream to the file
� Forwarding stdout of cmd1 to stdin of cmd2: cmd1 | cmd2
� Redirecting stderr to stout: cmd 2 >& 1
� Closing the stdin stream: cmd <&-
"Write programs to handle text streams, because
that is a universal interface." � UNIX Philosophy
         Are text streams really a universal interface?
                    Discuss with your neighbor(s)
cat stdout stdin stdout stdin
file.txt          sort                                             head
cat file.txt | sort | head
Case Study: Nametags
What happened here?
ASCII (American Standard Code for Information
Interchange)
� ASCII encodes each character as a value from 0 to 127 � storable as a seven-
  bit integer
non-printing codes         control characters  printable characters
primarily designed to
control peripheral
devices (like printers or
terminals) or to provide
meta-information about
a data stream
Read more here: https://www.ascii-code.com/
There are Several Different Variations
of ASCII Extensions to 8-bit
� Windows Code Pages aka. ANSI Code Pages (e.g., Windows-1252)
   � Standard for Western European languages in Microsoft Windows
   � Includes �, �, �, �, ...
� ISO 8859 Series (e.g., ISO 8859-1 for Latin-1, ISO 8859-2 for Latin-2)
   � Offers different extension parts for several languages
     (e.g., Greek, Arabic, Hebrew, Thai, ... )
Read more here: https://en.wikipedia.org/wiki/ISO/IEC_8859
UTF-8 (Unicode Transformation Format � 8-bit.)
� Unicode (aka The Unicode Standard) is designed to
  support all of the world's writing systems
  that can be digitized
� UTF-8 supports all 1,112,064 valid Unicode
  characters using variable-width encoding
  of one to four one-byte (8-bit) code units
� Emojis (e.g.,                    )
� Korean Alphabet (e.g.,)
� Chinese Characters (e.g., )
Read more here: https://home.unicode.org/
                 CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  19
                 Tobias D�rschmid
What exactly happened
here?
� in UTF-8 stored as bytes: (C3 BC)
In Latin-1 / Windows-1252:
          C3  �
          BC  �
Result: D�rschmid  Dürschmid
              UFT-8  Latin-1
Registration              Printing
System                      System
                     Why did all other                             Where does the ?
                     characters print correctly?                   come from?
The UNIX Shell lets you use UNIX commands
� A command-line interpreter that provides a command line user interface
� Typically, users interact with the UNIX shell using a terminal application
� Shell is the 6th most common tool used by practitioners
  (after JavaScript, HTML/CSS, Python, SQL, and TypeScript,
  most of which you will see in this course!)
 Continue this tutorial here: https://tobiasduerschmid.github.io/SEBook/tools/shell-tutorial
Shell Scripts
 Hello.sh  Tells the system to use the sh shell
#!/bin/sh  $1 reads the first command line argument
echo "Hello $1! Let's get scripting."
 Terminal                    Gives users the "read" and the
$chmod +x hello.sh                "execute" permissions
$./hello.sh students command line argument
Hello students! Let's get scripting
           CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  22
           Tobias D�rschmid
 Continue this tutorial here: https://tobiasduerschmid.github.io/SEBook/tools/shell-tutorial
Shell Scripts Support Variables
HelloVar.sh
#!/bin/sh
echo "What is your name?"
read USER_NAME      read stores the user input in
                       the USER_NAME variable
echo "Welcome, $USER_NAME! Let's get scripting."
Terminal         $USER_NAME gets the value of the
$ ./hellovar.sh                USER_NAME variable
What is your name?
Tobias
Welcome, Tobias! Let's get scripting.
             CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  23
             Tobias D�rschmid
 Continue this tutorial here: https://tobiasduerschmid.github.io/SEBook/tools/shell-tutorial
Shell Scripts Support For Loops
 for.sh
#!/bin/sh
for i in 1 2 3 4
do
   echo "Looping ... number $i"
done
Terminal
$ ./for.sh
Looping ... number 1
Looping ... number 2
Looping ... number 3
Looping ... number 4
          CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  24
          Tobias D�rschmid
 Continue this tutorial here: https://tobiasduerschmid.github.io/SEBook/tools/shell-tutorial
Shell Scripts Support If Then Else
 if.sh
#!/bin/sh
m=1
n=2
if [ $n -eq $m ]
then
              echo "Both variables are the same"
else
              echo "Both variables are different"
fi
 Continue this tutorial here: https://tobiasduerschmid.github.io/SEBook/tools/shell-tutorial
Shell Scripts Support Wildecards
 clean.sh  . Start the search in the current directory.
#!/bin/sh  -type f : Only match regular files (not directories or links).
           -iname "*.log": Match files ending in .log, ignoring case
           -delete : Permanently deletes the files found
find . -type f -iname "*.log" -delete
   $? holds the exit status (or return code) of the last executed command.
   0 means no errors, other numbers indicate the error code
if [ $? -eq 0 ]; then
         echo "Cleanup complete. All *.log files have been removed."
else
         echo "An error occurred during cleanup"
fi
           CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  26
           Tobias D�rschmid
Common Directories
�/bin: Common programs (short for binary)
�/dev: Peripheral hardware
�/etc: Configurations
�/home: Home directories of users
�/lib: Library files
�/mnt: Mount point for external FS
�/opt: Extra software
�/root: Home for root
�/tmp: Temporary files
�/usr: Files for user-related programs
�/var: Variable and temporary files
Paths
� Absolute path (starts from the root directory):
   � e.g., /usr/local/
� Relative path (to access files relative to your current working directoru):
   � ./ = current directory
   � ../ = parent directory
   � ~/ = home directory
� Use pwd (print working directory) to check your current path
The PATH Environment Variable
� The PATH environment variable contains an ordered list of directories that the
  operating system (like Windows, Linux, or macOS) searches for executable
  programs when a command is entered in a command line interface (CLI).
� If the CLI cannot find the program / file in your working directory, it will then
  search through the directories listed in the PATH.
� Add a new directory to the path via:
  export PATH="\$PATH:/path/to/your/new/directory"
   � This change will be temporary for your current session.
   � To make the change permanent change
     add it to the ~/.bashrc and/or ~/.zshrc files
Regular Expressions (RegEx)
Useful for:
� Data Validation: Checking if a user-entered email address or phone number is in
  the correct format.
� Text Processing: Extracting all URLs, dates, or specific identifiers from a large
  document.
� Search and Replace: Finding all instances of a misspelled word, regardless of
  capitalization, and replacing it.
� Log File Analysis: Searching through system logs for errors or specific events.
Literal Characters (The Simple Stuff )
Literals are characters that match themselves exactly
Pattern                    Matches
a                          "a"
123                        "123"
HeLLo                      "HeLLo"
         CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  31
         Tobias D�rschmid
Character Classes (Selecting one of many )
Character classes define a set of possible characters to be matched
Pattern Description                 Example                              Matches
[]    Character Set: Matches any [aeiou]                                 'a', 'e', 'i', 'o', or 'u'
      one of the characters inside
      the brackets
[]    Character Set: Matches any [0-4]                                   '0', '1', '2', '3', '4'
      one of the characters inside
      the brackets
[^ ]  Negated Set: Matches any [^0-9]                                    Any character that
                                                                         is NOT a digit.
      character NOT inside the
      brackets.
      CS 35L Software Construction: Lecture 2 � Files & Shell Scripting                              32
      Tobias D�rschmid
Meta Characters (To make your life easier )
Meta characters are short cuts for commonly used character classes
Pattern  Description
.        Matches any single character (except newline)
\d       Matches any digit (same as [0-9])
\D       Matches any non-digit (same as [^0-9])
\w       Any word character (same as [a-zA-Z0-9_]
\s       Any whitespace character (same as [ \t\r\n\f])
         CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  33
         Tobias D�rschmid
Quantifiers (How many times? )
Quantifiers describe how many occurrences of the preceding element
should be matched
Pattern Description                 Example Matches
X*    Matches X zero or more times  \s*     Any white spaces
X+    Matches X one or more times   \d+     Any number
X?    Matches X zero or one times   colou?r "color", "colour"
X{n}  Matches X exactly n times     \d{4}   Numbers with 4 digits
X{n,} Matches X n or more times     \w{5,}  Words with 5+ characters
X{n,m} Matches X n to m times       a{1,3}  "a", "aa", or "aaa"
      CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  34
      Tobias D�rschmid
Group Constructs (The Advanced Stuff )
Group constructs allow you to combine multiple characters or sub-
patterns into a single unit.
Pattern Description                Example         Matches
                                                   "ab", "abab", "ababab", ...
(...)    Simple group              (ab)*           "911", "9-1-1"
                                   (911|9-1-1)
(a | b)  Matches either a or                       "9a", "8z"
         b                         (?<num>[0-9])
                                   (?<char>[a-Z])            num char
(?<name>X) Gives group
                   matching X the
                   name `name'
         Helps with parsing files
         CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  35
         Tobias D�rschmid
Anchors
Regular expressions can match sub-strings in a longer sequence.
Anchors constrain matches based on their location inside the input
sequence
Pattern  Description
^        Start of string
         Does not allow any other characters to come before the ^
$        End of string
         CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  36
         Tobias D�rschmid
In-Class Exercise
Write a regular expression that can be used to validate a
password that must be at least 8 characters long and can
contain only letters and digits
^[a-zA-Z0-9]{8, }$
RegEx Reference: https://tobiasduerschmid.github.io/SEBook/tools/regex.html
In-Class Exercise
Write a regular expression that can be used to validate an
email address                         Dots outside of character
                                      classes need to be escaped
^[a-zA-Z0-9.-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
username can       domain can         top level
include letters,   include letters,   domain
digits, dots, and  digits, dots, and  can include
dashes.            dashes             letters
(e.g, cs-35L)      (e.g., ucla2025)
RegEx Reference: https://tobiasduerschmid.github.io/SEBook/tools/regex.html
Please Fill out your Exit Tickets on Bruin Learn!
Please briefly summarize the three key insights you have learned today. Do not copy
phrases from the lecture slides (and don't look back, answer purely from memory).
Which shell command goes into the folder "homework" and then lists all files
within this folder? Write it as a one-line command!
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slide use images from Flaticon.com (Creators: Freepik, Smashicons) and generated with Gemini
Additional Resources
� https://tobiasduerschmid.github.io/SEBook/tools/shell
� https://tobiasduerschmid.github.io/SEBook/tools/shell-tutorial
� https://tobiasduerschmid.github.io/SEBook/tools/regex
� https://tobiasduerschmid.github.io/SEBook/tools/regex-tutorial
� https://tobiasduerschmid.github.io/SEBook/tools/regex-tutorial-advanced
Summary
� The operating system abstract away hardware details
� UNIX commands offer you a large variety of useful tools
� The UNIX shell lets you write small programs that use UNIX commands
� Make sure you use correct character encoding when reading / writing files
� In most cases, UTF-8 is the most appropriate encoding to use
� RegEx is a useful tool for text processing
Credits: These slide use images from Flaticon.com (Creators: Freepik, Smashicons)

```
