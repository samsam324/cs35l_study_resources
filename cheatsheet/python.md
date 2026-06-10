# Python Cheatsheet (candidate items)

## Syntax basics
- No semicolons. Indentation = block.
- `:` ends if/for/while/def/class header
- `# comment` (single-line). Triple-quote `"""..."""` for docstring.

## Types
- `int float complex` numbers
- `str` strings (immutable)
- `list` mutable seq `[1,2,3]`
- `tuple` immutable seq `(1,2,3)`
- `dict` `{k:v}`
- `set` `{1,2,3}` (unordered, unique)
- `bool` `True / False`
- `None` null

## Truthiness
Falsy: `None`, `False`, `0`, `""`, `[]`, `{}`, `()`, `set()`

## Operators
- `==` value equality
- `is` identity (same object) — use for `None` checks
- `//` integer division
- `**` exponent
- `%` modulo / string format

## String ops
- `f"value is {x}"` f-string
- `"abc".upper() .lower() .strip() .split(',') .replace('a','b')`
- `",".join(["a","b"])`
- `"abc"[1:3]` slice → `"bc"`
- `"abc" * 3` → `"abcabcabc"`
- `len(s)` / `'a' in s`

## List ops
- `lst.append(x)` add to end
- `lst.extend([...])` extend
- `lst.insert(i, x)` insert at index
- `lst.pop()` remove + return last
- `lst.pop(0)` remove + return first
- `lst.remove(x)` remove first occurrence
- `lst.sort()` in place; `sorted(lst)` returns new
- `lst.reverse()` / `reversed(lst)`
- `lst[i]` access; `lst[-1]` last
- `lst[a:b]` slice; `lst[a:b:step]` step
- `lst + lst2` concat
- `[x for x in lst if cond]` list comp

## Dict ops
- `d[k] = v` set
- `d.get(k, default)` safe lookup
- `d.keys() / .values() / .items()`
- `del d[k]`
- `k in d` check key
- `{k:v for k,v in items}` dict comp
- `d.setdefault(k, [])` create-if-missing
- `collections.defaultdict(list)` auto-default

## Comprehensions
- List: `[x*2 for x in lst]`
- Filter: `[x for x in lst if x>0]`
- Dict: `{x: x**2 for x in lst}`
- Set: `{x for x in lst}`
- Generator (lazy): `(x*2 for x in lst)` — uses `()`, not `[]`

## Functions
```python
def f(x, y=10, *args, **kwargs):
    return x + y
```
- Default args evaluated ONCE at def time — **mutable default trap**
- `*args` collects positional into tuple
- `**kwargs` collects keyword into dict
- `lambda x: x*2` anonymous

## Classes
```python
class Foo:
    class_var = 0
    def __init__(self, x):
        self.x = x
    def method(self):
        return self.x
```
- `self` is explicit (vs C++ `this`)
- No real `private` — use `_name` convention; `__name` triggers name-mangling
- Dunder methods: `__init__ __str__ __repr__ __eq__ __len__ __iter__ __next__ __enter__ __exit__`

## Inheritance
```python
class Dog(Animal):
    def __init__(self, name):
        super().__init__(name)
```

## Control flow
- `if x: ... elif y: ... else: ...`
- `for x in iterable: ...`  (else clause runs if loop completes without break)
- `while cond: ...`
- `break / continue`
- `for i, x in enumerate(lst)` index + value
- `for a, b in zip(lst1, lst2)` parallel iteration

## Context managers
```python
with open("f.txt") as f:
    data = f.read()
```
Auto-closes on exit. Use for files, locks, DB connections.

## Exceptions
```python
try:
    risky()
except ValueError as e:
    handle(e)
except (TypeError, KeyError):
    ...
else:
    no_exception()
finally:
    cleanup()

raise ValueError("msg")
```

## EAFP vs LBYL
- **EAFP** (Easier to Ask Forgiveness): `try: x = d[k] except KeyError: ...`
- **LBYL** (Look Before You Leap): `if k in d: x = d[k]`
- Python idiom prefers EAFP.

## LEGB scoping (name resolution order)
1. **L** ocal
2. **E** nclosing function
3. **G** lobal (module)
4. **B** uiltin

Use `global x` or `nonlocal x` to write to outer scopes.

## Common gotchas
- Mutable default: `def f(x=[])` — list shared across all calls!
- `==` vs `is` — use `is` for `None`/`True`/`False` checks
- `a = b = []` — both point to SAME list
- Late-binding closures in loops
- Integer overflow: never (Python ints are arbitrary precision)

## sys.argv
```python
import sys
sys.argv[0]  # script name
sys.argv[1]  # first arg
```

## Useful stdlib
- `os.path.join`, `os.listdir`, `os.environ['VAR']`
- `re.search`, `re.findall`, `re.sub`, `re.compile`
- `json.dumps`, `json.loads`
- `collections.Counter`, `defaultdict`, `OrderedDict`, `namedtuple`
- `itertools.chain`, `combinations`, `permutations`, `product`

## Duck typing
"If it walks like a duck and quacks like a duck..." — no need for inheritance/interface; just need the right methods.

## File I/O
```python
with open("f.txt", "r") as f:
    contents = f.read()         # whole file
    lines = f.readlines()        # list of lines

with open("f.txt", "w") as f:    # 'w' truncates, 'a' appends
    f.write("text\n")
```

## Additional items (potentially missing)

### Conditional expression (ternary)
```python
x = a if cond else b
```

### Walrus operator (3.8+)
```python
if (n := len(a)) > 10:
    print(f"List too long: {n} elements")
```
Assigns AND uses value in one expression.

### Type hints (3.5+)
```python
def add(a: int, b: int) -> int: return a + b
names: list[str] = []
age: dict[str, int] = {}
```
Optional & ignored at runtime; static checkers (mypy) verify them.

### pathlib (modern path ops)
```python
from pathlib import Path
p = Path("data") / "file.txt"
p.exists()
p.read_text()
p.write_text("hi")
p.suffix         # .txt
p.parent
p.glob("*.csv")
```

### subprocess (run external commands)
```python
import subprocess
result = subprocess.run(["ls", "-l"], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)
```

### argparse
```python
import argparse
ap = argparse.ArgumentParser()
ap.add_argument("--count", type=int, default=10)
args = ap.parse_args()
print(args.count)
```

### Useful string methods
- `s.startswith("pre")` / `s.endswith(".py")`
- `s.find(sub)` returns -1 if not found
- `s.rfind(sub)` from the right
- `s.zfill(5)` pad with zeros
- `s.center(10)` `s.ljust(10)` `s.rjust(10)`
- `s.isdigit() s.isalpha() s.isalnum() s.isspace()`

### F-string formatting
- `f"{x:.2f}"` — 2 decimals
- `f"{x:>10}"` — right-pad
- `f"{x:0>4}"` — zero-pad
- `f"{x:#x}"` — hex
- `f"{x:,}"` — thousands separator
- `f"{x:%}"` — percent
- `f"{x!r}"` — use repr()
- `f"{name=}"` — debug ("name=value")

### List methods full
- `lst.count(x)` — number of occurrences
- `lst.index(x)` — first index of x (raises if missing)
- `lst.clear()`
- `lst.copy()` — shallow copy
- Or `lst[:]` or `list(lst)` for shallow copy
- `copy.deepcopy(lst)` for deep copy

### Dict methods full
- `d.update(other)` — merge
- `d.pop(k, default)` — remove + return
- `d.popitem()` — remove arbitrary item
- `d.clear()`
- `d.copy()` — shallow
- `{**d1, **d2}` — merge into new dict (3.5+)
- `d1 | d2` — merge (3.9+)

### Sets
- `s.add(x) / s.remove(x) / s.discard(x)`
- `s1 | s2` union; `s1 & s2` intersection
- `s1 - s2` difference; `s1 ^ s2` symmetric diff
- `s1 <= s2` subset

### Iteration helpers
- `enumerate(lst, start=0)` → (i, x)
- `zip(a, b, c)` — parallel iteration
- `zip(*pairs)` — unzip
- `map(fn, lst)`, `filter(pred, lst)`
- `sorted(lst, key=fn, reverse=True)`
- `min(lst, key=fn) / max(lst, key=fn)`
- `sum(lst, start=0)`
- `any(lst) / all(lst)`
- `reversed(lst)`

### Generators
```python
def gen():
    for i in range(3):
        yield i * 2

for x in gen(): print(x)   # 0, 2, 4
```
- `yield` makes a generator
- `yield from other_gen` delegate

### Decorators
```python
def loud(fn):
    def wrapper(*args, **kw):
        print(f"calling {fn.__name__}")
        return fn(*args, **kw)
    return wrapper

@loud
def greet(name): print(f"hi {name}")
```

### Common dunder methods
- `__init__` constructor
- `__repr__` formal string (debug)
- `__str__` informal string (user)
- `__eq__` equality
- `__hash__` hashable (for set/dict keys)
- `__lt__ __le__ __gt__ __ge__` comparisons (or use `@functools.total_ordering`)
- `__len__` `len()` support
- `__iter__ __next__` iterator support
- `__contains__` `in` support
- `__getitem__ __setitem__` `[]` support
- `__call__` makes object callable
- `__enter__ __exit__` context manager (`with`)

### Common stdlib quick refs
- `os.environ.get("VAR")`
- `os.path.join(a, b)`
- `os.makedirs(path, exist_ok=True)`
- `glob.glob("*.py")`
- `json.dumps(obj, indent=2)`
- `pickle.dumps(obj)` (binary serialization)
- `time.time()` (epoch seconds)
- `datetime.now()`, `datetime.strptime(s, fmt)`

### sys.argv basics
```python
import sys
sys.argv         # list
sys.argv[0]      # script name
sys.argv[1:]     # args
len(sys.argv)
```

### Conditional import / try-import pattern
```python
try:
    import simplejson as json
except ImportError:
    import json
```
