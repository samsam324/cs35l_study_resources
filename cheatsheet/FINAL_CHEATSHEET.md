# CS35L FINAL CHEATSHEET (hand-write candidate)

Organized for transcription. Front side = high-frequency lookup. Back = examples + templates.

---

## 1. ACRONYMS (define each)

**SOLID** — design principles
- **S**ingle Responsibility — one reason to change
- **O**pen-Closed — open for extension, closed for modification
- **L**iskov Substitution — subclass usable where parent is expected
- **I**nterface Segregation — many small interfaces > one big one
- **D**ependency Inversion — depend on abstractions, not concretions

**CIA** — security
- **C**onfidentiality — data accessed by authorized only
- **I**ntegrity — data modified by authorized only; accurate & trustworthy
- **A**vailability — services available when needed

**ACID** — DB transactions
- **A**tomicity — all-or-nothing
- **C**onsistency — DB stays in valid state (constraints hold)
- **I**solation — concurrent txns appear sequential
- **D**urability — once committed, survives crashes

**CAP** — distributed systems (at most 2 of 3)
- **C**onsistency — every read sees latest write
- **A**vailability — every request gets a response
- **P**artition tolerance — works despite network splits
- Since P is unavoidable → choose CP or AP

**FIRST** — good tests
- **F**ast — milliseconds
- **I**ndependent — order doesn't matter
- **R**epeatable — same result every time
- **S**elf-validating — pass/fail without human eye
- **T**imely — written close to / before production code

**DRY** — Don't Repeat Yourself
**KISS** — Keep It Simple, Stupid
**YAGNI** — You Aren't Gonna Need It
**MVP** — Minimum Viable Product
**EAFP** vs **LBYL** — Python: Ask Forgiveness vs Look Before Leap

---

## 2. DESIGN PRINCIPLES

**Information Hiding** (Parnas 1972)
- Define interface that reveals WHAT a module does
- Hide implementation details that may change
- Test: "If this decision changes, how many modules edit?" >1 = leaked
- Interface = stable contract; Implementation = changeable
- Encapsulation = mechanism (private); Info Hiding = the principle

**Single Choice Principle**
- ONE module knows the exhaustive list of alternatives (e.g., supported DBMSes)
- Adding new alternative = edit ONE module
- Not the same as Single Responsibility

**Separation of Concerns** — different aspects → different modules

**Composition over Inheritance** — prefer "has-a" over "is-a"

**Dependency Injection** — pass dependencies in, don't construct internally (enables testing + flexibility)

**Bus Factor** — # people who can leave before project stalls
- INCREASES it: code review, clean code, tests, docs, pair programming
- Does NOT: AI-generated code, working alone

---

## 3. TESTING & TDD

### TDD Cycle: Red → Green → Refactor
1. **Red**: write failing test FIRST
2. **Green**: write MINIMUM code to pass
3. **Refactor**: clean up while staying green

### Uncle Bob's 3 Rules
1. **No production code without a failing test** (every line justified)
2. **No more test than sufficient to fail** (tight cycles, one increment)
3. **No more production code than sufficient to pass** (no untested code)

### WHY each rule matters
- **Test must fail first** = "test the test"; prove the test detects failures
- **Min test** = stay focused, instant feedback, diagnose failures cleanly
- **Min code** = every line is justified by a test; no untested speculation

### Testing vs Debugging
- **Testing** = DETECTS presence of bugs
- **Debugging** = LOCATES + FIXES known bugs

### Test Doubles (exam phrasing)
| Type | Job |
|---|---|
| **Stub** | Injects indirect **INPUTS** (canned data) |
| **Mock** | Verifies indirect **OUTPUTS** (asserts on calls) |
| **Spy** | Records outputs without asserting |
| **Fake** | Working but simplified impl (in-mem DB) |

- "Indirect input" = data flowing INTO function from a dependency
- "Indirect output" = data your function sends OUT to a dependency

### V&V
- **Verification** = "did we build it right?" (matches spec)
- **Validation** = "did we build the right thing?" (matches user need)

### Test Pyramid
- Many UNIT (fast, ms)
- Some INTEGRATION (sec)
- Few E2E (min, Playwright)

### Test Design
- **Equivalence classes** — group inputs with same expected behavior
- **Boundary value analysis** — test edges (min−1, min, min+1, max−1, max, max+1)
- **Edge cases** — empty, null, max, min, unicode, 0, negative

### CI/CD
- CI = run tests on every push (GitHub Actions, Jenkins, Travis)
- CD = auto-deploy on green CI

### BDD (Cucumber/Gherkin)
Given / When / Then format → executable specs by non-devs

---

## 4. DEBUGGING

### 4-step process (ORDER MATTERS)
1. **Reproduce** the bug (FIRST — without this, can't observe or verify)
2. **Locate** the faulty code
3. Determine the **root cause**
4. Implement and **verify** the fix

### Fault vs Error vs Failure
- **Fault** = erroneous code (e.g., uninitialized var)
- **Error** = wrong runtime state
- **Failure** = externally visible bad behavior
- Flow: Fault → Error → (reaches boundary) → Failure

### Tactics
- Search error message in Google/AI FIRST
- Reproduce: problem environment + problem history → automate as test
- Locate: logging (DEBUG/INFO/WARNING/ERROR/CRITICAL), assertions, code smells
- Root cause: rubber duck, breakpoints, conditional breakpoints
- Fix: add assertions, doc commit message, keep test as regression

---

## 5. GIT

### Commands by safety
- **Additive (safe on shared):** `commit`, `merge`, `cherry-pick`, `revert`, `pull`, `push`
- **Rewriting (UNSAFE on shared):** `rebase`, `rebase -i`, `commit --amend`, `reset --hard`, `push -f`

### Force-push danger
- Rewrites remote history; teammates' commits made between your last pull and push get DESTROYED

### Detached HEAD
- HEAD points to a commit, NOT a branch
- Caused by: `checkout <sha>`, `checkout HEAD~N`, `checkout <tag>`, mid-`bisect`
- Get out: `git switch <branch>` or `git switch -c <new-branch>` (preserves work)

### Detached HEAD analysis rules
| Command sequence | Detached after? |
|---|---|
| Just commits | DEPENDS (on prior state) |
| `checkout HEAD~5` | YES |
| `checkout main` | NO |
| `stash` / `stash pop` (doesn't move HEAD) | DEPENDS |

### Git bisect
- Binary search for which commit introduced bug
- Workflow: `bisect start` → `bisect bad` → `bisect good <sha>` → `bisect run <test>` → `bisect reset`
- 256 commits → max 8 tests (log₂)

### Bisect traps
- **TDD red commits break bisect** — tests fail on red commits unrelated to the bug
- **Committed bisect-test files get deleted** when checking out old commits (don't commit them OR keep outside repo)
- **Squash merge fixes both** — only "green" commits reach main, intermediate red commits never appear

### Squash merge
- `git merge --squash <branch>` then `git commit`
- Collapses all branch commits into ONE on main
- Main has ONE parent (not a merge commit); branch is orphaned from main's POV

### Reverting a squashed feature commit
- Reverts the WHOLE feature, not just the bug
- Solution: bisect to find commit, then MANUALLY fix the bug inside the feature

### Relative addresses
- `HEAD~N` = N steps back via first parent
- `HEAD^` = first parent (same as ~1)
- `HEAD^2` = SECOND parent (merge commits only)

---

## 6. DESIGN PATTERNS (5)

### Observer (L6)
- **Subject** sends updates; **Observer** listens
- One-to-many notification; subject keeps list of observers
- Lecture example: NewsChannel + Subscriber (MobileApp, EmailDigest)
- Use: UI ↔ domain state sync

### Publish-Subscribe (L10)
- N-to-M messaging via **broker/topic**
- Publisher → Topic → Subscriber (decoupled)
- Differs from Observer: decoupled via broker, often cross-process
- Use: Kafka, RabbitMQ, microservices

### Adapter (L17)
- Wraps incompatible interface to expose target interface
- **Target** (what client expects), **Adaptee** (existing), **Adapter** (wraps)
- Use: bridging APIs, legacy integration

### Strategy (L7)
- Interchangeable algorithms via composition
- **Context** holds **Strategy**; delegates to it
- Supports Open-Closed Principle
- Use: swap sort algorithms, payment methods

### State (EXAM)
- Context delegates state-dependent behavior to State object
- One subclass per state (e.g., InJailState, NormalState, BankruptState)
- When state changes: swap state object (`this.state = new InJailState(...)`)
- Each state class implements the interface differently
- **How it supports Separation of Concerns:** state-dependent behavior lives in state classes; concerns are separated by state type

```
   Context                AbstractState
+---------+              +--------------+
| state   |─────────────►| op()         |   <-- italic = abstract
| op()    | delegates    +--------------+
+---------+                     △
                                |
                ┌───────────────┼───────────────┐
            StateA           StateB           StateC
            +op()            +op()            +op()
```

---

## 7. UML NOTATION (modeling questions)

### Visibility
- `-` private
- `+` public
- `#` protected
- `~` package-private

### Class diagram arrows
- Solid + **hollow triangle** = inheritance (Child→Parent)
- Dashed + hollow triangle = realization (implements interface)
- Solid line = association
- Solid + arrow = directed association
- Line + **hollow diamond** at whole = aggregation (weak "has-a")
- Line + **filled diamond** at whole = composition (strong "owns-a")
- Dashed + arrow = dependency

### Multiplicity
- `1`, `0..1`, `*`, `1..*`, `n..m`

### Abstract classes/methods
- Use **italic** for class name AND method name
- All abstract methods MUST be implemented by concrete subclasses

### Sequence diagrams
- Lifelines = vertical dashed lines
- Names use **objects** with `:` (e.g., `:Player`, NOT `Player`)
- Don't include abstract classes as objects (they can't be instantiated)
- Method signatures must match the class diagram

### Common mistakes (exam fodder)
- Filled inheritance arrow → should be hollow
- Public state member `+state` → should be private `-state`
- Concrete state class missing required method
- Abstract class without italics
- Sequence diagram uses class name not object (missing `:`)

---

## 8. PROCESS

### Three process types
| | Plan-Driven | Agile | Risk-Driven |
|---|---|---|---|
| Upfront design | Heavy | Tiny | Proportional |
| Iteration | None | 1-4 wks | Mixed |
| Feedback | Late | Continuous | Risk-focused |
| Best for | Spacecraft, medical | Web apps, startups | Mid-range, design docs |

### Process selection
| Domain | Process |
|---|---|
| Radiation therapy, spacecraft | Plan-driven |
| Saturn probe | Plan-driven |
| Startup | Agile |
| Google-scale web | Risk-driven |

### Scrum events / artifacts
- Product Backlog → Sprint Planning → Sprint Backlog → Sprint → Increment → Sprint Review
- Daily Standup (15 min, every day)
- Sprint = 2-3 wks, locked
- Planning Poker — story points in effort, not time

### Reuse: 3 design principles to memorize
1. **Pin dependency versions** (Pipenv, Pipfile, lock files)
2. **Update dependencies** for security patches — but aware of side effects
3. **Strive for fewer dependencies** — avoid trivial code (left-pad lesson)
4. (Alt) **Identify violated assumptions** before reusing (Ariane 5 lesson)
5. (Alt) **Prefer popular + maintained** packages — but fit-to-context > popularity

---

## 9. CODE SMELLS + REFACTORINGS

| Smell | Refactoring |
|---|---|
| Duplicate code | Extract Method / Class |
| Long method (>30 lines) | Extract Method |
| Large class | Extract Class |
| Long parameter list (5+) | Introduce Parameter Object |
| Magic numbers | Named constant |
| Dead code | Delete it |
| Feature envy | Move Method |
| Data clumps | Extract Class |
| Conditional dispatch on type | **Replace Conditional with Polymorphism** ← canonical exam answer |
| Shotgun surgery | Move methods to consolidate |
| God class | Split by responsibility |

### Replace Conditional with Polymorphism (template)
```
BEFORE:                          AFTER:
if obj.type == "X": ...          class Base: def op(self): pass
elif obj.type == "Y": ...        class X(Base): def op(self): ...
                                 class Y(Base): def op(self): ...
                                 obj.op()
```
Supports Open-Closed Principle.

---

## 10. CLEAN CODE (4 Pillars)

1. **Meaningful Naming** — intention-revealing, pronounceable, searchable
2. **Simplified Structure** — guard clauses, shallow nesting, small functions
3. **Purposeful Documentation** — explain WHY not WHAT
4. **Design by Contract** — preconditions / postconditions / invariants (use `assert`)

---

## 11. NETWORKING

### TCP/IP layers
| Layer | Job | Examples |
|---|---|---|
| Application | App-to-app | HTTP, FTP, SSH, DNS |
| Transport | End-to-end reliable + ORDERING + SEGMENTATION | TCP, UDP |
| Internet/Network | Addressing, routing | IPv4, IPv6 |
| Link | Physical | Ethernet, Wi-Fi |

### HTTP verbs
- `GET` — read (idempotent)
- `POST` — create (not idempotent)
- `PUT` — replace (idempotent)
- `PATCH` — partial update
- `DELETE` — remove (idempotent)
- `HEAD` — headers only

### Status codes
- `2xx` success: 200 OK, 201 Created, 204 No Content
- `3xx` redirect: 301, 304 Not Modified
- `4xx` client error: 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many
- `5xx` server error: 500, 502, 503

### Client-server vs P2P
- Client-server: centralized, client initiates
- P2P: decentralized, peers equal
- Hybrid common (Zoom: P2P with C-S fallback)

---

## 12. SECURITY

### SQL Injection defense
- **Parameterized queries / prepared statements**
- Vulnerable: `"SELECT * FROM u WHERE name='" + name + "'"`
- Safe: `"SELECT * FROM u WHERE name=?"; execute(sql, [name])`
- f-strings still vulnerable

### XSS (Cross-Site Scripting)
- Attacker injects JS into trusted site → executes in victim's browser → steals cookies/tokens
- Most prone: where user TEXT is rendered to OTHER users in a browser (reviews, comments, forums)
- Defense: output encoding/escaping, CSP, HttpOnly cookies, SameSite

### Authentication
- Session cookies — stateful, HttpOnly + SameSite
- JWT — stateless, signed by server, watch for XSS theft
- Always HTTPS + bcrypt/Argon2 for passwords, never MD5/SHA1

### Principle of Least Privilege
- Every component gets minimum necessary access
- Reduces blast radius of compromise

---

## 13. REGEX

### Anchors / classes / quantifiers
- `^` start, `$` end, `\b` word boundary
- `.` any (not newline by default), `\d` digit, `\D` non-digit, `\w` word, `\s` whitespace, `\S` non-whitespace
- `[abc]` any of, `[^abc]` not, `[a-z]` range
- `*` 0+, `+` 1+, `?` 0/1, `{n}` exact, `{n,}` n+, `{n,m}` range
- Lazy: `*?`, `+?`

### Groups / alternation
- `(...)` capture, `(?:...)` non-capture, `(?P<name>...)` named (Python)
- `|` alternation
- `\1` backreference (sed/Python), `$1` (JS)

### Lookarounds
- `(?=...)` lookahead, `(?!...)` neg lookahead
- `(?<=...)` lookbehind, `(?<!...)` neg lookbehind

### Flags
- `i` case-insensitive, `g` global, `m` multiline, `s` dotall

### Examples
- 8-20 chars no whitespace: `^\S{8,20}$` or `^[^\s]{8,20}$`
- Email-ish: `[\w.+-]+@[\w-]+\.[\w.-]+`
- IPv4: `\b\d{1,3}(\.\d{1,3}){3}\b`
- Non-empty word: `\b\w+\b`

---

## 14. GREP

```bash
grep "pattern" file         # basic
grep -i "pat" file          # case-insensitive
grep -v "pat" file          # invert (lines NOT matching)
grep -n "pat" file          # show line numbers
grep -r "pat" dir/          # recursive
grep -c "pat" file          # count matches
grep -l "pat" *.txt         # list files with matches
grep -E "pat1|pat2" file    # extended regex (use ( ) { } | + ? unescaped)
grep -A 3 "pat" file        # 3 lines after
grep -B 3 "pat" file        # 3 lines before
grep -C 3 "pat" file        # 3 lines around
grep -w "word" file         # whole word only
grep -F "literal" file      # fixed string (no regex)
```

---

## 15. SED

```bash
sed 's/old/new/' file       # replace first per line
sed 's/old/new/g' file      # replace all on line
sed -i 's/o/n/g' file       # in-place (GNU)
sed -i.bak 's/o/n/g' file   # in-place with backup
sed -n '5,10p' file         # print lines 5-10 only (with -n)
sed '/pat/d' file           # delete matching lines
sed '/^$/d' file            # delete blank lines
sed -e 'cmd1' -e 'cmd2'     # multiple commands
sed -E 's/(\w+) (\w+)/\2 \1/'  # swap with capture groups
sed 's|/a/b|/c/d|g'         # alternate delimiter (paths)
```

### sed trap
- Single quotes: `sed 's/$user/x/'` — literal `$user`
- Double quotes: `sed "s/$user/x/"` — shell expands `$user`

---

## 16. SHELL EXAMPLE (with all the weird variables)

```bash
#!/bin/bash
set -e          # exit immediately on error
set -u          # error on undefined variables
set -o pipefail # pipeline fails if any command fails

# Special variables
echo "Script: $0"
echo "Arg 1: $1"
echo "Arg 2: $2"
echo "All args: $@"     # all as separate words
echo "All args: $*"     # all as one string
echo "Number of args: $#"
echo "Exit code of last: $?"
echo "PID: $$"
echo "User: $USER"

# Variable defaults
NAME="${1:-default}"           # use default if $1 unset/empty
: "${REQUIRED:?must be set}"   # error if unset

# Conditionals
if [ "$#" -lt 2 ]; then
    echo "Usage: $0 arg1 arg2" >&2     # >&2 = to stderr
    exit 1
fi

# Loop over arguments
for arg in "$@"; do
    echo "Processing: $arg"
done

# Command substitution
FILES=$(ls *.txt 2>/dev/null)

# String checks
if [[ -z "$NAME" ]]; then echo "empty"; fi      # -z = empty
if [[ -n "$NAME" ]]; then echo "not empty"; fi  # -n = non-empty

# File checks
[[ -f "/etc/hosts" ]] && echo "file exists"
[[ -d "/tmp" ]] && echo "dir exists"
[[ -r "/etc/hosts" ]] && echo "readable"

# Heredoc
cat <<EOF
Hello $NAME
EOF

# Arithmetic
COUNT=$((10 + 5))
((COUNT > 10)) && echo "big"
```

---

## 17. MAKE EXAMPLE

```makefile
# Variables
CC = clang
CFLAGS = -Wall -g
LDFLAGS =

# Auto-discover sources
SOURCES = $(wildcard *.c)
OBJECTS = $(SOURCES:.c=.o)         # substitution

# Phony targets (not files)
.PHONY: all clean check

# Default target (first one!)
all: myprog

# Link
myprog: $(OBJECTS)
	$(CC) $(LDFLAGS) $^ -o $@      # $^ = all prereqs, $@ = target

# Pattern rule
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@    # $< = first prereq

# Static analysis (phony)
check:
	$(CC) --analyze $(SOURCES)

# Cleanup (phony)
clean:
	rm -f $(OBJECTS) myprog
```

### Essential Make vars
- `$@` = target
- `$^` = all prereqs
- `$<` = first prereq
- `$?` = prereqs newer than target
- `$*` = stem from `%`

### `=` vs `:=`
- `=` lazy (re-eval at use)
- `:=` eager (evaluated now, frozen)

### Trap
- Tab required for command line (not spaces)
- Without `.PHONY`, file named `clean` blocks the rule

---

## 18. C MALLOC EXAMPLE

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]) {
    int n = 5;

    // Allocate: returns NULL on failure
    int *arr = (int*)malloc(n * sizeof(int));
    if (arr == NULL) {
        fprintf(stderr, "malloc failed\n");
        return 1;
    }

    // Use
    for (int i = 0; i < n; i++) {
        arr[i] = i * 2;
    }

    // Realloc
    int *bigger = (int*)realloc(arr, 10 * sizeof(int));
    if (bigger == NULL) {
        free(arr);    // realloc failed; original still valid
        return 1;
    }
    arr = bigger;

    // Print
    for (int i = 0; i < 10; i++) {
        printf("%d\n", arr[i]);
    }

    // Always free + NULL out
    free(arr);
    arr = NULL;

    // String version
    char *s = (char*)malloc(strlen("hello") + 1);  // +1 for \0
    strcpy(s, "hello");
    free(s);

    // calloc — zeroed
    int *zeros = (int*)calloc(n, sizeof(int));
    free(zeros);

    return 0;
}
```

### Mistakes to avoid
- Memory leak: forget `free`
- Double-free: `free` twice
- Dangling pointer: use after `free`
- Buffer overflow: write past array end
- Forgetting `\0` byte in strings
- Returning pointer to local variable

---

## 19. PYTEST EXAMPLE (with @decorators)

```python
import pytest

# ==== FIXTURE ====
@pytest.fixture
def sample_user():
    """Provides fresh user dict to any test that asks."""
    return {"name": "Alice", "age": 30, "balance": 100}

@pytest.fixture(scope="module")
def db_connection():
    """One DB connection for the whole module. yield for teardown."""
    conn = create_connection()
    yield conn                  # tests use this
    conn.close()                # teardown runs after all tests

# ==== BASIC TEST ====
def test_user_name(sample_user):
    assert sample_user["name"] == "Alice"

# ==== TEST EXPECTED EXCEPTION ====
def test_negative_balance():
    with pytest.raises(ValueError):
        withdraw(account=user, amount=-5)

# ==== PARAMETRIZE — same test, many inputs ====
@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (-1, 1, 0),
    (0, 0, 0),
    (100, 200, 300),
])
def test_add(a, b, expected):
    assert add(a, b) == expected

# ==== SKIP / XFAIL ====
@pytest.mark.skip(reason="not yet implemented")
def test_future(): ...

@pytest.mark.skipif(sys.version_info < (3, 9), reason="3.9+")
def test_newer(): ...

@pytest.mark.xfail   # expected to fail
def test_known_bug(): ...

# ==== MOCK (test doubles) ====
from unittest.mock import Mock, patch

def test_signup_calls_email():
    mock_email = Mock()
    signup("alice@x.com", mock_email)
    mock_email.send.assert_called_once_with("alice@x.com", "Welcome!")

@patch("mymod.requests.get")
def test_external_api(mock_get):
    mock_get.return_value.status_code = 200
    result = fetch_user(1)
    assert result.status_code == 200
```

### Run
```bash
pytest                          # auto-discover test_*.py
pytest -v                       # verbose
pytest -k "name"                # filter by test name
pytest test_x.py::test_one      # specific test
pytest --cov=mypkg              # coverage report
pytest -x                       # stop on first failure
pytest --pdb                    # drop to debugger on failure
```

---

## 20. JAVA QUICK EXAMPLE

```java
import java.util.*;

public class Example {
    public static void main(String[] args) {
        // Collections
        List<String> names = new ArrayList<>();
        names.add("Alice");
        names.size();                   // method, NOT .length

        // Map with safe lookup
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        int grade = scores.getOrDefault("Bob", 0);   // no NPE

        // == vs .equals()
        String a = new String("hi");
        String b = new String("hi");
        System.out.println(a == b);        // false (identity)
        System.out.println(a.equals(b));   // true (value)

        // Try-with-resources
        try (var f = new FileReader("x.txt")) {
            // f auto-closed
        } catch (IOException e) { ... }

        // Lambda
        Runnable r = () -> System.out.println("hi");
        names.sort((x, y) -> x.compareTo(y));
        names.forEach(System.out::println);

        // Array.length (field) vs .length() (string method)
        int[] arr = {1, 2, 3};
        arr.length;                       // 3 (field, no parens)
        "hello".length();                  // 5 (method, parens)
    }
}
```

### Java traps
- `==` on objects = identity check; use `.equals()` for value
- Integer cache: -128 to 127 cached, `==` accidentally works for small ints
- `map.get("missing")` returns null → unbox to int = NPE
- Default access = package-private (NOT private like C++)

---

## 21. ANSWER FORMAT TEMPLATES

### User story
```
As a [role],
I want to [do something],
so that [benefit].
```

### Acceptance criteria (Gherkin)
```
Given [precondition],
When [action],
Then [expected outcome]
And [additional outcome]
```

### INVEST justification template
For V (Valuable):
> "Valuable because [user benefit tied to feature]. Specifically, AC[X] ensures [concrete outcome]. Many users benefit because [scope reason]."

For S (Small):
> "Small because implementation requires only [list scope]. No [out-of-scope thing]. Fits in [time]."

For T (Testable):
> "Testable because we can write Gherkin scenarios: Given [setup], When [action], Then [outcome]. We can verify by [test method]."

For I (Independent):
> "Independent because no other story needs to complete first."

For N (Negotiable):
> "Negotiable because details like [X, Y] can be refined during sprint planning."

For E (Estimable):
> "Estimable because we know roughly the LOC and the team has built similar features."

### Refactoring justification template
> Change 1: [what], because [clean code principle / smell removed].
> Change 2: [what], because [pillar — naming / structure / docs / contract].
> Change 3: [what], because [reduces cognitive load / increases beacons / removes magic numbers].

### Design pattern description template
> **Key idea**: [intent in one sentence]
> **Structure**: [roles + relationships]
> **Behavior**: [runtime operation]
> **Why it supports [principle]**: [explicit connection]

### Bisect debugging answer template
> **Why doesn't it work?** [Technical reason — e.g., "tests fail on red commits / files don't exist in old commits"]
> **Fix idea A**: [step 1] → [step 2] → [step 3]
> **Fix idea B**: [step 1] → [step 2] → [step 3]

### Process selection template
> Use [Plan-Driven / Agile / Risk-Driven] because:
> - The cost of failure is [high/low]
> - Requirements are [stable/changing]
> - The system is [safety-critical / web-app / startup]

---

## 22. QUICK MAPS — high-frequency lookup

### "Which design principle does X violate?"
| Symptom | Violated principle |
|---|---|
| Class does many things | Single Responsibility |
| Adding a feature requires editing existing code | Open-Closed |
| Big interface with unused methods | Interface Segregation |
| Subclass breaks parent contract | Liskov Substitution |
| Hard-coded dependency on concrete class | Dependency Inversion |
| Duplicate code | DRY |
| Change to one decision touches many files | Information Hiding |
| Multiple modules know the list of supported types | Single Choice |

### "Which test double for this scenario?"
| Scenario | Double |
|---|---|
| Function READS from dependency, want canned data | Stub |
| Function WRITES to dependency, want to verify call | Mock |
| Need real-ish behavior cheaply (in-mem DB) | Fake |
| Want real behavior + audit trail | Spy |

### "Which git command?"
| Need | Command |
|---|---|
| Undo last local commit | `git reset --soft HEAD~1` |
| Undo a pushed commit (safely) | `git revert <sha>` |
| Find who wrote line 42 | `git blame -L 42,42 file` |
| Find which commit broke it | `git bisect` |
| Save WIP to switch branches | `git stash` |
| Copy one commit to another branch | `git cherry-pick` |
| Fix commit message | `git commit --amend` (local only) |

### Quick decisions
- Python vs C? → close to humans = Python; close to hardware = C
- Library vs Framework? → you call IT = library; IT calls you = framework
- Stub vs Mock? → injects INPUTS = stub; verifies OUTPUTS = mock
- CP vs AP? → consistency-critical = CP; availability-critical = AP
- Agile vs Plan-driven? → low cost of change = Agile; high cost of failure = Plan

---

## 23. DIAGRAMS TO RECREATE BY HAND

These are the canonical exam-shape diagrams. Practice drawing each so you can reproduce them under pressure.

### 23.1 UML Class Diagram — full notation reference

```
        +----------------------+
        |    <<abstract>>      |        <- write class name in ITALICS
        |       Vehicle        |
        +----------------------+
        | - make: String       |        - = private
        | - year: int          |        + = public
        | # validate(): bool   |        # = protected
        | ~ helper(): void     |        ~ = package-private
        +----------------------+
        | + getMake(): String  |
        | + describe(): String |        <- italic = abstract method
        +----------------------+
                   △                    HOLLOW triangle = inheritance
                   |
        +----------------------+
        |        Car           |        (concrete subclass)
        +----------------------+
        | - numDoors: int      |
        +----------------------+
        | + describe(): String |        <- must implement abstract method
        +----------------------+
```

### 23.2 Relationship arrows (memorize)

```
   A ─────►  B       directed association  (A knows B)
   A ─────   B       plain association
   A ─────△  B       inheritance (A extends B)  HOLLOW triangle
   A ┄┄┄┄△  B       realization (A implements interface B)  DASHED + hollow triangle
   A ─────◇  B       aggregation (A has B, weak — B can outlive A)  HOLLOW diamond at WHOLE
   A ─────◆  B       composition (A owns B, strong — B dies with A)  FILLED diamond at WHOLE
   A ┄┄┄┄►  B       dependency  DASHED + arrow
```

### 23.3 Multiplicity (on association ends)

```
   1                exactly one
   0..1             optional (zero or one)
   *                zero or more
   1..*             one or more
   2..5             between 2 and 5

   Example:
   Library  1 ─────────── *  Book        (one Library has many Books)
   Person   1 ─────────── 0..1 Passport  (one Person has at most one Passport)
   Student  *  ────────── *  Course      (many-to-many)
```

### 23.4 State Pattern — full UML (the practice-exam shape)

```
   +-----------------+                          +-------------------------+
   |     Player      |       1         *        |     <<abstract>>        |
   |-----------------|◆-------------------------|     AbstractState       |   <- italic
   | - state:        |   (composition)          |-------------------------|
   |     AbsState    |                          | # player: Player        |
   |-----------------|                          |-------------------------|
   | + takeTurn()    |                          | + takeTurn(): void      |   <- italic (abstract)
   +-----------------+                          +-------------------------+
                                                          △
                                                          | (HOLLOW triangle)
                              ┌───────────────────────────┼───────────────────────────┐
                              |                           |                           |
                  +-------------------+      +-------------------+      +-------------------+
                  |   InJailState     |      |   NormalState     |      |   BankruptState   |
                  +-------------------+      +-------------------+      +-------------------+
                  | + takeTurn(): void|      | + takeTurn(): void|      | + takeTurn(): void|
                  +-------------------+      +-------------------+      +-------------------+

   COMMON MISTAKES (the exam tests these):
   - state attribute must be PRIVATE (-state, not +state)
   - inheritance arrow must be HOLLOW triangle (not filled)
   - AbstractState class name + takeTurn() must be ITALIC
   - EVERY concrete state must implement takeTurn() (BankruptState often forgotten!)
```

### 23.5 UML Sequence Diagram (for State pattern call)

```
      :Player              :NormalState
        │                      │
        │  takeTurn(player)    │            <- objects use ":Type" format
        │─────────────────────►│               (NOT just "Player")
        │                      │
        │   [activation bar]   │■            <- thin rectangle = activation
        │                      │■
        │                      │■  (does work)
        │                      │■
        │       return         │
        │◄- - - - - - - - - - -│            <- DASHED arrow = return
        │                      │
        ↓                      ↓             <- vertical DASHED lines = lifelines

   Notation rules:
   - Each participant gets a box at top with ":Type" or "name:Type"
   - Vertical dashed line = lifeline
   - Solid arrow = synchronous message
   - Open arrow ─────▻ = asynchronous message
   - Dashed arrow with open head = return value
   - Thin rectangle on lifeline = activation (active period)

   Fragment frames (for control flow):
   ┌────────────────────────────────┐
   │ alt [condition]                │     alt = if/else
   │   ... messages ...             │
   │ - - - - - - - - - - - - - - - -│
   │ [else]                         │
   │   ... messages ...             │
   └────────────────────────────────┘

   ┌────────────────────────────────┐
   │ loop [condition]               │     loop = iteration
   │   ... messages ...             │
   └────────────────────────────────┘

   opt = optional, par = parallel
```

### 23.6 Observer Pattern UML

```
   +-----------------+      observers      +-------------------+
   |     Subject     |    1         *      |   <<abstract>>    |
   |-----------------|◆- - - - - - - - - - |     Observer      |   <- italic
   | - observers     |                     |-------------------|
   |-----------------|                     | + update(): void  |   <- italic
   | + register(o)   |                     +-------------------+
   | + unregister(o) |                              △
   | + notify()      |                              |
   +-----------------+                       ┌──────┴──────┐
                                             |             |
                                    +----------------+  +----------------+
                                    | ConcreteObs A  |  | ConcreteObs B  |
                                    +----------------+  +----------------+
                                    | + update()     |  | + update()     |
                                    +----------------+  +----------------+

   Lecture example: NewsChannel (Subject) + MobileApp / EmailDigest (Observers)
```

### 23.7 UML State Machine Diagram (for behavioral view)

```
                       ●  <- filled dot = INITIAL state
                       │
                       ▼
              ┌───────────────────┐
              │       Idle        │   <- state = rounded rectangle
              └────────┬──────────┘
                       │ start()         <- transition labeled with event[guard]/action
                       ▼
              ┌───────────────────┐      pause()
              │     Running       │ ◄────────────────┐
              └────────┬──────────┘                  │
                       │  stop()                     │
                       │      ┌──────────────────────┘
                       │      │ resume()
                       ▼      │
              ┌───────────────────┐
              │     Stopped       │
              └────────┬──────────┘
                       │
                       ▼
                       ◉  <- bullseye = FINAL state
```

### 23.8 ER Diagram (Data Management)

```
   "Chen" notation (entities + diamonds):

      ┌────────────┐               ┌──────────────┐
      │  STUDENT   │      *   *    │   COURSE     │
      │            │─────◇ENROLLED◇│              │
      └─────┬──────┘     in        └──────┬───────┘
            │                              │
       ─────┼─────                  ───────┼──────
       │    │    │                  │      │      │
    (sid) name email             (cid)  title  credits
     ^^^                          ^^^
   (underline = primary key)

   "Crow's foot" notation (more common in modern tools):

       ┌──────────┐                     ┌──────────┐
       │ STUDENT  │ 1 ─────────────  *  │  COURSE  │
       │  sid PK  │     enrolled in     │ cid PK   │
       │  name    │                     │ title    │
       └──────────┘                     └──────────┘

   Cardinality on line ends:
   1        ─||─        exactly one
   0..1     ─o|─        zero or one
   *        ─}─         many (zero or more)
   1..*     ─|}─        one or more
```

### 23.9 TCP/IP Stack — package wrapping (commonly tested!)

```
   Application data:                            ┌─────────────────────┐
                                                │      Payload        │
                                                └─────────────────────┘
                                                          │ wrapped by TCP
                                                          ▼
   Transport (TCP):                       ┌──────┬─────────────────────┐
                                          │ TCP  │      Payload        │
                                          │header│                     │
                                          └──────┴─────────────────────┘
                                                          │ wrapped by IP
                                                          ▼
   Internet (IP):                 ┌──────┬──────┬─────────────────────┐
                                  │  IP  │ TCP  │      Payload        │
                                  │header│header│                     │
                                  └──────┴──────┴─────────────────────┘
                                                          │ wrapped by Ethernet
                                                          ▼
   Link (Ethernet):       ┌──────┬──────┬──────┬─────────────────────┬──────┐
                          │ ETH  │  IP  │ TCP  │      Payload        │ ETH  │
                          │header│header│header│                     │footer│
                          └──────┴──────┴──────┴─────────────────────┴──────┘
                                                          │
                                                          ▼ goes on the wire

   READING: outermost = LINK layer (lowest); innermost = APPLICATION data (highest)
   Each layer's header is wrapped INSIDE the next layer's payload.
```

### 23.10 Test Pyramid

```
                          /\
                         /  \              FEW E2E tests
                        / E2E\             - slow (min)
                       /──────\            - brittle
                      /        \           - Playwright, Cucumber
                     / Integ-   \
                    /  ration    \         SOME integration
                   /              \        - moderate speed (sec)
                  /────────────────\       - test multi-unit interactions
                 /                  \
                /        Unit        \     MANY unit tests
               /                      \    - fast (ms)
              /────────────────────────\   - test single function/class
                                          - pytest, JUnit

   Anti-patterns:
   - "Ice cream cone" (many E2E, few unit) = slow CI, brittle
   - "Hourglass" (many unit + many E2E, few integration) = gaps in middle
```

### 23.11 TDD Cycle

```
                  ┌──────────────────┐
              ┌──►│       RED        │   write a failing test FIRST
              │   │ (test must fail) │   - test proves it can detect bugs
              │   └────────┬─────────┘
              │            │
              │            ▼
              │   ┌──────────────────┐
              │   │      GREEN       │   minimum code to pass
              │   │ (make it pass)   │   - cheat if needed
              │   └────────┬─────────┘
              │            │
              │            ▼
              │   ┌──────────────────┐
              │   │     REFACTOR     │   clean up while green
              │   │ (improve design) │   - tests must STAY green
              │   └────────┬─────────┘
              │            │
              └────────────┘   loop for next behavior
```

### 23.12 Fault → Error → Failure

```
   ┌────────────┐         ┌──────────────┐                ┌─────────────────┐
   │   FAULT    │ ──────► │    ERROR     │ ──────► ┌───►  │     FAILURE     │
   │ (bad code) │         │ (wrong state │  reaches│      │ (visible wrong  │
   │            │ causes  │   at runtime)│ boundary│      │   behavior)     │
   └────────────┘         └──────────────┘         │      └─────────────────┘
                                                   │
        WHERE                  WHAT                │            OBSERVED
   (in source code)      (internal state)    (escapes        (user-visible
                                              the module)     output / crash)

   Example — circumference function:
   - FAULT:   input_radius = sys.argv[1]    (string, not converted to float)
   - ERROR:   2 * "10" produces "1010"      (string repetition, not arithmetic)
   - FAILURE: TypeError when multiplied with math.pi  (crash visible to user)
```

### 23.13 Scrum Process

```
   Customer Input
        │
        ▼
   ┌──────────────────────┐
   │   Product Backlog    │     all user stories, prioritized
   │  (User Stories)      │
   └──────────┬───────────┘
              │
              │ Sprint Planning (pick stories)
              ▼
   ┌──────────────────────┐
   │   Sprint Backlog     │     LOCKED for this sprint
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │       SPRINT         │     2-3 weeks of dev
   │   (development)      │  ◄── Daily Standup runs DAILY
   └──────────┬───────────┘     (15 min, all devs)
              │
              ▼
   ┌──────────────────────┐
   │  Product Increment   │     working software at end
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │   Sprint Review      │     demo to stakeholders
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │   Retrospective      │     improve process
   └──────────┬───────────┘
              │
              └──────────────► loop back to Sprint Planning
```

### 23.14 Information Hiding (mental model)

```
   Outside callers
        │
        │   depend ONLY on interface
        ▼
   ┌──────────────────────────────────────┐
   │           INTERFACE                   │  ← public, STABLE
   │     (what the module does)            │     - method signatures
   │     - getBalance(): double            │     - never breaks callers
   │     - withdraw(amount): bool          │
   ├──────────────────────────────────────┤
   │       IMPLEMENTATION                  │  ← hidden, CAN CHANGE
   │       (how it does it)                │     - data structure
   │     - balance stored as double        │     - algorithm
   │     - balance stored in database      │     - external deps
   │     - balance stored in cache         │     - performance tricks
   └──────────────────────────────────────┘

   TEST: "If I change THIS, how many callers break?"
   - 0/1 = secret is well-hidden ✅
   - 2+ = secret has LEAKED ❌
```

### 23.15 Test Doubles Decision Tree

```
                  Does my code WRITE to / CALL the dependency?
                                  │
                ┌─────────────────┼─────────────────┐
                │ YES                               │ NO (READS from dep)
                ▼                                   ▼
       Verify the CALLS happened?        Need realistic behavior?
                │                                   │
        ┌───────┼───────┐                ┌──────────┼──────────┐
        │ YES           │ YES + real     │ NO (fixed)          │ YES (logic)
        ▼               ▼  behavior      ▼                     ▼
      MOCK             SPY             STUB                   FAKE
   (records         (wraps real      (canned                (simplified
    + asserts)       + records)       values)               working impl)
```

### 23.16 Client-Server vs Peer-to-Peer

```
   CLIENT-SERVER (centralized):

        Client ───┐
        Client ───┼─────► [ Server ]      ← single source of truth
        Client ───┘                        ← initiates connection: client
        Client ───┘                        ← all traffic through server

   PEER-TO-PEER (decentralized):

           Peer ───── Peer
            │  ╲    ╱   │
            │   ╲  ╱    │              ← all peers are equals
            │    ╳      │              ← both supplier AND consumer
            │   ╱  ╲    │              ← no central server
            │  ╱    ╲   │
           Peer ───── Peer

   HYBRID example (Zoom):
   - Starts client-server (registration, signaling)
   - 1-on-1 video tries P2P direct (lower latency)
   - Falls back to client-server if P2P fails (NAT, firewall)
```

### 23.17 Diagram-drawing checklist (for exam)

When drawing UML class diagrams under time pressure:
- [ ] Box with 3 sections: Name | Attributes | Methods
- [ ] Abstract class name in ITALICS (or note "<<abstract>>")
- [ ] Visibility prefix on EVERY member: `-` `+` `#` `~`
- [ ] Inheritance = HOLLOW triangle (not filled)
- [ ] Composition (filled diamond) vs Aggregation (hollow diamond)
- [ ] Multiplicity on BOTH ends of association
- [ ] Concrete subclasses implement ALL abstract methods

When drawing sequence diagrams:
- [ ] Names use `:Type` format (e.g., `:Player`, NOT `Player`)
- [ ] Don't include abstract classes (no objects of those!)
- [ ] Vertical dashed line for each lifeline
- [ ] Solid arrow = sync call; dashed arrow = return
- [ ] Match signatures to class diagram
