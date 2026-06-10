# Debugging Cheatsheet (candidate items)

## Quote
> "Debugging is like being the detective in a crime movie where you are also the murderer." — Filipe Fortes

## Stats
- Bugs cost ~$60B/year
- Validation (including debugging) = 50–75% of dev time
- Best debuggers are 3× more efficient than others

## Pre-step 0: "Search for the error message"
Pattern (Problem/Context/Solution):
1. Strip project-specific identifiers from input/output
2. Paste into search engine / AI
3. Carefully study results, experiment with suggestions
4. (Only after) ask experienced co-worker

## 4-step debugging process
1. **Reproduce** the bug (problem environment + history; write automated test)
2. **Locate** the faulty code (logs, diagrams, code smells, assertions)
3. **Determine the root cause** (rubber duck, breakpoints, conditional breakpoints)
4. **Implement & verify** a fix (add assertions, document, keep test as regression)

## Bug terminology
- **Fault** — erroneous code (e.g., uninitialized var)
- **Error** — incorrect state at runtime (var has wrong value)
- **Failure** — externally visible incorrect behavior
- Flow: `Fault → Error → reaches boundary → Failure`

## Reproducing the bug
- **Problem environment** = OS, deps, settings, hardware, versions
- **Problem history** = inputs, user interactions, comms with other components
- Other variables: timing, randomness, physical influences
- Write an automated bug reproduction test → run repeatedly during debugging
- Simplify the test case (remove irrelevant steps)

## Locating faulty code
- **Code smells** — bugs cluster in poorly-written code (refactor first!)
- **Common bugs** — uninitialized vars, unused values, unreachable code, memory leaks, interface misuse, null pointers, inconsistent types
- **Assertions** — add `assert input > 0`, `assert file != None`
- **Logging** — capture inputs (especially unexpected), state changes, comms
- **Visual diagrams** — symptoms like "no message X received" reveal missing flow

## Logging levels (low → high severity)
1. DEBUG
2. INFO
3. WARNING
4. ERROR
5. CRITICAL

## Python logging
```python
import logging
logging.warning("disk low")
logging.error("connection failed")
```

## Root cause: Rubber Duck Debugging
- Place rubber duck (or any object) on desk
- Explain your code line-by-line to it
- You'll catch the bug mid-explanation
- Why: "Curse of Knowledge" — you read what you INTENDED, not what's there

## Breakpoints (VS Code)
- Click in gutter to set
- Run with debugger attached
- Inspect Variables panel: **Locals**, **Globals**, **Watch**
- **Debug Console** — evaluate expressions in current scope
- Step Over (F10), Step Into (F11), Step Out (Shift+F11), Continue (F5)

## launch.json (VS Code config)
```json
{
  "configurations": [{
    "name": "Python: Current File",
    "type": "debugpy",
    "request": "launch",
    "args": ["10"],
    "program": "${file}",
    "console": "integratedTerminal"
  }]
}
```

## Conditional breakpoints
- Right-click breakpoint → Edit → enter expression
- Breakpoint triggers only when expression is true (e.g., `i > 100`)
- Use when bug only appears in specific conditions

## Step 4: After fixing
- **Add assertions** near the fix to detect nearby bugs
- **Document** in:
  - Code comment (why fix was necessary)
  - Git commit message (reference bug)
  - Bug report (solution + history)
- **Keep new tests** as **regression tests** — prevent reintroduction

## Regression testing
Re-running existing tests after changes to make sure nothing else broke. Bug reproduction test stays in the suite forever.

## Quick decision table
| Symptom | Tool |
|---|---|
| Don't know what error means | Search engine / AI |
| Wrong value mid-execution | Breakpoint + Locals |
| Bug only with specific input | Conditional breakpoint |
| Can't pin which commit broke it | `git bisect` |
| Wrong line, who wrote it | `git blame` |
| Can't explain code → bug | Rubber duck |
| Reproduce on different env | Document problem environment |

## Common code smells (debug context)
- Uninitialized variables
- Unused values
- Unreachable code
- Memory leaks (forgot to free)
- Null/None pointer access
- Type confusion (string vs int)
- Off-by-one errors
- Race conditions

## Don't suppress errors
```python
try: ...
except: pass     # BAD — silences ALL exceptions
```
Always catch specific exceptions, log, and re-raise or handle deliberately.

## Additional items (potentially missing)

### Print debugging (sometimes best!)
```python
print(f"DEBUG: x={x}, type={type(x)}")
```
Pros: simple, no setup. Cons: scattered, must remove later.

### Python pdb (post-mortem debugger)
```python
import pdb; pdb.set_trace()      # add this line; runs interactive debugger
# or just: breakpoint()          # Python 3.7+
```
- `n` next line
- `s` step into
- `c` continue
- `p var` print
- `l` list code
- `q` quit
- `pp obj` pretty-print

### Run script under pdb
- `python -m pdb script.py`

### Post-mortem
```python
try: risky()
except: import pdb; pdb.post_mortem()
```

### Python -X dev (for dev warnings)
- `python -X dev script.py` — extra warnings, runtime checks

### gdb basics (for C)
- `gcc -g main.c -o myprog` — compile with debug symbols
- `gdb ./myprog`
- `run [args]` start
- `break main` set breakpoint
- `break file.c:42`
- `n` next, `s` step into
- `p var` print
- `bt` backtrace (call stack)
- `frame N` go to stack frame
- `info locals` show locals
- `c` continue
- `q` quit

### Common C bug detectors
- **Valgrind** — memory leaks, use-after-free, uninitialized reads
- **AddressSanitizer** (`-fsanitize=address`) — memory bugs
- **ThreadSanitizer** (`-fsanitize=thread`) — race conditions
- **Clang Static Analyzer** (`clang --analyze`)
- **gcc -Wall -Wextra` — extra warnings

### Core dumps (Linux)
- `ulimit -c unlimited` — enable core dumps
- After crash → `core` file in cwd
- `gdb ./myprog core` — investigate

### Strace / ltrace
- `strace ./myprog` — trace system calls
- `ltrace ./myprog` — trace library calls
- Useful for "why is my program slow / hanging"

### Bisect (review)
- `git bisect` — narrow which commit broke things
- See git cheatsheet for details

### Useful Python warnings
- `python -Werror` — turn warnings into errors
- `import warnings; warnings.filterwarnings("error")`

### JS debugging
- `debugger;` statement — pause in browser DevTools / VS Code
- `console.log` / `console.dir` / `console.table` / `console.trace`
- Chrome DevTools breakpoints

### Common bug archetypes (recognize on sight)
- **Off-by-one** — `<` vs `<=`, `i+1` confusion
- **Null/None deref** — forgot to check before use
- **Integer overflow** — small int type (C/Java)
- **Race condition** — concurrent access without sync
- **Use after free** — C, dangling pointer
- **Memory leak** — forgot to free / close
- **Resource leak** — file/socket/lock not closed
- **Type confusion** — string vs number, list vs single
- **Floating-point comparison** — `0.1 + 0.2 != 0.3`
- **Time-zone** — local vs UTC mismatch
- **Encoding** — UTF-8 vs ASCII vs Latin-1

### When stuck — diagnostic checklist
- Re-read the error message carefully
- Reproduce on a simpler input
- Look at git log for recent changes
- Search the error message online
- Take a break (rubber duck after)
- Pair with a colleague
- Bisect / binary search the failure
