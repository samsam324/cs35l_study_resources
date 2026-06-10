# Java Cheatsheet (candidate items)

## Entry point
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hi");
    }
}
```
- `public`: callable from outside
- `static`: no instance needed
- `void`: returns nothing
- `String[] args`: command line args

## 8 Primitives + Wrappers
| Primitive | Size | Wrapper |
|---|---|---|
| `byte` | 8 | `Byte` |
| `short` | 16 | `Short` |
| `int` | 32 | `Integer` |
| `long` | 64 | `Long` |
| `float` | 32 | `Float` |
| `double` | 64 | `Double` |
| `char` | 16 | `Character` |
| `boolean` | 1 | `Boolean` |

- Generics only work with objects → use wrappers in collections
- Autoboxing: `int → Integer` automatic
- Unboxing: `Integer → int` automatic

## Autoboxing traps
```java
Integer x = null; int n = x;   // NullPointerException
```
- Boxing in loops = slow (Integer allocations every iteration)

## == vs .equals()
| Use | When |
|---|---|
| `==` | Primitives, identity check of objects |
| `.equals()` | Object value comparison |

### Trap
```java
String a = new String("hi");
String b = new String("hi");
a == b;        // false (different objects)
a.equals(b);   // true

String x = "hi"; String y = "hi"; x == y;  // true (interned literals)

Integer p = 127, q = 127; p == q;  // true (cached -128..127)
Integer p = 128, q = 128; p == q;  // false
```
- **Rule**: `==` for primitives, `.equals()` for everything else.

## Access modifiers
| Modifier | Class | Package | Subclass | World |
|---|---|---|---|---|
| `private` | ✓ | ✗ | ✗ | ✗ |
| (default) package-private | ✓ | ✓ | ✗ | ✗ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| `public` | ✓ | ✓ | ✓ | ✓ |

- C++ default = private; Java default = **package-private** (trap!)
- UML symbols: `-` private, `+` public, `#` protected, `~` package

## Class
```java
public class BankAccount {
    private double balance;
    public BankAccount(double init) { this.balance = init; }
    public void deposit(double amt) { balance += amt; }
    public double getBalance() { return balance; }
    public String toString() { return "Bal=" + balance; }   // like Python __str__
}
```

## Interface
```java
public interface Shape {
    double getArea();
}
public class Circle implements Shape {
    public double getArea() { return ...; }
}
```
- Multiple inheritance allowed: `implements Shape, Comparable`
- Java 8+ allows `default` methods in interfaces

## Abstract class
```java
public abstract class Vehicle {
    public abstract String describe();   // must override
    public String getType() { return "vehicle"; }   // concrete
}
```

## Interface vs Abstract class
| | Interface | Abstract Class |
|---|---|---|
| Fields | only `static final` | any |
| Constructor | no | yes |
| Multiple? | yes | no |
| Use for | unrelated classes sharing behavior | related, sharing state |

## Inheritance
```java
public class Car extends Vehicle {
    public Car(String make) { super(make); }   // must be first
    @Override                                  // catches typos
    public String describe() { return "Car"; }
}
```
- Methods are **virtual by default** (no `virtual` keyword)
- `super(args)` must be first statement in constructor
- Single inheritance for classes

## Generics
```java
public class Box<T> { private T item; ... }
Box<String> b = new Box<>("Alice");

public static <T extends Comparable<T>> T max(T a, T b) { ... }
```
- **Type erasure**: `List<String>` and `List<Integer>` are SAME at runtime
- Can't do `new T()`, `ArrayList<int>` (must be `Integer`), `instanceof List<String>`

## Collections (interfaces)
| Type | Class examples | Notes |
|---|---|---|
| `List` | `ArrayList`, `LinkedList` | ordered, dupes ok |
| `Set` | `HashSet`, `TreeSet` | unique, unordered/sorted |
| `Map` | `HashMap`, `TreeMap` | key→value, unique keys |

| Class | Backing | Ordered | Sorted | Best for |
|---|---|---|---|---|
| `ArrayList` | array | insert | no | random access |
| `LinkedList` | doubly linked | insert | no | insert/delete ends |
| `HashSet` | hash | no | no | fast contains |
| `TreeSet` | red-black | sorted | yes | sorted iteration |
| `HashMap` | hash | no | no | fast lookup |
| `TreeMap` | red-black | sorted | yes | range queries |

## Common idioms
```java
List<String> names = new ArrayList<>();
names.add("Alice");
names.size();              // method, not .length

Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.getOrDefault("Bob", 0);   // safe lookup
```

## Length / size — INCONSISTENT
| Thing | How to get size |
|---|---|
| Array | `arr.length` (field, no parens) |
| String | `s.length()` (method) |
| Collection | `c.size()` (method) |

## Exceptions
```java
try {
    risky();
} catch (IOException e) {
    handle(e);
} catch (Exception e) {
    log(e);
} finally {
    cleanup();
}
throw new IllegalArgumentException("msg");
```
- **Checked** exceptions (extend Exception) must be caught or declared with `throws`
- **Unchecked** (extend RuntimeException) need not be caught

## Common traps
- `int x = map.get(missingKey);` → NPE (boxed Integer null unboxes)
- Default access = package-private, not private
- `arr.length()` is WRONG (use `.length` without parens)
- `s.length` is WRONG (use `.length()` with parens)
- Integer cache: -128..127 cached, == accidentally works for small ints

## Additional items (potentially missing)

### final keyword
- `final int x = 5;` — variable can't be reassigned
- `final class Foo` — class can't be subclassed
- `final void method()` — method can't be overridden
- Parameter `final` — can't reassign within method

### static
- `static` fields — shared across all instances
- `static` methods — called on class, not instance: `Math.max(...)`
- `static` block — runs once when class loaded
  ```java
  static { System.out.println("init"); }
  ```

### Enums (full class!)
```java
public enum Day {
    MON, TUE, WED, THU, FRI, SAT, SUN;

    public boolean isWeekend() {
        return this == SAT || this == SUN;
    }
}
Day d = Day.MON;
switch (d) { case MON: ... }
```

### Switch
```java
switch (x) {
    case 1: doA(); break;
    case 2: case 3: doB(); break;   // fall-through grouping
    default: doC();
}

// Java 14+ switch expression
String msg = switch (status) {
    case 200, 201 -> "ok";
    case 404 -> "not found";
    default -> "?";
};
```

### Casting / instanceof
```java
Object o = "hello";
if (o instanceof String) {
    String s = (String) o;             // downcast
    System.out.println(s.length());
}

// Java 16+ pattern variable
if (o instanceof String s) {
    System.out.println(s.length());     // s auto-cast
}
```

### Try-with-resources (auto-close)
```java
try (FileReader fr = new FileReader("f.txt");
     BufferedReader br = new BufferedReader(fr)) {
    String line = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
// fr and br closed automatically
```
Resource must implement `AutoCloseable`.

### Lambdas / functional interfaces
```java
Runnable r = () -> System.out.println("hi");
Function<Integer, Integer> sq = x -> x * x;
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
Predicate<String> nonEmpty = s -> !s.isEmpty();
Consumer<String> print = System.out::println;
Supplier<Date> now = Date::new;
```

### Streams API (functional collection ops)
```java
List<Integer> nums = List.of(1, 2, 3, 4, 5);

int sum = nums.stream()
              .filter(n -> n % 2 == 0)
              .mapToInt(Integer::intValue)
              .sum();

List<String> upper = words.stream()
                          .map(String::toUpperCase)
                          .collect(Collectors.toList());

Map<String, Long> count = words.stream()
                               .collect(Collectors.groupingBy(
                                   w -> w, Collectors.counting()));
```

### String building
```java
// BAD — creates new String each iteration (slow)
String s = "";
for (...) s += part;

// GOOD
StringBuilder sb = new StringBuilder();
for (...) sb.append(part);
String s = sb.toString();
```

### Iterators / Iterable
```java
for (String item : list) { ... }   // enhanced for, uses Iterator under hood

Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String item = it.next();
    if (cond) it.remove();          // safe removal during iteration
}
```

### Exception hierarchy
```
Throwable
├── Error (don't catch! JVM problems)
└── Exception
    ├── RuntimeException (unchecked)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── ClassCastException
    │   └── IllegalArgumentException
    └── (other) Exception types (checked — must catch or throws)
        ├── IOException
        └── SQLException
```

### throws clause
```java
public void load() throws IOException {
    // checked exceptions must be declared or caught
}
```

### Records (Java 14+) — immutable data classes
```java
public record Point(int x, int y) { }
Point p = new Point(1, 2);
p.x();   // accessor, auto-generated
// equals, hashCode, toString auto-generated
```

### Sealed classes (Java 17+)
```java
public sealed interface Shape permits Circle, Square { }
```

### Annotations beyond @Override
- `@Deprecated`
- `@SuppressWarnings("unchecked")`
- `@FunctionalInterface`
- `@SafeVarargs`

### Common Math methods
- `Math.max(a, b) / Math.min(a, b)`
- `Math.abs(x) / Math.sqrt(x) / Math.pow(b, e)`
- `Math.floor(x) / Math.ceil(x) / Math.round(x)`
- `Math.random()` — 0.0 ≤ x < 1.0

### Comparators
```java
list.sort(Comparator.comparingInt(Foo::getAge));
list.sort(Comparator.comparing(Foo::getName).reversed());
list.sort((a, b) -> a.getX() - b.getX());
```

### Implements Comparable for natural ordering
```java
class Pair implements Comparable<Pair> {
    public int compareTo(Pair other) {
        return this.x - other.x;
    }
}
```
