Want to practice? Try the Official Python Tutorial — Run it directly on your own machine.

Welcome to Python! Since you already know C++, you have a strong foundation in programming logic, control flow, and object-oriented design. However, moving from a compiled, statically typed systems language to an interpreted, dynamically typed scripting language requires a shift in how you think about memory and execution.

To help you make this transition, we will anchor Python’s concepts directly against the C++ concepts you already know, adjusting your mental model along the way.

The Execution Model: Scripts vs. Binaries
In C++, your workflow is Write 
 Compile 
 Link 
 Execute. The compiler translates your source code directly into machine-specific instructions.

Python is a scripting language. You do not explicitly compile and link a binary. Instead, your workflow is simply Write 
 Execute.

Under the hood, when you run python script.py, the Python interpreter reads your code, translates it into an intermediate “bytecode”, and immediately runs that bytecode on the Python Virtual Machine (PVM).

What this means for you:

No main() boilerplate: Python executes from top to bottom. You don’t need a main() function to make a script run, though it is often used for organization.
Rapid Prototyping: Because there is no compilation step, you can write and test code iteratively and quickly.
Runtime Errors: In C++, the compiler catches syntax and type errors before the program ever runs. In Python, syntax and indentation errors are caught at parse time before any code executes, but most other errors (e.g., TypeError, NameError, AttributeError) are caught at runtime only when the interpreter actually reaches the problematic line.
C++:

#include <iostream>
int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
Python:

print("Hello, World!")
The Mental Model of Memory: Dynamic Typing
This is the largest paradigm shift you will make.

In C++ (Statically Typed), a variable is a box in memory. When you declare int x = 5;, the compiler reserves 4 bytes of memory, labels that specific memory address x, and restricts it to only hold integers.

In Python (Dynamically Typed), a variable is a name tag attached to an object. The object has a type, but the variable name does not.

You can inspect the type of any object at runtime using the built-in type() function:

x = 42
print(type(x))        # <class 'int'>

x = "hello"
print(type(x))        # <class 'str'>

x = 3.14
print(type(x))        # <class 'float'>
This is useful for debugging, but note that checking types explicitly is often un-Pythonic — prefer Duck Typing (see below) for production code.

Let’s look at an example:

x = 1_000_000  # Python creates an integer object '1000000'. It attaches the name tag 'x' to it.
print(x)      

x = "Hello"   # Python creates a string object '"Hello"'. It moves the 'x' tag to the string.
print(x)      # The integer '1000000' is now nameless and will be garbage collected.
Note: CPython caches small integers (roughly -5 through 256) in a permanent pool, so they are not eligible for garbage collection even when no user variable references them. We deliberately use 1_000_000 above to illustrate the general principle.

Because variables are just name tags (references) pointing to objects, you don’t declare types. The Python interpreter figures out the type of the object at runtime.

Syntax and Scoping: Whitespace Matters
In C++, scope is defined by curly braces {} and statements are terminated by semicolons ;.

Python uses indentation to define scope, and newlines to terminate statements. This enforces highly readable code by design. PEP 8 recommends 4 spaces per level — never mix tabs and spaces, as this raises a TabError (a kind of IndentationError) when Python parses the file (before any code runs) that can be hard to diagnose (tabs and spaces look identical in many editors).

C++:

for (int i = 0; i < 5; i++) {
    if (i % 2 == 0) {
        std::cout << i << " is even\n";
    }
}
Python:

for i in range(5):
    if i % 2 == 0:
        print(f"{i} is even") # Notice the 'f' string, Python's modern way to format strings
The range() function generates a sequence of integers and has three forms:

range(stop) — from 0 up to (but not including) stop: range(5) → 0, 1, 2, 3, 4
range(start, stop) — from start up to (not including) stop: range(2, 6) → 2, 3, 4, 5
range(start, stop, step) — with a custom stride: range(0, 10, 2) → 0, 2, 4, 6, 8; range(5, 0, -1) → 5, 4, 3, 2, 1
⚠️ Scoping: The LEGB Rule (A “False Friend” from C++)
In C++, a variable declared inside a for or if block is scoped to that block. In Python, variables created inside a loop or if block are visible in the enclosing function scope — there are no block-level scopes. This is one of the most common “false friend” traps for C++ programmers.

for i in range(5):
    last = i

print(last)  # 4 — 'last' and 'i' are STILL accessible here!
# In C++, this would be a compile error: 'last' was declared inside the for block
Python resolves variable names using the LEGB rule — it searches scopes in this order:

Local — inside the current function
Enclosing — inside enclosing functions (for nested functions/closures)
Global — module-level
Built-in — Python’s built-in names (print, len, etc.)
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)    # "local" — L wins

    inner()
    print(x)        # "enclosing" — E level

outer()
print(x)            # "global" — G level
Key difference from C++: If you want to modify a variable from an enclosing scope, you must use the nonlocal (for enclosing functions) or global keyword. Without it, Python creates a new local variable instead of modifying the outer one.

Defining Functions with def
Python functions are defined with the def keyword. Unlike C++, there is no return type declaration — the function just returns whatever the return statement provides, or None implicitly if there is no return.

# Basic function — no type declarations needed
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))   # Hello, Alice!
Default Parameters: Parameters can have default values, making them optional at the call site:

def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

print(greet("Alice"))            # Hello, Alice!
print(greet("Bob", "Hi"))        # Hi, Bob!
Implicit None Return: A function with no return statement (or a bare return) returns None, Python’s equivalent of void:

def log_message(msg):
    print(msg)
    # No return — implicitly returns None

result = log_message("test")
print(result)   # None
Docstrings: The Python convention for documenting functions is a triple-quoted string immediately after the def line. Tools and IDEs display this as help text:

def calculate_area(width, height):
    """Return the area of a rectangle given its width and height."""
    return width * height
Type Hints (optional): Python 3.5+ supports optional type annotations. They are not enforced at runtime but improve readability and enable static analysis tools:

def add(x: int, y: int) -> int:
    return x + y
Passing Arguments: “Pass-by-Object-Reference”
In C++, you explicitly choose whether to pass variables by value (int x), by reference (int& x), or by pointer (int* x).

How does Python handle this? Because everything in Python is an object, and variables are just “name tags” pointing to those objects, Python uses a model often called “Pass-by-Object-Reference”.

When you pass a variable to a function, you are passing the name tag.

If the object the tag points to is Mutable (like a List or a Dictionary), changes made inside the function will affect the original object.
If the object the tag points to is Immutable (like an Integer, String, or Tuple), any attempt to change it inside the function simply creates a new object and moves the local name tag to it, leaving the original object unharmed.
# Modifying a Mutable object (similar to passing by reference/pointer in C++)
def modify_list(my_list):
    my_list.append(4) # Modifies the actual object in memory

nums = [1, 2, 3]
modify_list(nums)
print(nums) # Output: [1, 2, 3, 4]

# Modifying an Immutable object (behaves similarly to pass by value)
def attempt_to_modify_int(my_int):
    my_int += 10 # Creates a NEW integer object, moves the local 'my_int' tag to it

val = 5
attempt_to_modify_int(val)
print(val) # Output: 5. The original object is unchanged.
String Formatting: The Magic of f-strings
In C++, building a complex string with variables traditionally requires chaining << operators with std::cout, using sprintf, or utilizing the modern std::format. This can get verbose quickly.

Python revolutionized string formatting in version 3.6 with the introduction of f-strings (formatted string literals). By simply prefixing a string with the letter f (or F), you can embed variables and even evaluate expressions directly inside curly braces {}.

C++:

std::string name = "Alice";
int age = 30;
std::cout << name << " is " << age << " years old and will be " 
          << (age + 1) << " next year.\n";
Python:

name = "Alice"
age = 30

# The f-string automatically converts variables to strings and evaluates the math
print(f"{name} is {age} years old and will be {age + 1} next year.")
Pedagogical Note: Under the hood, Python calls the object’s __format__() method (passing the format spec, if any). For most built-in types __format__() delegates to __str__(), so the two appear interchangeable — but a custom class can override __format__() to support format specifiers like f"{value:>10}".

String Quotes: "..." and '...' Are Interchangeable
In C++, single quotes and double quotes mean completely different things: 'A' is a char, while "Alice" is a const char* (or std::string). Mixing them up is a compile error.

In Python, there is no char type — single quotes and double quotes both create str objects and are fully interchangeable:

name = "Alice"    # str
name = 'Alice'    # also str — identical result
This is especially handy when your string itself contains quotes, because you can pick whichever style avoids escaping:

msg = "It's easy"          # double quotes avoid escaping the apostrophe
html = '<div class="box">' # single quotes avoid escaping the double quotes
In C++ you would need to escape: "It\'s easy" or "<div class=\"box\">". Python lets you sidestep the backslashes entirely by choosing the other quote style.

Convention: PEP 8 accepts either style but recommends picking one and being consistent throughout a project. Both are equally common in the wild.

Common String Methods
Python strings come with a rich set of built-in methods (no #include required). Unlike C++ where std::string methods are relatively few, Python strings behave more like a full text-processing library:

text = "  Hello, World!  "

# Case conversion
print(text.upper())        # "  HELLO, WORLD!  "
print(text.lower())        # "  hello, world!  "

# Whitespace removal
print(text.strip())        # "Hello, World!"  (both ends)
print(text.lstrip())       # "Hello, World!  " (left end only)
print(text.rstrip())       # "  Hello, World!" (right end only)

# Splitting — returns a list of substrings
csv_line = "Alice,90,B+"
fields = csv_line.split(",")      # ['Alice', '90', 'B+']

log = "error: disk full\nwarning: low memory\n"
lines = log.splitlines()          # ['error: disk full', 'warning: low memory']

# Splitting on whitespace (default) collapses multiple spaces:
words = "  hello   world  ".split()   # ['hello', 'world']

# Checking content
print("hello".startswith("he"))   # True
print("hello".endswith("lo"))     # True
print("ell" in "hello")           # True

# Replacement
print("foo bar foo".replace("foo", "baz"))  # "baz bar baz"
strip() is especially important when reading files — lines from a file end with \n, so stripping removes the trailing newline before processing.

Core Collections: Lists, Sets, and Dictionaries
Because Python does not enforce static typing, its built-in collections are highly flexible. You do not need to #include external libraries to use them; they are native to the language syntax.

Lists (C++ Equivalent: std::vector)
A List is an ordered, mutable sequence of elements. Unlike a C++ std::vector<T>, a Python list can contain objects of entirely different types. Lists are defined using square brackets [].

# Heterogeneous list
my_list = [1, "two", 3.14, True]

my_list.append("new item") # Adds to the end (like push_back)
my_list.pop()              # Removes and returns the last item

# Other common operations
my_list.remove("two")      # Removes the first occurrence of "two" (like std::remove + erase)
my_list.clear()            # Empties the entire list (like std::vector::clear)

print(len(my_list))        # len() gets the size of any collection (Output: 0)
Sets (C++ Equivalent: std::unordered_set)
A Set is an unordered collection of unique elements. It is implemented using a hash table, making membership testing (in) exceptionally fast—
 on average. Sets are defined using curly braces {}, or by passing any iterable to the set() constructor.

unique_numbers = {1, 2, 2, 3, 4, 4}
print(unique_numbers) # Output: {1, 2, 3, 4} - duplicates are automatically removed

# Fast membership testing
if 3 in unique_numbers:
    print("3 is present!")

# Deduplication idiom — convert a list to a set and back:
words = ["apple", "banana", "apple", "cherry", "banana"]
unique_words = list(set(words))  # removes duplicates (order not preserved)

# Count unique items:
ip_list = ["10.0.0.1", "10.0.0.2", "10.0.0.1"]
print(len(set(ip_list)))  # 2 — number of distinct IP addresses
Dictionaries (C++ Equivalent: std::unordered_map)
A Dictionary (or “dict”) is a mutable collection of key-value pairs. Like Sets, they are backed by hash tables for incredibly fast 
 lookups. Dicts are defined using curly braces {} with a colon : separating keys and values.

player_scores = {"Alice": 50, "Bob": 75}

# Accessing and modifying values
player_scores["Alice"] += 10 
player_scores["Charlie"] = 90 # Adding a new key-value pair

print(f"Bob's score is {player_scores['Bob']}")
“Pythonic” Iteration
While C++ traditionally relies on index-based for loops (though modern C++ has range-based loops), Python strongly encourages iterating directly over the elements of a collection. This is considered writing “Pythonic” code.

C++ (Index-based iteration):

std::vector<std::string> fruits = {"apple", "banana", "cherry"};
for (size_t i = 0; i < fruits.size(); i++) {
    std::cout << fruits[i] << std::endl;
}
Python (Pythonic Iteration):

fruits = ["apple", "banana", "cherry"]

# Do not do: for i in range(len(fruits)): ...
# Instead, iterate directly over the object:
for fruit in fruits:
    print(fruit)

# Iterating over dictionary key-value pairs:
student_grades = {"Alice": 95, "Bob": 82}

for name, grade in student_grades.items():
    print(f"{name} scored {grade}")
Memory Management: RAII vs. Garbage Collection
In C++, you are the absolute master of memory. You allocate it (new), you free it (delete), or you utilize RAII (Resource Acquisition Is Initialization) and smart pointers to tie memory management to variable scope. If you make a mistake, you get a memory leak or a segmentation fault.

In Python, memory management is entirely abstracted away. You do not allocate or free memory. Instead, Python primarily uses Reference Counting backed by a Garbage Collector.

Every object in Python keeps a running tally of how many “name tags” (variables or references) are pointing to it. When a variable goes out of scope, or is reassigned to a different object, the reference count of the original object decreases by one. When that count hits zero, Python immediately reclaims the memory.

C++ (Manual / RAII):

void createArray() {
    // Dynamically allocated, must be managed
    int* arr = new int[100]; 
    // ... do something ...
    delete[] arr; // Forget this and you leak memory!
}
Python (Automatic):

def create_list():
    # Creates a list object in memory and attaches the 'arr' tag
    arr = [0] * 100 
    # ... do something ...
    
    # When the function ends, 'arr' goes out of scope. 
    # The list object's reference count drops to 0, and memory is freed automatically.
Object-Oriented Programming: Explicit self and “Duck Typing”
If you are used to C++ classes, Python’s approach to OOP will feel radically open and simplified.

No Header Files: Everything is declared and defined in one place.
Explicit self: In C++, instance methods have an implicit this pointer. In Python, the instance reference is passed explicitly as the first parameter to every instance method. By convention, it is always named self.
No True Privacy: C++ enforces public, private, and protected access specifiers at compile time. Python operates on the philosophy of “we are all consenting adults here”. There are no true private variables. Instead, developers use a convention: prefixing a variable with a single underscore (e.g., _internal_state) signals to other developers, “This is meant for internal use, please don’t touch it”, but the language will not stop them from accessing it.
Duck Typing: In C++, if a function expects a Bird object, you must pass an object that inherits from Bird. Python relies on “Duck Typing”—If it walks like a duck and quacks like a duck, it must be a duck. Python doesn’t care about the object’s actual class hierarchy; it only cares if the object implements the methods being called on it.
C++:

class Rectangle {
private:
    int width, height; // Enforced privacy
public:
    Rectangle(int w, int h) : width(w), height(h) {} // Constructor
    
    int getArea() {
        return width * height; // 'this->' is implicit
    }
};
Python:

class Rectangle:
    # __init__ is Python's constructor. 
    # Notice 'self' must be explicitly declared in the parameters.
    def __init__(self, width, height):
        self._width = width   # The underscore is a convention meaning "private"
        self._height = height # but it is not strictly enforced by the interpreter.

    def get_area(self):
        # You must explicitly use 'self' to access instance variables
        return self._width * self._height

# Instantiating the object (Note: no 'new' keyword in Python)
my_rect = Rectangle(10, 5)
print(my_rect.get_area())
Dunder Methods: __str__ vs. operator<<
In the OOP section, we covered the __init__ constructor method. Python uses several of these “dunder” (double underscore) methods to implement core language behavior.

In C++, if you want to print an object using std::cout, you have to overload the << operator. In Python, you simply implement the __str__(self) method. This method returns a “user-friendly” string representation of the object, which is automatically called whenever you use print() or an f-string.

Python:

class Book:
    def __init__(self, title, author, year):
        self.title = title
        self.author = author
        self.year = year
        
    def __str__(self):
        # This is what print() will call
        return f'"{self.title}" by {self.author} ({self.year})'

my_book = Book("Pride and Prejudice", "Jane Austen", 1813)
print(my_book) # Output: "Pride and Prejudice" by Jane Austen (1813)
Substring Operations and Slicing
In C++, if you want a substring, you call my_string.substr(start_index, length). Python takes a much more elegant and generalized approach called Slicing.

Slicing works not just on strings, but on any ordered sequence (like Lists and Tuples). The syntax uses square brackets with colons: sequence[start:stop:step].

start: The index where the slice begins (inclusive).
stop: The index where the slice ends (exclusive).
step: The stride between elements (optional, defaults to 1).
Negative Indexing: This is a crucial Python paradigm. While index 0 is the first element, index -1 is the last element, -2 is the second-to-last, and so on.

text = "Software Engineering"

# Basic slicing
print(text[0:8])    # Output: 'Software' (Indices 0 through 7)

# Omitting start or stop
print(text[:8])     # Output: 'Software' (Defaults to the very beginning)
print(text[9:])     # Output: 'Engineering' (Defaults to the very end)

# Negative indexing
print(text[-11:])   # Output: 'Engineering' (Starts 11 characters from the end)
print(text[-1])     # Output: 'g' (The last character)

# Using the step parameter
print(text[0:8:2])  # Output: 'Sfwr' (Every 2nd character of 'Software')

# The ultimate Pythonic trick: Reversing a sequence
print(text[::-1])   # Output: 'gnireenignE erawtfoS' (Steps backwards by 1)
Because variables in Python are references to objects, it is important to note that slicing a list always creates a shallow copy—a brand new list object containing references to the sliced elements. Slicing a string normally also returns a new string, but because strings are immutable, CPython is allowed to optimize the whole-string slice s[:] to return the same object — that’s a harmless implementation detail, not something to rely on.

Tuple Unpacking and Variable Swapping
The lecture introduces the concept of Syntactic Sugar—language features that don’t add new functional capabilities but make programming significantly easier and more readable.

A prime example is unpacking. In C++, swapping two variables requires a temporary third variable (or utilizing std::swap). Python handles this natively with multiple assignment.

C++:

int temp = a;
a = b;
b = temp;
Python:

a, b = b, a # Syntactic sugar that swaps the values instantly
Exception Handling: try / except
While we discussed that Python catches errors at runtime, the Week 2 materials highlight how to handle these errors gracefully using try and except blocks (Python’s equivalent to C++’s try and catch).

In C++, exceptions are often reserved for critical failures, but in Python, using exceptions for control flow (like catching a ValueError when a user inputs a string instead of an integer) is standard practice.

try:
    guess = int(input("> "))
except ValueError:
    print("Invalid input, please enter a number.")
EAFP vs. LBYL: A Python Philosophy Shift
In C++, the standard approach is LBYL — “Look Before You Leap”: check preconditions before performing an operation (e.g., check if a key exists before accessing it). Python encourages the opposite: EAFP — “Easier to Ask Forgiveness than Permission”: just try the operation and handle the exception if it fails.

# C++ instinct (LBYL — Look Before You Leap):
if "key" in my_dict:
    value = my_dict["key"]
else:
    value = "default"

# Pythonic (EAFP — Easier to Ask Forgiveness than Permission):
try:
    value = my_dict["key"]
except KeyError:
    value = "default"

# Even more Pythonic — dict.get() with a default:
value = my_dict.get("key", "default")
EAFP is idiomatic Python by convention. Setting up a try/except block in CPython 3.11+ has essentially zero cost on the no-exception path, so using try/except for expected cases like missing dictionary keys or file-not-found is standard practice, not an anti-pattern. (Modern C++ also uses zero-cost exception handling, so the contrast you may have heard between “cheap Python exceptions” and “expensive C++ exceptions” is mostly a cultural difference, not a performance one.)

Common Built-in Exception Types
Knowing the standard exception types makes it easier to write targeted except clauses and understand error messages:

Python table
Exception	When it occurs
SyntaxError	Code that cannot be parsed — caught before execution
IndentationError	Inconsistent indentation (e.g., mixed tabs and spaces)
TypeError	Operation on incompatible types (e.g., "5" + 3)
ValueError	Right type but inappropriate value (e.g., int("hello"))
IndexError	Sequence index out of range (e.g., my_list[99] on a short list)
KeyError	Dictionary key does not exist (e.g., d["missing"])
FileNotFoundError	open() called on a path that does not exist
ZeroDivisionError	Division or modulo by zero
AttributeError	Accessing a non-existent attribute on an object
Robust Command-Line Arguments (argparse)
In C++, you typically handle command-line inputs by parsing int argc and char* argv[] directly in main(). While Python does have a direct equivalent (sys.argv), the course materials emphasize using the built-in argparse module. It automatically generates help/usage messages, enforces types, and parses flags, saving you from writing boilerplate C++ parsing code.

Division Operators: / vs //
A common negative-transfer trap from C++: in C++, 7 / 2 gives 3 (integer division when both operands are ints). In Python 3, / always returns a float:

7 / 2     # 3.5  (float division — different from C++!)
7 // 2    # 3    (integer/floor division — like C++'s /)
7 % 2     # 1    (modulo — same as C++)
Use // when you explicitly want integer division. Use / when you want precise results.

The ** Exponentiation Operator
Python uses ** for exponentiation. In C++ you would use pow() or std::pow(). Be careful: ^ is bitwise XOR in Python, not exponentiation:

2 ** 8    # 256  ✓  (exponentiation)
9 ** 0.5  # 3.0  ✓  (square root)
2 ^ 8     # 10   ✗  (bitwise XOR — NOT exponentiation!)
Dynamic ≠ Weak: Python’s Strong Typing
Python is dynamically typed (you don’t declare types) but also strongly typed (it won’t silently convert between incompatible types). This is different from JavaScript, which is dynamically typed AND weakly typed:

x = "5" + 3    # TypeError: can only concatenate str to str
Unlike JavaScript (which would give "53"), Python refuses to guess. You must be explicit: int("5") + 3 → 8 or "5" + str(3) → "53".

enumerate() — Index and Value Together
In C++ you use index-based loops to get both the position and the value. Python’s enumerate() provides this more elegantly:

fruits = ["apple", "banana", "cherry"]

# Instead of: for i in range(len(fruits)): ...
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
List Comprehensions
List comprehensions are a compact, idiomatic way to build lists in Python — a pattern you will see everywhere in Python code:

# C++ equivalent:
# std::vector<int> squares;
# for (int i = 1; i <= 5; i++) squares.push_back(i * i);

# Python: one line
squares = [x**2 for x in range(1, 6)]          # [1, 4, 9, 16, 25]

# With a filter condition:
evens = [x for x in range(10) if x % 2 == 0]   # [0, 2, 4, 6, 8]
The general form is [expression for variable in iterable if condition]. Use comprehensions when the transformation is simple — they are more readable and slightly faster than equivalent for loops.

Generator Expressions: Lazy Comprehensions
Replacing the square brackets [...] with parentheses (...) creates a generator expression — it produces values one at a time (lazy evaluation) instead of building the entire list in memory:

# List comprehension — builds a full list in memory:
squares = [x**2 for x in range(1_000_000)]      # ~8 MB in memory

# Generator expression — produces values on demand:
squares = (x**2 for x in range(1_000_000))       # near-zero memory
Use generators when you only need to iterate once and don’t need to store the full collection — for example, passing directly to sum(), max(), or a for loop.

Reading Files with open() and with
In C++ you fopen, check for NULL, process, and fclose. Python’s with statement handles the close automatically — even if an exception occurs:

# C++: FILE *f = fopen("data.txt", "r"); ... fclose(f);

# Python — the 'with' block closes the file automatically:
with open("data.txt") as f:
    for line in f:
        print(line.strip())   # .strip() removes the trailing newline
There are several ways to read a file’s content depending on your needs:

with open("data.txt") as f:
    content = f.read()              # Entire file as one string
    lines = content.splitlines()    # Split into a list of lines (no trailing \n)

with open("data.txt") as f:
    lines = f.readlines()           # List of lines, each ending with \n

with open("data.txt") as f:
    for line in f:                  # Memory-efficient: one line at a time
        process(line.strip())
Prefer iterating line-by-line for large files — f.read() loads the entire file into memory at once, which can be problematic for gigabyte-scale logs.

The with statement is Python’s context manager idiom — just like RAII in C++, the file is guaranteed to be closed when the block exits. This also works with database connections, locks, and other resources.

Command-Line Arguments with sys.argv and sys.stderr
C++’s argc/argv maps directly to Python’s sys.argv:

import sys

# sys.argv[0] is the script name (like argv[0] in C++)
# sys.argv[1], [2], ... are the arguments

if len(sys.argv) < 2:
    print("Error: no filename given", file=sys.stderr)  # stderr, like std::cerr
    sys.exit(1)                                          # exit code 1, like exit(1)

filename = sys.argv[1]
print() writes to stdout by default. Use file=sys.stderr to send error messages to stderr, keeping output and diagnostics separate — the same reason C++ separates std::cout from std::cerr.

Regular Expressions (re module)
Since Python is a scripting language, it is heavily utilized for text processing. Python’s built-in re module provides the same power as grep and sed inside a script:

import re

text = "Error 404: page not found. Error 500: server crash."

# re.search() — find the FIRST match (like grep -q)
m = re.search(r'Error \d+', text)
if m:
    print(m.group())     # "Error 404"

# re.findall() — find ALL matches (like grep -o)
codes = re.findall(r'\d+', text)   # ['404', '500']

# re.sub() — replace matches (like sed 's/old/new/g')
clean = re.sub(r'Error \d+', 'ERR', text)
# "ERR: page not found. ERR: server crash."
Always use raw strings (r'...') for regex patterns — they prevent Python from interpreting backslashes before the re module sees them.

Top 10 Python Best Practices
These are the most important conventions and idioms that experienced Python programmers follow. Internalizing them will make your code more readable, less error-prone, and immediately recognizable as “Pythonic”.

1. Use f-Strings for String Formatting
F-strings (Python 3.6+) are the preferred way to embed values in strings. They are faster, more readable, and more concise than older approaches.

name = "Alice"
score = 95.678

# ✓ Pythonic: f-string
print(f"{name} scored {score:.1f}")

# ✗ Avoid: concatenation (verbose, error-prone with types)
print(name + " scored " + str(round(score, 1)))

# ✗ Avoid: %-formatting (old Python 2 style)
print("%s scored %.1f" % (name, score))
2. Use with for Resource Management
The with statement guarantees cleanup (closing files, releasing locks) even if an exception occurs — just like RAII in C++.

# ✓ Pythonic: guaranteed close
with open("data.txt") as f:
    content = f.read()

# ✗ Avoid: manual close (leaks on exception)
f = open("data.txt")
content = f.read()
f.close()
3. Iterate Directly Over Collections
Python’s for loop iterates over items, not indices. Never use range(len(...)) when you only need the elements.

fruits = ["apple", "banana", "cherry"]

# ✓ Pythonic: iterate directly
for fruit in fruits:
    print(fruit)

# ✗ Avoid: C-style index loop
for i in range(len(fruits)):
    print(fruits[i])
4. Use enumerate() When You Need the Index
When you need both the index and the value, enumerate() is the Pythonic solution.

# ✓ Pythonic: enumerate
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# ✗ Avoid: manual counter
i = 0
for fruit in fruits:
    print(f"{i}: {fruit}")
    i += 1
5. Follow PEP 8 Naming Conventions
Consistent naming makes Python code instantly readable across any project.

Python table
Entity	Convention	Example
Variables, functions	snake_case	total_count, get_area()
Classes	PascalCase	HttpResponse, Rectangle
Constants	UPPER_SNAKE_CASE	MAX_RETRIES, DEFAULT_PORT
“Private” attributes	Leading underscore	_internal_state
6. Use List Comprehensions for Simple Transformations
List comprehensions are more concise and slightly faster than equivalent for + append loops. Use them when the logic is simple and fits on one line.

# ✓ Pythonic: list comprehension
squares = [x**2 for x in range(10)]
evens = [x for x in numbers if x % 2 == 0]

# ✗ Avoid for simple cases: explicit loop
squares = []
for x in range(10):
    squares.append(x**2)
When to stop: If the comprehension needs nested loops or complex logic, use a regular for loop instead — readability always wins.

7. Catch Specific Exceptions
Never use bare except: or except Exception:. Catching too broadly hides real bugs and makes debugging much harder.

# ✓ Pythonic: specific exception
try:
    value = int(user_input)
except ValueError:
    print("Please enter a valid integer")

# ✗ Avoid: bare except (catches everything, including KeyboardInterrupt)
try:
    value = int(user_input)
except:
    print("Something went wrong")
8. Use None as a Sentinel for Mutable Default Arguments
Mutable default arguments (lists, dicts) are shared across all calls — one of Python’s most common pitfalls.

# ✓ Correct: None sentinel
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# ✗ Bug: mutable default is shared across calls
def add_item(item, items=[]):
    items.append(item)    # Second call sees items from the first call!
    return items
9. Use Truthiness for Empty Collection Checks
Empty collections ([], {}, "", set()) are falsy in Python. Use this directly instead of checking length.

my_list = []

# ✓ Pythonic: truthiness
if not my_list:
    print("list is empty")

if my_list:
    print("list has items")

# ✗ Avoid: explicit length check
if len(my_list) == 0:
    print("list is empty")
Exception: Use explicit is not None checks when 0, "", or False are valid values that should not be treated as “empty”.

10. Use is for None Comparisons
None is a singleton object in Python. Always compare with is / is not, never ==.

result = some_function()

# ✓ Pythonic: identity check
if result is None:
    print("no result")

if result is not None:
    process(result)

# ✗ Avoid: equality check (can be overridden by __eq__)
if result == None:
    print("no result")
This matters because a class can override __eq__ to return True when compared with None, which would break the equality check. The is operator checks identity (same object in memory), which cannot be overridden.

Practice
Python Syntax — What Does This Code Do?
You are shown Python code. Explain what it does and what it returns or prints.

Difficulty:
Intermediate
You are shown Python code. Explain what it does and what it returns or prints.

2 ** 8
2 ^ 8
Show Answer
Python Syntax — Write the Code
You are given a task description. Write the Python code that accomplishes it.

Difficulty:
Advanced
Write a script that prints an error to stderr and exits with code 1 if no command-line argument is provided.

Show Answer
Python Concepts Quiz
Test your deeper understanding of Python's design choices, paradigm differences from C++, and when to use which tool.

Difficulty:
Intermediate
A student uses re.findall(r'ERROR', text) to count errors in a log. Their teammate suggests text.count('ERROR') instead. When is re.findall() the better choice?


A
re.findall() is always better because it’s faster for string searching


B
text.count() only works on single characters, not multi-character strings


C
They are always interchangeable — use whichever you prefer


D
When the target is a pattern (timestamps, IPs, variable text), not a fixed literal.

---

# Appendix: Lecture 3 Slides (raw extracted text)

The following is the full extracted text of Lecture 3 (L3-PythonScripting.pdf). Preserved verbatim so no information is lost. Note: the opening section on INVEST user stories + Adversarial Thinking is process material — also covered in people-and-processes.md.

```text
CS 35L Software
Construction
L3 � Python
Scripting
Assistant Teaching Professor
Computer Science Department
Study Tips for Homework and Project
� Do the tutorials we recommend in the homework and slides
� Read man pages for all commands that you are using
   � If they are ambiguous:
       1. Find the Answer Online (Should be your first choice)
       2. Go test it yourself (works particularly well with less popular code or
           uncommon questions)
� Find a study partner or study group to discuss topics of this course. Learning
  together is more fun, and you get exposed to different ideas
   � Make sure all submitted work is your own!
Recap
User Stories
CS 35 L
SOFTWARE CONSTRUCTION
                                                                                                                          3
What is the INVEST Principle?
� Independent: Should be self-contained in a way that allows a user story to be
  developed & released in any order. (If dependencies cannot be avoided, they
  need to be referenced explicitly, e.g., "depends on user story 4")
� Negotiable: Captures the essence of users' needs. The user story only captures
  requirements without making design decisions.
� Valuable: Delivers value to the user. The feature is useful on its own.
� Estimable: Developers should be able to estimate the effort it takes to implement
  the user story. The scope of the user story is well-defined and clear.
� Small: A user story is a small chunk of work. A great user story cannot be split
  into smaller user stories that still satisfy other INVEST criteria.
� Testable: A user story has to be confirmed via objective acceptance criteria.
Sidebar: Adversarial Thinking
� Look at your ideas from the perspective of a critic to identify weaknesses and
  violations of common design principles
   � Putting yourself in the shoes of a critic
   � Searching for vulnerabilities or weaknesses.
   � Actively challenging the assumptions of your ideas
� Apply adversarial thinking to the INVEST principle:
   � For each user story, try to find reasons why they might violate the INVEST
     principle
   � If you find weaknesses, improve your user stories
   � If even when trying very hard, you cannot find weaknesses, write down the
     reasons that convince the inner critic in yourself
Use Adversarial Thinking to Check Whether your
User Stories Satisfy the INVEST principle
� Independent: Should be self-contained in a way that allows a user story to be
  developed & released in any order.
  Try to find dependencies between your user stories.
  Does this user story A assume that some portion of user story B is
  already implemented?
  If yes, try to re-write your user stories
  If you cannot re-write them to be independent, make the
  dependency explicit!
Use Adversarial Thinking to Check Whether your
User Stories Satisfy the INVEST principle
� Negotiable: Captures the essence of users' needs. The user story only captures
  requirements without making design decisions.
  Try to find elements in your user story that are design decisions
  or that over-constraint the solution space (e.g., algorithms,
  technology, or other ideas on how to implement a feature)
  If they are not really required by a user, remove them
Use Adversarial Thinking to Check Whether your
User Stories Satisfy the INVEST principle
� Valuable: Delivers value to the user. The feature is useful on its own.
  Does the user story add one increment to the software provide
  value on its own?
  Are the acceptance criteria missing something important? (e.g.,
  only entering data without any means to retrieve data might be
  considered not valuable in many cases)
Use Adversarial Thinking to Check Whether your
User Stories Satisfy the INVEST principle
� Estimable: Developers should be able to estimate the effort it takes to implement
  the user story
  Try to find unintentionally vague wording that makes it ambiguous
  how much effort it would be to implement the user story (e.g., just
  saying the system should be support "some foreign languages" is
  not estimable since it is unclear how many and to what degree
  they should be supported)
Use Adversarial Thinking to Check Whether your
User Stories Satisfy the INVEST principle
� Small: A user story is a small chunk of work. A great user story cannot be split
  into smaller user stories that still satisfy other INVEST criteria.
  Try to break up your user story into smaller ones that are still
  "valuable" and "independent". Repeat until you reached a point
  where smaller user stories are not "valuable" and "independent"
  anymore.
Use Adversarial Thinking to Check Whether your
User Stories Satisfy the INVEST principle
� Testable: A user story has to be confirmed via objective acceptance criteria.
  Try to think ambiguities in your acceptance criteria. Is there any
  subjective or vague language that could be made more precise
  while still staying "negotiable"?
Recap
Regular
Expressions
CS 35 L
SOFTWARE CONSTRUCTION
                                                                                                                        12
In-Class Exercise
Write a regular expression that can be used to validate a
password that must be at least 8 characters long and can
contain only letters and digits
^[a-zA-Z0-9]{8, }$
RegEx Reference: https://tobiasduerschmid.github.io/SEBook/tools/regex.html
In-Class Exercise
Write a regular expression that can be used to find all phone
numbers in varying formats:
� 123-456-7890
� (123) 456-7890
� 123.456.7890
� 1234567890
\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}
RegEx Reference: https://tobiasduerschmid.github.io/SEBook/tools/regex.html
In-Class Exercise
Write a regular expression that can be used to validate an
email address                         Dots outside of character
                                      classes need to be escaped
^[a-zA-Z0-9.-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
username can       domain can         top level
include letters,   include letters,   domain
digits, dots, and  digits, dots, and  can include
dashes.            dashes             letters
(e.g, cs-35L)      (e.g., ucla2025)
RegEx Reference: https://tobiasduerschmid.github.io/SEBook/tools/regex.html
Write a Regular Expression to Find All Users
<h1>Bulk Download for Reviewers</h1>
As a reviewer (or administrator), I want to be able to download all submitted
files for a specific application in a single compressed folder (ZIP) so that I
can review the materials efficiently offline.
       Given I am logged in as a registered reviewer and viewing the details of a
specific submission,
When I click the "Download All Submission Files" button,
       Then The system should generate and prompt me to download a single .zip
file containing every file uploaded by the applicant for that submission.
<h1>Applicant Registration</h1>
As an applicant, I want to be able to register and create a profile so that I
can easily track my submissions and receive official communications.
Given I am a first-time user and I am on the system's registration page,
When I fill in all required fields and click the "Register" button,
       Then My acCcSou35nLtSosfhtwoaurledCobnestruccrteioant: eLdec,turaen2d� IFilessh&ouSlhdell rSecrcipetiinvge an email  16
confirmation wTiotbihas aD�rlsichnmkidto my new profile dashboard.
Write a Regular Expression to Find All Users
Reg ex: As a .+, <h1>Bulk Download for Reviewers</h1>
As a reviewer (or administrator), I want to be able to download all submitted
files for a specific application in a single compressed folder (ZIP) so that I
can review the materials efficiently offline.
       Given I am logged in as a registered reviewer and viewing the details of a
specific submission,
       When I click the "Download All Submission Files" button,
   We don't want this, so we need an Then The system should generate and prompt me to download a single .zip
anchor so that we only match file containing every file uploaded by the applicant for that submission.
<h1>Applicant Registration</h1>
As an appliceanxtp, rIewsanstiotonbse atbhlae ttosrteagristttehr eandlicnreeatwe aithpro"fAilse a" so that I
can easily track my submissions and receive official communications.
Given I am a first-time user and I am on the system's registration page,
When I fill in all required fields and click the "Register" button,
       Then My acCcSou35nLtSosfhtwoaurledCobnestruccrteioant: eLdec,turaen2d� IFilessh&ouSlhdell rSecrcipetiinvge an email  17
confirmation wTiotbihas aD�rlsichnmkidto my new profile dashboard.
Write a Regular Expression to Find All Users
Reg ex: ^As a .+, <h1>Bulk Download for Reviewers</h1>
As a reviewer (or administrator), I want to be able to download all submitted
files for a specific application in a single compressed folder (ZIP) so that I
can review the materials efficiently offline.
       Given I am logged in as a registered reviewer and viewing the details of a
specific submission,
When I click the "Download All Submission Files" button,
  We miss users that start with a vowel Then The system should generate and prompt me to download a single .zip
file containing every file uploaded by the applicant for that submission.
<h1>Applicant Registration</h1>
As an applicant, I want to be able to register and create a profile so that I
can easily track my submissions and receive official communications.
Given I am a first-time user and I am on the system's registration page,
When I fill in all required fields and click the "Register" button,
       Then My acCcSou35nLtSosfhtwoaurledCobnestruccrteioant: eLdec,turaen2d� IFilessh&ouSlhdell rSecrcipetiinvge an email  18
confirmation wTiotbihas aD�rlsichnmkidto my new profile dashboard.
Write a Regular Expression to Find All Users
Reg ex: ^As an? .*, <h1>Bulk Download for Reviewers</h1>
As a reviewer (or administrator), I want to be able to download all submitted
files for a specific application in a single compressed folder (ZIP) so that I
can review the materials efficiently offline.
       Given I am logged in as a registered reviewer and viewing the details of a
specific submission,
When I click the "Download All Submission Files" button,
       Then The system should generate and prompt me to download a single .zip
file containing every file uploaded by the applicant for that submission.
<h1>Applicant Registration</h1>
As an applicant, I want to be able to register and create a profile so that I
can easily track my submissions and receive official communications.
Given I am a first-time user and I am on the system's registration page,
When I fill in all required fields and click the "Register" button,
       Then My acCcSou35nLtSosfhtwoaurledCobnestruccrteioant: eLdec,turaen2d� IFilessh&ouSlhdell rSecrcipetiinvge an email  19
confirmation wTiotbihas aD�rlsichnmkidto my new profile dashboard.
Introduction to
Python
CS 35 L
SOFTWARE CONSTRUCTION
                                                                                                                        20
Python is an Interpreted Language
� Interpreted languages do not have a compiler (compared to C/C++)
� Code is executed line by line at runtime by an interpreter
� Rapid prototyping (no compiler step needed)
� Execution can be slower than compiled code (interpreter creates runtime
  overhead)
� Requires Python to be installed on the target system
  (in contrast to C/C++ which just compiles directly to the target platform)
   � Software is delivered with the original source code
                  In-Class Discussion: What challenges might arise
                  from writing systems in interpreted languages
                         (compared to compiled languages)?
help � the man of Python
                 When running the python command without  The * means you can pass
Terminal specifying a script filename, the Python         multiple arguments to print
                                                                   separated by a
$python  interpreter starts in Interactive Mode.
>>> help(print)
Help on built-in function print in module builtins:
print(*args, sep=' ', end='\n', file=None, flush=False)
Prints the values to a stream, or to sys.stdout by default.
sep: string inserted between values, default a space.
end: string appended after the last value, default a newline.
file: a file-like object (stream); defaults to the current sys.stdout.
flush: whether to forcibly flush the stream.
         CS 35L Software Construction: Lecture 2 � Files & Shell Scripting             22
         Tobias D�rschmid
Hello World       No main function needed.
                  It runs the script from
                           top to bottom
 simplehello.py
print("Hello, World!") #The simple print
 Hello Works in C++
#include <iostream>
int main() {
       std::cout << "Hello, World!" << std::endl;
       return 0;
}
Hello World
simplehello.py
print("Hello, World!") #The simple print
multiple values     Separator symbol      end symbol displayed
hello.py                   used when        at the end of the
                                            concatenated list
                    concatenating the
                       list elements
print("Hello", "World", sep=', ', end='!\n')
 Terminal
$python ./hello.py
Hello_World!
Python is a Dynamically Typed Language
� Variables can change their types dynamically
 Terminal       type(var) returns the
>>> x = 4          class of which the
>>> type(x)
<class 'int'>  variable is an instance of
>>> x = "4"
>>> type(x)                                           In-Class Discussion: What
<class 'str'>                                   challenges might arise from writing
                                                     systems in a language with
                                                             dynamic typing?
Python Arithmetic on Basic Types
 Terminal
>>> "foo" + "bar"
'foobar'
>>>2 + 3
5
>>> 2.0 + 3
5.0
>>> 2.0 + "3.0"
TypeError: unsupported operand type(s) for +: 'float'
and 'str'
Data Types are Objects in Python
� Basic Types: int, float, bool (similar to C).
   � arithmetic operation with int and float --> float
� Strings: Strings are immutable (cannot be changed after creation)
  and a distinct type (str), not just an array of characters like in C
   � Build-in sub string operator: string_example[start:end:stride]
       � Default of start is 0
       � Default of stride is 1
   � Strings can be written either as "string" or as 'string'
       � Allows you to add single or double quotation marks without escaping them as
         special characters e.g., "my name is 'Tobias'"
                                    or 'my name is "Tobias"'
Substring Operations are Really Useful
Terminal
>>> "foobar"[:3] (same as "foobar"[0:3])
'foo'                        Negative end values
>>> "foobar"[:-1]           count backwards from
'fooba'
>>> "foobar"[:-2]           the last character
'foob'
>>> "foobar"[::-1]          Reverses the string
'raboof'
          CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  28
          Tobias D�rschmid
f-Strings are Super Useful
� f-Strings (formatted strings) are a concise and efficient way to embed
   Python expressions inside string literals for formatting
Terminal  f-Strings start with f""
>>> print(f"5 * 11 = {5*11}.")
'5 * 11 = 55.'              F-strings convert expressions inside braces
>>> PI = 3.14159265               to strings (calling the str operator)
>>> print(f"Pi rounded is {PI:.2f}.")
'Pi rounded is 3.14' Float with 2 decimals
>>> print(f"42 in binary is {42:b}.")
'42 in binary is 101010'
          CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  29
          Tobias D�rschmid
Python has built-in List Data Structures
� Create a new list via: my_list=list() or new_list=[]
� Query the number of elements via: len(my_list)
� Add an element via: my_list.append(4) or my_list.insert(index, 4)
� You can remove an element via my_list.pop(index)
� Sort elements that support an < operator via my_list.sort()
 Terminal               >>> print(new_list)
>>> new_list = []
>>> print(new_list)     [3]
[]                      >>> print(len(new_list))
>>> new_list.append(3)  1
    CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  30
    Tobias D�rschmid
Defining Sequences in Python
� Sequences are defined via [expr(x) for x in sequence]
 Terminal                           range(start,end)defines an
>>> [x for x in range(1,10)]     integer sequence starting at start
[1, 2, 3, 4, 5, 6, 7, 8, 9]                and ending at end-1
>>> [2*x for x in range(4)]      The default for start is 0
[0, 2, 4, 6]
>>> min([_ for _ in range(10)])  min, max, and sum are
0                                   built-in functions
Python has great Syntactic Sugar                                    "Syntactic Sugar" refers to
                                                                      programming language
� You can unpack a list into variables: [a, b] = [1, 2]                  features that make
� You can swap variables via: a, b = b, a
                                                                   programming easier but are
                                                                    not adding any additional
                                                                   capabilities to the language
 Terminal            >>> a, b = b, a
>>> [a, b] = [1, 2]  >>> a
>>> a                2
1                    >>> b
>>> b                1
2
Python is an Object-Oriented Language
simplehello.py
class Book:  Class comment                              # Create an instance of the class
"""Represents a book with a title and an author."""     my_book = Book("Pride and Prejudice",
                    __init__ is the constructor         "Jane Austen", 1813)
def __init__(self, title, author, year):                                               Create a Book object
self.title = title                Member variables are  # 1. Using print()
self.author = author                    accessed via
self.year = year                                        print(my_book)          Calls mybook.__str__()
                                  self.varName          # Output: "Pride and Prejudice" by Jane
                                                        Austen (1813)
def __str__(self):                Method comment
      """
Implements the str() operator (the user-friendly        # 2. Using the str() function explicitly
                                                        print(str(my_book))
string representation).
                                                                            Equivalent to print(my_book)
This is what print() calls.
                                                        # 3. Using in an f-string explicitly
"""                                                     print(f"The book is: {my_book}")
return f'"{self.title}" by {self.author}
({self.year})'
Incorrect indentation results in                                        Calls mybook.__str__()
compiler errors!
             CS 35L Software Construction: Lecture 2 � Files & Shell Scripting                               33
             Tobias D�rschmid
Let's use Python to read user stories via RegEx
 regex.py  Imports the module with name "re" (regex)
import re
import os     Convention to name constants in all upper case
FILE_PATH = 'L3_file_example.html'
What next???
Check out the documentation at https://docs.python.org/3/library/re.html
RegEx Reference: https://tobiasduerschmid.github.io/SEBook/tools/regex.html
           CS 35L Software Construction: Lecture 2 � Files & Shell Scripting  34
           Tobias D�rschmid
Please fill out your Exit Tickets
on Bruin Learn!
Please summarize three key insights you learned about Python today
                         Imagine you are a developer at NASA and you should develop software for a drone that is supposed to fly
                         many missions autonomously in the atmosphere of Saturn's moon Titan. What challenges might arise from
                         using Python as programming language for this project? Please describe at least one challenge that
                         specifically results from the use of the Python programming language?
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)

```
