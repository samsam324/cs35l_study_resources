# Code Comprehension Cheatsheet (candidate items)

## Why
- Developers spend **~58% of time** reading code (not writing)
- Skill gets MORE valuable with AI-generated code

## Two reading strategies
| | Bottom-Up (novice) | Top-Down (expert) |
|---|---|---|
| Approach | line-by-line | hypothesis-driven |
| Speed | slow | fast |
| Order | small → big | big → small |
| Requires | nothing | domain knowledge |

> Goal: **move from Bottom-Up toward Top-Down** as expertise grows

## The 3 enablers for top-down
1. **Build Domain Knowledge** — read docs, architecture, system summaries BEFORE the code
2. **Hypothesis-Driven Reading** — form primary + subsidiary hypotheses BEFORE reading
3. **Hunt for Beacons** — scan for the 5 beacon types

## The 5 Beacon types
| Type | What to look for |
|---|---|
| **Identifier Names** | Variable / function / class names reveal DOMAIN |
| **Code Structures** | Loops, recursion, guard clauses, map/reduce shape |
| **Tests** | Tests = executable spec; read tests first |
| **Assertions** | preconditions / postconditions / invariants |
| **Diagrams** | UML diagrams build top-down mental model (may be outdated) |

## Beacon 1: Identifier Names (most critical)
Domain by naming examples:
- `principal`, `interest_rate`, `term_months`, `apr` → **Finance / loans**
- `items`, `taxRate`, `subtotal`, `onPlaceOrder` → **E-commerce**
- `token`, `verifyJwt`, `401`, `Unauthorized` → **Auth / web security**
- **Tip:** if you don't recognize a name (APR, JWT) — LOOK IT UP. Domain knowledge gap.

## Beacon 2: Code Structures (recognize algorithm by shape)
Common shapes:
- **Guard clause** — `if bad: return` at top
- **Recursive** — function calls itself + base case
- **Map/filter/reduce** — chained array methods
- **Two-pointer** — `lo, hi` indexes converging
- **Sliding window** — `left`/`right` pointers maintaining range
- **DFS** — `stack.append(neighbor)` after `pop`
- **BFS** — `queue.popleft()` after `append`
- **Binary search** — `lo, hi, mid = (lo+hi)//2`
- **Retry loop** — bounded while with try/catch

## Beacon 3: Tests
- Tests are EXECUTABLE SPECS
- Read tests BEFORE production code
- Verify tests pass on current code first (they may be outdated)

```python
def test_no_discount():     assert apply(100, 0) == 100
def test_half_off():        assert apply(100, 50) == 50
def test_rejects_negative():with pytest.raises(ValueError): apply(100, -5)
```
Spec: `apply(price, percent)` returns discounted price; rejects negatives.

## Beacon 4: Assertions
- `assert amount > 0` → precondition
- `assert account.balance >= amount` → precondition
- `assert account.balance >= 0` → postcondition / invariant
- Tells you what inputs would crash + what's guaranteed after

## Beacon 5: Diagrams
- UML class / sequence / state machine
- Look FIRST when entering codebase
- ⚠️ may be outdated — trust the code over the diagram

## The Continuous Cycle of Hypothesis Testing
```
Hypothesis Generation
        ↓
    Beacon Hunting
        ↓
 ┌──────┴──────┐
 ↓             ↓
Found         Missing
 ↓             ↓
Verify        Reject + adjust
        ↓
   [loop back]
```

## Opportunistic Processing
- Experts toggle between strategies fluidly
- **Top-down** for navigation + global picture
- **Bottom-up** as a precision tool — drop in for complex chunks, return to top-down

## Reading checklist (apply to every snippet)
1. Read function/variable/class **names first**
2. Identify **domain** from names
3. Identify **structural shape**
4. Form **one-sentence hypothesis**
5. **Read selectively** to confirm
6. If lost, drop to bottom-up FOR THAT CHUNK only

## Exam application
For each code snippet:
- "What does this do?" → identifier beacons + shape
- "Predict output" → trace selectively after hypothesis
- "What's the bug?" → fault/error/failure thinking
- "Which design principle violated?" → code smell recognition
- "What domain is this?" → identifier names

## Common shapes — memorize
| Shape | Signature |
|---|---|
| Linear search | for loop + early return on match |
| Binary search | lo/hi/mid pointers, halving |
| BFS | queue + visited set |
| DFS | stack/recursion + visited set |
| Two-pointer | left + right indexes |
| Sliding window | left + right bounds |
| Sort + Two-pointer | `arr.sort()` then lo/hi inside loop |
| Memoization | `cache` dict, early return on cache hit |
| Map/reduce | `.map(...).reduce(...)` |
| Topological sort | indegree count + queue |

## Tips for the exam
- Don't trace line-by-line on first read
- Identify the algorithm first (5-second pattern recognition)
- Only trace details if needed to verify hypothesis
- Practice the two tutorials linked by the prof

## Additional items (potentially missing)

### Practice the algorithm from sight — extended

| Shape | Cues |
|---|---|
| Sorting | nested loops + swap, or partition function |
| Linked list traversal | `while node:` + `node = node.next` |
| Stack-based | explicit stack + push/pop |
| Queue-based | explicit queue / deque + appendleft/popleft |
| Dynamic programming | `dp[i] = ...` table, often 2D |
| Memoization | `cache` dict + early return on hit |
| Graph traversal (DFS) | recursion + visited set, or stack |
| Graph traversal (BFS) | queue + visited set |
| Topological sort | indegree count + queue |
| Union-Find | parent[] array, find/union ops |
| Bit manipulation | `&` `|` `^` `<<` `>>` patterns |
| Mathematical | factorial / gcd / modular arithmetic |

### Identifier "domain dictionary" — quick recognition
| Identifier | Domain |
|---|---|
| `principal, apr, interest` | Finance/loans |
| `cart, items, taxRate, checkout` | E-commerce |
| `token, jwt, oauth, 401, unauthorized` | Auth/security |
| `node, parent, children, leaf` | Tree structure |
| `vertex, edge, adjacency` | Graph |
| `inorder, postorder, preorder` | Tree traversal |
| `lo, hi, mid, target` | Binary search |
| `dp, memo, cache` | DP |
| `request, response, headers, body` | HTTP/REST |
| `query, params, rows, columns` | SQL/DB |
| `pixel, rgb, canvas, draw` | Graphics |
| `socket, port, host, listen` | Networking |
| `train, model, predict, loss` | ML |
| `event, emit, listener, subscribe` | Event-driven |

### Patterns of bad code (smell recognition)
- Many `if/elif` on `.type` field → use polymorphism
- Many nested `if`s → guard clauses / early return
- Same logic in multiple branches → extract method
- Magic numbers → named constants
- Long parameter lists → object param
- Function > 30 lines → split it

### Practice strategies
- Read OPEN-SOURCE code (small libraries, ~500 lines)
- Each week: read a function you've never seen, predict + verify
- Track your accuracy over time

### Read tests FIRST when entering a codebase
- Tests are the spec
- They show happy path + edge cases
- They show how to USE the API

### Common gotchas — code that LOOKS like one thing but isn't
- Python `default arg = []` mutation
- Java `==` on Integer / String
- JS `==` vs `===`
- C `if (a = 0)` is assignment, always false
- Shell `[ "$x" = "y" ]` needs spaces around `=`

### Reading tip: name → hypothesis → confirm
For each function:
1. Read the name
2. Guess what it does
3. Skim for confirming beacons
4. Trace only if hypothesis wrong

### Bottom-up triggers (drop to line-by-line when...)
- Unfamiliar APIs / domain
- Complex math
- Tricky concurrency
- Bit manipulation
- Performance-critical inner loops

### Exam-time strategy
- Don't trace line-by-line until you've hypothesized
- Spend 5-10 sec on names + shape
- If hypothesis confident, only verify key lines
- If lost, drop to bottom-up for that snippet

### Common output prediction errors
- Mishandling `==` semantics
- Wrong scope (LEGB in Python)
- Async timing (JS event loop)
- Off-by-one in loops
- Mutating shared state
- Boundary cases (empty input, single elem)

### The "what would I expect" trick
- Read function name + signature
- Write down YOUR expected behavior
- Then read code; mismatches reveal bugs OR your misunderstanding
