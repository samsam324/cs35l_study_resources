# Testing, TDD, Refactoring Cheatsheet (candidate items)

## Why test (the case)
- Knight Capital 2012: $440M lost in 45 min from untested code → bankruptcy
- Validation + debugging = 50–75% of dev time

## Verification vs Validation
- **Verification** = "Did we build the thing right?" (code matches spec)
- **Validation** = "Did we build the right thing?" (spec matches need)

## Test pyramid
```
       /\
      /E2E\        few, slow
     /------\
    /  Int.  \    some
   /----------\
  /    Unit    \  many, fast
 /--------------\
```

## Test types
| Type | Tests | Speed |
|---|---|---|
| Unit | single function/class | ms |
| Integration | multiple units together | sec |
| System / E2E | whole app from user perspective | min |

## Tools
- **Unit**: pytest (Python), JUnit (Java), Jest (JS)
- **E2E web**: **Playwright** (lecture)
- **BDD**: **Cucumber** with **Gherkin**
- **CI/CD**: Jenkins, Travis, **GitHub Actions**

## pytest basics
```python
def test_add():
    assert add(2, 3) == 5

def test_raises():
    with pytest.raises(ValueError):
        bad_input()
```
- File name `test_*.py`, function name `test_*`
- Just use `assert`

## pytest fixtures
```python
@pytest.fixture
def sample_user():
    return {"name": "Alice"}

def test_name(sample_user):
    assert sample_user["name"] == "Alice"
```
- Fixture function NAME used as test parameter → auto-injection

## TDD: Red — Green — Refactor
1. 🔴 Write a failing test (must fail)
2. 🟢 Write MINIMUM code to pass (cheat if needed)
3. 🔁 Refactor (clean up, tests stay green)

## Uncle Bob's 3 Rules of TDD
1. Don't write production code unless to make a failing test pass.
2. Don't write more of a test than is sufficient to fail.
3. Don't write more production code than is sufficient to pass.

## FIRST characteristics
- **F**ast — runs in milliseconds
- **I**ndependent — order doesn't matter
- **R**epeatable — same result every time
- **S**elf-validating — pass/fail, no human eye
- **T**imely — written close to (or before) production code

## Test design strategies
- **Equivalence classes** — group inputs that should behave the same; test one per class
- **Boundary value analysis** — test edges (`min-1, min, min+1, max-1, max, max+1`)
- **Edge cases** — empty, max, min, zero, negative, unicode, null
- **Indirect inputs/outputs** — anything not visible from the call signature (DB, network, files)

## Test doubles
| Type | What | Use |
|---|---|---|
| **Stub** | canned return values | code READS from dep |
| **Mock** | records calls, assert on them | code WRITES to dep |
| **Fake** | simplified working impl | when stub can't be realistic (in-memory DB) |
| **Spy** | wraps real obj + records | want real behavior + audit trail |

## Refactoring
**Definition:** change code STRUCTURE without changing BEHAVIOR.
- Test suite enables safe refactoring (tests stay green)

## Code smells
- Duplicate code
- Long method (>30 lines)
- Large class / God class
- Long parameter list (5+)
- Magic numbers (unexplained literals)
- Dead code
- Feature envy (method uses other class's data more)
- Data clump (same params always together)
- Shotgun surgery (one change → many files)
- Conditional dispatch on `type` field → use polymorphism

## Replace Conditional with Polymorphism
**Before:**
```python
def pay(emp):
    if emp.type == "hourly": ...
    elif emp.type == "salaried": ...
```
**After:**
```python
class Employee:
    def pay(self): pass
class Hourly(Employee):
    def pay(self): ...
class Salaried(Employee):
    def pay(self): ...

emp.pay()    # polymorphic dispatch
```
Aligns with **Open-Closed Principle** (SOLID).

## CI / CD
- **CI** = Continuous Integration — every push triggers tests
- **CD** = Continuous Deployment — auto-deploy on green CI
- Tools: **Jenkins, Travis, GitHub Actions**

## Gherkin / BDD
```gherkin
Feature: Login
  Scenario: Valid login
    Given a registered user "alice"
    When she submits username "alice" and password "secret"
    Then she sees the dashboard
```
- Given = setup
- When = action
- Then = expected outcome
- And = chain more steps

## V&V vs testing
- **V&V** = Verification + Validation (broader)
- Testing is one technique to do V&V; others: reviews, formal methods, static analysis

## Test naming conventions
- `test_<feature>_<scenario>_<expected>`
- e.g., `test_login_with_wrong_password_returns_401`

## Common acronyms (mnemonics)
- **TDD**: Test-Driven Development
- **BDD**: Behavior-Driven Development
- **CI/CD**: Continuous Integration / Continuous Deployment
- **V&V**: Verification & Validation
- **FIRST**: Fast, Independent, Repeatable, Self-validating, Timely
- **DRY**: Don't Repeat Yourself
- **YAGNI**: You Aren't Gonna Need It
- **KISS**: Keep It Simple, Stupid

## Additional items (potentially missing)

### pytest setup / teardown
```python
@pytest.fixture
def db():
    conn = connect()                # setup
    yield conn                       # provide to test
    conn.close()                     # teardown
```

### Fixture scopes
- `@pytest.fixture(scope="function")` — default; new instance per test
- `@pytest.fixture(scope="class")` — once per test class
- `@pytest.fixture(scope="module")` — once per test file
- `@pytest.fixture(scope="session")` — once for whole test run

### Parametrize tests
```python
@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (-1, 1, 0),
    (0, 0, 0),
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```

### Skip / xfail
```python
@pytest.mark.skip(reason="not impl")
@pytest.mark.skipif(sys.version_info < (3, 9), reason="Py3.9+")
@pytest.mark.xfail   # expected to fail
```

### Run pytest CLI
- `pytest`
- `pytest -v` verbose
- `pytest -k "name"` filter by test name
- `pytest test_foo.py::test_one` run specific test
- `pytest --pdb` drop to debugger on failure
- `pytest -x` stop on first failure
- `pytest --cov=mypkg` coverage report

### Code coverage
- **Line coverage**: % of lines executed
- **Branch coverage**: % of if/else branches taken
- **Path coverage**: % of unique paths
- 100% line ≠ correct; just means executed
- Coverage tools: pytest-cov, Istanbul (JS), JaCoCo (Java)

### Property-based testing (brief)
- Tools: Hypothesis (Python), QuickCheck (Haskell-origin)
- Generate random inputs; assert properties hold
- Found bugs traditional tests miss

### Mutation testing (brief)
- Tool changes (mutates) your code; runs tests
- If tests still pass, mutation survived → tests too weak
- Tools: mutmut (Python), Stryker (JS)

### unittest.mock (Python)
```python
from unittest.mock import Mock, patch

mock = Mock()
mock.method.return_value = 42
mock.method("arg")
mock.method.assert_called_once_with("arg")

@patch("mymod.requests.get")
def test_x(mock_get):
    mock_get.return_value.status_code = 200
```

### Continuous Integration providers
- GitHub Actions (most popular now)
- GitLab CI
- Jenkins (self-hosted classic)
- CircleCI, Travis CI (legacy)

### GitHub Actions example
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with: { python-version: '3.11' }
      - run: pip install -r requirements.txt
      - run: pytest
```

### Test smells (anti-patterns)
- **Conditional test logic** — tests with if/else
- **Mystery guest** — relies on external state (file, DB)
- **Eager test** — one test checks multiple unrelated things
- **Test logic in production** — code has `if testing` branches
- **Sleeping** — `time.sleep(5)` instead of polling/wait
- **Flaky tests** — sometimes pass, sometimes fail

### AAA pattern (Arrange-Act-Assert)
```python
def test_something():
    # Arrange — set up state
    user = User(name="Alice", balance=100)
    # Act — do the thing
    user.withdraw(40)
    # Assert — verify
    assert user.balance == 60
```

### Given-When-Then (BDD form of AAA)
Just a different naming convention; same idea.

### Refactoring catalog (Martin Fowler, brief)
- Extract Method
- Inline Method
- Rename Variable / Method
- Move Method / Field
- Extract Class / Inline Class
- Replace Magic Number with Constant
- Replace Conditional with Polymorphism
- Replace Loop with Pipeline
- Introduce Parameter Object

### Test pyramid violations
- **Ice cream cone** anti-pattern — many E2E, few unit (slow, brittle)
- **Hourglass** anti-pattern — many unit, many E2E, few integration (gaps in middle)
- Goal: bottom-heavy **pyramid**
