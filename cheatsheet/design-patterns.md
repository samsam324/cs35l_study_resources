# Design Patterns Cheatsheet (candidate items)

## What's a design pattern?
> A common, acceptable solution to a recurring problem in a specific context.

Pattern description has at minimum:
- **Problem** — recurring design problem
- **Context** — scenario where it arises
- **Solution** — template / roles
- **Consequences** — trade-offs

- Patterns are TEMPLATES, not code
- Patterns assign ROLES to your classes/objects
- "Start by considering existing solutions" — most problems are already solved

## 4 patterns explicitly covered in CS35L
1. **Observer** (L6)
2. **Publish-Subscribe** (L10)
3. **Adapter** (L17 — interoperability)
4. **Strategy** (L7 — supports Open-Closed Principle)

---

## 1. Observer Pattern (L6)

**Intent:** one-to-many notification. When subject changes, all observers are notified.

**Roles:**
- **Subject** (Observable) — sends updates
- **Observer** — listens for updates

**Structure:**
```
Subject                          AbstractObserver
- observers: list                +update(event)
+register(o)
+unregister(o)                       ▲
+notify(event)                       |
                              ConcreteObserver
                              +update(event)
```

**Lecture code (NewsChannel / Subscribers):**
```python
class Subscriber(ABC):                # Observer
    @abstractmethod
    def update(self): pass

class NewsChannel:                    # Subject
    def __init__(self):
        self._subscribers = []
        self._latest_post = ""
    def follow(self, sub):
        if sub not in self._subscribers:
            self._subscribers.append(sub)
    def unfollow(self, sub):
        self._subscribers.remove(sub)
    def publish_post(self, text):
        self._latest_post = text
        self._notify_subscribers()
    def get_latest_post(self):
        return self._latest_post
    def _notify_subscribers(self):
        for sub in self._subscribers:
            sub.update()

class MobileApp(Subscriber):
    def __init__(self, channel):
        self._channel = channel
    def update(self):
        post = self._channel.get_latest_post()
        print(f"[MobileApp] Push: {post}")
```

**Use case:** UI updates when domain state changes; event handling; React/Vue reactivity.

---

## 2. Publish-Subscribe Pattern (L10)

**Intent:** N-to-M messaging via an intermediary (broker/topic).

**Roles:**
- **Publisher** — emits events to a topic
- **Topic / Queue (Broker)** — routes events
- **Subscriber** — registers interest in a topic

**Difference from Observer:**
- Observer: direct subject↔observer coupling, in-process
- Pub-Sub: decoupled via broker, often across processes/services

**Use case:** event-driven architectures, microservices, message queues (Kafka, RabbitMQ, AWS SNS/SQS).

---

## 3. Adapter Pattern (L17 — interoperability)

**Intent:** make two incompatible interfaces work together by wrapping one.

**Roles:**
- **Target** — interface client expects
- **Adaptee** — existing class with incompatible interface
- **Adapter** — wraps Adaptee, exposes Target

```python
class EuropeanPlug:
    def round_pin_input(self): ...

class USOutlet:
    def flat_pin_output(self): ...

class PlugAdapter(USOutlet):    # exposes the US interface
    def __init__(self, plug: EuropeanPlug):
        self.plug = plug
    def flat_pin_output(self):
        return self.plug.round_pin_input()    # translation
```

**Use case:** integrating legacy code; bridging APIs; making third-party libs match your interface.

---

## 4. Strategy Pattern (L7 — Open-Closed)

**Intent:** swap interchangeable algorithms via composition; supports Open-Closed Principle.

**Roles:**
- **Strategy** (interface) — algorithm contract
- **ConcreteStrategy** — specific implementation
- **Context** — holds a Strategy, delegates to it

```python
class SortStrategy(ABC):
    @abstractmethod
    def sort(self, lst): pass

class QuickSort(SortStrategy):
    def sort(self, lst): ...
class MergeSort(SortStrategy):
    def sort(self, lst): ...

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self.strategy = strategy
    def run(self, lst):
        return self.strategy.sort(lst)
```

**Use case:** when adding behavior via delegation rather than modification (OCP).

---

## When to use which
| Pattern | Use when |
|---|---|
| Observer | one-to-many notifications, in-process |
| Pub-Sub | N-to-M messaging, decoupled, often cross-service |
| Adapter | bridge two incompatible interfaces |
| Strategy | swap interchangeable algorithms; supports OCP |

## Quick associations
- Observer = subject + listeners + notify
- Pub-Sub = publisher + topic + subscriber
- Adapter = wrapper that translates interfaces
- Strategy = interchangeable algorithm via composition

## Additional items (potentially missing)

### Pattern categories (GoF taxonomy — context)
- **Creational** — Factory, Singleton, Builder, Prototype
- **Structural** — Adapter, Decorator, Facade, Proxy, Composite
- **Behavioral** — Observer, Strategy, Command, Iterator, State, Template Method

### Singleton (brief — may be tested)
**Intent:** ensure only ONE instance exists.

```python
class Config:
    _instance = None
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

**Use case:** logger, config, DB connection pool.
**Critique:** often considered an anti-pattern (global state, hard to test). Use dependency injection instead.

### Factory pattern (brief)
**Intent:** encapsulate object creation.

```python
class ShapeFactory:
    @staticmethod
    def create(kind):
        if kind == "circle": return Circle()
        if kind == "square": return Square()
        raise ValueError(kind)
```
Hides instantiation logic from caller.

### Decorator pattern (brief)
**Intent:** add behavior to an object dynamically without subclassing.
```python
class Coffee:
    def cost(self): return 5

class WithMilk:
    def __init__(self, c): self.c = c
    def cost(self): return self.c.cost() + 1

class WithSugar:
    def __init__(self, c): self.c = c
    def cost(self): return self.c.cost() + 0.5

c = WithSugar(WithMilk(Coffee()))
c.cost()    # 6.5
```
**Distinct from** Python `@decorator` syntax (which is a different thing using the same name).

### Facade pattern (brief)
**Intent:** simplify a complex subsystem behind a single interface.
```python
class CarFacade:
    def __init__(self):
        self.engine = Engine()
        self.battery = Battery()
        self.fuel = FuelSystem()
    def start(self):
        self.battery.power_on()
        self.fuel.prime()
        self.engine.ignite()
```
Caller doesn't need to know about internals.

### Comparison: Observer vs Pub-Sub (deep)
| | Observer | Pub-Sub |
|---|---|---|
| Coupling | Subject knows observers | Decoupled via broker |
| Scale | In-process, same machine | Cross-process, distributed |
| Topology | One subject, many observers | Many publishers, many subscribers, via topic |
| Examples | React state, GUI events | Kafka, RabbitMQ, AWS SNS |

### Comparison: Strategy vs Template Method
- **Strategy** — composition: hold a Strategy object, delegate to it (preferred)
- **Template Method** — inheritance: parent class defines skeleton, subclasses override steps

### Pattern vs Anti-pattern
- **Pattern** = recurring solution that WORKS
- **Anti-pattern** = recurring solution that's TEMPTING but bad
- Examples of anti-patterns:
  - God Class
  - Spaghetti code
  - Magic numbers
  - Premature optimization
  - Cargo-cult programming
  - Hammer-and-nail ("when all you have is a hammer...")

### When NOT to use patterns
- When a simple function would do
- When you're forcing the pattern (cargo cult)
- Premature pattern adoption = unnecessary complexity
- **Rule:** apply patterns when the problem ARISES, not preemptively

### Lecture's 4 patterns mapped to where they appear
- **Observer** → L6 (UI / React)
- **Pub-Sub** → L10 (debugging/git lecture, separate slides)
- **Adapter** → L17 (API design / interoperability)
- **Strategy** → L7 (modeling, supports OCP in SOLID)

### Pattern → Principle mapping
- Observer / Pub-Sub → low coupling, separation of concerns
- Adapter → interface segregation, interoperability
- Strategy → Open-Closed, composition over inheritance
- Decorator → Open-Closed
- Facade → information hiding, low coupling
