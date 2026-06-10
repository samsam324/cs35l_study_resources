# Clean Code Cheatsheet (candidate items)

## Definitions from the lecture

> "Clean code can be read, and enhanced by a developer other than its original author." — **Dave A. Thomas** (godfather of Eclipse IDE)

> "Clean code always looks like it was written by someone who cares." — **Michael Feathers** (Working Effectively with Legacy Code)

> "You know you are working on clean code when each routine you read turns out to be pretty much what you expected." — **Ward Cunningham** (inventor of the Wiki)

> "Clean code is simple and direct. Clean code reads like well-written prose. Clean code never obscures the designer's intent but rather is full of crisp abstractions and straightforward lines of control." — **Grady Booch** (co-inventor of UML)

> **Instructor's definition:** "Clean code effectively convinces the reader that the code is likely free of bugs and gives contributors the confidence that everything will be fine. Clean code never betrays this trust."

## Why clean code matters
- Cost of fixing bugs grows over time
- Most dev time is reading existing code
- Clean code = lower maintenance cost + lower onboarding cost

## "Broken Window" effect
- Once code starts to look messy, devs let other parts get messy
- Maintain cleanliness deliberately

## Comparison of overlapping concepts
| Principle | Focus |
|---|---|
| **Information Hiding** | Hide design decisions |
| **Separation of Concerns** | Each module = one responsibility |
| **SOLID** | 5 design principles for OO |
| **Clean Code** | Readable, maintainable code at line/function level |

---

## The 4 Pillars of Clean Code

### 1. Meaningful Naming
- **Intention-revealing names** — `daysSinceLastBackup` not `d`
- **Avoid disinformation** — don't name a list `accountList` if it's a `Set`
- **Make meaningful distinctions** — `productInfo` vs `productData` is bad (no real difference)
- **Pronounceable names** — `genymdhms` is unreadable
- **Searchable names** — `MAX_RETRIES` not `5`
- **Avoid encodings** — Hungarian notation (`strName`) is dated
- **Class names = nouns** — `Customer`, `Order`
- **Method names = verbs** — `save`, `calculateTax`

### Examples
```python
# BAD
def f(d): return d * 86400
# GOOD
def days_to_seconds(days): return days * SECONDS_PER_DAY
```

### 2. Simplified Structure
- **Guard clauses** — early return on bad input
  ```python
  def process(x):
      if x is None: return None       # guard
      # main logic
  ```
- **Reduce nesting** — flatten via early returns
- **Cognitive load** — keep functions short, single-purpose
- **Chunking via method extraction** — break long methods into named pieces
- **Single Responsibility per function**
- Functions should do ONE thing

### 3. Purposeful Documentation
- **Comments explain WHY, not WHAT**
  - BAD: `i++; // increment i`
  - GOOD: `i++; // skip header row`
- **Self-documenting code first** — good names reduce need for comments
- **Document non-obvious decisions** — workarounds, performance trade-offs
- **Don't comment out code** — delete it (version control remembers)
- **Update comments** when code changes (stale comments are worse than none)

### 4. Design by Contract
*(Bertrand Meyer / Eiffel)*

- **Preconditions** — what the CALLER must satisfy
- **Postconditions** — what the METHOD guarantees on completion
- **Invariants** — what must ALWAYS hold (class-level)

Example:
```python
def withdraw(account, amount):
    assert amount > 0                  # precondition
    assert account.balance >= amount   # precondition
    account.balance -= amount
    assert account.balance >= 0        # postcondition / invariant
    return account.balance
```

**Assertions in different languages:**
- Python: `assert x > 0` (disabled with `python -O`)
- C/C++: `assert(x > 0);` (disabled with `-DNDEBUG`)
- Java: `assert x > 0;` (disabled by default! enable with `-ea`)
- JS: `console.assert(...)` or `if (!cond) throw ...`

---

## "Likely Free of Bugs" — prioritize simplicity
- Simple code → readers can verify correctness
- Complex code → bugs hide
- "When in doubt, choose the simpler design"

## Boy Scout / Girl Scout Rule
> "Leave the campground cleaner than you found it."

When you touch a file:
- Improve a name
- Extract a method
- Delete dead code
- Add a missing assertion

Small improvements accumulate.

## Anti-patterns to flag
- Magic numbers (use named constants)
- Long parameter lists (5+ args — group into object)
- Deep nesting (3+ levels — extract methods, use guards)
- Dead code
- Commented-out code
- Empty `catch` blocks
- Returning `null` instead of an empty collection
- Boolean flag parameters (`process(true, false, true)` — unreadable)

## Good practices summary
- One function = one thing
- One file = one logical unit
- Names communicate intent
- Comments explain WHY
- Assertions document contracts
- Keep nesting shallow
- Eliminate duplication

## Test-Case Version A vs Version B
The lecture shows side-by-side comparisons:
- Version A: complex, nested, magic numbers, no comments
- Version B: same behavior, clean structure, named constants, guards, contracts
- Reviewer's confidence in B >> A

## Quick checklist before merging
- [ ] Are all names intention-revealing?
- [ ] Any methods > 30 lines?
- [ ] Nesting > 3 levels?
- [ ] Magic numbers?
- [ ] Comments explain WHY?
- [ ] Assertions on preconditions / postconditions?
- [ ] Dead code?
- [ ] Duplicate logic?

## Additional items (potentially missing)

### Naming heuristics (Robert C. Martin)
- Use intention-revealing names
- Avoid disinformation (don't name a Set as `accountList`)
- Make meaningful distinctions
- Use pronounceable names (`genYmdHms` → `generationTimestamp`)
- Use searchable names (don't use single-letter for non-trivial vars)
- Avoid Hungarian notation (`strName`, `intCount`)
- Class names = nouns; method names = verbs
- One word per concept (don't use `fetch`/`retrieve`/`get` interchangeably)
- Use solution domain names (algorithms, data structures)
- Use problem domain names when no programmer term exists

### Function rules
- Functions should be SMALL (~20 lines max)
- Do ONE thing
- One level of abstraction per function
- Use descriptive names (long is OK if clear)
- Few arguments (0 ideal, 3+ is suspect)
- Avoid flag arguments (`render(true, false)`)
- Avoid side effects (functions should EITHER mutate OR return)
- Command-Query Separation:
  - Command — does something, returns nothing
  - Query — returns something, no side effects

### Bad comment types
- **Mumbling** — comments that aren't clear
- **Redundant** — comments restating the code
- **Misleading** — out of date
- **Mandated** — required but uninformative (`@param x x`)
- **Journal** — change log (use git instead)
- **Noise** — `// Default constructor`
- **Position markers** — `// === SECTION ==='` (signal of long file)
- **Closing brace** comments — `} // end if`
- **Commented-out code** — DELETE IT

### Good comment types
- **Legal** comments (copyright)
- **Informative** (regex explanation)
- **Explanation of intent** (why a workaround)
- **Clarification** of obscure code (when you can't simplify)
- **Warning** of consequences (`// Don't run this on prod!`)
- **TODO** comments (clear and tracked)
- **Amplification** of importance

### Error handling
- Use exceptions (in OO languages), not error codes
- Provide context with exceptions (what failed and why)
- Define exception types based on caller's needs
- Don't return null (use empty collection / Optional)
- Don't pass null
- Fail fast (assert early)

### Boundaries
- Wrap third-party libraries to insulate your code
- This lets you swap them out + test independently

### Classes
- Small (under ~50 methods)
- Single Responsibility (one reason to change)
- High cohesion (members work together)
- Open for extension, closed for modification (OCP)

### Code smells reference
| Smell | Refactor |
|---|---|
| Duplicate code | Extract Method / Class |
| Long method | Extract Method |
| Large class | Extract Class |
| Long param list | Introduce Parameter Object |
| Divergent change | Extract Class |
| Shotgun surgery | Move Method/Field |
| Feature envy | Move Method |
| Data clumps | Extract Class |
| Primitive obsession | Replace with Class |
| Switch statements | Replace with Polymorphism |
| Lazy class | Inline Class |
| Speculative generality | Inline Class / Remove Param |
| Temporary field | Extract Class |
| Message chains | Hide Delegate |
| Middle man | Remove Middle Man |
| Inappropriate intimacy | Move Method / Extract Class |

### The Single Responsibility test
Ask: "What's the ONE reason this would change?" If you can list multiple → SRP violation.

### DRY done wrong (cautionary)
- Don't deduplicate code that's COINCIDENTALLY similar
- Two pieces that look alike but represent different concepts → keep separate (they'll diverge)

### KISS (Keep It Simple, Stupid)
- Choose the simplest design that works
- Don't add abstraction until you have 3+ use cases

### YAGNI (You Aren't Gonna Need It)
- Don't build for hypothetical future requirements
- Build for today; refactor later

### Boy/Girl Scout Rule restated
"Leave the campground cleaner than you found it." When touching a file:
- Rename a confusing identifier
- Extract a magic number
- Add a missing assertion
- Delete dead code

### Assertions across languages
| Lang | Syntax | Disable |
|---|---|---|
| Python | `assert x > 0` | `python -O` |
| C/C++ | `assert(x > 0)` | `-DNDEBUG` |
| Java | `assert x > 0` | enabled with `-ea`, off by default! |
| JS | `console.assert(x > 0)` | no off switch (prints only) |
| Rust | `assert!(x > 0)` | `--release` (debug_assert) |

### Design by Contract recap (Bertrand Meyer / Eiffel)
- **Precondition** — caller's responsibility
- **Postcondition** — function's guarantee
- **Invariant** — class-level property that always holds
- Document explicitly + enforce with assertions
