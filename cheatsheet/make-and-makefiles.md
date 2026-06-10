# Makefile Cheatsheet (candidate items)

## Basic syntax
```makefile
target: prerequisites
<TAB>command
```
- Command line MUST start with literal **TAB**, not spaces.

## Smallest example
```makefile
hello: hello.c
	gcc hello.c -o hello
```

## Automatic variables (memorize!)
- `$@` = target
- `$^` = all prereqs
- `$<` = first prereq
- `$?` = prereqs newer than target
- `$*` = stem matched by `%`

## Pattern rules
```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```
`%` matches anything; same value used on both sides per-word.

## Variables
- Assignment: `CC = gcc`
- Use: `$(CC)` or `${CC}`
- `=` recursive (lazy) — re-evaluates each use
- `:=` simple (eager) — evaluated ONCE at definition
- `?=` assign if not already set
- `+=` append (adds a single space)

## Variable flavors trap
```makefile
A := hello
B = $(A) world      # stores formula
A = goodbye

target: ;@echo $(B)   # → "goodbye world" (re-eval at use)
```
With `:=`, B would be frozen as "hello world".

## .PHONY
For targets that are NOT files (always run the recipe):
```makefile
.PHONY: all clean test check
clean:
	rm -f *.o myprog
```

## Default target
The **first** target wins when you run bare `make`. Conventionally `all`.

## $(wildcard) — auto-discover files
```makefile
SOURCES = $(wildcard *.c)        # → "main.c utils.c"
OBJECTS = $(SOURCES:.c=.o)       # substitution: → "main.o utils.o"
```

## Substitution shortcut
- `$(VAR:from=to)` replaces `from` with `to` at the end of every word.
- Same as `$(patsubst %from,%to,$(VAR))`.

## Implicit rules
- Make has built-in `%.o: %.c` recipe — you can omit it if `CC`/`CFLAGS` are set.

## Common targets (conventions, not keywords)
- `all` — build everything (usually first)
- `clean` — remove build artifacts
- `test` — run tests
- `check` — static analysis / lint
- `install` — copy to system dirs

## Stereotypical C Makefile
```makefile
CC = clang
CFLAGS = -Wall -g
CHECKFLAGS = --analyze -Xanalyzer -analyzer-checker=core

.PHONY: all clean check
all: myprog

myprog: main.o utils.o
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

check:
	$(CC) $(CHECKFLAGS) *.c

clean:
	rm -f *.o myprog
```

## Multi-line / chained commands
- Each line in a recipe runs in a **separate shell** by default
- Use `\` to continue, or `;` to chain:
```makefile
target:
	cd dir; pwd            # works (one line)
	cd dir \
	  && pwd               # works (line continuation)
```

## Quick exam traps
- TAB indentation required for recipes
- `=` lazy vs `:=` eager
- `$(VAR)` not `$VAR`
- Without `.PHONY`, a file named `clean` blocks the recipe
- Make uses TIMESTAMPS to decide what to rebuild
- Touch a `.c` file → `.o` rebuilds → linker re-runs

## $$ vs $
- `$@` in Makefile = target
- `$$VAR` in Makefile = shell variable `$VAR` (escaping)

## Additional items (potentially missing)

### Built-in functions (sometimes tested)
- `$(subst from,to,text)` — substring substitution
- `$(patsubst pat,repl,text)` — pattern substitution (full form of `:=`)
- `$(filter %.c,$(SRC))` — keep matches
- `$(filter-out %.h,$(FILES))` — drop matches
- `$(notdir path/to/file.c)` → `file.c`
- `$(basename file.c)` → `file`
- `$(addsuffix .o,foo bar)` → `foo.o bar.o`
- `$(addprefix src/,foo bar)` → `src/foo src/bar`
- `$(shell ls *.c)` — run shell, capture output
- `$(foreach var,list,body)`

### Conditional execution
```makefile
ifeq ($(DEBUG),1)
    CFLAGS += -g -O0
else
    CFLAGS += -O2
endif

ifdef VERBOSE
    Q =
else
    Q = @
endif
```

### Include other Makefiles
```makefile
include common.mk
-include optional.mk      # don't error if missing
```

### Echo silencing
- `@echo "hi"` — `@` suppresses the "echo hi" line itself
- `make -s` — silent mode (no command echoing)

### Order-only prerequisites
```makefile
$(OBJDIR)/main.o: main.c | $(OBJDIR)
$(OBJDIR):
	mkdir -p $@
```
Prereqs after `|` aren't checked for timestamps — they just need to exist.

### Special / built-in targets
- `.PHONY:` — phony targets
- `.SUFFIXES:` — suffix rules
- `.DEFAULT_GOAL := all` — set default target explicitly

### Useful make flags
- `make -n` — dry run (print commands, don't execute)
- `make -j 4` — parallel build with 4 jobs
- `make -B` — force rebuild everything
- `make -k` — keep going on error
- `make -C dir` — change directory first
- `make -f Makefile.custom` — use a different file

### Recursive make
```makefile
all:
	$(MAKE) -C subdir1
	$(MAKE) -C subdir2
```
(use `$(MAKE)` not `make` so flags propagate)

### Variable assignment trap (deep)
```makefile
A := $(shell date)   # snapshot at parse time
B = $(shell date)    # NEW snapshot every reference (slow!)
```

### Multi-line variable
```makefile
define BANNER
echo "===================="
echo "  Building project"
echo "===================="
endef

show:
	@$(BANNER)
