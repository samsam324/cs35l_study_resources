# CS35L Design Patterns Study Guide

This guide covers the design patterns explicitly taught in CS35L lectures: the
general concept of a Design Pattern (L6), the **Observer Pattern** (L6), the
**Publish-Subscribe Pattern** (L10), the **Adapter Pattern** (L17), and a brief
mention of the **Strategy Pattern** as a delegation-based pattern supporting
the Open-Closed Principle (L7).

---

## 1. What Is a Design Pattern?

### Definition (from L6)

> "A *pattern* is a **common, acceptable solution** to a reoccurring
> **problem** that arises in a specific **context**."

Three key qualifiers from the definition:

- **"found in many instances"** â€” the problem occurs repeatedly across many
  real-world cases (it generalizes beyond a few concrete cases).
- **"a good solution, but not always the best one"** â€” patterns are
  *acceptable*, not silver bullets.
- **"specific context"** â€” patterns always refer to a specific **situation,
  goal, or trade-off**. **Patterns are not universally good.**

### Pattern Description Structure

Pattern descriptions have many different formats. **At minimum it should
describe problem, context, and solution.** The fuller structure used in the
lectures:

- **Context** â€” the situation / scenario in which the problem arises.
- **Problem** â€” the recurring design problem you face.
- **Solution** â€” the template / arrangement of roles that addresses it.
- **Consequences** â€” trade-offs implied by adopting the solution.

### Key Properties of Design Patterns

- A design pattern is **not code** â€” it is a **template** for solving design
  problems. You instantiate it in your code by assigning the pattern's roles
  to concrete classes/objects.
- Design patterns **add roles to classes / objects** (e.g., a class becomes a
  "Subject" or an "Observer" because of the role it plays in the pattern).
- **Lesson Learned (L6): Start By Considering Existing Solutions.**
  - Most problems have been **solved already** and described in a
    **well-documented** way.
  - **Knowing existing solutions and patterns in your field** can save you a
    lot of time and effort.

### Why Design Patterns Matter Here

In L6, patterns are introduced together with **Separation of Concerns** and
the separation of the **Presentation Layer** from the **Application Layer**.
The Observer pattern is the canonical way to keep a UI synchronized with
domain state without coupling the domain to the UI:

- Allows the presentation layer to **register a callback** for state changes
  (e.g., `onBalanceChanged`).
- Allows the presentation layer to retrieve information using **getter
  functions** (e.g., `getCurrentBalance()`).
- Allows the presentation layer to **forward user events** (e.g.,
  `buyProperty(name, user)`).

The Application Layer "doesn't have a clue that there even is a UI" â€” the
Observer pattern enables this decoupling.

---

## 2. The Observer Design Pattern (L6)

### Intent / Roles

The Observer pattern introduces two main roles:

- **Subject** (also called *Observable*) â€” the object **sending updates after
  it has changed**.
- **Observer** â€” the object **listening to the updates of Subjects**.

It establishes a **one-to-many** dependency: when one Subject changes, **all**
registered Observers are notified.

### Context

> Scenarios requiring **distributed event handling** systems or **highly
> decoupled architectures** like GUI, and signal processing.

### Problem

> How can a **one-to-many dependency** between objects be maintained
> efficiently **without making the objects tightly coupled**?

### Solution

Introduce a `Subject` that maintains a list of `Observer`s. Observers
register themselves with the subject. When the subject's state changes, it
iterates over the observer list and calls `update()` on each one.

### Structure (UML-style)

```
       Subject                          AbstractObserver  Â«abstractÂ»
+----------------------+        *      +------------------+
| Subject              |--------------> | update(Event)    |
+----------------------+                +------------------+
| register(Observer)   |                        â–²
| unregister(Observer) |                        |
| notify(Event)        |                +------------------+
| Data getData()       |                | Concrete Observer|
| doSomething()        |                +------------------+
+----------------------+                | update(Event)    |
                                        +------------------+
```

- `Subject` is e.g., your **Domain Object** with a lot of useful business
  logic.
- `Concrete Observer` is e.g., the **presentation-layer UI element** that
  displays the data of the subject whenever it changes.

### Sequence (interaction)

```
  Observer                       Subject
     |  register(self)             |
     |---------------------------->|     <-- Adds observer to list
     |                             |
     |                       doSomething()
     |                             |
     |                       notify(someEvent)   <-- Iterates over observer list
     |       update()              |
     |<----------------------------|
     |       getData()             |
     |---------------------------->|
     |        data                 |
     |<- - - - - - - - - - - - - - |
```

### Full Lecture Code Example â€” News Channel / Subscribers

The L6 lecture frames the Observer pattern with a concrete example using
**`NewsChannel`** (the Subject) and concrete subscribers **`MobileApp`** and
**`EmailDigest`**.

#### UML of the Example

```
                       NewsChannel
                  -_subscribers: list[Subscriber]
                  -_latest_post: str
                  +follow(subscriber: Subscriber)
                  +unfollow(subscriber: Subscriber)
                  +publish_post(text: str)
                  +get_latest_post(): str
                  -_notify_subscribers()
                          | 0..*  _subscribers
                          |
                       Â«ABCÂ»
                      Subscriber
                       +update()
                          â–²
              ____________|____________
             |                         |
         MobileApp                EmailDigest
       -_channel: NewsChannel   -_channel: NewsChannel
       +update()                +update()
```

Notes:

- `_notify_subscribers()` body is essentially:
  `for subscriber in self._subscribers: subscriber.update()`
- A `MobileApp.update()` does e.g.:
  `post = self._channel.get_latest_post()`
  `print(f"[MobileApp] Push notification: {post}")`

#### Code â€” Observer Interface (`Subscriber`)

```python
# ==================================
# OBSERVER INTERFACE
# ==================================
class Subscriber(ABC):
    """The Observer interface."""
    @abstractmethod
    def update(self):
        pass
```

#### Code â€” Subject (`NewsChannel`)

```python
# SUBJECT
class NewsChannel:
    """The Subject that maintains a list of subscribers."""
    def __init__(self):
        self._subscribers: list[Subscriber] = []
        self._latest_post: str = ""

    def follow(self, subscriber: Subscriber):
        if subscriber not in self._subscribers:
            self._subscribers.append(subscriber)

    def unfollow(self, subscriber: Subscriber):
        self._subscribers.remove(subscriber)
```

```python
# SUBJECT (continued)
class NewsChannel:

    [...]

    def publish_post(self, text: str):
        self._latest_post = text
        self._notify_subscribers()

    def get_latest_post(self) -> str:
        return self._latest_post

    def _notify_subscribers(self):
        for subscriber in self._subscribers:
            subscriber.update()
```

#### Code â€” Concrete Observers

```python
# CONCRETE OBSERVERS
class MobileApp(Subscriber):
    """A concrete observer that pulls state from the channel."""
    def __init__(self, channel: NewsChannel):
        self._channel = channel

    def update(self):
        post = self._channel.get_latest_post()
        print(f"[MobileApp] Push notification: {post}")


class EmailDigest(Subscriber):
    """Another concrete observer with different behavior."""
    def __init__(self, channel: NewsChannel):
        self._channel = channel

    def update(self):
        post = self._channel.get_latest_post()
        print(f"[EmailDigest] New email queued: {post}")
```

#### Code â€” Client Code

```python
# CLIENT CODE
channel = NewsChannel()

app   = MobileApp(channel)
email = EmailDigest(channel)

channel.follow(app)
channel.follow(email)

channel.publish_post("New video uploaded!")
# [MobileApp]   Push notification: New video uploaded!
# [EmailDigest] New email queued:  New video uploaded!

channel.unfollow(email)

channel.publish_post("Live stream starting!")
# [MobileApp]   Push notification: Live stream starting!
```

### Key Mechanics Observed in the Example

- The **Subject (`NewsChannel`)** owns the list of subscribers and the data
  (`_latest_post`).
- Observers **pull** state from the subject in their `update()` (they call
  `get_latest_post()` â€” the notification itself carries no payload here).
- `follow` / `unfollow` are this example's names for `register` / `unregister`.
- `publish_post` is the trigger that mutates state **and** then calls
  `_notify_subscribers()`.
- Different concrete observers (`MobileApp`, `EmailDigest`) implement the
  **same `update()` interface** but with **different behavior** â€” this is what
  makes the subject able to talk to many heterogeneous observers without
  knowing their types.

### When to Use / Consequences

**Use when:**

- You have **distributed event handling** systems or highly decoupled
  architectures like GUI and signal processing.
- One object's state change must be reflected in many others, but you do not
  want the subject to be tightly coupled to those others.
- You want the **Presentation Layer** to listen to the **Application Layer**
  without the Application Layer knowing about the UI.

**Consequences / Trade-offs:**

- Loose coupling: subject only knows the abstract `Observer` interface.
- Supports broadcast communication (one-to-many).
- The order of notifications is generally not guaranteed.
- Risk of unexpected updates / cascading updates if observers themselves
  mutate state during `update()`.

### Related Patterns

- **Publish-Subscribe** (below) â€” a generalization that introduces a broker
  / topic, decoupling publishers from subscribers entirely.

---

## 3. The Publish-Subscribe Pattern (L10)

### Intent

Publish-Subscribe implements an **N-to-M messaging channel**: multiple
publishers can publish to a *topic*, and multiple subscribers can subscribe
to that topic, without publishers and subscribers knowing each other.

### Problem

> How to keep the **state of cooperating components synchronized**?

### Context

> Components should be **loosely coupled** and **reduce direct dependencies**.

### Components / Roles

- **Publisher** â€” produces data or sends notifications about state updates.
- **Topic** â€” the named channel through which messages flow. **Topics are
  usually identified via string names.**
- **Message Queue** â€” buffer attached to each subscriber, holding messages
  delivered through the topic until the subscriber consumes them.
- **Subscriber** â€” consumes data or listens to state updates.

### Structure (Conceptual Diagram)

```
   Publisher â”€â”                                   â”Œâ”€â”€ Message Queue â”€â”€ Subscriber
              â”‚       publishes to    subscribes  â”‚
              â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º ( Topic ) â—„â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
              â”‚      (string-named)               â”‚
   Publisher â”€â”˜                                   â””â”€â”€ Message Queue â”€â”€ Subscriber
```

### Code Sketch (ROS-style example from lecture)

The lecture illustrates pub/sub with ROS-like syntax:

```cpp
// Publisher side
pub = nh.advertise("topic_name");
...
pub.publish(msg);

// Subscriber side
nh.subscribe("topic_name", 10, callback);
```

The topic is identified by the **string** `"topic_name"`. Publishers and
subscribers never reference each other directly â€” they only reference the
topic name.

### How It Differs From Observer

Both deal with one-or-more producers notifying one-or-more consumers about
state, but:

| Aspect              | Observer                          | Publish-Subscribe                                  |
|---------------------|-----------------------------------|----------------------------------------------------|
| Cardinality         | One-to-many (1 Subject : N Obs.)  | **N-to-M** (many publishers, many subscribers)     |
| Coupling            | Subject knows its observer list   | Publishers/subscribers know **only the topic**     |
| Indirection         | Direct method call `update()`     | Goes through a **broker / topic / message queue**  |
| Identification      | Object references                 | Usually **string names** of topics                 |
| Sync vs Async       | Typically synchronous             | Often asynchronous (queues decouple in time)       |
| Typical use         | In-process GUI, signal processing | Event-driven architectures, distributed systems    |

The lecture explicitly asks: *"How does this relate to the Observer?"* â€”
Publish-Subscribe is essentially Observer extended with a **broker (the
topic) and message queues**, which lets it scale to N-to-M and to
distributed systems where the publisher and subscriber may not even be in
the same process.

### When to Use

- **Event-driven architectures** where many independent components produce
  and consume events.
- **Distributed systems** where components run in different processes or on
  different machines (the topic abstraction works across process / network
  boundaries).
- When you want components to be **loosely coupled** and to **reduce direct
  dependencies** â€” neither side needs to know the other exists; they only
  need to agree on a topic name and message format.
- When you want to synchronize the state of cooperating components without
  forcing one to manage subscriber lists directly.

---

## 4. The Adapter Pattern (L17)

### Where It Comes From

Lecture 17 ("Code Comprehension & API Design") presents the Adapter as the
**"Design Pattern for Interoperability: Use Adapters to Connect
Interfaces."** It is taught as the solution when two systems must talk but
have **incompatible interfaces** (e.g., different data formats, different
protocols).

### Problem

> Two systems use **different interfaces** (e.g., different data formats,
> different protocols, ...).

For example, your system speaks **XML** but their system speaks **JSON** â€”
they cannot communicate directly.

```
   Your System â”€â”€â”€â–º [XML]   âš¡   [JSON] â—„â”€â”€â”€ Their System
                                  (mismatch)
```

### Solution

> Create an **adapter component** that **translates between the two
> interfaces**.

```
   Your System â”€â”€â”€â–º  Adapter  â—„â”€â”€â”€ Their System
                  (XML  â†”  JSON
                   translation
                   is localized here)
```

The adapter wraps one interface and exposes another. Crucially, **the
translation logic is localized in the adapter**, so neither "Your System"
nor "Their System" needs to change to interoperate.

### Use Case: Interoperability

The Adapter pattern is the structural pattern that **connects systems with
different interfaces** so they can interoperate. In the GDS (Global
Distribution System) case study used in L17, adapters allow each individual
airline / booking system to keep its own internal interface while still
participating in the shared GDS protocol â€” the adapter translates between
its native API and the shared format.

### How an Adapter Works (Mechanically)

1. **One side** of the adapter implements / exposes the interface expected
   by the consumer ("Your System").
2. **The other side** of the adapter calls / consumes the interface
   provided by the producer ("Their System").
3. Inside the adapter, all **format translation, protocol mapping, and
   semantic mapping** between the two sides happens.

Because the adapter sits between the two systems, all the cross-system
translation code lives in one place â€” easy to evolve, test, and replace if
either side's interface changes.

### Question Raised in the Lecture

The lecture asks: *"Does this only work for Syntactic Interoperability?"* â€”
the implicit answer is **no**: an adapter can also bridge **semantic**
differences (e.g., units, currencies, vocabularies, MM/DD/YY vs DD/MM/YY,
coordinate frames) between systems, not just syntactic format differences.

### When to Use

- You need to integrate with an existing system whose interface you cannot
  change.
- Two systems have incompatible data formats, protocols, or interface
  shapes, but you want them to interoperate.
- You want to **localize** all translation logic in a single component so
  the rest of your codebase only deals with one consistent interface.

---

## 5. The Strategy Pattern (briefly, from L7)

The Strategy pattern is mentioned in L7 (Modeling & Design Principles) as a
canonical example of a **delegation-based pattern** that supports the
**Open-Closed Principle** ("software entities should be open for extension,
but closed for modification").

### Key Idea

Instead of hard-coding a particular algorithm or behavior into a class, you
**delegate** the behavior to a separate **Strategy** object that implements
a common interface. New behaviors are added by writing **new Strategy
classes** (extension), not by modifying the class that uses them
(modification).

### Why It Supports Open-Closed

- The client class (the "context") is **closed for modification** â€” its
  source code does not change when behavior changes.
- The system is **open for extension** â€” adding a new behavior means adding
  a new Strategy class that implements the strategy interface.
- This is achieved through **composition** (the context holds a reference
  to a Strategy) rather than **modification** (editing the context to add a
  new `if` branch or subclass).

### Contrast With Modification

If you instead added each new behavior by editing the class or adding
`if/else` branches for every new variant, you would violate Open-Closed.
Strategy lets you **add behavior via composition rather than modification**.

---

## Quick Comparison Table

| Pattern               | Lecture | Intent                                                              | Cardinality      | Key Roles                                       | Primary Benefit                              |
|-----------------------|---------|---------------------------------------------------------------------|------------------|-------------------------------------------------|----------------------------------------------|
| **Observer**          | L6      | Auto-notify dependents when subject changes                         | 1 : N            | Subject, (Abstract)Observer, ConcreteObserver   | Loose coupling between domain and UI         |
| **Publish-Subscribe** | L10     | N-to-M messaging via a named channel                                | N : M            | Publisher, Topic, Message Queue, Subscriber     | Distributed / decoupled event-driven systems |
| **Adapter**           | L17     | Connect two incompatible interfaces                                 | 1 : 1            | Client interface, Adapter, Adaptee              | Interoperability, localized translation      |
| **Strategy**          | L7      | Swap behaviors via composition; support Open-Closed                 | 1 : 1 (per call) | Context, Strategy interface, ConcreteStrategy   | Extensibility without modifying the context  |

---

## Summary of Lecture Takeaways

- A **design pattern** = common, acceptable solution to a recurring problem
  in a specific context. Described by **Problem / Context / Solution** (and
  often Consequences). Patterns are templates, not code, and they assign
  **roles** to classes/objects.
- **Observer** decouples a Subject from many Observers via a registration
  list and a uniform `update()` callback â€” see the full `NewsChannel` /
  `MobileApp` / `EmailDigest` example.
- **Publish-Subscribe** generalizes Observer to N-to-M via a **broker /
  topic / message queue**, identified by string topic names â€” ideal for
  event-driven and distributed systems.
- **Adapter** bridges **incompatible interfaces** by wrapping one and
  exposing another, localizing all translation in one place â€” the core
  design pattern for **interoperability**.
- **Strategy** is a **delegation-based** pattern that lets you add behavior
  via **composition** rather than modification, directly supporting the
  **Open-Closed Principle**.


---

# Appendix A: Lecture 6 Slides (raw extracted text) â€” Observer Pattern source

```text
CS 35L Software
Construction
Lecture 5 ­ React, UI
Development & Design
Pattern Introduction
Assistant Teaching Professor
Computer Science Department
Motivating Case Study: Digital Monopoly Game
There are dozens of implementations of the game "Monopoly"
with vastly different User Interfaces
Motivating Case Study: Digital Monopoly Game
There are dozens of implementations of the game "Monopoly"
with vastly different User Interfaces. Even with just plain text UI!
Next Turn (Mohamed)                          Dice Rolled (Mohamed)
It is Mohamed`s turn.                    The first die shows 6. The second die shows 2.
       Roll Dice                                                    Okay
                       Player Decision Requested (Mohamed)
                       You landed on the unpurchased property "Royce Hall".
                           Do you want to buy "Royce Hall" for Bruin$800?
                                  Your current inventory is Bruin$4250.
                       Yes (-Bruin$800)  No
What Should They All Have In Common?
                     All of them should implement
                       the exact same game logic!
               So at least in theory they could all run
           the exact same code just with a different UI
         (ideally even with multiple UIs simultaneously)
Next Turn (Mohamed)     Dice Rolled (Mohamed)     Player Decision Requested (Mohamed)
 It is Mohamed`s turn
                         The first dice shows 6.  You landed on the unpurchased property "Royce Hall".
        Roll Dice      The second dice shows 2.       Do you want to buy "Royce Hall" for Bruin$800?
                                                             Your current inventory is Bruin$4250.
                                    Okay
                                                  Yes (-Bruin$800)  No
Separation of Concerns                                   Front-
                                                          End
Systems should be divided into distinct
sections, or concerns, where each section
addresses a separate, specific goal,
purpose, or responsibility.
The goal is to make the system easier to
develop, maintain, and evolve.
Applies to all                                           Back-
  software                                                End
development.
  Not just UI
Read more here: https://www.geeksforgeeks.org/software-
engineering/separation-of-concerns-soc/
                CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  5
                Tobias Dürschmid
Separate the UI Implementation
from the Domain Logic of your Application
              Presentation Layer                                Doesn't have a
                                                                clue that there
     Displays information to the user and collects input
(e.g., positions of players on the board, style of the boards,   even is a UI.
                                                                  I mean who
                               buttons, ...)                     really needs a
                                                                  UI? Can't we
               Application Layer
                                                                    just have
         Implements the domain logic and behavior                command line
    (e.g., effects of community chest cards, player turns,       interfaces with
                                                                  the shell???
                   interactions between players)
Separate the UI Implementation
from the Domain Logic of your Application
              Presentation Layer                           Doesn't have a
                                                           clue that there
               Application Layer
                                                            even is a UI.
- Allows presentation later to register a                    I mean who
    callback for state changes (e.g., onBalanceChanged)     really needs a
                                                             UI? Can't we
- Allows presentation layer to retrieve information using
    getter functions (e.g., getCurrentBalance())               just have
                                                            command line
- Allows presentation layer to forward user events          interfaces with
    (e.g., buyProperty(name, user))                          the shell???
The Observer      Pattern descriptions have many different
Design Pattern     formats. At minimum it should describe
                       problem, context, and solution.
  Context
Scenarios requiring distributed event handling systems or highly
decoupled architectures like GUI, and signal processing
  Problem
How can a one-to-many dependency between objects be maintained
efficiently without making the objects tightly coupled?
  Solution
The pattern introduces two main roles: the Subject (the object
sending updates after it has changed) and the Observer (the object
listening to the updates of Subjects).
The Observer Design Pattern                             Read more here:
                                                        https://tobiasduerschmid.github.io/SE
                                                        Book/designpatterns/observer.html
                       *                                Subject                     Observer
Subject                    AbstractObserver                      register(self)
                          update(Event)                             Adds observer
register(Observer)
unregister(Observer)      doSomething()                          to list
notify(Event)
Data getData()            Concrete Observer                      notify(someEvent)
doSomething()             update(Event)
                                                                     Iterates over
                                                                      observer list
                                                                           update()
  e.g., your Domain       e.g., the presentation-later           getData()
 Object with a lot of      UI element that displays                         data
useful business logic       the data of the subject
                             whenever it changes
Design Patterns
                  found in many      It is a good solution,
                    instances        but not always the
                                            best one
   "A pattern is a common, acceptable solution to a
reoccurring problem that arises in a specific context"
  The problem is generic enough      Patterns always refer to a
so that the it generalizes beyond a  specific situation, goal, or
                                     trade-off. Patterns are not
          few concrete cases
                                           universally good
Design Patterns Add Roles To Classes / Objects
This is an Example of the Observer Design Pattern
_channel           NewsChannel                             for subscriber in self._subscribers:
                                                            subscriber.update()
          -_subscribers: list[Subscriber]
          -_latest_post: str                                                      _channel
          +follow(subscriber: Subscriber)
          +unfollow(subscriber: Subscriber)
          +publish_post(text: str)
          +get_latest_post(): str
          -_notify_subscribers()
                          0..* 1_subscribers
                            «ABC»
                      Subscriber
                       +update()
                MobileApp              EmailDigest
          -_channel: NewsChannel  -_channel: NewsChannel
          +update()               +update()
          post = self._channel.get_latest_post()
          print(f"[MobileApp] Push notification: {post}")
          CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React     11
          Tobias Dürschmid
Observer Pattern ­ Subscriber Interface
  # ==================================
  # OBSERVER INTERFACE
  # ==================================
  class Subscriber(ABC):
         """The Observer interface."""
         @abstractmethod
         def update(self):
                pass
Observer Pattern ­ Subject (NewsChannel)
# SUBJECT
class NewsChannel:
       """The Subject that maintains a list of subscribers."""
       def __init__(self):
              self._subscribers: list[Subscriber] = []
              self._latest_post: str = ""
def follow(self, subscriber: Subscriber):
       if subscriber not in self._subscribers:
              self._subscribers.append(subscriber)
def unfollow(self, subscriber: Subscriber):
       self._subscribers.remove(subscriber)
Observer Pattern ­ Subject (NewsChannel)
# SUBJECT
class NewsChannel:
[...]
def publish_post(self, text: str):
       self._latest_post = text
       self._notify_subscribers()
def get_latest_post(self) -> str:
       return self._latest_post
def _notify_subscribers(self):
       for subscriber in self._subscribers:
              subscriber.update()
Observer Pattern ­ Concrete Observers
  # CONCRETE OBSERVERS
  class MobileApp(Subscriber):
         """A concrete observer that pulls state from the channel."""
         def __init__(self, channel: NewsChannel):
                self._channel = channel
         def update(self):
                post = self._channel.get_latest_post()
                print(f"[MobileApp] Push notification: {post}")
  class EmailDigest(Subscriber):
         """Another concrete observer with different behavior."""
         def __init__(self, channel: NewsChannel):
                self._channel = channel
         def update(self):
                post = self._channel.get_latest_post()
                print(f"[EmailDigest] New email queued: {post}")
Observer Pattern ­ Client Code
  # CLIENT CODE
  channel = NewsChannel()
  app = MobileApp(channel)
  email = EmailDigest(channel)
  channel.follow(app)
  channel.follow(email)
  channel.publish_post("New video uploaded!")
  # [MobileApp] Push notification: New video uploaded!
  # [EmailDigest] New email queued: New video uploaded!
  channel.unfollow(email)
  channel.publish_post("Live stream starting!")
  # [MobileApp] Push notification: Live stream starting!
Lesson Learned:
Start By Considering Existing Solutions
· Most problems have been solved already and described in a
 well-documented way
· Knowing existing solutions and patterns in your field can
 save you a lot of time and effort
Use SVGs Whenever You Can
Scalable Vector Graphics (SVG) are     Bitmaps (e.g., JPEG, PNG, BMP, GIF,
vector-based graphics defined by       ...) are pixel-based graphics composed of
mathematical equations for shapes.     a fixed grid of colored squares. Due to
They scale to any size and often have  their general-purpose representation, they
smaller file sizes.                    are supported by more editing software.
There are many Different Color Representations
· RGB (Red, Green, Blue) is the most common format to represent digital colors.
  The resulting color is a mix of red, green, and blue light.
   · The Hex-Color representation uses two hexadecimal digits
     for each color (#RRGGBB e.g., #2774AE, #FFD100)
· HSL/BSH (Hue, Saturation, Luminance/Brightness)
  lets you represent colors based on their
  color tone (hue), and color intensity (saturation)
  and brightness (luminance)
· CMYK (Cyan, Magenta, Yellow, Keycolor (Black))
  represents colors based on the subtractive
  color model used in printing and is less common for screen-based rendering
                    See https://www.w3schools.com/tags/default.asp
Important HTML Tag
  Content Tags                           Top Heading
· Headings (<h1> to <h6>)                Example
    e.g., <h1>Top Heading</h1>
                                         1. first item
· Paragraphs (<p>) e.g., <p>Example</p>  2. second item
· Lists (<ul> unordered list
                                         · First item
    and <ol> ordered lists)              · Second item
    e.g., <ol>
                <li>first item</li>
                <li>second item</li>
         </ol>
                    See https://www.w3schools.com/tags/default.asp
Important HTML Tag
  Text Formatting Tags                     Important Text
                                           emphasis
· Bold (<strong> previously: <b> or <B>)   inserted
    e.g., <strong>Important Text</strong>  deleted
· Italics (<em> previously: <i> or <I>)
    e.g., <em>emphasis</em>
· Underline (<ins> previously: <U>)
    e.g., <ins>inserted</ins>
· Strikethrough (<del>)
    e.g., <del>deleted</del>
                    See https://www.w3schools.com/tags/default.asp
Important HTML Tag
  Text Formatting Tags                              Click me
                                                   anchor
· Button (<button>)
    e.g., <button>Click me</button>                    Check me
· Links (<a>)
    e.g., <a href="ucl.edu">anchor</a>
· Images(<img>)
    e.g., <img src="https://u.edu/l">
· Inputs (<input> with various types)
    e.g., <input type="checkbox">check
    me</input> (or text, file, email, radio, ...)
HTML DOM (Document Object Model) is a
programming interface for HTML documents.
It represents the structure of an HTML page as a tree of Document
objects, where each object corresponds to a part of the Root Element
document, such as an element, attribute, or text.
                                                         <html>
<html>                       Element                                  Element
<head>                       <head>                                   <body>
   <title>My Title</title>   Element Element                          Element
</head>                      <title> <h1>                               <a>
<body>
                                                   Text  Text         Attribute       Text
   <h1>Hello</h1>                                                     "href"        "Link"
   <a href="[...]">Link</a>  "Title" "Hello"
</body>                                                                Value
</html>                                                                "[...]"
CSS (Cascading Style Sheets)
· Declarative-style computer programming language used to define the visual
  presentation and layout of web pages.
· Works in conjunction with markup languages like HTML (which provide the
  structure and content of a webpage)
               CSS                      HTML
Visual presentation & Layout    Content & Structure
More details here:
https://www.w3schools.com/css/
CSS can define style classes for HTML divs
(divisions, used as containers)
  example.html                       example.css
<!DOCTYPE html>                    h1 { /* Heading 1 will be navy colored
<html>                             and underlined */
<head>
                                          color: navy;
   <title>CSS Example</title>             text-decoration: underline;
   <link rel="stylesheet"          }
                                   .rounded-box {/* Will become a class
              href="example.css">
</head>                                                           for divs*/
<body>                                    width: 200px;
                                          background-color: #FFD100;
   <h1>CSS Example</h1>                   border: 5px solid #2774AE;
   <div class="rounded-box">              border-radius: 10px; /* corners */
                                          text-align: center;
       <p>This will be styled             font-family: sans-serif;
            with CSS.</p>          }
   </div>
</body>
</html>
CSS can define style classes for HTML divs
(divisions, used as containers)
  example.html
<!DOCTYPE html>
<html>
<head>
   <title>CSS Example</title>
   <link rel="stylesheet"
              href="example.css">
</head>
<body>
   <h1>CSS Example</h1>
   <div class="rounded-box">
       <p>This will be styled
            with CSS.</p>
   </div>
</body>
</html>
React.js (aka. React, ReactJS)
· React is a free and open-source front-end JavaScript library for building user
  interfaces
· maintained by Meta (formerly Facebook)
· React has a declarative nature
   · you describe what you want the UI to look like for a given state
   · React handles the steps to achieve that UI and keep it updated efficiently
     when the state changes.
   · In contrast, imperative programming focuses on describing how to achieve a
     result by manually manipulating the UI elements
React uses JSX (JavaScript XML)
· JSX is a syntax extension that allows developers to write HTML-like code
within JavaScript. This makes it easier to define the structure and content of UI
components.  React.createElement('h1',null,'Hello, ',name,'!');
const name = "CS 35L"
[...]
JavaScript constant (or variable)
defined somewhere
return <h1>Hello, {name}!</h1>;    Hello, CS 35L
Use JavaScript within HTML-like
       tags via curly braces.
CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  35
Map function
 const names = ["Royce Hall", "Powell Library",
 "Kerckhoff Hall"]
 [...]
return (<ul>                        · Royce Hall
   {names.map((name, index) => (    · Powell Library
       <li key={index}>{name}</li>  · Kerckhoff Hall
   ))}
</ul>);
CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  37
React Applications use Components
· Components are self-contained, reusable pieces of UI   Can be referenced via:
  that manage their own rendering logic.                 <WelcomeMessage
                                                         name="CS35L"/>
· This modularity promotes code reusability, simplifies
  development, and improves maintainability.
· React Component are Defined as Functions with a Single Return Element
· Inputs to the functions are called proCposmponent Parameters
function WelcomeMessage(props) {                         Component Parameter
                                                         Reference
   return <h1>Hello, {props.name}!</h1>;
}                    Makes the component visible to outside modules
export default WelcomeMessage;
   CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  38
   Tobias Dürschmid
CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  39
React Components manage their own State
import React, { useState } from 'react';       Initial value of the state
function Counter(props) {
const [count, setCount] = useState(props.initialCount);
State variable     State change function name
const increment = () => {         Reference to the state variable. Changes
   setCount(count + 1);           whenever the state variable is changed via
                                  the state change function
};
return (
   <>
<p>Current Count: {count}</p>
<button onClick={increment}>
        Increment
</button> Click event handler (the name of the function to
   </>             be called when the user clicks on the button)
);
} export default Counter; Makes the component visible to outside modules
                CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  40
                Tobias Dürschmid
React Components manage their own State
import React, { useStatUen}wrfarpompr'orpesact';  Initial value of the state
function Counter({initialCount}) {
const [count, setCount] = useState(initialCount);
State variable  State change function name
const increment = () => {         Reference to the state variable. Changes
   setCount(count + 1);           whenever the state variable is changed via
                                  the state change function
};
return (
   <>
    <p>Current onClick={increment}>
     Increment
    </button> Click event handler (the name of the function to
</>                 be called when the user clicks on the button)
);
Count: {count}</p>                Makes the component visible to outside modules
    <button} export default Counter;
                CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  41
                Tobias Dürschmid
React Components manage their own State
import React, { useState Y}ofurocman'raedadctp'r;ops  Initial value of the state
function Counter({initialCount, countName}) {
const [count, setCount] = useState(initialCount);
State variable     State change function name
const increment = () => {         Reference to the state variable. Changes
   setCount(count + 1);           whenever the state variable is changed via
                                  the state change function
};
return (
   <>
<p>{countName}: {count}</p>
<button onClick={increment}>
        Increment
</button> Click event handler (the name of the function to
   </>             be called when the user clicks on the button)
);
} export default Counter; Makes the component visible to outside modules
                CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  42
                Tobias Dürschmid
CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  43
React employs a Virtual DOM
· Virtual DOM is a lightweight representation of the actual DOM.
· When data changes, React first updates the Virtual DOM, then efficiently
  calculates the differences with the real DOM and applies only the necessary
  changes to the real DOM.
· Minimizes direct DOM manipulation and improves performance
CS 35L Software Construction: Lecture 6 ­ UI Development, Design Patterns, & React  45
Single Page Application (SPA)                                   Most common in React
· SPAs are structured as a single HTML page that has no preloaded content.
· Content is loaded dynamically via JavaScript for the entire application and
  housed within a single HTML page.
· The JavaScript code houses all the data relating to the application logic, UI, and
  communication with the server
Client                                                          Server
                                                     GET (`/')
        Single Page App (contains all pages)
More details here:             /current_news
https://hygraph.com/blog/diff  /about
erence-spa-ssg-ssr
Server-Side Rendering (SSR)                       Frameworks like Next.js
                                                  (which use React) do this
· Clients receive a fully rendered page (rendered on the server)
· Requires more server compute
· Faster initial load time (since only the current page is sent)
· Longer response time for user interaction
· Better search engine optimization
Client                                                      Server
         Main page                                GET (`/')
More details here:                                GET (`/current_news')
https://hygraph.com/blog/diff
erence-spa-ssg-ssr             Current news page
Please Fill out your Exit Tickets on Bruin Learn!
Please summarize three insights you learned about UI Development, Design Patterns,
or React today.
Please explain how using the Observer Design Pattern for UI elements supports the design
principle of Separation of Concerns
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slides use images generated with Gemini, from Flaticon.com (Creators: Freepik)
and www.svgrepo.com (Creators: Solar Icons, Iconsax, Giovana, Esri)

```

---

# Appendix B: Lecture 10 Slides (raw extracted text) â€” Publish-Subscribe source

```text
CS 35L Software
Construction
Lecture 10 ­
Debugging & Git
                              detective in a crime movie where
Assistant Teaching Professor
Computer Science Department        you are also the murderer."
                                                              ­ Filipe Fortes
You see this Error. What do you do next?!
Talk to your Neighbor(s)!
mysql> CREATE DATABASE my_awesome_database
         -> CHARACTER SET utf8mb4
         -> COLLATE utf8mb4_unicode_ci;
ERROR 3680 (HY000): Failed to create schema directory
'my_awesome_database' (errno: 2 - No such file or directory)
                     Ask search engines (e.g., Google)
                    and AI tools (e.g., Gemini, ChatGPT)
Software Construction Practice:
Search For The Error Message
  Problem
You see an error message from your framework, library, or external service (not your
own code) that does not directly point you towards a solution.
  Context
As a human developer, you do not have infinite knowledge & experience. Nobody
expects you to understand every error message.
  Solution
1. Remove all project-specific identifiers from the input & output
2. Copy & paste input & output into search engine / AI tool
3. Carefully study the results, (cautiously) experiment with suggested solutions
4. (Only) after you cannot get any more useful information from online sources,
     ask your more experienced co-worker
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
· 1) Investigating symptoms to
  Reproduce the Bug
· 2) Locating the faulty code
· 3) Determining the root cause  "Debugging is like being the
  of the bug
· 4) Implementing and verifying  detective in a crime movie where
  a fix                               you are also the murderer."
                                                                     ­ Filipe Fortes
Do Debugging
Skills Matter?
· Software bugs cost  60 billion
  USD / year
· Validation (including debugging)
  can take up to 50-75% of
  development time
· When debugging, some
developers are three times as           "Debugging is like being the
efficient as others                 detective in a crime movie where
Hopefully you!                      you are also the murderer."
                                                                ­ Filipe Fortes
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
· 1) Investigating symptoms to
  Reproduce the Bug
· 2) Locating the faulty code
· 3) Determining the root cause  "Debugging is like being the
  of the bug
· 4) Implementing and verifying  detective in a crime movie where
  a fix                               you are also the murderer."
                                                                     ­ Filipe Fortes
Bug Terminology
  Fault                                             import sys
                                                    import math
The erroneous location in the code                  def cal_circumference(radius):
(e.g., variable not initialized)
                                                       diameter = 2 * radius
Error    Program execution                             circumference = diameter * math.pi
                                                       return circumference
An incorrect state during program execution
                                                    def __main__():
(e.g., variable has the wrong value)                   try:
                                                           input_radius = sys.argv[1]
         Error Reaches                                     C = cal_circumference(input_radius)
                                                           print(f"The circumference of a circle
Failure  System Boundary                                   with radius {input_radius} is: {C}")
                                                       except:
Observed incorrect outside behavior                        print("An error occurred
                                                                       but there is no failure")
(e.g., system crashes, incorrect output displayed)
How can we prevent this error from becoming a failure? __main__()
         CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git  7
         Tobias Dürschmid
How can we Debug these Xerox Scans?
Document:         Scan:
                  Are we looking at a
                  fault, error, or failure?
Source: https://www.dkriesel.com/en/blog/2013/0802_xerox-workcentres_are_switching_written_numbers_when_scanning
                                                         How to reproduce the bug?
Step 1: Reproduce the Bug What should we ask the
                   customer?
Goal / Motivation
Reproduce the bug to be able to observe it and check whether you've fixed it
Document:          Scan:
Source: https://www.dkriesel.com/en/blog/2013/0802_xerox-workcentres_are_switching_written_numbers_when_scanning
Reproduce the Bug & Write a Bug Report
   Reproduce the problem environment
· Problem environment = setting in which the problem occurs
· Configuration of the machine (e.g., hardware, operating system, settings,
   run-time dependencies, software versions, ...)
· Try to re-create the problem environment on a different machine
   Reproduce the problem history
· Problem history = steps necessary to re-create the problem
· Record a sequence of data inputs, user interactions, and
   communications with other components
· Additional variables: timing, randomness, physical influences, ...
See "Why Programs Fail ­ A Guide to Systematic Debugging" by Andreas Zeller 2009
If Possible, Write an Automated
Bug Reproduction Test
      Goal / Motivation
  Automated Tests allow you to check faster whether you've fixed the bug
      Write a Bug Reproduction Test
  Automate the steps of reproducing the bug & checking if the bug is present.
  Run your test several times during debugging
      Simplify your Test Case
  Find out what is relevant & remove all apparently irrelevant steps (e.g.,
  details in the input that do not seem to be connected to the bug and that do not
  affect whether the failure is present or not)
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
   · 1) Investigating symptoms to
     Reproduce the Bug
   · 2) Locating the faulty code
   · 3) Determining the root cause
     of the bug
   · 4) Implementing and verifying
     a fix
Investigate Symptoms                           import logging                                              Python Logging
Using Logs
                                               # Create a log format
· Logging Libraries help recording             fmt = logging.Formatter(
  symptoms
                                                      "%(name)s: %(asctime)s | %(levelname)s |
                                               %(filename)s:%(lineno)s | %(process)d >>>
                                               %(message)s"
                                               )
      · Inputs (particularly unexpected ones)  logging.debug("A debug message")
      · State changes                          logging.info("An info message")
      · Communication with other components    logging.warning("A warning message")
                                               logging.error("An error message")
Output                                         logging.critical("A critical message")
example: 2023-07-23 14:42:18,599 | INFO | main.py:30 | 187901 >>> Server started listening
on port 8080
example: 2023-07-23 14:14:47,578 | WARNING | main.py:28 | 143936 >>> Disk space on drive
'/var/log' is running low. Consider freeing up space
example: 2023-07-23 14:14:47,578 | ERROR | main.py:34 | 143936 >>> Failed to connect to
database: 'my_db'
Traceback (most recent call last):
File "/home/ayo/dev/betterstack/demo/python-logging/main.py", line 32, in <module>
raise Exception("Failed to connect to database: 'my_db'")
Exception: FailedCSto35LcoSnonftewcatretCoondsatrtuacbtiaons:eL:ec'tumrye_1d0b'­ Debugging & Advanced Git  14
              Tobias Dürschmid
Use Visual Diagrams to Locate Bugs
                                                                     Symptoms:
                                                                     · No
                                                                         sim_pose
                                                                         messages
                                                                     · Many fix
                                                                         messages
Focus on the Most Likely Origins
   Code Smells
Bugs are more likely to be in poorly written code.
Get rid of code smells via refactoring before debugging
   Look for common bugs
e.g., uninitialized variables, unused values, unreachable code, memory leaks,
interface misuse, null pointers, inconsistent types, ...
   Assertions
Add assertions to your code to prevent errors from propagating
(e.g., assert input > 0, assert file != null)
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
   · 1) Investigating symptoms to
     Reproduce the Bug
   · 2) Locating the faulty code
   · 3) Determining the root cause
     of the bug
   · 4) Implementing and verifying
     a fix
Your most valuable root
   cause analysis tool
Rubber Duck Debugging                         And then we call the function
Can Help you Find Bugs                        recursively until ... oh wait! I
  Curse of Knowledge                               forgot the base case!
                                               Thanks for listening, Duck
Having a mental model of your solution,
you read what you intended to write, not
what you actually wrote.
  Explain Your Code to Your Duck
· Place a rubber duck on your desk.
· Explain to the duck what your code is
   supposed to do, line by line.
· At some point, you will tell the duck what
   your code should be doing next & realize
   that this is not what it actually does.
Use Break Points                       1 import sys
For Debugging
                                       2 import math
  Goal
                                       3 def cal_circumference(radius):
You want to observe the execution
of the program step by step and see    4 diameter = 2 * radius
variable values
                                       5 circumference = diameter * math.pi
  Break Point
                                       6 return circumference
An intentional stopping point
                                       7
  How it works
                                       8 def __main__():
Whenever the program execution
reaches the break point, it stops and  9 try:
opens interactive control over the
program to step through and watch      10                   input_radius = sys.argv[1]
                                       11                   C = cal_circumference(input_radius)
                                       12                   print(f"The circumference of a circle
                                       13                   with radius {input_radius} is: {C}")
                                       14 except:
                                       15                   print("An error occurred
                                       16                   but there is no failure")
                                       17
                                       18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
   Define Inputs & Run                 1 import sys
   Configuration
                                       2 import math
                                       3 def cal_circumference(radius):
                                       4 diameter = 2 * radius
      Debug Configuration              5 circumference = diameter * math.pi
      in VS Code
                                       6 return circumference
                                       7
launch.json                            8 def __main__():
"version": "0.2.0",                    9 try:
"configurations": [                    10                   input_radius = sys.argv[1]
   {                                   11                   C = cal_circumference(input_radius)
      "name": "Python Debugger:        12                   print(f"The circumference of a circle
      Current File",                   13                   with radius {input_radius} is: {C}")
      "type": "debugpy",               14 except:
      "request": "launch",  inputs     15                   print("An error occurred
      "args": ["10"],
      "program": "${file}",            16                   but there is no failure")
      "console": "integratedTerminal"  17
   }                                   18 __main__()
]
See https://code.visualstudio.com/docs/debugtest/debugging
             CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git        21
             Tobias Dürschmid
Observe Program                         1 import sys
Step by Step
                                        2 import math
             Indicates where we are in
             the program execution      3 def cal_circumference(radius):
Allows you to evaluate                  4 diameter = 2 * radius
expressions in the
scope of the program                    5 circumference = diameter * math.pi
           Debug Console                6 return circumference
 sys.argv[1]
                                        7
     '10'
 input_radius                           8 def __main__():
      NameError: name                   9 try:
      'input_radius'
      is not defined                    10                  input_radius = sys.argv[1]
                                        11                  C = cal_circumference(input_radius)
                                        12                  print(f"The circumference of a circle
                                        13                  with radius {input_radius} is: {C}")
                                        14 except:
                                        15                  print("An error occurred
                                        16                  but there is no failure")
                                        17
                                        18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Observe Program            1 import sys
Step by Step
                           2 import math
     Variables
                           3 def cal_circumference(radius):
      Locals
                           4 diameter = 2 * radius
     input_radius = '10'
                           5 circumference = diameter * math.pi
      Globals
                           6 return circumference
Current values of
variables in the scope of  7
the current statement
                           8 def __main__():
                           9 try:
                           10                               input_radius = sys.argv[1]
                           11                               C = cal_circumference(input_radius)
                           12                               print(f"The circumference of a circle
                           13                               with radius {input_radius} is: {C}")
                           14 except:
                           15                               print("An error occurred
                           16                               but there is no failure")
                           17
                           18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Observe Program                 1 import sys
Step by Step
                                2 import math
    Variables
      Locals                    3 def cal_circumference(radius):
     radius = '10'              4 diameter = 2 * radius
      Globals
                                5 circumference = diameter * math.pi
     We have stepped into this
     Function                   6 return circumference
                                7
                                8 def __main__():
                                9 try:
                                10                          input_radius = sys.argv[1]
                                11                          C = cal_circumference(input_radius)
                                12                          print(f"The circumference of a circle
                                13                          with radius {input_radius} is: {C}")
                                14 except:
                                15                          print("An error occurred
                                16                          but there is no failure")
                                17
                                18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Observe Program          1 import sys
Step by Step
                         2 import math
     Variables
      Locals             3 def cal_circumference(radius):
     radius = '10'       4 diameter = 2 * radius
     diameter = '1010'
      Globals            5 circumference = diameter * math.pi
New variables appear as
they are assigned        6 return circumference
                         7
                         8 def __main__():
                         9 try:
                         10                                 input_radius = sys.argv[1]
                         11                                 C = cal_circumference(input_radius)
                         12                                 print(f"The circumference of a circle
                         13                                 with radius {input_radius} is: {C}")
                         14 except:
                         15                                 print("An error occurred
                         16                                 but there is no failure")
                         17
                         18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Observe Program                  1 import sys
Step by Step
                                 2 import math
    Variables
      Locals                     3 def cal_circumference(radius):
     input_radius = '10'         4 diameter = 2 * radius
      Globals
                                 5 circumference = diameter * math.pi
     Exception handling is
     triggered by multiplying a  6 return circumference
     string with a float
                                 7
                                 8 def __main__():
                                 9 try:
                                 10                         input_radius = sys.argv[1]
                                 11                         C = cal_circumference(input_radius)
                                 12                         print(f"The circumference of a circle
                                 13                         with radius {input_radius} is: {C}")
                                 14 except:
                                 15                         print("An error occurred
                                 16                         but there is no failure")
                                 17
                                 18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Conditional Break Points ­ Skip Directly to the
Execution You Want to Observer
    Goal
  You want to observe the program only if certain conditions are true (e.g., the
  input is very large or the input contains a certain character, or the loop variable
  has reached 10).
  Conditional Break Points
Break points that trigger only when a given expression evaluates to true.
  How it works
Right-click on the break point to edit it. The expression can include functions and
variables that exist in the code of this statement.
See https://code.visualstudio.com/docs/debugtest/debugging
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
   · 1) Investigating symptoms to
     Reproduce the Bug
   · 2) Locating the faulty code
   · 3) Determining the root cause
     of the bug
   · 4) Implementing and verifying
     a fix
Document The Bug Fix & Run Your Tests
    Add Assertions
After fixing the bug, add assertions to detect nearby bugs & run tests
   Document your Bug Fix
In the code, explain why your fix was necessary in a comment
Reference the bug in your git commit message
Document your solution in the bug report to allow historical traceability
   After Fixing, Keep all new Tests for Regression Testing
Regression Testing = re-running existing tests after code changes to ensure
that new updates, bug fixes, or features haven't introduced new problems into
existing functionality
CS 35L Software
Construction
Lecture 10 ­ Advanced
Git Techniques
Assistant Teaching Professor
Computer Science Department
Rapid Fire Quiz on Git
  What is HEAD?
The pointer to the currently selected commit / branch
  What is the staging area?
    f.
Code that is about to be committed in the next commit
  What is a branch?
A pointer to the most recent commit of a sequence of separately developed code
  What is a merge conflict?
The attempted merge of lines in the code that have been changed by both versions
  What makes a merge commit different?
It has two parents
Detached HEAD                    git checkout main  main HEAD
                                                    E main HEAD
    Non-Detached HEAD                ABCD
  HEAD is attached to a branch.  git commit
  Most Git commands, such as
  git commit, will move the          ABCD
  branch & HEAD.
  Detached HEAD                  git checkout D     main
                                                     HEAD
HEAD is attached to a commit,       ABCD
not a branch. Git commands,                         main
will NOT move the branch.        git commit         E HEAD
                                    ABCD
You are in 'detached HEAD' state. You can look around, make experimental changes and commit them, and you can
discard any commits you make in this state without impacting any branches by switching back to a branch.
Relative Commit Addresses
  Goal
You want to describe the parent commit (or grand parent commit, or great grand parent
commit) of a commit without having to get the commit ID from the log.
  How it works
BRANCH~n is the n-th commit before BRANCH.
feature~2 main~2           HEAD~1
A                 B        C       D
                                      HEAD main
                  f feature
                 See https://www.geeksforgeeks.org/git/git-cherry-pick/
Git Cherry-Pick
  Goal
You want to selectively "copy" one particular commit from one branch to another
branch without bringing over the entire history.
  How it works
git cherry-pick  calculates the difference (patch) for commit  and creates a new
commit that applies the same patch to the current base.
Note: This may result in merge conflicts if the patch cannot be applied directly.
           feature                           feature
                git cherry-pick                  
ABCD
                                  A B C D 
HEAD main                                                                          HEAD  main
              CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git               34
              Tobias Dürschmid
                                              See https://www.geeksforgeeks.org/git/git-stash/
Git Stash & Pop
  Problem
You want to checkout a branch or commit while you have uncommited changes
  Context
You might need your uncommited changes later
  Solution
Use git stash to move your changes to a separate stashing location
and use git stash pop to bring them back
  How it works
Git maintains a stack of stashes changes. Every time you call git stash it adds these
patches to the stashing stack. Pop removes them from the stack. git stash list
shows you the entire stack. git stash apply <id> applies a non-top stash patch
git blame ­ Which Commit                                        See https://www.geeksforgeeks.org/git/how-
Last Changed a Dubious Line?                                                                         to-use-git-blame/
  Goal
For a given line you want to identify why it exists / why it has been implemented the way
it has, or who has most recently changed this line.
  How it works                    git blame .\my_stack.py
git blame <filename> shows        ae721935 (TD 10-25 15:04 1) class MyStack:
you, for every line in the file:
· Commit id of the last commit    562ad4a6 (TD 10-25 15:06 2)   """
    that modified this line       562ad4a6 (TD 10-25 15:06 3)   A simple implementation of a Stack data str
· Author of that commit
· Timestamp of that commit        562ad4a6 (TD 10-25 15:06 4)
GitHub visualizes this very well  562ad4a6 (TD 10-25 15:06 5)   A Stack operates on the Last-In, First-Out
                                  562ad4a6 (TD 10-25 15:06 6)   the last element added to the stack is the
                                  562ad4a6 (TD 10-25 15:06 7)   """
                                  562ad4a6 (TD 10-25 15:06 8)
                                  6f33a948 (TD 10-25 18:58 9)   class PopOnEmptyStackException(Exception):
                                  6f33a948 (TD 10-25 18:58 10)       """
                                  6f33a948 (TD 10-25 18:58 11)       Exception raised when trying to call po
                                  6f33a948 (TD 10-25 18:58 12)       """
                                  6f33a948 (TD 10-25 18:58 13)       def __init__(self, message="Cannot pop
                                  6f33a948 (TD 10-25 18:58 14)            super().__init__(message)
                                  6f33a948 (TD 10-25 18:58 15)
                                  07768051 (TD 10-25 16:29 16)  def __init__(self):
                                  47c50cfe (TD 10-25 16:44 17)       """
git bisect ­ Which Commit                           See https://www.geeksforgeeks.org/git/git-bisect/
Introduced the Fault?
  Goal
Out of hundreds of commits, how to find the one that broke the tests?
How it works                                     A  A                         A  A
1. Identify a past commit c_good that worked     B  B                         B  B
2. Write a command test that tests whether a     C  C                         C  C
commit works and that can run on past
commits                                          D  D                         D  D
3. Run: git bisect start c_good                  E  E                         E  E
&& git bisect run test
4. Git will run a binary search and stop on the  F  F                         F  F
commit that broke the tests                      G  G                         G  G
         CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git        37
         Tobias Dürschmid
git revert ­ The Undo Button  See https://www.geeksforgeeks.org/git/how-
For Changes in the Past                     to-revert-a-commit-with-git-revert/
  Goal
You want to "undo" a commit in the history of your branch (e.g., it introduced a bug)
  How it works
git revert B calculates the difference (patch) for commit B and creates a new
commit that applies the inverse patch to the current base. B is still in the commit
history. The default commit message for the reverting commit is "Revert B".
Note: This may result in merge conflicts if the inverse patch cannot be applied directly.
This command is non-destructive (preserves the entire history).
ABCD  git revert B            A B C D -B
HEAD main                                                                          HEAD  main
              CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git               38
              Tobias Dürschmid
git rebase ­i                                              See https://git-scm.com/book/en/v2/Git-
Rewrite Your Own History                                                        Tools-Rewriting-History
  Goal
You want to "undo" one or more commits in without introducing a revert commit.
   How it works                                         pick B "Let's see if they find my bug"
                                                        pick C "Harmless changes"
git rebase -i A rewrites the commit history             pick D "Implement magical feature"
starting from commit A. If changes are made (e.g.,
commits should be dropped or added within a break) git  # Rebase A..D onto A (3 commands)
creates new commits with the same patches. This         # Commands:
command is destructive (the result is a new commit      # p, pick <commit> = use commit
history which may require push --force to update)       # r, reword <commit> = use commit, but
                                                        edit the commit message
                                                        # b, break = stop here (continue rebase
                                                        later with 'git rebase --continue')
                                                        # d, drop <commit> = remove commit
                                                        [...]
ABCD              git rebase -i A                       A  C'        D'  Destructive
    HEAD main        Then drop B                                         New commits
                                                        HEAD main
git rebase ­i                                 See hTttapsm://gpit-esTcrmoion.clsog-mRe/wbworioittkihn/egn-H/vi2s/tGority-
Rewrite Your Own History                       time could destroy
    Data Loss Risk                                  everything!
  If you get confused, mess up a step, or
  accidentally drop a commit you needed, you
  could lose work. Always ensure your
  changes are committed (or stashed) before
  starting a rebase.
    The Safe Zone
  When working in teams, only use git
  rebase -i on commits that exist only on
  your local machine or on a feature branch
  that only you are working on.
                                                                                                                             See https://www.geeksforgeeks.org/git/rebasing-
                                                                                                                                                                       of-branches-in-git/
Simple Rebase ­ An Alternative to Merge
  Goal
You want to merge a feature branch but instead of a merge commit with two parents you
just want to "copy" the individual commits
   How it works
git rebase f combines the patches from all commits on the feature branch into one
instead of creating a merge commit with two parents
                   feature HEAD                                      ' ' '
ABCD                   git rebase  A B C D feature
                 main       main                                 HEAD
                                                    main
                                                                                                                             See https://www.geeksforgeeks.org/git/rebasing-
                                                                                                                                                                       of-branches-in-git/
Simple Rebase ­ An Alternative to Merge
  Goal
You want to merge a feature branch but instead of a merge commit with two parents you
just want to "copy" the individual commits
   How it works
git rebase f combines the patches from all commits on the feature branch into one
instead of creating a merge commit with two parents
                   feature                                                feature
                            ABC                                      
ABCD       git rebase                                                D' main
             feature                                                       HEAD
HEAD main                                                                                42
                                                                                                                                     See https://www.geeksforgeeks.org/git/git-
                                                                                                                                                                                     squash/
Squash Merge keeps your History Simple and Clean
  Goal
You want to merge a feature branch but instead of keeping all individual commit you want
just one single commit for the new feature
   How it works
git merge --squash combines the patches from all commits on the feature branch
into one instead of creating a merge commit with two parents
                 feature                                                 feature
ABCD                         ABCD                                    
                  git merge
    HEAD main       feature                HEAD                       
                   --squash                                           main
Git Submodules ­ Perfect for  See https://www.geeksforgeeks.org/git/git-
                                                                        submodule/
Large Projects with Multiple Repositories
   Goal
 You want to include external libraries or shared modules within your project while
 maintaining their history and keeping them separate from your main repository.
  How it works
Conceptually, git submodules keep a Git repository as a subdirectory of another Git
repository. Internally, they are represented as a file that points to the commit ID that
should be checked out in the submodule. You change the content of the submodule by
committing this file (which has the same name as the directory).
   Examples
https://github.com/cmu-rss-lab/rosdiscover-evaluation/tree/main/deps
https://github.com/cmu-rss-lab/rosdiscover-evaluation/tree/main/deps
https://github.com/UCLA-CS-35L/submodule_example_parent/
https://github.com/UCLA-CS-35L/submodule_example_parent/
Common Submodule Commands                              See https://www.geeksforgeeks.org/git/git-
(Yes, this is initially confusing)                                                               submodule/
  Add a Submodule
git submodule add <repo-url> <path>
Clones <repo-url> into <path> and starts keeping track of it as a submodule
  Clone a Repository that has Submodules
git clone --recursive <repo-url>
Clones <repo-url> and their transitive submodules and checks out the selected commit
  Reset Submodule Content to the Selected Commit
git submodule update
Clones all submodule repos (and their submodules) and checks out the selected commit
  Commit Content inside the Submodule
Commit the changes within the submodule folder & push
Change the directory to outside of the submodule, commit the submodule & push
Tutorials for this Lecture
· Advanced Git Tutorial
   · https://tobiasduerschmid.github.io/SEBook/tools/git-advanced-tutorial
   · Run rebase, bisect, and other commands in git with our grit graph visualization
· Debugging Tutorial
   · https://tobiasduerschmid.github.io/SEBook/tools/python-debugging
   · Use our time traveling debugger to step go back and forth in time
Please fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three key insights you learned about Debugging
& Git today. (Do not copy phrases from the lecture slides)
What is the difference between Testing & Debugging? Please describe the difference based
on an example.
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slide use images from Flaticon.com (Creators: Freepik)
Publish-Subscribe implements an N-to-M
Messaging Channel
    Problem
How to keep the state of cooperating components synchronized?
    Context
Components should be loosely coupled and reduce direct dependencies
Publisher            Usually identified  Message                     Subscriber
                     via string names     Queue
       publishes to
                           Topic         subscribes to
Publisher
                                         Message                     Subscriber
                                          Queue
Publish-Subscribe implements an N-to-M
Messaging Channel
Produces data or sends     How does this   Consumes data or
notifications about state   relate to the  listens to state
updates                      Observer?     updates
Publisher                                  Message                   Subscriber
                                            Queue
       publishes to
                           Topic  subscribes to
Publisher
                                           Message                   Subscriber
                                            Queue
Publish-Subscribe implements an N-to-M
Messaging Channel
pub = nh.advertise("topic_name");
...
                                 nh.subscribe("topic_name", 10, callback);
pub.publish(msg)
Publisher                   Message                                  Subscriber
                             Queue
       publishes to
                     Topic  subscribes to
Publisher
                            Message                                  Subscriber
                             Queue

```

---

# Appendix C: Lecture 17 Slides (raw extracted text) â€” Adapter Pattern source

```text
CS 35L Software
Construction
L17 ­ Code
Comprehension &
API Design
Assistant Teaching Professor
Computer Science Department
Why Comprehension Skills Matter
· Developers spend 58% of their time
 on understanding code
· A speedup in code understanding has a
 large impact on productivity
· With AI writing more and more code, developers might
spend even more time on reading code in the near future
Code Comprehension Approaches
"Bottom-Up" (Novices)                 "Top-Down" (Experts)
· Reading code line by line     · Driven by a hypothesis of what the
    grouping statements into        code does
    logical "chunks" of higher
    abstraction                 · Looking for "beacons" (key lines of
                                    code that reveal the core purpose
· Takes more time because           of a function or test the hypothesis)
    each statement needs to be
    analyzed                    · First getting the big picture,
                                    then looking for details
Move from here to there
Tips on How to Transition from Bottom-Up
Towards Top-Down Code Reading
     Build Domain Knowledge
   The top-down approach is only possible when you are familiar with the
   application domain.
   Without domain knowledge, programmers lack the necessary schemas to form
   expectations and are forced to fall back on bottom-up chunking.
   Tip: Before looking at the source code, invest time in reading requirements,
   architectural documents, and high-level system summaries.
   Understanding what the software is supposed to do in the real world provides
   the foundation for your top-down expectations.
Tips on How to Transition from Bottom-Up
Towards Top-Down Code Reading
     Practice Hypothesis-Driven Reading
   Instead of tracing execution flows immediately, practice generating an initial,
   general hypothesis about what a program or module does based on its
   name, context, or documentation.
   Tip: Once you have a primary hypothesis (e.g., "This program produces
   invoices"), subdivide it into finer subsidiary hypotheses (e.g., "It must have a
   data retrieval method and a formatting method") before looking at the code.
   You then only read the code to validate or reject these specific expectations.
Tips on How to Transition from Bottom-Up
Towards Top-Down Code Reading
     Hunt for "Beacons"
  A top-down reader does not read every line; they scan for "beacons".
  Beacons are stereotypical segments of code, recognizable variable names, or
  distinct structures that reliably indicate the presence of a specific operation or
  algorithm. At its core, a beacon is a recognizable, familiar point in the
  source code that serves as a mental shortcut for the programmer. They
  are defined as "signs standing close to human thinking that may give a
  hint for the programmer about the purpose of the examined code"
Tip: Train yourself to scan files for recognizable patterns, method signatures, and comments. When
you spot a beacon, use it to instantly confirm your hypothesis without needing to trace the granular logic.
Beacons ­ The Structural Signs
Guiding Your Through the Code
     Identifier Names Are Beacons
  The most frequent and most critical beacons are the names developers
  assign to variables, functions, and classes. Comprehension often depends
  domain information carried by identifier names.
  Tip: First look for the names of important classes, functions, and variables. If
  you don not know their meaning (e.g., due to lack of domain knowledge) look
  up their meaning first!
Identifier Names -- Code Examples
Spot the Domain in the Names ­ What do the functiond do?
             Python                     React (JSX)                 Node.js (Express)
def calculate_apr(             function CartCheckout({       function authenticate(
      principal,                  items, taxRate,                  req, res, next) {
      interest_rate,              onPlaceOrder }) {
      term_months):               const subtotal = items        const token = req
                                     .reduce((s,i) =>              .headers.authorization;
   monthly = interest_rate/12            s + i.price, 0);
   n = term_months                return <button onClick=       if (!verifyJwt(token))
   return (                          {onPlaceOrder}>               return res.status(401)
                                     Pay {subtotal*taxRate}            .send('Unauthorized');
      (1+monthly)**n - 1          </button>; }
   ) * 100                                                      next();
                                                             }
Exercise: What domain does each snippet operate on? Which identifiers told you?
Beacons ­ The Structural Signs
Guiding Your Through the Code
     Identifier Names Are Beacons
  The most frequent and most critical beacons are the names developers
  assign to variables, functions, and classes. Comprehension often depends
  domain information carried by identifier names.
  Tip: First look for the names of important classes, functions, and variables. If
  you don not know their meaning (e.g., due to lack of domain knowledge) look
  up their meaning first!
Think-Pair-Share: What are other beacons besides identifier names?
Beacons ­ The Structural Signs
Guiding Your Through the Code
     Code Structures Are Beacons
   Physical layout of these statements--often referred to as text-structure
   knowledge--serves as a visual beacon. Loops help you identify structural
   patterns. Guard clauses help you identify early exit paths.
   Tip: Look for the algorithms used to solve the problem.
Code Structures -- Code Examples
Recognize the Algorithm by Its Shape
                Python                   React (JSX)                         Node.js
def max_depth(node):         useEffect(() => {              app.post('/orders', (req,res) => {
   # guard clause               let attempts = 0;              // guard
   if node is None:             // retry loop                  if (!req.user)
      return 0                                                    return res.status(401).end();
   # recursive shape            while (attempts < 3) {         // map/reduce shape
   return 1 + max(                 try {                       const total = req.body.items
      max_depth(node.left),            return fetchData();        .map(i => i.price * i.qty)
      max_depth(node.right)        } catch {                      .reduce((a,b) => a+b, 0);
   )                                   attempts++;             res.json({ total });
                                   }}                       });
                             }, [url]);
Exercise: Predict -- what does each snippet compute? Which structural shape (loop,
                                      recursion, guard) gave it away?
Beacons ­ The Structural Signs
Guiding Your Through the Code
     Tests Are Beacons
   When reading unfamiliar code, a developer's primary challenge is deducing
   the original author's intent. Tests act as explicit beacons that illuminate this
   intent by providing an executable, unambiguous specification of how the
   production code should work
   Tip: When reviewing code, look at the tests first. Verify that the tests pass on
   the current code before spending too much time reading them
Tests -- Code Examples
Tests Spell Out the Spec
        Python (pytest)                  React (Playwright)                  Node.js (Jest + supertest)
def test_no_discount():         import { test, expect }               test('GET /users  200', async () => {
   assert apply(100, 0) == 100     from '@playwright/test';              const res = await request(app)
                                                                            .get('/users');
def test_half_off():            test('checkout disables', async          expect(res.status).toBe(200);
   assert apply(100, 50) == 50     ({ page }) => {                       expect(res.body).toHaveLength(3);
                                   await page.goto('/cart');
def test_rejects_negative():       const btn = page.getByRole(        });
   with pytest.raises(                'button', { name: /pay/i });
      ValueError):                 await expect(btn).toBeDisabled();  test('unknown id  404', async () => {
      apply(100, -5)               await page.getByLabel('Terms')        const res = await request(app)
                                      .check();                             .get('/users/999');
                                   await expect(btn).toBeEnabled();      expect(res.status).toBe(404);
                                });                                   });
Exercise: From the tests alone, describe a one-sentence spec for each function.
Beacons ­ The Structural Signs
Guiding Your Through the Code
     Assertions Are Beacons
   Zooming in from the file level to the statement level, the individual assertions
   within a test (or embedded within production code) act as highly localized
   beacons. Check for pre- and post conditions (and invariants) specified in the
   code.
   Tip: Add assertions in your own code to make it easier for others to use top
   down reading approaches.
Assertions -- Code Examples
Pre/Postconditions Pin Down Intent
                Python                           React                             Node.js
def withdraw(account, amount):  function useSlider(min, max) {    const assert = require('assert');
   # preconditions                 // precondition                function transfer(from, to, amt) {
   assert amount > 0               invariant(min < max);
   assert account.balance >=       const [v, setV] =                 // preconditions
                                                                     assert(from !== to);
amount                          useState(min);                       assert(amt > 0);
                                   const update = (n) => {           assert(from.balance >= amt);
   account.balance -= amount          invariant(                     from.balance -= amt;
                                          n >= min && n <= max);     to.balance += amt;
   # postcondition (invariant)        setV(n);                       // postcondition
   assert account.balance >= 0     };                                assert(from.balance >= 0);
   return account.balance          return [v, update];            }
                                }
Exercise: List an input that would crash each function.
        The assertions tell you -- point to the line.
Beacons ­ The Structural Signs
Guiding Your Through the Code
     Diagrams Are Beacons
  UML Diagrams help build a top-down mental model of the system.
  When reading a code base, first look for diagrams. Visual diagrams often help
  understanding important complex concepts more efficiently.
  However: Sometimes diagrams are outdated, which can lead to incorrect
  hypotheses.
Tips on How to Transition from Bottom-Up
Towards Top-Down Code Reading
     Continuous Cycle of Hypothesis Testing
  Hypothesis Generation: e.g., the developer assumes the system
  must have a "database connection" module.
  Beacon Hunting: The developer scans the code looking for beacons,
  such as an SQL library import, a connectionString variable, or a
  db_connect() method.
  Verification or Rejection: If the anticipated beacons are found, the
  hypothesis is verified and becomes a part of the mental model; if the
  beacons are missing, the hypothesis is declined, and the programmer
  must adjust their assumptions.
Tips on How to Transition from Bottom-Up
Towards Top-Down Code Reading
     Embrace "Opportunistic Processing"
   Experts rarely use top-down processing in strict isolation; instead, they act as
   "opportunistic processors" who continuously toggle between top-down and
   bottom-up strategies.
   Tip: Use top-down methods to quickly navigate and build the big picture. When
   you encounter a highly complex, unfamiliar, or localized piece of logic where
   your hypotheses fail, seamlessly drop into a bottom-up approach to
   understand that specific chunk, and then immediately return to your top-down
   overview
API Design &
Interoperability
Assistant Teaching Professor
Computer Science Department
Bob Wants To Book
Flights So That He
Can Go On Vacation
Let me
Tell You
A Story...
Bob's Flight has been
Cancelled
The Airline Agent Tries
To find an Alternative
Flight
Bob has been
Re-Booked
Onto a Flight with A
Different Airline
THE END
What makes this story difficult to implement?
                                There needs to be
                                a system that can
                                interoperate with
                                all airlines and all
                                booking systems
     Bob sees flights from                            Bob can get re-booked to a
hundreds of different airlines                               different airline
The Global Distribution System (GDS) lets Almost
all Airlines & Booking System Interoperate
The invention of GDS is closer historically to the invention of flight by
the Wright brothers than to today
          Request Flights       Register Flights
          Book Flights                Sell Flights
Booking                                                                    Airline 1
System 1                                                                   Airline 2
                           GDS
Booking                         Manage Passengers Airline 3
System 2
                                   Manage Baggage
What makes GDS Interoperable?
          Invent a Design Principle to Design Systems for
          Interoperability. Talk to Your Neighbor(s)
          Request Flights       Register Flights
          Book Flights                Sell Flights
Booking                                             Airline 1
System 1
                           GDS                      Airline 2
Booking                         Manage Passengers Airline 3
System 2
                                   Manage Baggage
Design Principle for Interoperability:
Create Shared Interfaces / Data Formats
1. List all data that needs to be exchanged
2. Define an interface / data format that supports all data
3. Implement serialization & deserialization
Design Principle for Interoperability:
Create Shared Interfaces / Data Formats
Build Abstractions  The interface / document format should
                    not expose any implementation details
                    of systems / components.
Ensure Language- and The format should be supported by all
                                         programming languages, operating systems,
Platform-Independence and devices.
Common Technique to Implement
Shared Interfaces: REST APIs
· REST (representational state transfer) is a stateless protocol (no server-side
  session) to exchange data in client-server systems via HTTP / HTTPS:
   · POST ­ Creates a Resource
   · GET ­ Reads a Resource
   · PUT ­ Updates a Resource
   · DELETE ­ Deletes a Resource
· Naming convention for URLs, based on resource identifiers
   · "/users/123" instead of e.g., many variations of /getUser?id=123
· Resources are often described via XML, YAML, JSON, or HTML
Example REST API:            ...
GDS API                      "travelPreferences": {
· Request: POST /getFlights            "flightType": "Direct",
                                       "maximumStopsQuantity": 1
                             },
                             "itineraryParts": [ {
                                       "departureAirportCode": "LAX",
                                       "arrivalAirportCode": "PIT",
                             ... ]}
                             ...
Alternative Techniques to Implement Shared Interfaces
(besides REST)
RPC                Calling a function on a remote server as if it were local. Can be
                   stateful. Functions and actions beyond CRUD. Good for
(Remote Procedure  complex calculations. Can use multiple document formats.
Call)
SOAP               Can be stateful. More complex. Sometimes slower. Has
                   more security features. Has built-in error handling. Good for
(Simple Object     distributed enterprise environments. Uses XML.
Access Protocol)
GraphQL            Server-side schema defines types, enabling checking of data
                   structure conformance. Good for large, complex, and
                   interrelated data sources. Uses JSON.
                  CS 35L Software Construction: Lecture 17 ­ Code Comprehension & API Design  32
                  Tobias Dürschmid
Common Technique to Test Compatibility:
XML / JSON / YAML Schema
· A schema describes the structure of document
· Schemas list attributes, their possible values, and complex types (e.g.,
  sequences, recursive types) for a document
· Validation of a document against a schema can be done automatically
  to test compatibility at run-time
Example JSON Schema
"properties": {
"departureAirportCode": {
             "type": "string"   Constraints on the Type of a Property
             "pattern": "^[A-Z]{3}$"
},               RegEx!
"price": {
             "type": "number",
             "minimum": 0,      Constraints on the Values of a Property
             "exclusiveMinimum": true
          }  Constraints on Document Structure
},
"required": ["departureAirportCode", "price"]
             CS 35L Software Construction: Lecture 17 ­ Code Comprehension & API Design  34
             Tobias Dürschmid
Case Study
Mars Climate Orbiter
         Question: How could they have
                 prevented this bug?
     Spacecraft Lost Due to Lack of
         Semantic Interoperability
(Shared Interpretation of Shared Data)
Flight System Software                      Ground Software
     Developed by NASA JPL                Supplied by Lockheed Martin
                                           (US-based sub-contractor)
Expected commands in Command
                                           Sent commands in
N (SI units)                Interface   lbf (US Customary units)
Lesson Learned:
Syntactic Interoperability is Not Enough
                In-Class Activity: Invent a Design Principle
                          for Semantic Interoperability
Data exchanged between systems /   <plant>
components must be interpreted in
the same way by all systems /
components.
Design Principle for Interoperability:                                      <plant>
Define the Semantics of Shared Data
· Document Interfaces and their Semantics
  (e.g., What units are implied? Does price include tax?
  MM/DD/YY or DD/MM/YY?
  What coordinate system is used a reference frame?)
· Use Shared Dictionary of Items or Agree on Vocabulary
  (e.g., doughnut or donut?  or?)
· Write Integration Tests for the Systems
Interface Descriptions
Syntactic Describe document format, the actions that can be
View      performed, their parameters, and outputs.
Semantic  Describe the purpose / meaning of the resource / action:
View          · Side-effects: Changes to the state of a resource or
                 environment
              · Usage restrictions: Who can perform this action?
              · Error Handling: What errors can occur and why?
              · Examples: Examples of outputs for a given input
      CS 35L Software Construction: Lecture 17 ­ Code Comprehension & API Design  38
      Tobias Dürschmid
(Simplified)
Syntactic View of the GDS Flight Booking API
POST /createBooking {       The flight number using the code
  · flightNumber: str              of the operating airline
  · date: dateTime
  · seat: str          The date of the departing flight according to
                           the time zone of the departure airport
                       Seat number (same label as on the plane)
· paymentReference: PaymentMethod
· travelerName: str    Full name of the person traveling in ACSII
}
     In-Class Activity: Describe The Semantic
           View of GDS Flight Booking API
Example: Semantic View of GDS Booking
· Purpose: The airline confirms a requested booking
· Side-effects: money is charged, seat is marked as sold, can
  be canceled within 1 hour
· Usage restrictions: Authorized booking systems
· Errors: Invalid Format, Unauthorized, too many requests, ...
See here: https://developer.sabre.com/docs/rest_apis/trip/orders/booking_management
Guidelines on Interface Documentation
· Focus on how elements interact and their externally visible
 effects, not their implementation
· To support changeability of the implementation, expose only
 what is needed to use the interface
· Keep the documentation minimal and use-case oriented to
 increase readability
GDS Interface Documentation
                             In-Class Activity: Describe Pros and Cons of
                                     the GDS Interface Documentation
https://developer.sabre.com/docs/rest_apis/trip/orders/booking_management
GDS Interface Documentation
                               In-Class Activity: Describe Pros and Cons of
                                       the GDS Interface Documentation
                                       Semantic
                                   Interoperability
                              Via Shared Dictionary
                  Semantics of Level Are Unclear
However ... Now Many Airlines are Moving Away
From GDS
· GDS only lets you book seats
· Airlines make more money up-selling "priority boarding", or other add-ons
· Southwest doesn't have assigned seating
· Some airlines have seats that can be transformed into a sky couch, resulting in
  the unavailability in GDS
· Airlines do not get valuable customer data via GDS
What makes GDS Less Changeable?
In-Class Activity: Generate Design Options to Increase
          the Changeability of GDS Booking Options
          Request Flights       Register Flights
          Book Flights                Sell Flights
Booking                                             Airline 1
System 1
                           GDS                      Airline 2
Booking                         Manage Passengers Airline 3
System 2
                                   Manage Baggage
Making GDS More Changeable
Extensible Interfaces:
    · Offers can contain a dynamically-defined add-ons
    · Add-on: (price, name, description, id)
        · price(int): The price in cents (excl. tax) additionally charged when this add-on is
           selected
        · name(str): The name of the add-on as shown to the user (in UTF-8)
        · description(str): A short description shown to the user in order to decide if they want
           to purchase the add-on (in UTF-8)
        · id(str): Unique identifier of this add-on starting with the flight number (in ASCII)
    · A list of add-ons is added to a flight listing. The booking API needs to add an optional list of
       add-on ids to identify requested add-ons.
In-Class Activity: What Disadvantage does this have
                    over the existing GDS?
Lesson Learned: Interoperability
Often Conflicts with Changeability
· Shared interfaces and data format mean changes have to be
 implemented in all cooperating systems
· We cannot localize the interface change to a single system,
 so extensions / changes require a new version of the
 interface, which breaks interoperability
How to Evaluate Practical Interoperability?
These Design-Time Measures evaluate how likely a design for interoperability is to
 succeed in a practical setting. They can be used in quality attribute specifications.
                                          Often in Conflict
     Measure the Effort to
                                                                      Measure the Variability
Implement the Interface in all
                                                                Allowed by the Interface / Format
    Systems / Components
A more complex interface / data      An interface / data format that
format is less likely to be adopted  supports more use cases and
by many systems due to higher        potential extensions is more
implementation cost.                 likely to be adopted.
Design Pattern for Interoperability:                      Question: Does
Use Adapters to Connect Interfaces                         this only work
                                                            for Syntactic
Problem: Two systems use different interfaces             Interoperability?
(e.g., different data formats, different protocols, ...)
Your System  XML  JSON     Their System
Solution: Create an adapter component that translates between
the two interfaces.
Your System       Adapter  Their System
             XML-JSON Translation is localized here
Please Fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three key insights you learned about Code
Comprehension today. (Do not copy phrases from the lecture slides)
                      For one API that is used in your project for client server communication, write a first draft of an
                      interface specification. Include syntax (parameters and their type) and semantics (what is
                      the meaning of the overall request & the parameters? Which Errors could occur, ...)
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slides use images generated with Gemini and from Flaticon.com (Creators: Freepik)

```
