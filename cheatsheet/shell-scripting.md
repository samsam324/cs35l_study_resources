# Shell Scripting Cheatsheet (candidate items)

## File navigation
- `pwd` print working dir
- `cd <dir>` change dir; `cd -` previous; `cd ~` home
- `ls -l -a -h -t -S -R` long / all / human / time / size / recursive
- `mkdir -p` create with parents
- `rm -r -f` recursive / force
- `cp -r` recursive
- `mv` move/rename
- `touch <file>` create empty / update timestamp
- `cat / less / head -n / tail -n / tail -f`
- `find . -name "*.c" -type f -mtime -7`

## Text processing
- `grep -i -v -E -r -n -c -l "pat" file` ignore-case / invert / extended-regex / recursive / line-num / count / files-with-matches
- `sed 's/old/new/g'` global replace
- `sed -i 's/o/n/g' file` in-place
- `sed -n '5,10p'` print lines 5-10
- `sed '/pat/d'` delete matching lines
- `awk '{print $2}'` field 2 ($0 = whole line, NF = num fields, NR = line num)
- `awk -F',' '{...}'` custom delimiter
- `sort -n -r -u -k2` numeric / reverse / unique / key column
- `uniq -c` count dupes (input must be sorted)
- `tr 'a-z' 'A-Z'` translate
- `tr -d '\r'` delete chars
- `tr -s ' '` squeeze runs
- `cut -d',' -f1,3` field 1 and 3 by comma
- `wc -l -w -c` lines / words / chars

## I/O & redirection
- `cmd > file` overwrite stdout
- `cmd >> file` append stdout
- `cmd < file` stdin from file
- `cmd 2> err.log` stderr to file
- `cmd > out 2>&1` or `cmd &> out` stdout+stderr combined
- `cmd1 | cmd2` pipe stdout of cmd1 -> stdin of cmd2
- `cmd1 && cmd2` run cmd2 only if cmd1 succeeds
- `cmd1 || cmd2` run cmd2 only if cmd1 fails
- `cmd1; cmd2` run both regardless

## Here-docs / here-strings
```sh
cat <<EOF
multi-line
EOF

grep "pat" <<<"single string"
```

## Quoting
- Single quotes `'...'` — literal, no expansion
- Double quotes `"..."` — `$var` and `` `cmd` `` expand, `*` does NOT
- Backticks / `$(cmd)` — command substitution
- `$()` preferred over backticks (nestable)

## Variables
- `VAR=value` (no spaces)
- `$VAR` or `${VAR}` use it
- `export VAR=v` make available to child processes
- `$1 $2 ...` script args
- `$#` number of args
- `$@` all args
- `$?` exit code of last command
- `$$` PID

## Conditionals
```sh
if [ "$x" -eq 5 ]; then ...
elif [ -f file ]; then ...
else ...
fi

[[ "$str" == "yes" ]]    # bash extended
```
- Numeric: `-eq -ne -lt -le -gt -ge`
- String: `=` `!=` `-z` (empty) `-n` (non-empty)
- File: `-f` (regular file) `-d` (dir) `-e` (exists) `-r -w -x` perms

## Loops
```sh
for f in *.c; do ...; done
for i in {1..10}; do ...; done
while [ $i -lt 10 ]; do ...; done
until cond; do ...; done
```

## Functions
```sh
greet() { echo "Hi $1"; }
greet "Alice"
```

## Permissions
- `chmod 755 file` (rwx r-x r-x)
- `chmod u+x file` (add exec for user)
- Octal: 4=r 2=w 1=x → 7=rwx, 5=r-x, 6=rw-

## Special chars in paths
- `~` home dir
- `.` current dir
- `..` parent dir
- `/` root

## Misc useful
- `man <cmd>` manual
- `which <cmd>` find binary path
- `history` shell history
- `!!` last command
- `!$` last arg
- `ssh user@host`
- `scp file user@host:/path`
- `htop / top` process viewer
- `kill -9 <pid>` force kill
- `ps aux | grep ...`

## Common trap
`echo 'Hello $USER'` → `Hello $USER` (no expansion — single quotes!)
`echo "Hello $USER"` → `Hello alice` (double quotes — expansion)

## Additional items (potentially missing)

### Process / job control
- `ps aux | grep <name>` see running processes
- `ps -ef`
- `kill <pid>` send SIGTERM
- `kill -9 <pid>` SIGKILL (force)
- `pkill <name>` kill by name
- `jobs` list background jobs
- `cmd &` run in background
- `fg %1` foreground job 1
- `bg %1` background job 1
- `nohup cmd &` run independent of shell
- `Ctrl+C` SIGINT (interrupt)
- `Ctrl+Z` suspend (then `fg`/`bg`)

### Useful tools
- `xargs` — build commands from stdin (`ls | xargs rm`)
- `tee file` — write stdout to file AND stdout
- `curl -L URL` — fetch URL, follow redirects
- `wget URL` — download file
- `diff a b` — line-by-line diff
- `cmp a b` — byte-by-byte compare
- `md5sum / sha256sum file`
- `du -sh dir` — disk usage
- `df -h` — disk free space
- `date` / `date +%Y-%m-%d`
- `time cmd` — measure runtime

### File globbing
- `*` — any chars (incl. empty)
- `?` — single char
- `[abc]` — one of a/b/c
- `[!abc]` — NOT one of these
- `{a,b,c}` — brace expansion (any of)
- `**/*.c` — recursive glob (needs `shopt -s globstar` in bash)

### Arithmetic
```sh
x=$((5 + 3))      # arithmetic expansion
((x > 5)) && echo yes
let "y = x * 2"
```

### case statement
```sh
case "$var" in
    a)  echo "is a";;
    b|c) echo "is b or c";;
    *)  echo "other";;
esac
```

### read input
```sh
read -p "Name: " name
read -s pwd       # silent (passwords)
```

### Process substitution
```sh
diff <(sort a) <(sort b)
```

### Standard streams reminder
- stdin = 0
- stdout = 1
- stderr = 2

### `true` and `false` as commands
- `true` always exits 0
- `false` always exits 1
- Useful in `while true; do ...; done`

### Common exit codes
- 0 = success
- 1 = general error
- 2 = misuse of shell builtin
- 126 = command not executable
- 127 = command not found
- 130 = terminated by Ctrl+C

### Variable defaults / fallbacks
- `${var:-default}` — use default if var unset/empty
- `${var:=default}` — assign default if unset/empty
- `${var:+alt}` — alt if var IS set
- `${var:?error}` — error out if unset

### String manipulation
- `${#var}` — length
- `${var:offset:length}` — substring
- `${var/pattern/replace}` — replace first
- `${var//pattern/replace}` — replace all
- `${var#prefix}` — strip prefix
- `${var%suffix}` — strip suffix

### Brace expansion examples
- `mv file{.txt,.bak}` → `mv file.txt file.bak`
- `mkdir -p src/{main,test}/{java,resources}`
