This is a reference page for Java, designed to be kept open alongside the Java Tutorial. Use it to look up syntax, concepts, and comparisons while you work through the hands-on exercises.

New to Java? Start with the interactive tutorial first — it teaches these concepts through practice with immediate feedback. This page is a reference, not a teaching resource.

Basics
Entry Point and Syntax
Java forces everything into a class. There are no free functions. The entry point is a static method called main — the JVM looks for it by name:

public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
Every word in the signature has a purpose:

Basics table
Keyword	Why
public	The JVM must be able to call it from outside the class
static	No instance of the class is created before main runs
void	Returns nothing; use System.exit() for exit codes
String[] args	Command-line arguments, like C++’s argv
Quick mapping from Python and C++:

Basics table
Feature	Python	C++	Java
Entry point	if __name__ == "__main__":	int main() (free function)	public static void main(String[] args) (class method)
Typing	Dynamic (x = 42)	Static (int x = 42;)	Static (int x = 42;)
Memory	GC + reference counting	Manual (new/delete) or RAII	GC (generational)
Free functions	Yes	Yes	No — everything lives in a class
Multiple inheritance	Yes (MRO)	Yes	No — single class inheritance + interfaces
// Variables — declare type like C++
int count = 10;
double pi = 3.14159;
String name = "Alice";     // String is a class, not a primitive
boolean done = false;      // not 'bool' (C++) or True/False (Python)

// Printing
System.out.println("Count: " + count);

// Arrays — fixed size, .length is a field (no parentheses)
int[] scores = {90, 85, 92};
System.out.println(scores.length);  // 3 — NOT .length() or len()

// Enhanced for — like Python's "for x in list"
for (int s : scores) {
    System.out.println(s);
}
Size inconsistency: Arrays use .length (field). Strings use .length() (method). Collections use .size() (method). This is a well-known Java wart.

The Dual Type System: Primitives and Wrappers
Java has 8 primitive types that live on the stack (like C++ value types), and corresponding wrapper classes that live on the heap:

Basics table
Primitive	Size	Default	Wrapper
byte	8-bit	0	Byte
short	16-bit	0	Short
int	32-bit	0	Integer
long	64-bit	0L	Long
float	32-bit	0.0f	Float
double	64-bit	0.0	Double
char	16-bit	'\u0000'	Character
boolean	1-bit	false	Boolean
Why wrappers exist: Java generics only work with objects, not primitives. You cannot write ArrayList<int> — you must write ArrayList<Integer>.

Autoboxing is the automatic conversion between primitive and wrapper:

ArrayList<Integer> numbers = new ArrayList<>();
numbers.add(42);              // autoboxing: int → Integer
int first = numbers.get(0);   // unboxing: Integer → int
Autoboxing Traps
Trap 1 — Null unboxing causes NullPointerException:

Integer count = null;
int n = count;    // NullPointerException! Can't unbox null.
Trap 2 — Boxing in loops is slow:

// BAD — creates a new Integer object on every iteration
Integer sum = 0;
for (int i = 0; i < 1_000_000; i++) {
    sum += i;  // unbox sum, add i, box result — every iteration!
}

// GOOD — use primitive type for accumulation
int sum = 0;
for (int i = 0; i < 1_000_000; i++) {
    sum += i;  // pure arithmetic, no boxing
}
The Identity Trap: == vs .equals()
⚠ False Friend: In Python, == compares values. In Java, == on objects compares identity (are these the exact same object in memory?), not value equality.

String c = new String("hello");
String d = new String("hello");
System.out.println(c == d);       // false — different objects in memory
System.out.println(c.equals(d));  // true  — same characters
String literals appear to work with == because Java interns them into a shared pool:

String a = "hello";
String b = "hello";
System.out.println(a == b);  // true — but only because both point to the interned literal!
Do not rely on this. Always use .equals() for string comparison.

The Integer cache trap: Java caches Integer objects for values −128 to 127, making == accidentally work for small numbers:

Integer x = 127;
Integer y = 127;
System.out.println(x == y);     // true (cached — same object)

Integer p = 128;
Integer q = 128;
System.out.println(p == q);     // false (not cached — different objects)
System.out.println(p.equals(q)); // true (always use .equals())
The golden rule:

Use == for primitives (int, double, boolean, char)
Use .equals() for everything else (objects, strings, wrapper types)
Object-Oriented Programming
Classes and Encapsulation
A Java class bundles private fields with public methods that control access. Unlike Python (where self.balance is always accessible) and C++ (where you control access at the class level), Java enforces encapsulation at compile time.

public class BankAccount {
    private String owner;    // private — only accessible within this class
    private double balance;

    public BankAccount(String owner, double initialBalance) {
        this.owner = owner;          // 'this' disambiguates field from parameter
        this.balance = initialBalance;
    }

    public void deposit(double amount) {
        if (amount > 0) {            // validation — callers can't bypass this
            balance += amount;
        }
    }

    public boolean withdraw(double amount) {
        if (amount > 0 && balance >= amount) {
            balance -= amount;
            return true;
        }
        return false;               // returns false instead of allowing overdraft
    }

    public double getBalance() { return balance; }
    public String getOwner()   { return owner; }

    // Called automatically by System.out.println(account) — like Python's __str__
    public String toString() {
        return "BankAccount[owner=" + owner + ", balance=" + balance + "]";
    }
}
Access Modifiers
Java has four access levels. The default (no keyword) is different from C++:

Object-Oriented Programming table
Modifier	Class	Package	Subclass	World
private	✓	✗	✗	✗
(none) = package-private	✓	✓	✗	✗
protected	✓	✓	✓	✗
public	✓	✓	✓	✓
⚠ False Friend from C++: In C++, the default access in a class is private. In Java, the default is package-private — accessible to any class in the same package. Always be explicit.

In UML class diagrams: - means private, + means public, # means protected, ~ means package-private.

Information Hiding
Encapsulation (using private fields) is a mechanism. Information hiding is a design principle.

A module hides its secrets — design decisions that are likely to change. When a secret is properly hidden, changing that decision modifies exactly one class. When a secret leaks, a single change cascades across many classes.

Object-Oriented Programming table
Secret to Hide	Example	Why
Data representation	int[] vs ArrayList vs database	Storage format may change
Algorithm	Bubble sort vs quicksort	Optimization may change
Business rules	Grading thresholds, capacity limits	Policy may change
Output format	CSV vs JSON vs text	Reporting needs may change
External dependency	Which API or library to call	Vendor may change
The Getter/Setter Fallacy
Fields can be private and yet still leak design decisions:

// Fully encapsulated — but leaking the "ISBN is an int" decision
class Book {
    private int isbn;
    public int getIsbn() { return isbn; }
    public void setIsbn(int isbn) { this.isbn = isbn; }
}
When the spec changes to support international ISBNs with hyphens (String), every caller of getIsbn() breaks. The module is encapsulated but hides nothing.

Better design — expose behavior, not data:

// Hides the representation; callers depend on behavior only
class GradeReport {
    private ArrayList<Integer> scores;  // hidden

    public String getLetterGrade(int score) { ... }  // hides the grading policy
    public double getAverage()             { ... }  // hides the data representation
    public String formatReport()           { ... }  // hides the output format
}
Test for information hiding: For each design decision, ask: “If this changes, how many classes must I edit?” If the answer is more than one, the secret has leaked.

Interfaces: Design by Contract
An interface defines what a class can do, without specifying how. Java’s philosophy:

Program to an interface, not an implementation.

// Defining an interface — method signatures only
public interface Shape {
    double getArea();
    double getPerimeter();
    String describe();
}

// Implementing an interface — must provide ALL methods
public class Circle implements Shape {
    private double radius;

    public Circle(double radius) { this.radius = radius; }

    public double getArea()      { return Math.PI * radius * radius; }
    public double getPerimeter() { return 2 * Math.PI * radius; }
    public String describe()     { return "Circle(r=" + radius + ")"; }
}
Declare variables as the interface type so you can swap implementations without changing calling code:

Shape s = new Circle(5.0);    // interface type on the left
Shape r = new Rectangle(3, 4);
// s and r can be used interchangeably anywhere Shape is expected
Compared to C++ and Python:

Object-Oriented Programming table
Aspect	C++	Python	Java
Mechanism	Pure virtual functions / abstract class	Duck typing (no enforcement)	interface keyword, compiler-enforced
Multiple inheritance	Yes (virtual base classes)	Yes (MRO)	A class can implement multiple interfaces
Default methods	No	No	Java 8+: default methods can have implementations
Inheritance and Polymorphism
Java supports single class inheritance with abstract classes for sharing both state and behavior:

// Abstract class — cannot be instantiated, may have concrete fields and methods
public abstract class Vehicle {
    private String make;
    private int year;

    public Vehicle(String make, int year) {  // abstract classes have constructors
        this.make = make;
        this.year = year;
    }

    public String getMake() { return make; }
    public int getYear()    { return year; }

    // Subclasses MUST implement these
    public abstract String describe();
    public abstract String startEngine();
}

public class Car extends Vehicle {
    private int numDoors;

    public Car(String make, int year, int numDoors) {
        super(make, year);  // MUST call parent constructor first — like C++ initializer lists
        this.numDoors = numDoors;
    }

    @Override  // optional but recommended — compiler verifies you're actually overriding
    public String describe() {
        return getYear() + " " + getMake() + " Car (" + numDoors + " doors)";
    }

    @Override
    public String startEngine() { return "Vroom!"; }
}
Polymorphism — a parent reference can point to any subclass:

Vehicle[] fleet = {
    new Car("Toyota", 2024, 4),
    new Motorcycle("Harley", 2023, true),
};

for (Vehicle v : fleet) {
    System.out.println(v.describe());  // calls Car.describe() or Motorcycle.describe()
    //                                    based on the actual runtime type — dynamic dispatch
}
Key differences from C++:

Java methods are virtual by default — no virtual keyword needed
@Override annotation is optional but the compiler validates it catches typos
super(args) must be the first statement in a constructor (C++ uses initializer lists)
When to use interface vs abstract class:

Object-Oriented Programming table
Aspect	Interface	Abstract Class
Methods	Abstract (+ default in Java 8+)	Abstract AND concrete
Fields	Only static final constants	Instance fields allowed
Constructor	No	Yes
Inheritance	implements (multiple OK)	extends (single only)
Use when…	Unrelated classes share behavior	Related classes share state + behavior
Generics
Generics: Not C++ Templates
Java generics look like C++ templates but work completely differently:

Generics table
Feature	C++ Templates	Java Generics
Mechanism	Code generation (monomorphization)	Type erasure (single shared implementation)
Runtime type info	Yes — vector<int> ≠ vector<string>	No — List<String> = List<Integer> at runtime
Primitive types	Yes — vector<int> works	No — must use List<Integer>
new T()	Yes	No — type is unknown at runtime
// A generic class — T is a type parameter
public class Box<T> {
    private T item;

    public Box(T item) { this.item = item; }
    public T getItem()  { return item; }
}

// The compiler ensures type safety — no casts needed
Box<String> nameBox = new Box<>("Alice");
String name = nameBox.getItem();  // compiler knows it's String

Box<Integer> numBox = new Box<>(42);
int num = numBox.getItem();       // unboxing Integer → int
Generic methods declare their own type parameters:

// <X, Y> before the return type — method's own type parameters
public static <X, Y> Pair<Y, X> swap(Pair<X, Y> pair) {
    return new Pair<>(pair.getSecond(), pair.getFirst());
}
Bounded type parameters — restrict what types are allowed:

// T must implement Comparable<T> — like C++20 concepts
public static <T extends Comparable<T>> T findMax(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}
Type Erasure
When Java 5 added generics (2004), billions of lines of pre-generics code already existed. To maintain binary compatibility, generic types are erased after compilation:

// What you write:
List<String> names = new ArrayList<>();
String first = names.get(0);

// What the compiler generates (roughly):
List names = new ArrayList();
String first = (String) names.get(0);  // cast inserted by compiler
Consequences:

ArrayList<int> is illegal — use ArrayList<Integer> instead
new T() is illegal — type is unknown at runtime
if (list instanceof List<String>) is illegal — generic type is erased
Collections Framework
Choosing the Right Collection
Java Collections are organized by interfaces. Declare variables as the interface type:

«interface»
Collection
«interface»
List
«interface»
Set
«interface»
Map
«resizable array=""»
ArrayList
«doubly-linked»
LinkedList
«unordered, fast»
HashSet
«sorted»
TreeSet
«unordered, fast»
HashMap
«sorted by="" key=""»
TreeMap
Detailed description

UML class diagram with 6 classes (ArrayList, LinkedList, HashSet, TreeSet, HashMap, TreeMap), 4 interfaces (Collection, List, Set, Map). List extends Collection. Set extends Collection. ArrayList implements List. LinkedList implements List. HashSet implements Set. TreeSet implements Set. HashMap implements Map. TreeMap implements Map.

Interfaces

Collection — Attributes: none declared — Operations: none declared
List — Attributes: none declared — Operations: none declared
Set — Attributes: none declared — Operations: none declared
Map — Attributes: none declared — Operations: none declared
Relationships

List extends Collection
Set extends Collection
ArrayList implements List
LinkedList implements List
HashSet implements Set
TreeSet implements Set
HashMap implements Map
TreeMap implements Map
Predict each output. Then explain why Line A and Line B differ — what does each operator actually check?

Show Answer
Java — Write the Code
You are given a scenario or design problem. Write Java code that solves it. Questions target Apply, Evaluate, and Create levels — not just syntax recall.

Difficulty:
Expert
Write a WordCounter class that takes a String[] in its constructor and provides:

int getCount(String word) — returns 0 for unknown words, no NPE
int getUniqueCount() — number of distinct words
Use the most appropriate collection for each responsibility.

Show Answer
Java Concepts Quiz
Test your deeper understanding of Java's type system, OOP model, and design idioms. Covers false friends with C++/Python, encapsulation vs information hiding, generics, collections, and exception handling. Includes Parsons problems, technique-selection questions, and spaced interleaving across all concepts.

Difficulty:
Advanced
What is the bug in this code?

Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
int grade = scores.get("Bob");

A
Nothing — the default value 0 is returned for missing keys


B
Compile error — get() returns Object, which cannot be assigned to int


C
NullPointerException — get() returns null and unboxing null to int throws


D
HashMap does not support String keys