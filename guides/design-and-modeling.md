# CS35L Lecture 7 â€” Design & Modeling

Source: `L7-ModelingDesignPrinciples.pdf` (Tobias DÃ¼rschmid, UCLA CS Department)

---

## 1. Where Design Fits in Software Engineering Activities

The full lifecycle (recap):

| Activity | Question Answered |
|---|---|
| **Requirements Analysis** (driven by *User Stories*) | What should be built? |
| **Design** | How should we build it? |
| **Development** | Build it |
| **Testing** | Did we build the right system? |
| **Deployment** | Deliver to customer |

Today's topic = **Design**.

---

## 2. Notes on the Team Project (Process Context)

- **Start coding now.** Use **Pair Programming**:
  - Two developers use the same computer physically or via Zoom screen sharing to code together.
  - Works especially well for beginners.
  - Talking aloud about what you are doing can help you find issues in your design.
  - Having someone look over your shoulder means you find bugs more easily.
  - Knowledge is exchanged between team members.
- Every individual team member submits a **weekly progress report** on GradeScope (Fridays starting this week):
  - What have you done this week?
  - How did you apply concepts taught in the lectures to your project?

---

## 3. Why Design Matters â€” "Load-Bearing Walls"

**Common misconception about software**: *"Just keep coding. We can always fix it later."*

- Early design decisions often become **"load-bearing walls"** of your app â€” they are hard to change later.
- E.g., once you start implementing a **peer-to-peer architecture**, it's hard to transform into a **client-server architecture**.
- Spend some time thinking about design *before* starting to code.
- The more experience you have, the easier it becomes to identify which design decisions are "important."

**Great Software Engineers Communicate Their Ideas Well.** Without a shared model, every team member imagines a different system (e.g., one thinks "skyscraper," another thinks "mall," another thinks "house," another thinks "dog house") â€” modeling/diagrams put everyone on the same page.

---

## 4. The Four Software Views

Different views visualize different aspects of software. Reference: *Documenting Software Architectures: Views and Beyond* by Clements et al. (available in UCLA digital library).

| View | What It Describes | Example Diagram |
|---|---|---|
| **Code View / Module View** | Structure of Code | UML Class Diagram (e.g., `Shape` / `Rectangle` inheritance) |
| **Data View** | Structure of Data | Entity-Relationship Diagram (e.g., `Student` â€”studyâ€” `Course`, with `UID` attribute) |
| **Run-Time View / Component-Connector View** | How Run-Time Components are Connected | Component diagram with ball-and-socket connectors (e.g., `Client` <-> `Library Server`) |
| **Behavioral View** | Interactions | UML Sequence Diagram (e.g., `client_1: Client` <-> `server: LibraryServer`), UML State Machine Diagram |

### Summary Table (Usefulness of Each View)

| View | Describes | Useful For |
|---|---|---|
| Code View / Module View | Structure of Code | Code-level design discussions & documentation (can be inferred automatically from code) |
| Data View | Structure of Data | Structuring database tables |
| Behavioral View | Interactions | Reasoning about bottlenecks, messaging protocols, and complex interactions |

---

## 5. UML Class Diagrams (Code View)

Reference: https://tobiasduerschmid.github.io/SEBook/uml_class_diagram.html

### 5.1 Anatomy of a Class Box

A class is drawn as a rectangle divided into three compartments:

1. **Class name** (top). *Italicized* = abstract class.
2. **Attributes** (middle), with visibility marker + name + `:` + type.
3. **Functions / Methods** (bottom), with visibility marker + name + parameters + return type.

**Visibility markers:**
- `-` private
- `+` public
- `#` protected (standard UML; in this course `-`/`+` are the primary ones shown)

### 5.2 Example â€” Inheritance (Is-a)

```
   Shape                          <-- Abstract Class name (italic)
   -color: int                    <-- Attribute (private)
   +set_color(r: int, g: int, b:int)   <-- Method (public)
      ^
      |  (open white triangle = Inheritance / Is-a)
      |
   Rectangle                      <-- Class name
   -width: int
   -length: int
   +set_width(width: int)
   +set_height(height: int)
```

### 5.3 Python Code for the Inheritance Example

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    def __init__(self):
        self.__color: int = 0   # private  (-color)

    def set_color(self, r: int, g: int, b: int) -> None:
        self.__color = (r << 16) | (g << 8) | b

class Rectangle(Shape):
    def __init__(self):
        super().__init__()
        self.__width: int = 0    # private  (-width)
        self.__length: int = 0   # private  (-length)

    def set_width(self, width: int) -> None:
        self.__width = width

    def set_height(self, height: int) -> None:
        self.__length = height
```

Notice the mapping: `-color` UML attribute <-> `self.__color` Python attribute (double-underscore = name-mangled / private).

### 5.4 Association â€” A Relationship Between Classes

When a class has a reference to another class (instead of a primitive), draw an **Association** line. The role/field name (e.g., `-color`) is written along the line on the target end. Multiplicity (e.g., `1`, `*`, `0..1`, `1..*`) can be added at the ends.

Example with `Shape` referencing a `Color` object:

```
  Shape  <>------ -color ----->  Color
                                  -color: int
                                  +set_color(r: int, g: int, b:int)
  +set_color(r: int, g: int, b:int)
```

Python code for the Association example:

```python
from abc import ABC, abstractmethod

class Color:
    def __init__(self, r: int, g: int, b: int):
        self.set_color()

    def set_color(self, r: int, g: int, b: int) -> None:
        self.r: int = r
        self.g: int = g
        self.b: int = b

class Shape(ABC):
    def __init__(self):
        self.__color: Color = Color(0, 0, 0)   # (-color)

    def set_color(self, color: Color) -> None:
        self.__color = color
```

### 5.5 Kinds of Associations (Arrows / Diamonds)

| Symbol | Name | Meaning | Examples |
|---|---|---|---|
| `------>` (plain arrowhead) | **Navigable Association** | The left object has a reference to the right object | Employee <-> Boss, Vote -> Politician |
| `------X` (line with X) | **Non-Navigable Association** | The left object does **not** have a reference to the right object | Voter X- Vote, Account X-X ClearTextPassword |
| `<>-----` (open diamond at the owner end) | **Aggregation (has-a)** | The left object "owns" the right object | Car <>- Wheel, Library <>- Book |
| `<#>-----` (filled diamond at the owner end) | **Composition (has-a)** | Same as aggregation, **plus** the right object's existence depends on the left â€” it cannot exist without the whole | Person <#>- Head, House <#>- Room |

Notes:
- Navigability arrows tell you which side holds the reference (data flow direction in memory).
- Aggregation vs. Composition: both are "has-a" / "whole-part," but composition implies *lifetime ownership* (destroy whole => destroy parts).

---

## 6. Entity-Relationship Diagrams (ERDs) â€” Data View

Used to model **databases** / data structure.

### 6.1 ERD Building Blocks

| Symbol | Meaning |
|---|---|
| Rectangle | **Entity** (e.g., a database table) |
| Diamond | **Relationship** (between entities) |
| Oval / Ellipse | **Attribute** (of an entity or relationship) |
| Label on connecting line (`1`, `N`, `M`) | **Cardinality** |

### 6.2 Cardinalities

Common cardinality patterns on a relationship:
- **1-1** (one-to-one)
- **1-N** (one-to-many)
- **N-1** (many-to-one)
- **N-M** (many-to-many)

### 6.3 Worked Example

```
   (First name) (Last name)                                   (Number) (Title)
        \         /                                              \       /
         \       /                                                \     /
        +--------+         N    /------\    M         +----------+
        | Student|------------<  study  >------------|  Course  |
        +--------+              \------/             +----------+
        /        \                                              \
      (UID)   (Birthyear)                              /-------------\
                                                     <  PreRequirement >  (self-relationship on Course)
                                                      \-------------/
```

- `Student` and `Course` are **entities**.
- `study` is a **relationship** between them with cardinality **Nâ€“M** (a Student studies many Courses; a Course is studied by many Students).
- `First name`, `Last name`, `UID`, `Birthyear` are **attributes** of Student.
- `Number`, `Title` are **attributes** of Course.
- `PreRequirement` is a relationship (a Course is a prerequisite for another Course).

---

## 7. UML Sequence Diagrams â€” Behavioral View

Reference: https://tobiasduerschmid.github.io/SEBook/uml_sequence_diagram.html

Sequence diagrams **show examples of interactions between objects / component instances**.

### 7.1 Heads (Lifelines)

Each participant is drawn as a labeled rectangle at the top, with a dashed vertical **lifeline** descending. The label format is:

```
instance_name : ComponentTypeName
```

Example: `client_1 : Client`  and  `server : LibraryServer`.

### 7.2 Message Arrows

| Arrow | Meaning | Notes |
|---|---|---|
| Solid line with filled arrowhead `----->` | **Synchronous Call** | Caller **blocks** until response received |
| Dashed line with open arrowhead `- - ->` | **Response to Synchronous Call** | Goes back to the caller |
| Solid line with open (line) arrowhead | **Asynchronous Message** | **Non-blocking** communication |
| Arrow starting at a solid black dot | **Found Message** | The sender is unknown or irrelevant |

### 7.3 Activation Bars

- A thin tall rectangle drawn on a lifeline = **"activation bar"**.
- It indicates that the component is **executing a task**.
- **Activation bars can be stacked** to indicate that the component starts a sub-process or calls a sub-component.

Example: when `client_1: Client` sends a synchronous `getDBRecord(userID)` to `server: LibraryServer`, the server's activation bar appears, and a nested smaller activation bar on top of it can show an internal sub-call (e.g., a self-call back to itself).

### 7.4 Combined Fragments (Showing Multiple Execution Paths)

Sequence diagrams support **fragments** â€” labeled boxes around portions of the sequence â€” to show conditional/looping behavior:

**(a) `Alt` â€” Alternatives**
- Drawn as a box labeled `Alt` in the top-left tag with multiple compartments separated by dashed horizontal lines.
- Each compartment is guarded by a `[condition]`.
- **Alt executes exactly one of the cases based on their conditions.**
- Example: client sends `GET /book/id`; then `Alt [book found]` returns `responseCode=200, book`; `[else]` returns `responseCode=404`.

**(b) `Loop` â€” Repetition**
- Box labeled `Loop` with a `[condition]` guard.
- **Loop gets executed as long as the condition is true.**
- Example: `Loop [for each book]` containing `POST /book/{id}` followed by a response.

**(c) `Opt` â€” Optional**
- Box labeled `Opt` with a `[condition]` guard.
- **Opt gets executed if and only if the condition is true.**
- Example: `Opt [login successful]` containing `GET /borrowed_books/{user_id}` and its response.

---

## 8. UML State Machine Diagrams â€” Behavioral View

Reference: https://tobiasduerschmid.github.io/SEBook/uml_state_diagram.html

### 8.1 Elements

- **States** are drawn as rounded rectangles or circles with a name (e.g., `Initial State`, `Next State`).
- **Initial pseudo-state**: a solid filled black dot with an arrow into the starting state.
- **Transitions**: directed arrows labeled with `[condition] trigger / action` (any part is optional).
- **Self-transition**: arrow looping from a state back to itself.

### 8.2 Transition Label Syntax

```
[guard condition] trigger / action
```

- `[condition]` â€” guard; transition fires only if the condition is true.
- `trigger` â€” event that causes the transition (e.g., `button`).
- `action` â€” side effect executed when transitioning (e.g., `i = i+1`, `alarm`).

### 8.3 Complex States (with Entry / Exit Actions)

A complex state has internal compartments:

```
+-------------------------+
|      Complex State      |
+-------------------------+
| Entry / i = 0           |
| Exit  /                 |
+-------------------------+
```

- `Entry / <action>` â€” runs whenever the state is entered.
- `Exit / <action>` â€” runs whenever the state is exited.

### 8.4 Worked Example

States: `Initial State`, `Next State`, `Complex State`.

Transitions:
- `(black dot)  -->  Initial State` (initial pseudo-state)
- `Initial State  --[condition] trigger / action-->  Next State`
- `Next State  -->  Complex State`
- `Complex State  --[i < 4] button / i = i+1-->  Complex State` (self-loop while counting up)
- `Complex State  --[i == 4] button / alarm-->  Initial State` (exit back, triggers `alarm`)
- `Complex State` has `Entry / i = 0` so the counter resets each entry; `Exit /` defined (empty).

---

## 9. Class-Design Sanity Checks (Setup for SOLID)

### 9.1 Bad Class Example

```
            Employee
   -----------------------------
   saveToDatabase(db)
   calculatePay(month)
   promote(levels)
   fire(date)
   recordHours(hours)
   assignToTeam(team)
```

Why this class is bad: it serves **many different actors** (Accounting wants `calculatePay`; HR wants `promote`/`fire`; the Manager wants `recordHours`/`assignToTeam`; the DBA wants `saveToDatabase`). Each actor can independently demand a change -> many reasons to change -> fragile.

### 9.2 Bad Hierarchy Example

```
   Rectangle
       ^
       |  (inheritance)
       |
   Square
```

Why this hierarchy is bad: `setWidth` / `setHeight` are not supposed to change the other dimension on a Rectangle, but `setWidth` / `setHeight` don't make sense for a Square (they'd have to mutate the other dimension to preserve the square invariant). A `Square` is **not substitutable** for a `Rectangle`.

---

## 10. SOLID Design Principles

The five SOLID principles:

- **S** â€” Single Responsibility Principle
- **O** â€” Open-Closed Principle
- **L** â€” Liskov Substitution Principle
- **I** â€” Interface-Segregation Principle
- **D** â€” Dependency Inversion Principle

### 10.1 S â€” Single Responsibility Principle (SRP)

- **A class should have only one reason to change.**
- Classes should be designed so that **only the needs of one single actor (user from the user story)** would require a change in the class.

Motivation (Employee example): the `Employee` class above mixes responsibilities serving multiple distinct actors:
- `calculatePay()` â€” Accounting actor
- `assignToTeam()` â€” Manager actor
- `saveToDatabase()` â€” Database / persistence actor
- `recordHour()` â€” Time-tracking actor
- `promote()` â€” HR actor
- `fire()` â€” HR actor

Each actor pulling the class in a different direction (visualized as a Swiss-army-knife with knives sticking out in many directions, crossed out in red) means many independent reasons to change -> violates SRP. Split into separate classes (e.g., `PayCalculator`, `EmployeeRepository`, `HRActions`, ...) each driven by exactly one actor.

### 10.2 O â€” Open-Closed Principle (OCP)

- **A module should be open for extension but closed for modification.**
- **Extensions** could be:
  - **Subclassing**
  - **Adding behavior via delegation-based design patterns (e.g., Strategy)**
- **Modifications** are:
  - **Large changes of the source code**

Interpretation: you should be able to add new behavior without rewriting existing, working code. Achieve this with polymorphism â€” either inheritance (subclasses override hooks) or composition (inject a different Strategy object).

*(Note: design patterns like Strategy, Observer, Adapter, Pub-Sub are covered in detail in `design-patterns.md`.)*

### 10.3 L â€” Liskov Substitution Principle (LSP)

- **Subclasses should be substitutable for their super classes without breaking the application's correctness.**

Motivation (Rectangle / Square example):
- For `Rectangle`: `setWidth` / `setHeight` are **not** supposed to change the other dimension.
- For `Square`: `setWidth` / `setHeight` **don't make sense** for a Square (must mutate both to preserve invariant).
- Therefore `Square` *is-a* `Rectangle` (inheriting setters) breaks correctness â€” code that holds a `Rectangle` reference and calls `setWidth` would behave wrong when given a `Square`. Hence the inheritance violates LSP.

### 10.4 I â€” Interface Segregation Principle (ISP)

- **A client module should not be forced to depend on methods it does not use.**
- Think of this as a **Single Responsibility Principle for interfaces**.
- **Split up large interfaces into smaller coherent interfaces.**

Practical consequence: prefer many small, focused interfaces (e.g., `Readable`, `Writable`, `Closeable`) over one fat interface (`MegaIO` that has dozens of methods most clients don't need).

### 10.5 D â€” Dependency Inversion Principle (DIP)

- **High-level (abstract) modules should not depend on low-level (concrete) modules. Both should depend on abstractions.**
- Instead of using information about a concrete class (low-level), you should **define an interface (high-level) and only depend on this interface.**

Net effect: depend on stable abstractions, inject concrete implementations from the outside. This is the foundation of dependency-injection, testable code, and pluggable architectures.

---

## 11. Code Smells & Refactoring

### 11.1 Code Smell #1 â€” Code Duplication

Reference: https://refactoring.guru/smells/duplicate-code

**Smelly code:**

```javascript
function calculateBookOrderTotal(items, discountRate = 0.1, taxRate = 0.05) {
  let subtotal = sumItems(items);
  let discountedTotal = subtotal * (1 - discountRate);
  let finalTotal = discountedTotal * (1 + taxRate);
  console.log(`Book order processed. Subtotal: $${subtotal.toFixed(2)}, Final: $${finalTotal.toFixed(2)}`);
  return finalTotal;
}

// --- Function 2: Calculate Total for Electronics ---
function calculateElectronicsOrderTotal(items, discountRate = 0.05, taxRate = 0.10) {
  let subtotal = sumItems(items);
  let discountedTotal = subtotal * (1 - discountRate);
  let finalTotal = discountedTotal * (1 + taxRate);
  console.log(`Electronics order processed. Subtotal: $${subtotal.toFixed(2)}, Final: $${finalTotal.toFixed(2)}`);
  return finalTotal;
}
```

**Problem:** the three highlighted lines (`subtotal`, `discountedTotal`, `finalTotal` computations) are **duplicated** between the two functions. If the tax/discount math changes, you must remember to update both copies â€” bug-prone.

**Refactored â€” Extract Function:**

```javascript
function applyDiscountAndTax(items, discountRate, taxRate) {
  let subtotal = sumItems(items);
  let discountedTotal = subtotal * (1 - discountRate);
  let finalTotal = discountedTotal * (1 + taxRate);
  return finalTotal;
}

function calculateBookOrderTotal(items, discountRate = 0.1, taxRate = 0.05) {
  let finalTotal = applyDiscountAndTax(item, discountRate, taxRate);
  console.log(`Book order processed. Subtotal: $${subtotal.toFixed(2)}, Final: $${finalTotal.toFixed(2)}`);
  return finalTotal;
}

function calculateElectronicsOrderTotal(items, discountRate = 0.05, taxRate = 0.10) {
  let finalTotal = applyDiscountAndTax(item, discountRate, taxRate);
  console.log(`Electronics order processed. Subtotal: $${subtotal.toFixed(2)}, Final: $${finalTotal.toFixed(2)}`);
  return finalTotal;
}
```

The shared logic now lives in **one** place (`applyDiscountAndTax`); the two callers differ only in their *messages and defaults*.

### 11.2 Code Smell #2 â€” Naming Smells

Reference: https://hilton.org.uk/blog/naming-smells

**Smelly code:**

```javascript
function fetchData(userId) {
  const url = `${API_BASE_URL}/users/${userId}`;
  try {
    const r = await fetch(url);
    if (!r.ok) {
      throw new Error(`API request failed with status: ${r.status}`);
    }
    const data = await r.json();
    return data;
  } catch (error) {
    console.error(`Error fetching user ID ${userId}:`, error.message);
    throw error;
  }
}
```

**Problems (annotated):**
- `url` â€” *What is this URL used for???* (Ambiguous â€” could be anything.)
- `r` â€” *What is r?!!!* (Cryptic single-letter â€” gives the reader no clue.)
- `data` â€” *What kind of data is this?!!* (Generic â€” could be anything.)
- `fetchData` â€” generic verb+noun; doesn't say *what* is being fetched.

**Refactored â€” names communicate intent & behavior:**

```javascript
function fetchUserData(userId) {
  const userApiUrl = `${API_BASE_URL}/users/${userId}`;
  try {
    const userApiResponse = await fetch(userApiUrl);
    if (!userApiResponse.ok) {
      throw new Error(`User API request failed with status:
                       ${userApiResponse.status}`);
    }
    const rawUserData = await userApiResponse.json();
    return rawUserData;
  } catch (error) {
    console.error(`Error fetching user ID ${userId}:`, error.message);
    throw error;
  }
}
```

Rename map:
- `fetchData` -> `fetchUserData`
- `url` -> `userApiUrl`
- `r` -> `userApiResponse`
- `data` -> `rawUserData`

Result: refactored code **better communicates the intent & behavior** without changing any logic.

---

## 12. Recommended Tutorials (Linked in Lecture)

- **UML Class Diagrams in Python**
- **UML Sequence Diagrams in Python**
- **SOLID Principles in Python with UML** (Preview Version)

(Linked from Tobias DÃ¼rschmid's SEBook site referenced throughout the slides.)

---

## 13. Exit-Ticket Self-Check Questions

1. Summarize three insights you learned about software modelling & design today.
2. **Describe in your own words what the Single Responsibility Principle is and why it is important to create clean, maintainable code.**
3. List any questions about today's material that are still unclear.

---

## 14. One-Page Cheat Sheet

| Concept | Key Idea |
|---|---|
| Code/Module View | Class diagrams â€” structure of code |
| Data View | ER diagrams â€” structure of data |
| Run-Time / Component-Connector View | Components + connectors â€” how runtime parts wire up |
| Behavioral View | Sequence + state diagrams â€” interactions and dynamics |
| Class diagram visibility | `+` public, `-` private |
| Inheritance | Open white triangle, "is-a" |
| Navigable association | Plain arrow -> (left has ref to right) |
| Non-navigable association | Line with X (no reference) |
| Aggregation | Open diamond â€” "owns" (has-a) |
| Composition | Filled diamond â€” owned part can't exist without whole |
| ERD cardinalities | 1-1, 1-N, N-1, N-M |
| Sequence msg: synchronous | Solid filled arrow; caller blocks |
| Sequence msg: response | Dashed open arrow back |
| Sequence msg: asynchronous | Solid open arrow; non-blocking |
| Sequence msg: found | Starts at solid black dot â€” unknown sender |
| Activation bar | Component is executing a task; can stack for sub-calls |
| `Alt` fragment | Exactly one branch by guard |
| `Loop` fragment | Repeat while condition true |
| `Opt` fragment | Run iff condition true |
| State transition label | `[guard] trigger / action` |
| State entry/exit | `Entry / ...` and `Exit / ...` actions inside compound state |
| **S**RP | One reason to change; serve one actor |
| **O**CP | Open for extension (subclass / Strategy), closed for modification |
| **L**SP | Subclasses substitutable for superclasses without breaking correctness |
| **I**SP | Don't force clients to depend on methods they don't use; split big interfaces |
| **D**IP | Depend on abstractions, not concretions |
| Code Duplication smell | Extract repeated logic into a shared function |
| Naming smell | Names like `r`, `data`, `url` hide intent; pick descriptive names |
| Separation of Concerns | (General design principle behind SRP/ISP) â€” each module owns one concern |


---

# Appendix: Lecture 7 Slides (raw extracted text)

The following is the full extracted text of Lecture 7. Preserved verbatim so no information is lost.

```text
CS 35L Software
Construction
Lecture 7 ­ Design
& Modeling
Assistant Teaching Professor
Computer Science Department
Notes on the Team Project
· You should start coding now. We recommend Pair Programming
   · two developers use the same computer physically or via Zoom screen sharing
     to code together.
   · Works especially well for beginners
   · Talking aloud about what you are doing can help you find issues in your design
   · Having someone look over your shoulder means you find bugs more easily
   · Knowledge is exchanged between team members
· Every individual team member will submit a weekly progress report on Grade
  Scope (Fridays starting this week)
   · What have you done this week?
   · How did you apply concepts taught in the lectures to your project?
Recap of Software Engineering Activities
                            User Stories
Requirements Analysis What should be built?
               Design            How should we build it?
                                 Development Build it
Today's topic                                                               Did we build the
                                                                            right system?
                                 Testing
                                 Deployment                                 Deliver to
                                                                            customer
               CS 35L Software Construction: Lecture 7 ­ Design & Modeling                    3
               Tobias Dürschmid
Coding Without Design Considerations
Often Turns Very Messy After Some Time
· Early design decisions often become
"load-bearing walls" of your app
(they are hard to change later)
· E.g., once you start implementing a
peer-to-peer architecture, it's hard to
transform into a client-server
architecture
· Spend some time thinking about
      design before starting to code     "Just keep coding.
· The more experience you have, the
easier it will become to identify which  We can always fix it later."
design decisions are "important"
CS 35L Software   Construction:  LCecoturme 7m­ oDnesimgni&scMoodneclinegption  about  software  4
Great Software Engineers
Communicate Their Ideas Well
Great Software Engineers
Communicate Their Ideas Well
Great Software Engineers
Communicate Their Ideas Well
Different Views Visualize                                                                                                                        Software
Different Aspects of Software
                                                                      Read more here: Documenting architecture
                                                                      Software Architectures: Views textbook cover
                                                                      and Beyond by Clements et al. Thumbnail image
                                                                                                                                                 of a software
                                                                      (available in UCLA digital library) architecture or
                                                                                                                                                 modeling textbook,
                                                                                                                                                 displayed as a
                                                                                                                                                 reference resource
Code View /                        Data View          Run-Time View /     Behavioral View
Module View                                           Component-
                                                      Connector View
Describes Structure                Describes          Describes how Run- Describes
of Code                            Structure of Data  Time Component Interactions
                                                      are Connected
                                              N
          Shape                    Course             Client              client_1:  server:
                                      study
            -color: int                                                   Client LibraryServer
+set_color(r: int, g: int, b:int)
     Rectangle                     Student            Library
                                       UID            Server
         -width: int
        -length: int
 +set_width(width: int)
+set_height(height: int)
             CS 35L Software Construction: Lecture 7 ­ Design & Modeling                        8
             Tobias Dürschmid
Class Diagrams                                               Read more here:
Visualize the Code View                                      https://tobiasduerschmid.github.io/
                                                             SEBook/uml_class_diagram.html
Shape                     Abstract Class name                Visibility:
-color: int               Attributes                         -  private
+set_color(r: int, g: int, b:int) Functions                  + public
Inheritance (Is-a)
      Rectangle           Class name
         -width: int
         -length: int
 +set_width(width: int)
+set_height(height: int)
Class Diagrams                                                           Read more here:
Visualize the Code View                                                  https://tobiasduerschmid.github.io/
                                                                         SEBook/uml_class_diagram.html
           Shape                   from abc import ABC, abstractmethod
             -color: int           class Shape(ABC):
+set_color(r: int, g: int, b:int)         def __init__(self):
                                                 self.__color: int = 0 # private (-color)
                                          def set_color(self, r: int, g: int, b: int) -> None:
                                                 self.__color = (r << 16) | (g << 8) | b
      Rectangle                    class Rectangle(Shape):               # private  (-width)
                                          def __init__(self):            # private  (-length)
         -width: int                             super().__init__()
         -length: int                            self.__width: int = 0
                                                 self.__length: int = 0
 +set_width(width: int)
+set_height(height: int)           def set_width(self, width: int) -> None:
                                          self.__width = width
                                   def set_height(self, height: int) -> None:
                                          self.__length = height
Class Diagrams                                               Read more here:
Visualize the Code View                                      https://tobiasduerschmid.github.io/
                                                             SEBook/uml_class_diagram.html
           Shape                                 -color              Color
+set_color(r: int, g: int, b:int)  Association                        -color: int
                                                         +set_color(r: int, g: int, b:int)
        Rectangle
                                   from abc import ABC, abstractmethod
             -width: int
            -length: int           class Color:
    +set_width(width: int)                def __init__(self, r: int, g: int, b: int):
   +set_height(height: int)                       self.set_color()
                                          def set_color(self, r: int, g: int, b: int) -> None:
                                                  self.r: int = r
                                                 self.g: int = g
                                                 self.b: int = b
                                   class Shape(ABC):
                                          def __init__(self):
                                                 self.__color: Color = Color(0, 0, 0) # (-color)
                                          def set_color(self, color: Color) -> None:
                                                 self.__color = color
Class Diagrams                                               Read more here:
Visualize the Code View                                      https://tobiasduerschmid.github.io/
                                                             SEBook/uml_class_diagram.html
           Shape                         -color: color              Color
+set_color(r: int, g: int, b:int)  Association                       -color: int
                                                        +set_color(r: int, g: int, b:int)
                                                           Navigable Association
                                   The left object has a reference to the right object
      Rectangle                                            Non-Navigable Association
         -width: int               The left object does not have a reference to the right object
         -length: int
                                                           Aggregation (has-a)
 +set_width(width: int)
+set_height(height: int)           The left object "owns" the right object
                                                           Composition (has-a)
                                   Same as aggregation, plus the right object's existence depends
                                   on the left object as it cannot exist without the whole
Class Diagrams                                               Read more here:
Visualize the Code View                                      https://tobiasduerschmid.github.io/
                                                             SEBook/uml_class_diagram.html
           Shape                                   -color              Color
+set_color(r: int, g: int, b:int)  Association                          -color: int
                                                           +set_color(r: int, g: int, b:int)
        Rectangle
                                                           Navigable Association
             -width: int
            -length: int           (e.g., Employee         Boss, Vote      Politician)
    +set_width(width: int)
   +set_height(height: int)        (e.g., Voter                   Non-Navigable Association
                                                           Vote, Account ClearTextPassword)
                                                           Aggregation (has-a)
                                   (e.g., Car              Wheel, Library  Book)
                                   (e.g., Person
                                                           Composition (has-a)
                                                           Head, House     Room)
Entity-Relationship Diagrams (ERDs)                          Relationship
Visualize the Data View
First name        Cardinality                   Pre-
Last name         (1-N, N-1, N-            Requirement
                  M, 1-1)
Student N              study     M Course
   UID      Attribute            Entity                      Number
Birthyear                        (e.g.                         Title
                                 Database
                                 table)
Sequence Diagrams              Read more here
Visualize the Behavioral View  https://tobiasduerschmid.github.io/SE
                               Book/uml_sequence_diagram.html
Sequence diagrams show examples of interactions between
objects / component instances Instance name Component type name
client_1: Client               server: LibraryServer
Caller blocks              Synchronous Call                                   Found
until response    Response to Synchronous Call                               Message
received
                      Asynchronous Message                                    Sender is
Non-blocking          Asynchronous Message                                    unknown or
communication                                                                 irrelevant
                CS 35L Software Construction: Lecture 7 ­ Design & Modeling                      15
                Tobias Dürschmid
Sequence Diagrams              Read more here
Visualize the Behavioral View  https://tobiasduerschmid.github.io/SE
                               Book/uml_sequence_diagram.html
client_1: Client               server: LibraryServer
                                      Synchronous Call                           getDBRecord(userID)
                          This is called an "activation bar". It                                    16
                          indicates that the component is
                          executing a task
Activation barscan be stacked to indicate that the
component starts a sub-process or calls a sub-component
                    CS 35L Software Construction: Lecture 7 ­ Design & Modeling
                    Tobias Dürschmid
Sequence Diagrams Support Fragments
To Show Multiple Execution Paths
client_1: Client                    server: LibraryServer
                            GET /book/id                     Alt executes
                                                             exactly one
Alt [book found]                                             of the cases
                                                             based on
                    responseCode=200, book                   their
                                                             conditions
[else]
                  responseCode=404
Sequence Diagrams Support Fragments
To Show Multiple Execution Paths
client_1: Client  server: LibraryServer
                    Loop gets executed as long
                    as the condition is true
Loop [for each book]
                            POST /book/{id}
Sequence Diagrams Support Fragments
To Show Multiple Execution Paths
client_1: Client  server: LibraryServer
Opt         Opt gets executed if and
            only if the conditions is true
     [login successful]
        GET /borrowed_books/{user_id}
State Machine Diagrams                                       Read more here:
Visualize the Behavioral View                                https://tobiasduerschmid.github.io
                                                             /SEBook/uml_state_diagram.html
Initial                  [condition] trigger /  Next
State                             action        State
[i == 4] button / alarm
                         Complex State          [i < 4] button / i = i+1
                           Entry / i = 0
                               Exit /
Different Views Visualize                                                                                                             Software
Different Aspects of Software
                                                          Read more here: Documenting architecture
Code View /                     Data View                 Software Architectures: Views textbook cover
Module View                                               and Beyond by Clements et al. Thumbnail image
Describes Structure of Code Describes Structure of                                                                                    of a software
                                             Data
                                                          (available in UCLA digital library) architecture or
                                                                                                                                      modeling textbook,
                                                                                                                                      displayed as a
                                                                                                                                      reference resource
                                                         Behavioral View
                                                         Describes Interactions
Useful for code-level design    Useful when structuring  Useful to reason about
discussions & documentation     data base tables         bottlenecks, messaging
(can be inferred automatically                           protocols, and complex
from code)                                               interactions
             CS 35L Software Construction: Lecture 7 ­ Design & Modeling         21
             Tobias Dürschmid
Is this Class a Good Class? Why or Why Not?
                                 Employee
                                 saveToDatabase(db)
                                 calculatePay(month)
                                 promote(levels)
                                 fire(date)
                                 recordHours(hours)
                                 assignToTeam(team)
Single Responsibility Principle
· A class should have only one reason to                         promote()
  change                                                               recorfidrHeo()ur()
· Classes should be designed so that only the
  needs of one single actor (user from the user
  story) would require a change in the class
                                                 assignToTeam() Employee calculatePay()
                                                                         Class
                                                 saveToDatabase
Is this a good class hierarchy? Why or Why Not?
                                      Rectangle
                  Square
Liskov Substitution Principle
· Subclasses should be substitutable for their super classes without breaking the
  application's correctness
Rectangle         setWidth / setHeight
                  are nor supposed to change
                  the other dimension
Square            setWidth / setHeight
                  Don't make sense for a
                  Square
SOLID Design Principles
· Single Responsibility Principle
· Open-Closed Principle
· Liskov Substitution Principle
· Interface-Segregation Principle
· Dependency Inversion Principle
Open-Closed Principle (OCP)
· A module should be open for extension but closed for modification
· Extensions could be:
   · Subclassing
   · Adding behavior via delegation-based design patterns (e.g., Strategy)
· Modification are:
   · Large changes of the source code
Interface Segregation Principle (ISP)
· A client module should not be forced to depend
  on methods it does not use
· think of this as a Single Responsibility Principle for interfaces
· Split up large interfaces into smaller coherent interfaces
Dependency Inversion Principle (DIP)
· high-level (abstract) modules should not depend on low-level
  (concrete) modules. both should depend on abstractions
· Instead of using information about a concrete class (low-level), you
  should define an interface (high-level) and only depend on this interface
What is Wrong With This Code?
Talk to your Neighbor(s)
function calculateBookOrderTotal(items, discountRate = 0.1, taxRate = 0.05) {
   let subtotal = sumItems(items);
   let discountedTotal = subtotal * (1 - discountRate);
   let finalTotal = discountedTotal * (1 + taxRate);
   console.log(`Book order processed. Subtotal: $${subtotal.toFixed(2)}, Final:
$${finalTotal.toFixed(2)}`);
   return finalTotal;
}
// --- Function 2: Calculate Total for Electronics ---
function calculateElectronicsOrderTotal(items, discountRate = 0.05, taxRate = 0.10) {
   let subtotal = sumItems(items);
   let discountedTotal = subtotal * (1 - discountRate);
   let finalTotal = discountedTotal * (1 + taxRate);
   console.log(`Electronics order processed. Subtotal: $${subtotal.toFixed(2)}, Final:
$${finalTotal.toFixed(2)}`);
   return finalTotal;
}
What is Wrong With This Code?                                See
Too much Code Duplication                                    https://refactoring.guru/smells
                                                             /duplicate-code
function calculateBookOrderTotal(items, discountRate = 0.1, taxRate = 0.05) {
   let subtotal = sumItems(items);
   let discountedTotal = subtotal * (1 - discountRate);
   let finalTotal = discountedTotal * (1 + taxRate);
   console.log(`Book order processed. Subtotal: $${subtotal.toFixed(2)}, Final:
$${finalTotal.toFixed(2)}`);
   return finalTotal;
}
// --- Function 2: Calculate Total for Electronics ---
function calculateElectronicsOrderTotal(items, discountRate = 0.05, taxRate = 0.10) {
   let subtotal = sumItems(items);
   let discountedTotal = subtotal * (1 - discountRate);
   let finalTotal = discountedTotal * (1 + taxRate);
   console.log(`Electronics order processed. Subtotal: $${subtotal.toFixed(2)}, Final:
$${finalTotal.toFixed(2)}`);
   return finalTotal;
}
Refactored Code
Avoids Unnecessary Code Duplication
function applyDiscountAndTax(items, discountRate, taxRate) {
   let subtotal = sumItems(items);
   let discountedTotal = subtotal * (1 - discountRate);
   let finalTotal = discountedTotal * (1 + taxRate);
   return finalTotal;
}
function calculateBookOrderTotal(items, discountRate = 0.1, taxRate = 0.05) {
   let finalTotal = applyDiscountAndTax(item, discountRate, taxRate);
   console.log(`Book order processed. Subtotal: $${subtotal.toFixed(2)}, Final:
$${finalTotal.toFixed(2)}`);
   return finalTotal;
}
function calculateElectronicsOrderTotal(items, discountRate = 0.05, taxRate = 0.10) {
   let finalTotal = applyDiscountAndTax(item, discountRate, taxRate);
   console.log(`Electronics order processed. Subtotal: $${subtotal.toFixed(2)}, Final:
$${finalTotal.toFixed(2)}`);
   return finalTotal;
}
   CS 35L Software Construction: Lecture 7 ­ Design & Modeling                          32
   Tobias Dürschmid
What is Wrong With This Code?
Talk to your Neighbor(s)
function fetchData(userId) {
   const url = `${API_BASE_URL}/users/${userId}`;
   try {
       const r = await fetch(url);
       if (!r.ok) {
          throw new Error(`API request failed with status: ${r.status}`);
       }
       const data = await r.json();
       return data;
   } catch (error) {
       console.error(`Error fetching user ID ${userId}:`, error.message);
       throw error;
   }
}
Naming Smells                                    See https://hilton.org.uk/blog/naming-smells
function fetchData(userId) {
   const url = `${API_BASE_URL}/users/${userId}`;
   try {  What is this URL used for???
      const r = await fetch(url);
      if (!r.oWk)ha{t is r?!!!
         throw new Error(`API request failed with status: ${r.status}`);
      }
      const data = await r.json();
                   What kind of data is this?!!
      return data;
   } catch (error) {
      console.error(`Error fetching user ID ${userId}:`, error.message);
      throw error;
   }
}
          CS 35L Software Construction: Lecture 7 ­ Design & Modeling                          34
          Tobias Dürschmid
Refactored Code better Communicates
The Intent & Behavior of the Code
function fetchUserData(userId) {
   const userApiUrl = `${API_BASE_URL}/users/${userId}`;
   try {
       const userApiResponse = await fetch(userApiUrl);
       if (!userApiResponse.ok) {
          throw new Error(`User API request failed with status:
                                                                ${userApiResponse.status}`);
       }
       const rawUserData = await userApiResponse.json();
       return rawUserData;
   } catch (error) {
       console.error(`Error fetching user ID ${userId}:`, error.message);
       throw error;
   }
}
Tutorials Recommended for Content from
This Lecture
· UML Class Diagrams in Python
· UML Sequence Diagrams in Python
· SOLID Principles in Python with UML (Preview Version)
Please fill out your Exit Tickets on Bruin Learn!
Please summarize three insights you learned about software modelling &
design today
Describe in your own words what the Single Responsibility Principle is
and why it is important to create clean, maintainable code.
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slide use images from Flaticon.com (Creators: Freepik)

```
