# Design & Modeling Cheatsheet (candidate items)

## Software views (4 perspectives)
| View | Describes | Diagram |
|---|---|---|
| **Code / Module View** | structure of code | UML Class |
| **Data View** | structure of data | ER Diagram |
| **Run-Time / Component-Connector** | how components connect at runtime | Component diagram |
| **Behavioral View** | interactions over time | UML Sequence, State Machine |

## UML Class Diagram
- Box with three sections: **Name | Attributes | Methods**
- Visibility:
  - `-` private
  - `+` public
  - `#` protected
  - `~` package-private
- Relationships:
  - **Inheritance** — solid line + hollow triangle (Child → Parent)
  - **Association** — solid line (with optional arrow for navigability)
  - **Aggregation** — line + hollow diamond at the whole (weak "has-a")
  - **Composition** — line + filled diamond at the whole (strong "owns-a", lifetime tied)
- Multiplicity: `1`, `0..1`, `*`, `1..*`, `2..5`

## UML Sequence Diagram
- Lifelines = vertical dashed lines per actor/object
- Time goes downward
- Messages = horizontal arrows
- **Synchronous** message — solid arrow with filled head
- **Asynchronous** — solid arrow with open head
- **Return** — dashed arrow
- **Activation bar** — thin rectangle on lifeline when active
- **Fragments**:
  - **alt** — alternatives (if/else)
  - **loop** — iteration
  - **opt** — optional execution

## UML State Machine Diagram
- States (rounded rectangles)
- Transitions (arrows with `event[guard]/action` label)
- Initial state (filled dot), final state (bullseye)

## ER Diagram (Entity-Relationship)
- Entity (rectangle), Attribute (oval), Relationship (diamond)
- Primary key underlined
- Foreign key links across entities
- Cardinality: 1:1, 1:N, M:N

## SOLID Principles
| Letter | Principle | One-line |
|---|---|---|
| **S** | Single Responsibility | A class should have ONE reason to change |
| **O** | Open-Closed | Open for extension, closed for modification |
| **L** | Liskov Substitution | Subclasses must be usable wherever parent is expected |
| **I** | Interface Segregation | Many small interfaces > one big one |
| **D** | Dependency Inversion | Depend on abstractions, not concretions |

### SRP examples
- BAD: `User` class that validates, persists, formats, emails
- GOOD: split into `UserValidator`, `UserRepository`, `UserFormatter`

### OCP — typical refactor
- Add new behavior via NEW subclass / strategy implementation
- DON'T modify existing function to add an `elif`

### LSP — violation example
- `Square extends Rectangle` and overrides `setWidth` to also set height — breaks Rectangle contract

### ISP example
- BAD: one `Worker` interface with `work() + eat()` — robots can't eat
- GOOD: split into `Workable` and `Feedable`

### DIP example
```python
# BAD: directly depends on MySQL
class OrderService:
    def __init__(self): self.db = MySQL()

# GOOD: depends on abstraction
class OrderService:
    def __init__(self, db: Database): self.db = db
```

## Other design principles
- **DRY** — Don't Repeat Yourself
- **KISS** — Keep It Simple, Stupid
- **YAGNI** — You Aren't Gonna Need It
- **Separation of Concerns** — different responsibilities → different modules
- **Law of Demeter** — talk only to immediate friends (`a.b().c().d()` is suspicious)
- **Composition over Inheritance** — prefer "has-a" relationships

## Code smells (refactor triggers)
- Duplicate code
- Long method / large class
- Long parameter list (5+)
- Magic numbers
- Dead code
- Feature envy
- Data clump
- Conditional dispatch on `type` field → polymorphism
- God class
- Shotgun surgery

## Refactoring: Replace Conditional with Polymorphism
**Before:** `if shape.type == "circle": ... elif type == "square": ...`
**After:** abstract `Shape.area()`; each subclass overrides

Aligns with OCP — new shape = new class, no edits to existing code.

## UML quick-draw cheats
- Class: `[ClassName | -attr: Type | +method(): Ret ]`
- Inheritance: arrow points to PARENT
- Association: plain line; arrow if directional
- Aggregation: ◇—— at the WHOLE end (weak)
- Composition: ◆—— at the WHOLE end (strong)
- Multiplicity always at the END of the line near the entity

## Design as collaborative activity
- AVOID Ivory Tower architects (design in isolation, ignore developers)
- Include domain experts, developers, stay close to codebase

## Document design decisions (Google "Design Doc" parts)
1. Context & Scope
2. Goals & Non-Goals
3. The Design (with diagrams)
4. Alternatives (with trade-offs)

## Additional items (potentially missing)

### Other UML diagram types
- **Use Case Diagram** — actors + use cases (stick figures + ovals)
- **Activity Diagram** — flowchart-like, business workflow
- **Component Diagram** — software components and their dependencies
- **Deployment Diagram** — physical nodes + which components run where
- **Package Diagram** — groupings of classes (namespaces)
- **Object Diagram** — instance snapshot of class diagram

### UML class diagram MULTIPLICITY
| Notation | Meaning |
|---|---|
| `1` | exactly one |
| `0..1` | zero or one (optional) |
| `*` | zero or more |
| `1..*` | one or more |
| `n..m` | between n and m |

### Class diagram association arrows
| Symbol | Meaning |
|---|---|
| ───── | Association (no direction) |
| ────► | Directed association (A knows B) |
| ────◇ (hollow diamond) | Aggregation (weak "has-a"; parts can live without whole) |
| ────◆ (filled diamond) | Composition (strong "owns-a"; parts die with whole) |
| ────▷ (hollow triangle) | Generalization / Inheritance (Child → Parent) |
| ┄┄┄┄▷ (dashed + triangle) | Realization (Class → Interface) |
| ┄┄┄┄► (dashed + arrow) | Dependency |

### Sequence diagram extras
- **alt** fragment — if/else alternatives
- **opt** — optional execution
- **loop** — iteration
- **par** — parallel execution
- **ref** — reference another sequence
- **break** — exit fragment on condition

### Composition vs Aggregation memory aid
- **Composition** (filled): "if I die, you die" — Car owns Wheels
- **Aggregation** (hollow): "we're related but independent" — Team has Players

### GRASP principles (related to SOLID)
- **Information Expert** — assign responsibility to the class with the info
- **Creator** — class that should create instances
- **Controller** — class that handles system events
- **Low Coupling** / **High Cohesion**
- **Polymorphism** — assign variation via polymorphism
- **Pure Fabrication** — invented class to achieve low coupling
- **Indirection** — intermediate to decouple
- **Protected Variations** — interface stable; impl can change

### Refactoring triggers (when to refactor)
- Right before adding a feature ("clean up so the change is easier")
- During code review
- When fixing a bug
- When TDD's refactor step
- When onboarding a new team member who finds it confusing

### Composition over Inheritance (rule of thumb)
- Inheritance creates tight coupling
- Composition (has-a) is more flexible
- Modern Java/Python style favors composition + interfaces

### Open-Closed in practice
```python
# Before (modify to add)
def calculate(shape):
    if shape == "circle": ...
    elif shape == "square": ...

# After (extend by adding new class — no modification)
class Shape: def area(self): ...
class Circle(Shape): def area(self): return ...
class Square(Shape): def area(self): return ...
```

### Dependency Inversion in practice
```python
# Before — depends on concrete
class App:
    def __init__(self): self.db = MySQL()

# After — depends on abstraction (injected)
class App:
    def __init__(self, db): self.db = db
# now App works with MySQL, Postgres, FakeDB for tests
```

### Code smell catalog (for design-review questions)
- Duplicate code (DRY violation)
- Long method (>30 lines)
- Large class (>500 lines)
- Long parameter list (>4 args)
- Divergent change (one class changed for many reasons — SRP violation)
- Shotgun surgery (one change → many files — coupling)
- Feature envy (method uses another class's data more than own)
- Data clumps
- Primitive obsession (using strings/ints where small classes would be clearer)
- Switch statements (often → polymorphism)
- Comments (when code can self-document)
- Speculative generality (over-engineering)

### Design quality attributes
- **Modifiability** — easy to change
- **Reusability** — usable in multiple contexts
- **Testability** — easy to test in isolation
- **Performance**
- **Security**
- **Scalability**
- **Availability**
- **Maintainability**
