# Managing Complexity Cheatsheet (candidate items)

## Why complexity matters
- The **Software Crisis** (1968 NATO conf) — projects unmanageable
- Modern systems: millions of LOC
- Complexity → harder to modify, more bugs, slower onboarding

## Big Ball of Mud (anti-pattern)
- No discernible structure
- Symptoms:
  - **Modifiability** problems — small change = many file edits
  - **Understandability** problems — no one knows how it works
  - **Fragility** — change one thing, three other things break

## Information Hiding Principle (Parnas 1972)
> "We propose [...] that one begins with a list of difficult design decisions or design decisions which are likely to change. Each module is then designed to hide such a decision from the others."

- One of THE most important concepts in CS
- The job of a module is to HIDE design decisions that might change
- Module changes are LOCAL when its secrets are well-hidden

## Module = Interface + Implementation
- **Interface** = stable contract (WHAT the module does) — should be visible
- **Implementation** = HOW it fulfills the contract — should be hidden
- Implementation can change freely as long as interface stays the same

## Power outlet analogy
- Wall outlet = interface (110V, 60Hz, 2 holes + ground)
- Power grid behind it = implementation (could be coal/solar/nuclear)
- Devices don't care about implementation; they depend on interface

## Deep vs Shallow modules (Ousterhout)
- **Deep module** = small interface hiding lots of functionality (good)
- **Shallow module** = large interface for little functionality (bad — leaks complexity)
- Target: small interface, big internal complexity

## Coupling and Cohesion
| Goal | Definition |
|---|---|
| **Low Coupling** | Few connections between modules |
| **High Cohesion** | Each module does ONE thing well |

## Syntactic vs Semantic dependencies
- **Syntactic** — direct code reference (import, call)
- **Semantic** — implicit assumption (e.g., relying on undocumented behavior)
- Semantic deps are dangerous; they're invisible until they break

## Modularity principles
- **Single Choice Principle** — each design decision lives in ONE place
- **DRY** (Don't Repeat Yourself) — duplication = duplicate maintenance burden
- **Single Responsibility** — class/module changes for ONE reason

## Change Impact Analysis
- Before making a change, ask: "If I change this decision, how many places must I edit?"
- Answer > 1 → the secret has leaked → module isn't hiding properly

## The encapsulation/info hiding distinction
- **Encapsulation** = MECHANISM (private fields, getter/setter)
- **Information Hiding** = PRINCIPLE (hide *what might change*)

## Getter/Setter Fallacy
```java
class Book {
    private int isbn;
    public int getIsbn() { return isbn; }
    public void setIsbn(int isbn) { this.isbn = isbn; }
}
```
- Encapsulated, but hides NOTHING
- Change ISBN to String? Every caller breaks
- Better: expose BEHAVIOR (`isValidIsbn()`, `formatForDisplay()`), not data

## Test for hiding
For each design decision in the module:
- "If this changes, how many classes break?"
- If > 1 → secret leaked

## Making design decisions
1. **Generate many alternatives** — research: more alternatives = better outcomes
2. **Delay decisions** that need more info
3. **Solve simpler problems first** — extend the solution later
4. **Rational vs intuitive** decision making (combine both)

## Google Design Docs (4 parts)
1. **Context & Scope** — background facts
2. **Goals & Non-Goals** — what to do / what to skip
3. **The Design** — selected solution (context diagram, data model, APIs)
4. **Alternatives** — what else was considered, trade-offs, why chose this one

## SOLID applied to managing complexity
- All 5 SOLID principles serve complexity reduction
- Especially: Single Responsibility, Open-Closed

## Other complexity-reducing patterns
- **Layered architecture** (e.g., TCP/IP stack)
- **Hexagonal / ports-and-adapters** architecture
- **Domain-Driven Design** — model the business domain
- **Microservices** — split by domain (trade-off: distributed complexity)

## Quick mental model
- Complex code = code with too many things to track at once
- Information hiding REDUCES what readers need to track
- A good module: you can use it without knowing how it works inside

## Additional items (potentially missing)

### Conway's Law
> "Organizations design systems that mirror their communication structure."
- Team boundaries become system boundaries
- Plan org structure to match desired architecture (or vice versa)

### Cyclomatic Complexity
- Counts independent execution paths
- Roughly: 1 + (number of if/else/for/while/case/&&/||)
- Tools (e.g., radon, sonarqube) report it
- Aim for <10 per function; refactor when higher

### Cognitive load
- # of things reader must track at once
- Reduce by: smaller functions, fewer params, better names, clear control flow

### Modularity benefits
- **Comprehensibility** — easier to understand smaller pieces
- **Independent development** — work in parallel
- **Testability** — test in isolation
- **Reuse** — pluggable modules
- **Changeability** — local changes

### What makes a "deep module"
- Small interface (few params, simple return)
- Large implementation hidden behind it
- e.g., `printf` — single function, complex internals
- Counter-example: shallow file API where you do EVERY step explicitly

### What makes a "shallow module"
- Big interface, little internal logic
- Leaks complexity to caller
- Adds little abstraction value

### Abstraction layers
- Each layer hides details from layers above
- TCP/IP, OS abstractions, ORMs, etc.
- Wrong abstraction is worse than no abstraction

### Layered architecture
```
Presentation (UI)
    |
Application (use cases)
    |
Domain (business logic, entities)
    |
Infrastructure (DB, network, FS)
```
Dependencies point INWARD only (clean architecture).

### Hexagonal / Ports & Adapters
- Core business logic at center
- Ports (interfaces) define how to interact
- Adapters implement ports for specific tech (DB, HTTP, CLI)

### Information hiding principle (Parnas restated)
- Start with: "what design decisions are LIKELY TO CHANGE?"
- Each module hides ONE such decision
- When that decision changes, only one module changes

### Examples of "secrets to hide"
| Secret | Why it might change |
|---|---|
| Data structure choice | Array vs hash map for perf |
| Storage backend | MySQL vs Postgres vs Mongo |
| Algorithm | bubble sort → quicksort |
| API vendor | Stripe vs PayPal |
| Output format | JSON vs XML vs MessagePack |
| Auth method | session vs JWT |
| Caching strategy | LRU vs LFU |

### "What" vs "How" rule
- Interface tells WHAT (stable)
- Implementation tells HOW (changes freely)
- If callers know HOW, they're coupled to it

### Test for good info hiding
- "If I change THIS, how many files do I edit?"
- 1 = excellent hiding
- 2-3 = OK
- 10+ = leaky

### Common complexity-induced bugs
- Cascading effects (small change → many breaks)
- Inconsistent updates (forgot one of the dependent files)
- Race conditions (more code paths → more interaction)
- Difficulty reasoning about state

### Decision: when to split a module
- Symptoms: long file, many concerns, conflicting changes
- Tip: extract by THE WHY of each piece
- Don't split prematurely; YAGNI applies

### Decision: when to merge modules
- Symptoms: lots of "feature envy", artificial separation, modules that always change together
- Sign that they should be ONE thing

### Quality attribute trade-offs
- Modularity vs Performance (calls have cost)
- Flexibility vs Simplicity (more options = more complexity)
- Generality vs Specificity (general tools are slower/heavier)
- Abstraction vs Discoverability (more layers = harder to find code)

### Tools to measure complexity
- Cyclomatic complexity (cc)
- Lines of code (LOC) per function
- Number of arguments
- Depth of nesting
- Halstead metrics
- Maintainability index
