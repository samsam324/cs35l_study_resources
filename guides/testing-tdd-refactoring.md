# CS 35L Lecture 9 â€” Testing, TDD & Refactoring

Instructor: Tobias DÃ¼rschmid (Assistant Teaching Professor, UCLA Computer Science Department)

The cycle motif used throughout this lecture:
- **Red Light** -> **Implement Feature** -> **Green Light** -> **Improve Code** -> **Refactor** -> **Add Test** -> back to Red Light
- Equivalent triangle: **Red** -> **Green** -> **Refactor** -> back to **Red**

---

## 1. Software Engineering Storytime: Knight Capital Group

"Once upon a Time, in the Kingdom of Wall Street there was a Knight..."

### Who was Knight Capital?

- **High-frequency Stock Trading** company.
  - **Buying** stocks at a **low** price and shortly after **selling** at a slightly **higher** price at **very high speeds**.
- **17 years of dedicated work** made Knight one of the leading trading houses on Wall Street.
- In 2012, Knight was the **largest trader** in U.S. stocks (**17% of daily trading volume**, **$21B/day**).

### A True Story on Losing $440 Million in 28 Minutes â€” Timeline (August 1st 2012)

| Time | Event |
| --- | --- |
| (Before 8:00am) | **New Software Version gets deployed** |
| **8:00am** | Automated **email warnings** are sent about some issue (**nobody cares**) |
| **9:30am** | Stock Market Opens |
| **9:31am** | The market is **flooded with buying orders** from Knight |
| (continuing) | Knight's trading program **starts buying stocks** at an **exceptionally high volume of $148M/min** |
| **9:32am** | Traders on Wall Street are puzzled why it **keeeeeeeeps buying** |
| (continuing) | Knight's program is responsible for **50% of all trades** on the major stock exchanges |
| (next) | Knight **rolls back** the trading program to a **previous version** |
| (after rollback) | Knight's trading program **buys 8x as many stocks** |
| **9:58am** | Knight **shuts down everything** |

### Server Diagram of the Disaster

- 8 Servers (Server 1 through Server 8).
- BEFORE rollback: Only Server 8 was buggy -> **Losing money at high speed**.
- AFTER **Rollback to an older version**: ALL 8 servers became buggy -> **Losing money at VERY high speed**.

### The Buggy Code

```
//DO NOT USE IN PRODUCTION!!!!
if (configurationFlag("PowerPeg")) {
    buyHigh();
    sellLow(); //loses money
}
```

This block had been **dead code for 9 years**. **"dead code" = code that will never be executed**.

```
//some new code for
//the new feature used the same name
activateConfigurationFlag("PowerPeg");
```

The new feature used the **same name** as the dormant flag, accidentally reactivating the old buy-high/sell-low logic.

### Discussion Prompt

> Propose a generalizable Software Engineering Principle that would have prevented this! Talk to your neighbor(s)!

---

## 2. Lesson Learned: Always Test Software Before Deployment

Visual metaphor: A factory where TestCase A, B, C are fed as INPUT into "The Application Engine (Code Under Test)", producing Result A, Result B, Result A which are compared against **Expected Output** (Expect A, Expect B) by a **Quality Assurance Inspector** robot that produces **PASS!** or **FAIL!** verdicts.

### Definition of Software Testing

> **Executing the program** with **selected inputs** in a **controlled environment** while checking **assertions on its behavior** & **outputs**.

### Goals of Software Testing

- **Primary: Reveal bugs**, so they can be fixed.
  - **Testing can only prove the existence of bugs (not their absence!)**
- **Secondary: Document** what the code **should** do.

### Concrete Example: pytest in Python

**`my_domain_class.py`** â€” buggy first version:

```python
def calculate_price(price, discount_percent):
    return price - discount_percent
```

**`my_domain_class.py`** â€” corrected version:

```python
def calculate_price(price, discount_percent):
    return price - discount_percent * price/100
```

**`my_test.py`**:

```python
def test_50_discount():
    price = calculate_price(100, 50)
    assert price == 50

def test_discount():
    price = calculate_price(20, 50)
    assert price == 10
â€¦
```

`assert` **fails if the condition is false**.

**Terminal output**:

```
$pytest
tests/test_main.py .F.....F...                                          [100%]
=================== 2 failed, 9 passed in 0.03s ===================
```

> **Pytest is a common testing framework for Python** (calls all functions starting with `test` in `.py` files in the `/tests` folder of a Python project).

---

## 3. Continuous Integration (CI)

### Tests Can Be Automated in Your Continuous Integration Workflow

- **Continuous Integration (CI)** is a software development practice in which changes are **frequently merged** into the main branch.
- Tool support for this allows you to automatically run tests in the CI environment (**on a server that has your code**) to see if a change broke the build.
- **Convention: Only push changes that should pass the tests.**
- **31.5% of all NPM repositories on GitHub** show badges for their build status. [Trockman et al., ICSE 2018]

### Common CI Tools

- **Jenkins**
- **Travis CI**
- **GitHub Actions**

### Badges

- GitHub badge: `Tests passing` (green) â€” indicating the latest build passes.
- GitHub badge: `Tests failing` (red) â€” indicating the latest build is broken.

Reference: [2] A. Trockman et al. *"Adding Sparkle to Social Coding: An Empirical Study of Repository Badges in the npm Ecosystem"*. ICSE 2018.

---

## 4. V&V: Verification vs Validation

### Not All V&V (Verification & Validation) Can be Automated

| Verification | Validation |
| --- | --- |
| **Did we build the system right?** | **Did we build the right system?** |
| Can often be **automated** | Must be done **manually** (by someone who represents real users) |

### Traditionally Testing Has Happened After Development (Waterfall)

The traditional sequential phases:

1. **Requirements Analysis** â€” *What should be built?*
2. **Design** â€” *How should we build it?*
3. **Development** â€” *Build it*
4. **Testing** â€” *Did we build the right system?* (highlighted yellow)
5. **Deployment** â€” *Deliver to customer*

---

## 5. Test-Driven Development (TDD) Puts Testing First

Originally proposed by **Kent Beck** (who later went on to work for Facebook/Meta).

More on this in *"Test Driven Development: By Example"* by Kent Beck.

### The Red-Green-Refactor Cycle

| Phase | Meaning |
| --- | --- |
| **Red** | For your new requirement write a **small test that fails**, and perhaps doesn't even compile at first. |
| **Green** | Make the **test pass** with **minimal coding effort**, potentially using simplifying shortcuts in the process. |
| **Refactor** | Make the design more **elegant**, cleaner, and potentially **faster** while **not changing the functionality** of the program. |

Cycle direction: Red -> Green -> Refactor -> Red -> ...

---

## 6. Refactoring Turns Code Smells into Clean Code

- **Motivation:** "*When a software system evolves, its complexity increases unless work is done to reduce it*" (**Second Law of Software Evolution** [Lehman et al., 1997]).
- Refactoring **reduces accidental complexity** in software systems and/or makes them easier to read / understand.
- **Code is read more often than it is written**, so make sure your code is readable.
- **Common themes in refactoring:**
  - Avoid **code duplication** by creating **abstractions**.
  - Remove **dead code** & improve **documentation**.
  - Replace **conditional logic** with **polymorphism**.
  - **Improve naming** of classes, functions, & variables.
  - Learn common refactoring techniques here: https://refactoring.guru/refactoring/catalog

Reference: [1] M. M. Lehman et al. *"Metrics and laws of software evolution-the nineties view,"* Proc. International Software Metrics Symposium 1997.

---

## 7. TDD Mantra

Loop diagram:

```
Add a Test
    |
    v
Run Tests  <-----+
    |            |
   Red Light     |
    |            |
    v            |
Make Small      Green Light (goes back to Add a Test)
Change ---------+
```

Equivalently, the cyclical pictogram: **Red Light** -> **Implement Feature** -> **Green Light** -> **Improve Code** -> **Refactor** -> **Add Test** -> Red Light again.

---

## 8. TDD Example: A Simple Stack Implementation

> A **stack** is a list data structure that operates on the **Last-In, First-Out (LIFO)** principle.

Walkthrough visual:
- Start with stack `[2, 1]` (bottom 1, top 2).
- `push(3)` -> `[3, 2, 1]`.
- `pop()` -> `[2, 1]`.

Walk through this at home: https://github.com/UCLA-CS-35L/TDD_stack_example

### Step 1 (RED): Start by Writing a Failing Test

`test_main.py`:

```python
from src.my_stack.my_stack import MyStack

def test_create_stack():
    stack = MyStack()
```

`my_stack.py`: (empty)

**Terminal**:

```
$pytest
tests/test_main.py:1: in <module>
    from src.my_stack.my_stack import MyStack
E   ImportError: cannot import name 'MyStack' from 'src.my_stack.my_stack'
Did you mean: 'my_stack'?
==================== short test summary info ====================
ERROR tests/test_main.py
!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!
======================== 1 error in 0.04s =======================
```

> **We want to see the test fail first!** This step is **"testing the test"**. Otherwise, we can't be sure that the test works. (maybe we accidentally referenced the stack class from a library?)

### Step 2 (GREEN): Then Add the Simplest Code that Passes the Test

`my_stack.py`:

```python
class MyStack:
    pass
```

> **Null operation** in Python (to tell the interpreter the class is empty).

`test_main.py`: unchanged.

**Terminal**:

```
$pytest
==================== test session starts ====================
tests/test_main.py .                                  [100%]
===================== 1 passed in 0.00s =====================
```

> If we had written more code than necessary to pass the test, we **would not know whether the additional code is correct**, since we don't have a test for it.

### Step 3 (REFACTOR): Improve the Quality of Your Code via Refactoring

`my_stack.py`:

```python
class MyStack:
    """
    A simple implementation of a Stack data structure.
    A Stack operates on the Last-In, First-Out (LIFO) principle.
    """
    pass
```

> Comments should **describe only a fixed contract** of what this class does across future versions (and ideally also why).

Tests still pass:

```
$pytest
==================== test session starts ====================
tests/test_main.py .                                  [100%]
===================== 1 passed in 0.00s =====================
```

> Tests give us the **confidence** that our **refactoring** didn't **break anything**. Everything still works!

### Step 4 (RED): Add Another Test for New Functionality

`test_main.py`:

```python
from src.my_stack.my_stack import MyStack

def test_create_stack():
    stack = MyStack()
    assert stack.is_empty()
```

> Writing the **code using** the method before its implementation leads to more **modular design**.

**Terminal**:

```
________________ test_create_stack ________________
def test_create_stack():
    stack = MyStack()
>   assert stack.is_empty()
           ^^^^^^^^^^^^^^
E   AttributeError: 'MyStack' object has no attribute 'is_empty'
===================== 1 failed in 0.02s ====================
```

### Step 5 (GREEN): Then Add the Simplest Code that Passes the Tests

`my_stack.py`:

```python
class MyStack:
    """
    A simple implementation of a Stack data structure.
    A Stack operates on the Last-In, First-Out (LIFO) principle.
    """
    def is_empty(self):
        return True
```

> This is the **simplest implementation** to pass the test. **We have not tested other non-empty stacks yet!** Hence, we will only implement it when we have a test for this.

Tests pass.

### Step 6 (REFACTOR): Improve the Quality of Your Code via Refactoring

`my_stack.py`:

```python
class MyStack:
    def is_empty(self):
        """
        Checks whether the stack is currently empty.
        Returns:
            bool: True if the stack contains no elements, False otherwise.
        """
        return True
```

> Our code is still very clean, the only thing we can do is **documenting** the contract of this method.

### Step 7 (RED): Add Another Test for New Functionality

`test_main.py`:

```python
from src.my_stack.my_stack import MyStack

def test_create_stack():
    stack = MyStack()
    assert stack.is_empty()

def test_add_element():
    stack = MyStack()
    stack.add_element(3)
    assert not stack.is_empty()
```

> We try to write **simple, small tests**, ideally **the simplest test that fails**.

**Terminal**:

```
$pytest
tests/test_main.py .F                                 [100%]
________________ test_add_element ________________
def test_add_element():
    stack = MyStack()
>   stack.add_element(3)
         ^^^^^^^^^^^^^^^^^
E   AttributeError: 'MyStack' object has no attribute 'add_element'
=================== 1 failed, 1 passed in 0.02s ==============
```

### Step 8 (GREEN): Now we Actually Need to Think and Add Lots of Code

`my_stack.py`:

```python
class MyStack:
    def __init__(self):
        self.elements = []

    def is_empty(self):
        """ â€¦ """
        return len(self.elements) == 0

    def add_element(self, element):
        self.elements.append(element)
```

Tests pass: 2 passed in 0.00s.

### Step 9 (REFACTOR): Choose Better Method Names

`test_main.py`:

```python
from src.my_stack.my_stack import MyStack

def test_create_stack():
    stack = MyStack()
    assert stack.is_empty()

def test_push():
    stack = MyStack()
    stack.push(3)
    assert not stack.is_empty()
```

`my_stack.py`:

```python
class MyStack:
    def __init__(self):
        self.elements = []

    def is_empty(self):
        """ â€¦ """
        return len(self.elements) == 0

    def push(self, element):
        self.elements.append(element)
```

> Thanks to our tests we know this **refactoring didn't break anything**.

### Step 10 (RED): Add Another Test for New Functionality

`test_main.py`:

```python
from src.my_stack.my_stack import MyStack
â€¦
def test_pop():
    stack = MyStack()
    stack.push(3)
    element = stack.pop()
    assert element == 3
    assert stack.is_empty()
```

**Terminal**:

```
$pytest
tests/test_main.py .F                                 [100%]
________________ test_add_element ________________
def test_pop():
>   element = stack.pop()
              ^^^^^^
E   AttributeError: 'MyStack' object has no attribute 'pop'
=================== 1 failed, 2 passed in 0.02s ==============
```

### Step 11 (GREEN attempt 1): Then Add the Simplest Code that Passes the Tests

`my_stack.py`:

```python
class MyStack:
    def __init__(self):
        self.elements = []
    â€¦
    def push(self, element):
        self.elements.append(element)

    def pop(self):
        element = self.elements[-1]
        return element
```

**Terminal**:

```
$pytest
tests/test_main.py .F                                 [100%]
________________ test_add_element ________________
>   assert stack.is_empty()
E   assert False
E    +  where False = is_empty()
E    +    where is_empty = <src.my_stack.my_stack.MyStack object at 0x10aa30690>.is_empty
=================== 1 failed, 2 passed in 0.02s ==============
```

> Thanks to our tests we know that **we made a mistake in the code**. We didn't change the size of the stack!

### Step 12 (GREEN attempt 2): Fix the Code Until All Tests Pass

`my_stack.py`:

```python
class MyStack:
    def __init__(self):
        self.elements = []
    â€¦
    def push(self, element):
        self.elements.append(element)

    def pop(self):
        element = self.elements[-1]
        self.elements = self.elements[0:-1]
        return element
```

**Terminal**:

```
$pytest
==================== test session starts ====================
tests/test_main.py ..                                 [100%]
===================== 3 passed in 0.02s =====================
```

> Now we have **confidence** that our stack pops correctly for **one element**.

### Step 13: Does the Stack work correctly with Multiple Elements too?

`test_main.py`:

```python
def test_multiple_pop():
    stack = MyStack()
    stack.push(1)
    stack.push(2)
    stack.push(3)
    element = stack.pop()
    assert element == 3
    element = stack.pop()
    assert element == 2
    element = stack.pop()
    assert element == 1
    assert stack.is_empty()
```

> **Already passes Yay!**

```
$pytest
==================== test session starts ====================
tests/test_main.py ..                                 [100%]
===================== 4 passed in 0.02s =====================
```

### Step 14: Is there any Redundant Code in the Tests?

Notice that every test starts with `stack = MyStack()`. This is redundant setup.

```python
def test_create_stack():
    stack = MyStack()
    assert stack.is_empty()

def test_push():
    stack = MyStack()
    stack.push(3)
    assert not stack.is_empty()

def test_multiple_pop():
    stack = MyStack()
    stack.push(1)
    â€¦
```

---

## 9. pytest Fixtures: Objects Created Freshly for Each Test Case

### Refactored test file using a fixture

`test_main.py`:

```python
@pytest.fixture
def stack():
    stack = MyStack()          # Setup Phase: Code run before each test
    yield stack                # Runs the test with fixture
                                # Teardown Phase: Code run after each test

def test_create_stack(stack):  # Uses stack fixture
    assert stack.is_empty()

def test_push(stack):          # Uses stack fixture
    stack.push(3)
    assert not stack.is_empty()
```

Annotations from the slide:
- `@pytest.fixture` / `def stack():` â€” **Defines `stack` fixture**.
- `stack = MyStack()` â€” **Setup Phase: Code run before each test**.
- `yield stack` â€” **Runs the test with fixture**.
- (After `yield`) â€” **Teardown Phase: Code run after each test**.
- `def test_create_stack(stack):` â€” **Uses `stack` fixture**.

### Why Fixtures Matter

> Fixtures are especially useful for **complex setup / teardown procedures**, such as setting up a test database.

```python
@pytest.fixture
def db_connection():
    # 1. Setup: Connect to the test database
    db = db_library.connect(TEST_DB_URL)
    db.add_data(TEST_DATA)
    # 2. Yield: Forward connection object to test
    yield conn
    # 3. Teardown: Close the connection
    db.rollback()
    db.close()
```

Read more here: https://docs.pytest.org/en/stable/how-to/fixtures.html

Tests still pass:

```
$pytest
==================== test session starts ====================
tests/test_main.py ..                                 [100%]
===================== 4 passed in 0.02s =====================
```

---

## 10. TDD Is Not Just About Finding Bugs

| Benefit | Explanation |
| --- | --- |
| **Tests are living documentation** | Tests describe what the code **should do** and are always **up-to-date**. |
| **TDD often leads to better design** | Writing the test first makes you think about **how to use a new method / class first**, which often leads to a **more modular design**. |
| **TDD can increase productivity** | TDD mantra keeps you **focused** on the current task. |

Read more in I. Karac & B. Tunhan *"What Do We (Really) Know about Test-Driven Development?"* IEEE Software 2018.

---

## 11. Rules of Test-Driven Development (According to "Uncle Bob" Robert C. Martin)

1. **Do not write a line of production code until you have a failing unit test.**
2. **Do not write more of a unit test than is sufficient for the test to fail.**
   (You cannot write a lot of unit test code before you switch back to writing production code.)
3. **Do not write more production code than is sufficient to pass the tests.**

=> You will end up writing unit tests & production code **somewhat concurrently**.

> **This is NOT a dogma.** Every rule in software engineering has its exceptions. Once you have mastered TDD, you can adjust it to fit your coding style & project. However, it's hard to master TDD without **first practicing it based on these rules**.

---

## 12. Limitations of TDD

Visual metaphor: "You'll never get to the moon simply by building taller and taller towers."

TDD does **not** work well for:

- **Extremely complex behavior** where the solution to your problem is **not an incremental improvement to simpler problems** (e.g., implementing **ACID-compliant DBMS**, **distributed databases**).
- Systems with **non-binary success outcomes** (e.g., **image recognition**, **image processing**).
- Testing **non-functional properties** (e.g., **response time**, **availability**, â€¦).

---

## 13. Unit Testing Is Not Enough

- Components that **work well in isolation** might **break once they are composed together**.
- **Integration tests** test the **interactions between two components**.

Revisiting the Knight Capital code:

```
//DO NOT USE IN PRODUCTION!!!!
if (configurationFlag("PowerPeg")) {
    buyHigh();
    sellLow(); //loses money
}

//some new code for
//the new feature used the same name
activateConfigurationFlag("PowerPeg");
```

> Works fine in isolation but **not together with the other code**.

Meme: "UNIT TESTS PASSING ... NO INTEGRATION TESTS" (image of a kitchen cabinet whose drawer collides catastrophically with a neighboring drawer).

---

## 14. Levels of Software Tests (The Test Pyramid)

| Level (bottom -> top) | Description |
| --- | --- |
| **Unit Testing** (base) | Tests **individually testable units** of an application (e.g., functions, classes, components) **in isolation**. |
| **Integration Testing** (middle) | Tests whether different components, modules, or services **work together correctly**. |
| **System Testing (End-to-End Testing)** (top) | Tests whether the **complete application** from user input to output works correctly. |

The pyramid shape implies: many unit tests at the base, fewer integration tests, fewest E2E tests at the top.

---

## 15. Cucumber: Executable System Tests in Gherkin (BDD)

Cucumber lets you write executable system tests in **Gherkin** (User Story Acceptance Criteria).

```gherkin
Feature: Withdrawing cash

  Rule: Customers cannot withdraw more than their balance

    Scenario: Successful withdrawal within balance
      Given Alice has 234.56 in their account
      When Alice tries to withdraw 200.00
      Then the withdrawal is successful

    Scenario: Declined withdrawal in excess of balance
      Given Hamza has 198.76 in their account
      When Hamza tries to withdraw 200.00
      Then the withdrawal is declined
```

> Can be **automatically translated** into **executable tests** using **natural language processing**.

See https://cucumber.io/

---

## 16. End-to-End Web Testing Frameworks (Playwright)

### Page Navigation, Entering Text, Clicking a Link

```javascript
await page.goto('https://localhost:8080/');                              // Page Navigation
await page.getByRole('textbox').fill('example value');                   // Entering Text
const submitLink = page.getByRole('link', { name: 'Submit' });
await submitLink.click();                                                // Clicking a Link
```

### Assertions for Visibility and Text Content

```javascript
await expect(page.getByText('Welcome')).toBeVisible();                   // Assertion for Visibility

// At least one of the two elements is visible, possibly both.
await expect(
  page.getByRole('button', { name: 'Sign in' })
    .or(page.getByRole('button', { name: 'Sign up' }))
    .first()
).toBeVisible();

const locator = page.locator('.title');                                  // Assertion for Text Content
await expect(locator).toContainText('substring');
await expect(locator).toContainText(/\d messages/);
```

See more details here: https://playwright.dev/docs/writing-tests

---

## 17. How to Write Good Tests?

- **Select Inputs that test the boundaries & equivalence classes** (e.g., positive numbers, negative numbers, zero, very large, very small).
- Try to cover common **edge cases**, not just the "**happy path**".
- **Write strong assertions!** Your assertion should test **ALL aspects of the specification** (remember the **acceptance criteria** from your user stories? If you've written good ones, this part is easier).
- **Test behavior, not implementation details!**
  - Your test should **pass any correct implementation**. Write it as a **specification of the behavior**, not the design decisions details, because those might change!

Learn this based on concrete Python examples: https://tobiasduerschmid.github.io/SEBook/tools/testing-foundations-tutorial

---

## 18. Indirect Inputs & Indirect Outputs

### Definitions

- **Indirect inputs** are values **returned by another component** whose services the SUT uses.
- **Indirect outputs** are actions that **cannot be observed through the public API of the SUT** which are seen or experienced by other systems or application components.

### Diagram of the Setup

```
+------+    Direct Inputs     +-----------+   Indirect Outputs   +-----------+
| Test |--------------------->|  System   |--------------------->| Depended- |
|      |                      | Under Test|                      | on        |
|      |<---------------------|   (SUT)   |<---------------------| Component |
+------+    Direct Outputs    +-----------+   Indirect Inputs    |  (DOC)    |
                                                                 +-----------+
```

### Indirect Inputs & Outputs Can Be Ordered in Many Different Sequences

Two sequence diagrams illustrate alternate orderings:

**Sequence A (SUT acts first):**
- `Inputs` -> SUT
- SUT -> DOC (`Indirect Outputs`)
- DOC -> SUT (`Indirect Inputs`)
- SUT -> `Outputs`

**Sequence B (DOC acts first):**
- `Inputs` -> SUT
- DOC -> SUT (`Indirect Inputs`)
- SUT -> DOC (`Indirect Outputs`)
- SUT -> `Outputs`

---

## 19. Test Doubles

### 19.1 Mock Object Pattern

- **Problem:** How to **observe indirect outputs** sent to separate DOCs?
- **Context:** The connector between SUT and DOC **cannot easily be intercepted**.
- **Solution:** Create a **Mock Object** that **replaces the DOC** and **only verifies the indirect outputs**.

Diagram:

```
SUT --(Indirect Outputs)--> Mock Object [Verification assert(â€¦)]
```

**Example:** "How to test that the trading system **sends well-formed buy / sell orders** to the stock exchange without sending them to the actual stock exchange?" (Trading System --Indirect Outputs--> Stock Exchange.)

**Different example:** Testing that your app sends the right requests to an **LLM-based AI provider API**.

More details: http://xunitpatterns.com/Mock%20Object.html

### 19.2 Test Spy Pattern

- **Problem:** How to **observe indirect outputs** sent to separate DOCs?
- **Context:** The connector between SUT and DOC cannot easily be intercepted.
- **Solution:** Create a **Test Spy** component that **replaces the DOC** and **forwards the indirect outputs to the test**.

Diagram:

```
SUT --(Indirect Outputs)--> Test Spy [Data Collection] --(Forwarding)--> Test
```

**Example:** "How to test that the trading system sends buy / sell orders to the stock exchange **in the correct order**?"

**Different example:** Testing that your system sends the requests in the correct order to an LLM provider via their API.

More details: http://xunitpatterns.com/Test%20Spy.html

### 19.3 Test Stub Pattern

- **Problem:** How to **control indirect inputs** sent from separate DOCs?
- **Context:** The connector between SUT and DOC cannot easily be intercepted.
- **Solution:** Create a **Test Stub** component that **replaces the DOC** and **sends the desired inputs to the SUT**.

Diagram:

```
SUT <--(Indirect Inputs)-- Test Stub [Return Values]
SUT --(Indirect Outputs)--> Test Stub
```

**Example:** "How to test that the trading system **responds correctly to updates on stock prices** in the stock exchange?"

**Different example:** Testing that your system behaves correctly when receiving various responses from the LLM provider API.

More details: http://xunitpatterns.com/Test%20Stub.html

### 19.4 Test Doubles Unification

> **Test Spies, Mock Components, and Test Stubs are all unified under the term Test Doubles.**

Hierarchy:

```
                Test Double
                    ^
        ____________|____________
       |            |            |
   Test Spy    Mock Object    Test Stub
```

In Node.js HTTP test doubles can be implemented using this library:
https://www.npmjs.com/package/node-mocks-http

---

## 20. "A Software Tester Walks Into A Bar" â€” The Bar Joke / Test Case Catalog

This recurring joke illustrates equivalence-class and edge-case testing. Setting: "THE BROKEN CODE INN".

| Scenario | What it tests |
| --- | --- |
| A Software Tester **WALKS** into a bar | The System Correctly Handles a **Standard Request** |
| A Software Tester **RUNS** into a bar | The System Correctly Handles a **Non-Standard Request** |
| A Software Tester **CRAWLS** into a bar | The System Correctly Handles a **Non-Standard Request** |
| A Software Tester **DANCES** into a bar | The System Correctly Handles a **Non-Standard Request** |
| A Software Tester **FLIES** into a bar | The System Correctly Handles a **Non-Standard Request** |
| A Software Tester orders **"One Cup of Tea Please"** | The System Correctly Handles a **Standard Request** |
| A Software Tester orders **"Two Cups of Tea Please"** | The System Correctly Handles a **Standard Request** |
| A Software Tester orders **"Zero Cups of Tea Please"** | The System Correctly Handles a **Non-Standard Request** |
| A Software Tester orders **"9999999 Cups of Tea Please"** | The System Correctly Handles a **Very Large Request** |
| A Software Tester orders **"-1 Cups of Tea Please"** -> "Invalid Input" | The System Correctly Handles an **Invalid Request** |
| A Software Tester orders **"'qwertyuiop' Cups of Tea Please"** -> "Invalid Input" | The System Correctly Handles an **Invalid Request** |
| A Software Tester orders **"A Lizard in a Teacup Please"** -> "Resource Unavailable" | The System Correctly Handles **Errors** |
| "**Let's Ship The Product!**" | **Testing Complete!** |

### The Twist: A Real Customer Walks Into The Bar

A Real Customer (an old lady) walks into the bar and asks: **"Excuse Me Where are the Bathrooms?"**

> **System Deployed In the Real World** â€” and then: **The Developers And Testers Didn't Expect This Request.** The bar is on fire (Woosh! Blaze!) because real users do unexpected things outside the test catalog. This dramatizes that no matter how thorough your test design, real users can ask questions you never anticipated.

---

## 21. Tutorials

- https://tobiasduerschmid.github.io/SEBook/tools/testing-foundations-tutorial
- https://tobiasduerschmid.github.io/SEBook/tools/tdd-tutorial
- **Test doubles:** Soon-to-come

---

## 22. Exit Ticket Questions (Bruin Learn)

1. In your own words, please summarize **three key insights** you learned about **Testing, TDD, or Refactoring** today. (Do not copy phrases from the lecture slides.)
2. Besides the examples from the lecture, please describe one other concrete **scenario in which applying TDD is a good idea** and one other concrete scenario in which **applying TDD is a bad idea**. (Do not use examples from the lecture.)
3. Please leave any questions that you have about today's material and things that are still unclear or confusing to you (if none, simply write N/A).

Credits: These slides use images from Flaticon.com (Creators: Freepik).


---

# Appendix: Lecture 9 Slides (raw extracted text)

The following is the full extracted text of Lecture 9. Preserved verbatim so no information is lost.

```text
Red Implement Green
Light Feature                 Light
CS 35L Software                            Improve
Construction                                    Code
Lecture 9 ­ Testing, Add
TDD & Refactoring Test        Refactor
Assistant Teaching Professor
Computer Science Department
It's Software Engineering Storytime!
Once upon a Time, in the Kingdom of Wall Street
There was a Knight...
Case Study: Knight Capital Group
A True Story
· High-frequency Stock Trading
   · Buying stocks at a low price and shortly after
      selling at a slightly higher price at very high speeds
· 17 years of dedicated work made Knight
  one of the leading trading houses on Wall Street
· In 2012, Knight was the largest trader in U.S. stocks
  (17% of daily trading volume, $21B/day)
New Software   Automated email    Stock Market  The market is flooded               Knight's trading program
 Version gets  warnings are sent      Opens      with buying orders                 starts buying stocks at
               about some issue                        from Knight                   an exceptionally high
   deployed
                 (nobody cares)                             f                        volume of $148M/min
August 1st
         2012  8:00am             9:30am                  9:31am
               CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring                            3
               Tobias Dürschmid
Case Study: Knight Capital Group
A True Story
· High-frequency Stock Trading
   · Buying stocks at a low price and shortly after
      selling at a slightly higher price at very high speeds
· 17 years of dedicated work made Knight
  one of the leading trading houses on Wall Street
· In 2012, Knight was the largest trader in U.S. stocks
  (17% of daily trading volume, $21B/day)
New Software   The market is flooded  Traders on Wall Street    Knight's program is    Knight rolls back the
 Version gets    with buying orders     are puzzled why it    responsible for 50% of   trading program to a
                      from Knight                             all trades on the major
   deployed                            keeeeeeeeps buying                                previous version
                           f                                      stock exchanges
August 1st                                         k                                              .
         2012            9:31am
                                                 9:32am                                                      4
               CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring
               Tobias Dürschmid
Case Study: Knight Capital Group
A True Story
· High-frequency Stock Trading
   · Buying stocks at a low price and shortly after
      selling at a slightly higher price at very high speeds
· 17 years of dedicated work made Knight
  one of the leading trading houses on Wall Street
· In 2012, Knight was the largest trader in U.S. stocks
  (17% of daily trading volume, $21B/day)
New Software   Knight rolls back the  Knight's trading program
 Version gets  trading program to a   buys 8x as many stocks
   deployed      previous version
August 1st                .
         2012
               CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  5
               Tobias Dürschmid
Case Study: Knight Capital Group
A True Story on losing $440 Million in 28 Minutes
Server 1 Server 2 Server 3 Server 4         Rollback           Server 1 Server 2 Server 3 Server 4
Server 5 Server 6 Server 7 Server 8  to an older version       Server 5 Server 6 Server 7 Server 8
Losing money at                                                Losing money at
    high speed                                                 VERY high speed
New Software
Version gets  Knight rolls back the  Knight's trading program  Knight shuts down
  deployed    trading program to a   buys 8x as many stocks         everything
August 1st      previous version                                      9:58am
        2012
                         .
              CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring                   6
              Tobias Dürschmid
Case Study: Knight Capital Group
A True Story on losing $440 Million in 28 Minutes
Server 1 Server 2 Server 3 Server 4         Rollback      Server 1 Server 2 Server 3 Server 4
Server 5 Server 6 Server 7 Server 8  to an older version  Server 5 Server 6 Server 7 Server 8
Losing money at                      Has been dead code   Losing money at
    high speed                        for 9 years. "dead  VERY high speed
                                      code" = code that
                                          will never be
                                             executed
//DO NOT USE IN PRODUCTION!!!!        //some new code for
if (configurationFlag("PowerPeg")) {  //the new feature used the same name
                                      activateConfigurationFlag("PowerPeg");
     buyHigh();
     sellLow(); //loses money
}
Case Study: Knight Capital Group
A True Story on losing $440 Million in 28 Minutes
Propose a generalizable Software Engineering
   Principle that would have prevented this!
               Talk to your neighbor(s)!
                  Has been dead code
                   for 9 years. "dead
                   code" = code that
                       will never be
                          executed
//DO NOT USE IN PRODUCTION!!!!        //some new code for
if (configurationFlag("PowerPeg")) {  //the new feature used the same name
                                      activateConfigurationFlag("PowerPeg");
     buyHigh();
     sellLow(); //loses money
}
Lesson Learned: Always Test Software
Before Deployment
Lesson Learned: Always Test Software
Before Deployment
   Definition of Software Testing
 Executing the program with selected inputs in a controlled
 environment while checking assertions on its behavior & outputs
   Goals of Software Testing
 · Primary: Reveal bugs, so they can be fixed
      · Testing can only prove the existence of bugs
          (not their absence!)
 · Secondary: Document what the code should do
Lesson Learned: Always Test Software
Before Deployment
  Definition of Software Testing
Executing the program with selected inputs in a controlled
environment while checking assertions on its behavior & outputs
my_domain_class.py                                 my_test.py
def calculate_price(price, discount_percent):      def test_50_discount():
return price - discount_percent                    price = calculate_price(100, 50)
                                                   assert price == 50
def calculate_price(price, discount_percent):
return price - discount_percent * price/100        def test_discount():
           assert fails if the condition is false  price = calculate_price(20, 50)
                                                   assert price == 10
 Terminal  Pytest is a common testing framework for...Python
           (calls all functions starting with "test" in .py files in the /tests folder of a Python project)
$pytest
tests/test_main.py .F.....F...                                                                               [100%]
=================================== 2 failed, 9 passed in 0.03s ===================================
           CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring                               11
           Tobias Dürschmid
Tests can be Automated in your Continuous
Integration Workflow
· Continuous Integration (CI) is a software            Common CI Tools
  development practice in which changes are
  frequently merged into the main branch             · Jenkins
                                                     · Travis CI
· Tool support for this allows you to automatically  · GitHub Actions
  run tests in the CI environment (on a server that
  has your code) to see if a change broke the build
· Convention: Only push changes that should
  pass the tests
· 31.5% of all NPM repositories on GitHub show
badges for their build status [2]
[2] A. Trockman et al. "Adding Sparkle to Social Coding: An Empirical Study of Repository Badges in the npm Ecosystem". ICSE 2018
Not All V&V (Verification & Validation)
Can be Automated
  Verification                    Validation
Did we build the system right?  Did we build the right system?
Can often be automated          Must be done manually
                                    (by someone who
                                 represents real users)
Traditionally Testing
Has Happened After Development
Requirements Analysis What should be built?
Design            How should we build it?
                  Development Build it
                  Testing                                            Did we build the
                                                                     right system?
                  Deployment                                         Deliver to
                                                                     customer
Test-Driven Development (TDD)                                       Red            Green
Puts Testing First
Originally proposed by Kent Beck                                         Refactor
(who later went on to work for Facebook/Meta)
 Red      For your new requirement write a small test that fails,
Green     and perhaps doesn't even compile at first
          Make the test pass with minimal coding effort,
          potentially using simplifying shortcuts in the process
Refactor  Make the design more elegant, cleaner, and potentially
          faster while not changing the functionality of the program
More on this in "Test Driven Development: By Example" by Kent Beck
Refactoring Turns Code Smells into Clean Code
· Motivation: "When a software system evolves, its complexity increases unless
  work is done to reduce it" (Second Law of Software Evolution [1])
· Refactoring reduces accidental complexity in software systems and/or makes
  them easier to read / understand
· Code is read more often than it is written, so make sure your code is readable
· Common themes in refactoring:
· Avoid code duplication by creating abstractions
· Remove dead code & improve documentation
· Replace conditional logic with polymorphism
· Improve naming of classes, functions, & variables
· Learn common refactoring techniques here: https://refactoring.guru/refactoring/catalog
[1] M. M. Lehman et al. "Metrics and laws of software evolution-the nineties view," Proc. International Software Metrics Symposium 1997
                         Red Implement Green
TDD Mantra               Light Feature                               Light
             Add a Test  Add                                         Improve
                         Test                                            Code
Green Light
              Run Tests
         Red Light
            Make Small
               Change
                               Refactor
TDD Example: A Simple Stack Implementation
                     3
2 push(3)            2  pop()                                           2
1                    1                                                  1
   A stack is a list data structure that operates on the
             Last-In, First-Out (LIFO) principle.
Walk through this at home: https://github.com/UCLA-CS-35L/TDD_stack_example
   CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring     18
   Tobias Dürschmid
Start by Writing a Failing Test  Red                                               Green
my_stack.py                    test_main.py                    Refactor
                               from src.my_stack.my_stack import MyStack
                               def test_create_stack():
                                  stack = MyStack()
             We want to see the test fail first! This step is "testing the test".
             Otherwise, we can't be sure that the test works.
Terminal     (maybe we accidentally referenced the stack class from a library?)
$pytest
tests/test_main.py:1: in <module>
       from src.my_stack.my_stack import MyStack
E ImportError: cannot import name 'MyStack' from 'src.my_stack.my_stack'
Did you mean: 'my_stack'?
===================================== short test summary info =====================================
ERROR tests/test_main.py
!!!!!!!!!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
======================================== 1 error in 0.04s =========================================
             CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring   19
             Tobias Dürschmid
Then Add the Simplest Code Red                                                 Green
 that Passes the Test                              test_main.py  Refactor
my_stack.py
class MyStack:                                     from src.my_stack.my_stack import MyStack
   pass Null operation in Python (to tell          def test_create_stack():
              the interpreter the class is empty)     stack = MyStack()
If we had written more code than necessary to pass the test,
we would not know whether the additional code is
correct, since we don't have a test for it.
Terminal
$pytest
======================================= test session starts =======================================
tests/test_main.py .                                                           [100%]
======================================== 1 passed in 0.00s ========================================
          CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  20
          Tobias Dürschmid
Improve the Quality of your Red                                                Green
 Code via Refactoring                    test_main.py        Refactor
my_stack.py
class MyStack:                           from src.my_stack.my_stack import MyStack
"""
A simple implementation of a Stack data  def test_create_stack():
structure.                                  stack = MyStack()
A Stack operates on the Last-In, First-
Out (LIFO) principle.                     Comments should describe only a fixed
"""                                       contract of what this class does across future
                                          versions (and ideally also why)
   pass
          Tests give us the confidence that our refactoring
Terminal  didn't break anything. Everything still works!
$pytest
======================================= test session starts =======================================
tests/test_main.py .                                                           [100%]
======================================== 1 passed in 0.00s ========================================
          CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  21
          Tobias Dürschmid
  Add Another Test                       Red                                      Green
  for New Functionality
                                         test_main.py            Refactor
 my_stack.py
class MyStack:                           from src.my_stack.my_stack import MyStack
"""
A simple implementation of a Stack data  def test_create_stack():
structure.                                  stack = MyStack()
A Stack operates on the Last-In, First-     assert stack.is_empty()
Out (LIFO) principle.
"""                                      Writing the code using the method before its
                                         implementation leads to more modular design
   pass
   Terminal
________________________________________ test_create_stack ________________________________________
def test_create_stack():
   stack = MyStack()
>  assert stack.is_empty()
             ^^^^^^^^^^^^^^
E  AttributeError: 'MyStack' object has no attribute 'is_empty'
======================================== 1 failed in 0.02s ========================================
             CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  22
             Tobias Dürschmid
Then Add the Simplest Code Red                                                 Green
 that Passes the Tests                   test_main.py  Refactor
my_stack.py
class MyStack:                           from src.my_stack.my_stack import MyStack
"""
A simple implementation of a Stack data  def test_create_stack():
structure.                                  stack = MyStack()
A Stack operates on the Last-In, First-     assert stack.is_empty()
Out (LIFO) principle.
"""
def is_empty(self):    This is the simplest implementation to pass the test.
   return True         We have not tested other non-empty stacks yet!
                       Hence, we will only implement it when we have a test for this.
Terminal
$pytest
======================================= test session starts =======================================
tests/test_main.py .                                                                   [100%]
======================================== 1 passed in 0.00s ========================================
          CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring          23
          Tobias Dürschmid
Improve the Quality of your Red                                                Green
 Code via Refactoring                         test_main.py  Refactor
my_stack.py
class MyStack:                                from src.my_stack.my_stack import MyStack
   def is_empty(self):
       """                                    def test_create_stack():
       Checks whether the stack is currently     stack = MyStack()
       empty.                                    assert stack.is_empty()
Returns:
bool: True if the stack contains no
   elements, False otherwise.                 Our code is still very clean, the only thing we can
"""                                           do is documenting the contract of this method
return True
Terminal
$pytest
======================================= test session starts =======================================
tests/test_main.py .                                                           [100%]
======================================== 1 passed in 0.00s ========================================
          CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  24
          Tobias Dürschmid
  Add Another Test                                 Red                                                                        Green
  for New Functionality
                                                   test_main.py  Refactor
 my_stack.py
class MyStack:                                     from src.my_stack.my_stack import MyStack
   def is_empty(self):                             def test_create_stack():
       """ ... """                                    stack = MyStack()
       return True                                    assert stack.is_empty()
             We try to write simple, small tests,  def test_add_element():
                                                      stack = MyStack()
             ideally the simplest test that fails  stack.add_element(3)
   Terminal                                        assert not stack.is_empty()
$pytest
tests/test_main.py .F                                                                                                         [100%]
________________________________________ test_add_element ________________________________________
def test_add_element():
         stack = MyStack()
>        stack.add_element(3)
         ^^^^^^^^^^^^^^^^^
E        AttributeErCrSor3:5L'SMoyfStwtaacrek'Coonbsjtreuccttiohna:sLencoturaet9tr­iTbeusttieng',aTdDdD_,eRleefmaecntotr'ing  25
===================T=o=b=ia=s=D==ür=s=c=h=m=id==== 1 failed, 1 passed in 0.02s ===================================
Now we Actually Need to           Red                                                Green
Think and Add Lots of Code                      Refactor
my_stack.py                       test_main.py
class MyStack:                    from src.my_stack.my_stack import MyStack
def __init__(self):
   self.elements = []             def test_create_stack():
def is_empty(self):                  stack = MyStack()
                                     assert stack.is_empty()
""" ... """                       def test_add_element():
return len(self.elements) == 0       stack = MyStack()
def add_element(self, element):   stack.add_element(3)
self.elements.append(element)     assert not stack.is_empty()
Terminal
$pytest
======================================= test session starts =======================================
tests/test_main.py ..                                                                [100%]
======================================== 2 passed in 0.00s ========================================
                CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  26
                Tobias Dürschmid
                                                      Red                            Green
Choose Better Method Names
my_stack.py                       test_main.py                    Refactor
class MyStack:                    from src.my_stack.my_stack import MyStack
def __init__(self):
self.elements = []                def test_create_stack():
                                                   stack = MyStack()
def is_empty(self):                                assert stack.is_empty()
""" ... """
return len(self.elements) == 0    def test_push():
                                                   stack = MyStack()
def push(self, element):                           stack.push(3)
self.elements.append(element)                      assert not stack.is_empty()
 Terminal       Thanks to our tests we know this
                refactoring didn't break anything
$pytest
======================================= test session starts =======================================
tests/test_main.py ..                                                                [100%]
======================================== 2 passed in 0.00s ========================================
                CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  27
                Tobias Dürschmid
  Add Another Test                          Red                                                                        Green
  for New Functionality
                                       test_main.py        Refactor
 my_stack.py
class MyStack:                         from src.my_stack.my_stack import MyStack
   def __init__(self):                 ...
       self.elements = []
                                       def test_pop():
   def is_empty(self):
       """ ... """                          stack = MyStack()
       return len(self.elements) == 0
                                            stack.push(3)
   def push(self, element):
       self.elements.append(element)        element = stack.pop()
                                            assert element == 3
                                            assert stack.is_empty()
   Terminal
$pytest
tests/test_main.py .F                                                                                                  [100%]
________________________________________ test_add_element ________________________________________
def test_pop():
>        element = stack.pop()
                       ^^^^^^
         AttributeErCrSor3:5L'SMoyfStwtaacrek'Coonbsjtreuccttiohna:sLencoturaet9tr­iTbeusttieng',pToDpD', Refactoring
E                                                                                                                      28
===================T=o=b=ia=s=D==ür=s=c=h=m=id==== 1 failed, 2 passed in 0.02s ===================================
   Then Add the Simplest Code Red                                            Green
    that Passes the Tests                test_main.py  Refactor
   my_stack.py
class MyStack:                           from src.my_stack.my_stack import MyStack
   def __init__(self):                   ...
       self.elements = []                def test_pop():
...                                         stack = MyStack()
   def push(self, element):                 stack.push(3)
      self.elements.append(element)      element = stack.pop()
                                         assert element == 3
   def pop(self):                        assert stack.is_empty()
      element = self.elements[-1]
      return element
   Terminal            Thanks to our tests we know that we made a mistake
$pytest                in the code. We didn't change the size of the stack!  [100%]
tests/test_main.py .F
________________________________________ test_add_element ________________________________________
>  assert stack.is_empty()
E  assert False
E            + where False = is_empty()
                whereCSis3_5eLmSpotfytw=are<sCrocn.smtryu_cstitoanc:kL.emcytu_rset9ac­kT.eMsytiSntga,cTkDDo,bRjeefcatctoarting0x10aa30690>.is_empty 29
E            +
===================T=o=b=ia=s=D==ür=s=c=h=m=id==== 1 failed, 2 passed in 0.02s ===================================
     Fix the Code Until All                                     Red                       Green
Tests Pass                                test_main.py               Refactor
my_stack.py
class MyStack:                            from src.my_stack.my_stack import MyStack
     def __init__(self):                  ...
     self.elements = []                   def test_pop():
...                                            stack = MyStack()
     def push(self, element):                  stack.push(3)
     self.elements.append(element)             element = stack.pop()
                                               assert element == 3
     def pop(self):                            assert stack.is_empty()
     element = self.elements[-1]
     self.elements = self.elements[0:-1]
     return element
                          Now we have confidence that our
 Terminal                 stack pops correctly for one element
$pytest
======================================= test session starts =======================================
tests/test_main.py ..                                                                     [100%]
======================================== 3 passed in 0.02s ========================================
                     CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  30
                     Tobias Dürschmid
     Does the Stack work correctly Red                                                    Green
     with Multiple Elements too?                         Refactor
my_stack.py                               test_main.py
class MyStack:                            def test_multiple_pop():
     def __init__(self):                  stack = MyStack()
     self.elements = []                   stack.push(1)
...                                       stack.push(2)
     def push(self, element):             stack.push(3)
     self.elements.append(element)        element = stack.pop()
                                          assert element == 3
     def pop(self):                       element = stack.pop()
     element = self.elements[-1]          assert element == 2
     self.elements = self.elements[0:-1]  element = stack.pop()
     return element                       assert element == 1
                     Already passes Yay!  assert stack.is_empty()
Terminal
$pytest
======================================= test session starts =======================================
tests/test_main.py ..                                                                     [100%]
======================================== 4 passed in 0.02s ========================================
                     CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  31
                     Tobias Dürschmid
Is there any Redundant Code Red                                                Green
 in the Tests?                       test_main.py   Refactor
my_stack.py
class MyStack:                       def test_create_stack():
   def __init__(self):                  stack = MyStack()
       self.elements = []               assert stack.is_empty()
...                                  def test_push():
   def push(self, element):
   self.elements.append(element)     stack = MyStack()
def pop(self):                       stack.push(3)
                                     assert not stack.is_empty()
element = self.elements[-1]          def test_multiple_pop():
self.elements = self.elements[0:-1]     stack = MyStack()
return element
                                     stack.push(1)
Terminal                             ...
$pytest
======================================= test session starts =======================================
tests/test_main.py ..                                                          [100%]
======================================== 4 passed in 0.02s ========================================
          CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring  32
          Tobias Dürschmid
Fixtures are Objects Created Red                                                                                        Green
Freshly for each Test Case                                                                          Refactor
Fmixytu_resstaacrek.epspyecially useful for complex setup / teardown               test_main.py
cplraocsesduMreysS, tsuacchka:s setting up a test database
                                                                                   @pytest.fixture
@pdyetfest_._fiinxitutr_e_(self):                                                  def stack(): Defines stack fixture
 def sdbe_lcfo.nenlecetmieonnt(s):= []                                              Setup Phase: Code run before each test
... # 1. Setup: Connect to the test database                                       stack = MyStack()
                                                                                   yield stack Runs the test with fixture
   dddebbf.a=dpddu_bsd_hla(itbasr(eaTlErfSy,T._cDeoAnlnTeeAmc)etn(tTE)S:T_DB_URL)
       self.elements.append(element)                                                   Teardown Phase: Code run after each test
# 2. Yield: Forward connection object to test                                      def test_create_stack(stack):
dyeifeldpocpo(nsnelf):                                                             assert stack.is_empty() Uses stack fixture
      element = self.elements[-1]                                                  def test_push(stack):
   # 3s. eTelafr.doewlne:mCelnostesth=e csoenlnefc.tieolnements[0:-1]                 stack.push(3) Uses stack fixture
   ddbbr..rcelotlouslrben(a)cke(l)ement                                               assert not stack.is_empty()
RTeeardmmionraelhere: https://docs.pytest.org/en/stable/how-to/fixtures.html
$pytest
======================================= test session starts =======================================
tests/test_main.py ..                                                                                                   [100%]
======================================== 4 passed in 0.02s ========================================
                        CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring                                      33
                        Tobias Dürschmid
TDD Is Not Just About Finding Bugs
  Tests are living documentation
Tests describe what the code should do and are always up-to-date.
  TDD often leads to better design
Writing the test first makes you think about how to use a new method /
class first, which often leads to a more modular design.
  TDD can increase productivity
TDD mantra keeps you focused on the current task.
Read more in I. Karac & B. Tunhan "What Do We (Really) Know about Test-Driven Development?" IEEE Software 2018
Rules of Test-Driven Development
(According to "Uncle Bob" Robert C. Martin)
1) Do not write a line of production code until you have a failing unit test
2) Do not write more of a unit test than is sufficient for the test to fail
    (You cannot write a lot of unit test code before you switch back to writing
    production code)
3) Do not write more production code than is sufficient to pass the tests
 You will end up writing unit tests & production code somewhat concurrently
                  This is NOT a dogma.
Every rule in software engineering has its exceptions.
Once you have mastered TDD, you can adjust it to fit your coding style & project.
However, it's hard to master TDD without first practicing it based on these rules.
Limitations of TDD
TDD does not work well for:
· extremely complex behavior where the solution to your
problem is not an incremental improvement to simpler
problems (e.g. implementing ACID-compliant DBMS, distributed
databases, )
· systems with non-binary success outcomes
(e.g., image recognition, image processing)
· testing non-functional properties                                  You'll never
(e.g., response time, availability, ...)     get to the moon simply by
                                          building taller and taller towers.
Unit Testing Is not Enough
· Components that work well in isolation
  might break once they are composed
  together
· Integration tests test the interactions between
  two components
//DO NOT USE IN PRODUCTION!!!!
if (configurationFlag("PowerPeg")) {
     buyHigh();                 Works fine in
     sellLow(); //loses money   isolation but not
}                               together with
//some new code for             the other code
//the new feature used the same name
activateConfigurationFlag("PowerPeg");
Levels of Software Tests
Tests whether the complete               System Testing
application from user input to           (End-to-End Testing)
output works correctly.
                                         Integration Testing
Tests whether different components,
modules, or services work together
correctly.
Tests individually testable units of an  Unit Testing
application (e.g., functions, classes,
components) in isolation.
Cucumber Lets you Write Executable System Tests
in Gherkin (User Story Acceptance Criteria)
Feature: Withdrawing cash
Rule: Customers cannot withdraw more than their balance
Scenario: Successful withdrawal within balance                            Can be
   Given Alice has 234.56 in their account                           automatically
   When Alice tries to withdraw 200.00                               translated into
   Then the withdrawal is successful
                                                                       executable
Scenario: Declined withdrawal in excess of balance                     tests using
   Given Hamza has 198.76 in their account
   When Hamza tries to withdraw 200.00                                    natural
   Then the withdrawal is declined                                      language
                                                                       processing
See https://cucumber.io/
End-to-end Web Testing Frameworks
await page.goto('https://localhost:8080/');
    Page Navigation
await page.getByRole('textbox').fill('example value');
                                   Entering Text
const submitLink = page.getByRole('link', { name:
'Submit' });
await submitLink.click();
                  Clicking a Link
See more details here: https://playwright.dev/docs/writing-tests
End-to-end Web Testing Frameworks
await expect(page.getByText('Welcome')).toBeVisible();
// At least one of the two elements                               Assertion for
is visible, possibly both.                                           Visibility
await expect(
page.getByRole('button', { name: 'Sign in' })
.or(page.getByRole('button', { name: 'Sign up' }))
.first()                                                          Assertion for Text
).toBeVisible();
const locator = page.locator('.title');                              Content
await expect(locator).toContainText('substring');
await expect(locator).toContainText(/\d messages/);
See more details here: https://playwright.dev/docs/writing-tests
How to Write Good Tests?
· Select Inputs that test the boundaries & equivalence classes
  (e.g., positive numbers, negative numbers, zero, very large, very small)
· Try to cover common edge cases, not just the "happy path"
· Write strong assertions! Your assertion should test ALL aspects of the
  specification (remember the acceptance criteria from your user stories? If you've
  written good ones, this part is easier)
· Test behavior, not implementation details!
   · Your test should pass any correct implementation. Write it as a specification of
     the behavior, not the design decisions details, because those might change!
Learn this based on concrete Python examples:
https://tobiasduerschmid.github.io/SEBook/tools/testing-foundations-tutorial
Indirect Inputs & Indirect Outputs
Can be Ordered in Many Different Sequences
Indirect inputs are values returned by another component whose services the SUT uses
Indirect outputs are actions that cannot be observed through the public API of the SUT
which are seen or experienced by other systems or application components
              Direct Inputs          System Indirect Outputs Depended-on
        Test  Direct Outputs         Under Test  Indirect Inputs                   Component
                                        (SUT)                                         (DOC)
         SUT                    DOC              SUT                               DOC
Inputs                                                Indirect Inputs
                                                      Indirect Outputs
              Indirect Outputs
Outputs       Indirect Inputs        Inputs
                                     Outputs
              CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring
              Tobias Dürschmid
Mock Object Pattern                                                    Different example: Testing that your app
                                                                         sends the right requests to an LLM-
                                                                                   based AI provider API
Problem: How to observe indirect outputs sent                            How to test that the trading system
to separate DOCs?                                                      sends well-formed buy / sell orders
                                                                       to the stock exchange without sending
Context: The connector between SUT and DOC                              them to the actual stock exchange?
cannot easily be intercepted.
Solution: Create a Mock Object that replaces                           Trading                     Stock
the DOC and only verifies the indirect outputs                         System                   Exchange
Direct Inputs  System                   Indirect Outputs  Depended-on                           Mock Object
Test Direct Outputs Under Test                            Component              Indirect        Verification
                                 (SUT)                                                            assert(...)
                                        Indirect Inputs   (DOC)        SUT Outputs
More Details here: http://xunitpatterns.com/Mock%20Object.html
                           CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring
                           Tobias Dürschmid
Test Spy Pattern                                                          Different example: Testing that your system
                                                                           sends the requests in the correct order to
                                                                                   an LLM provider via their API
Problem: How to observe indirect outputs sent to                          How to test that the trading system
separate DOCs?                                                            sends buy / sell orders to the stock
Context: The connector between SUT and DOC                                 exchange in the correct order?
cannot easily be intercepted.
Solution: Create a Test Spy component that                                Trading      Stock
replaces the DOC and forwards the indirect                                System    Exchange
outputs to the test.
Direct Inputs  System                   Indirect Outputs     Depended-on         Indirect Test Spy
Test Direct Outputs Under Test                               Component    SUT    Outputs Data
                                 (SUT)
                                        Indirect Inputs      (DOC)         Test                  Collection
                                                                                            Forwarding
More Details here: http://xunitpatterns.com/Test%20Spy.html
               CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring
               Tobias Dürschmid
Test Stub Pattern                                                        Different example: Testing that your system
                                                                          behaves correctly when receiving various
                                                                             responses from the LLM provider API
Problem: How to control indirect inputs sent from                          How to test that the trading system
separate DOCs?                                                            responds correctly to updates on
                                                                          stock prices in the stock exchange?
Context: The connector between SUT and DOC
cannot easily be intercepted.                                                  Trading      Stock
                                                                               System    Exchange
Solution: Create a Test Stub component that
replaces the DOC and sends the desired inputs to
the SUT.
Direct Inputs  System                   Indirect Outputs                       Indirect  Test Stub
                                                                               Outputs
                                                             Depended-on                 Return
                                                                               Indirect  Values
Test Direct Outputs Under Test                                Component   SUT  Inputs
                                 (SUT)
                                        Indirect Inputs       (DOC)
More Details here: http://xunitpatterns.com/Test%20Stub.html
               CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring
               Tobias Dürschmid
Test Spies, Mock Components, and Test Stubs are all
unified under the term Test Doubles
                                  Test Double
Test Spy  Mock Object  Test Stub
In Node.js HTTP test doubles can implemented using this library:
https://www.npmjs.com/package/node-mocks-http
                   CS 35L Software Construction: Lecture 9 ­ Testing, TDD, Refactoring
                   Tobias Dürschmid
A Software Tester
WALKS into a bar
The System
Correctly
Handles a
Standard
Request
A Software Tester
RUNS into a bar
The System
Correctly
Handles a
Non-Standard
Request
A Software Tester
CRAWLS into a bar
The System
Correctly
Handles a
Non-Standard
Request
A Software Tester
DANCES into a bar
The System
Correctly
Handles a
Non-Standard
Request
A Software Tester
FLIES into a bar
The System
Correctly
Handles a
Non-Standard
Request
A Software Tester  One Cup of
Orders ...         Tea Please
The System
Correctly
Handles a
Standard
Request
A Software Tester  Two Cups of
Orders ...          Tea Please
The System
Correctly
Handles a
Standard
Request
A Software Tester  Zero Cups of
Orders ...          Tea Please
The System
Correctly
Handles a
Non-Standard
Request
A Software Tester  9999999 Cups of
Orders ...             Tea Please
The System
Correctly
Handles a
Very Large
Request
A Software Tester     -1 Cups of
Orders ...           Tea Please
The System         Invalid
Correctly           Input
Handles an
Invalid
Request
A Software Tester  "qwertyuiop"
Orders ...            Cups of
The System          Tea Please
Correctly
Handles an          Invalid
Invalid              Input
Request
A Software Tester     A Lizard in
Orders ...             a Teacup
The System              Please
Correctly
Handles             Resource
Errors             Unavailable
Let's Ship
The Product!
Testing
Complete!
A Real Customer       Excuse Me
Walks into the Bar  Where are the
                     Bathrooms?
System
Deployed
In the Real
World
A Real Customer       Excuse Me
Walks into the Bar  Where are the
                     Bathrooms?
The Developers
And Testers
Didn't Expect
This Request
Tutorials
· https://tobiasduerschmid.github.io/SEBook/tools/testing-foundations-tutorial
· https://tobiasduerschmid.github.io/SEBook/tools/tdd-tutorial
· Test doubles: Soon-to-come
Please fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three key insights you learned about Testing,
TDD, or Refactoring today. (Do not copy phrases from the lecture slides)
Besides the examples from the lecture, please describe one other concrete scenario in
which applying TDD is a good idea and one other concrete scenario in which applying
TDD is a bad idea. (Do not use examples from the lecture)
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slides use images from Flaticon.com (Creators: Freepik)

```
