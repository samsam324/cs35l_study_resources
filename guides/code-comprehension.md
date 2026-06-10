# Code Comprehension

Source: CS 35L Lecture 17 â€“ Code Comprehension & API Design (Tobias DÃ¼rschmid). This guide covers ONLY the Code Comprehension half of L17. The API Design / Interoperability half is documented in `api-design-and-interoperability.md`.

---

## 1. Why Comprehension Skills Matter

- Developers spend **58% of their time** on **understanding code**.
- A speedup in code understanding has a **large impact on productivity**.
- With AI writing more and more code, developers might spend **even more time on reading code in the near future**.

Implication: Reading code, not writing it, is the dominant developer activity. Any improvement in comprehension speed compounds across the entire workday. As AI-generated code proliferates, the human role tilts further toward reading, reviewing, and integrating code rather than producing it from scratch.

---

## 2. Code Comprehension Approaches

There are two foundational strategies for reading code. Novices and experts tend to use different approaches, and the goal is to **move from "Bottom-Up" toward "Top-Down"** as expertise grows.

### 2.1 "Bottom-Up" (Novices)

- Reading code **line by line**.
- **Grouping** statements into logical "chunks" of higher abstraction.
- Takes **more time** because **each statement needs to be analyzed**.

Bottom-up reading is the default fall-back when a reader lacks the domain knowledge required to form higher-level expectations. The reader essentially reconstructs meaning by composing the meaning of individual statements upward into larger conceptual units.

### 2.2 "Top-Down" (Experts)

- Driven by a **hypothesis** of what the code does.
- Looking for **"beacons"** â€” **key lines of code** that **reveal the core purpose** of a function or **test the hypothesis**.
- First getting the **big picture**, then **looking for details**.

Top-down reading starts from a high-level guess about purpose (informed by names, context, documentation, domain familiarity) and then dives into the source only to confirm or refute that guess. Experts do not read every line â€” they skim for confirmation signals.

### 2.3 The Direction of Growth

**Move from here to there**: the explicit pedagogical message is to migrate your reading style from Bottom-Up toward Top-Down as you gain experience. The remainder of this guide gives concrete tips for doing exactly that.

---

## 3. Tips to Transition from Bottom-Up to Top-Down Code Reading

These tips form a cluster of techniques. Each one increases the reader's ability to skip line-by-line analysis in favor of hypothesis-driven scanning.

### 3.1 Build Domain Knowledge

- The top-down approach is **only possible when you are familiar with the application domain**.
- Without domain knowledge, programmers **lack the necessary schemas to form expectations** and are forced to fall back on bottom-up chunking.
- **Tip**: Before looking at the source code, invest time in **reading requirements, architectural documents, and high-level system summaries**.
- Understanding *what* the software is supposed to do in the real world provides the foundation for your top-down expectations.

Why this matters: schemas (mental templates) are what allow experts to predict the structure of unfamiliar code. Without domain context, no hypothesis can be generated, so beacons cannot be recognized, and you collapse into bottom-up chunking by default.

### 3.2 Practice Hypothesis-Driven Reading

- Instead of tracing execution flows immediately, **practice generating an initial, general hypothesis about what a program or module does** based on its name, context, or documentation.
- **Tip**: Once you have a primary hypothesis (e.g., **"This program produces invoices"**), **subdivide it into finer subsidiary hypotheses** (e.g., **"It must have a data retrieval method and a formatting method"**) before looking at the code.
- You then **only read the code to validate or reject these specific expectations**.

The discipline here is to think *before* you read. The hypothesis tree (primary hypothesis -> subsidiary hypotheses) gives you targeted questions; you read the source to answer those questions rather than to discover from scratch.

### 3.3 Hunt for "Beacons"

- A top-down reader **does not read every line**; they **scan for "beacons"**.
- Beacons are **stereotypical segments of code, recognizable variable names, or distinct structures** that reliably indicate the presence of a specific operation or algorithm.
- At its core, a beacon is a **recognizable, familiar point in the source code that serves as a mental shortcut for the programmer**.
- Defined as **"signs standing close to human thinking that may give a hint for the programmer about the purpose of the examined code"**.
- **Tip**: Train yourself to **scan files for recognizable patterns, method signatures, and comments**. When you spot a beacon, use it to **instantly confirm your hypothesis without needing to trace the granular logic**.

Beacons let you replace painstaking line-level analysis with pattern recognition. The next section catalogues the canonical types of beacons.

---

## 4. Beacons â€” The Structural Signs Guiding You Through the Code

There are five categories of beacons covered in the lecture: **Identifier Names, Code Structures, Tests, Assertions, and Diagrams**.

### 4.1 Identifier Names Are Beacons

- The **most frequent and most critical beacons** are the **names developers assign to variables, functions, and classes**.
- Comprehension often depends on the **domain information carried by identifier names**.
- **Tip**: First look for the names of important classes, functions, and variables. If you do not know their meaning (e.g., due to lack of domain knowledge) **look up their meaning first!**

#### Code Examples â€” Spot the Domain in the Names

The lecture presents three side-by-side snippets and asks: *What domain does each snippet operate on? Which identifiers told you?*

**Python â€” Finance / Loan domain (APR = Annual Percentage Rate)**

```python
def calculate_apr(
    principal,
    interest_rate,
    term_months):
    monthly = interest_rate/12
    n = term_months
    return (
        (1+monthly)**n - 1
    ) * 100
```

Identifiers that reveal the domain: `calculate_apr`, `principal`, `interest_rate`, `term_months`, `monthly`.

**React (JSX) â€” E-commerce / Shopping cart domain**

```jsx
function CartCheckout({
    items, taxRate,
    onPlaceOrder }) {
    const subtotal = items
        .reduce((s,i) =>
            s + i.price, 0);
    return <button onClick=
        {onPlaceOrder}>
        Pay {subtotal*taxRate}
    </button>; }
```

Identifiers that reveal the domain: `CartCheckout`, `items`, `taxRate`, `onPlaceOrder`, `subtotal`, `i.price`, `Pay`.

**Node.js (Express) â€” Authentication / Web-security domain**

```js
function authenticate(
    req, res, next) {
    const token = req
        .headers.authorization;
    if (!verifyJwt(token))
        return res.status(401)
            .send('Unauthorized');
    next();
}
```

Identifiers that reveal the domain: `authenticate`, `token`, `headers.authorization`, `verifyJwt`, `401`, `'Unauthorized'`.

**Exercise**: *What domain does each snippet operate on? Which identifiers told you?*

#### Think-Pair-Share

The lecture explicitly poses: **"What are other beacons besides identifier names?"** â€” leading into the rest of section 4.

### 4.2 Code Structures Are Beacons

- **Physical layout** of statements â€” often referred to as ***text-structure knowledge*** â€” serves as a **visual beacon**.
- **Loops** help you identify **structural patterns**.
- **Guard clauses** help you identify **early exit paths**.
- **Tip**: Look for the **algorithms** used to solve the problem.

#### Code Examples â€” Recognize the Algorithm by Its Shape

*Predict â€” what does each snippet compute? Which structural shape (loop, recursion, guard) gave it away?*

**Python â€” Tree depth (guard clause + recursive shape)**

```python
def max_depth(node):
    # guard clause
    if node is None:
        return 0
    # recursive shape
    return 1 + max(
        max_depth(node.left),
        max_depth(node.right)
    )
```

Structural shapes: **guard clause** (`if node is None: return 0`) and **recursive shape** (two recursive calls combined with `max`).

**React (JSX) â€” Retry loop**

```jsx
useEffect(() => {
    let attempts = 0;
    // retry loop
    while (attempts < 3) {
        try {
            return fetchData();
        } catch {
            attempts++;
        } }
}, [url]);
```

Structural shape: **retry loop** â€” bounded `while` with try/catch increment in the catch.

**Node.js â€” Guard + map/reduce**

```js
app.post('/orders', (req,res) => {
    // guard
    if (!req.user)
        return res.status(401).end();
    // map/reduce shape
    const total = req.body.items
        .map(i => i.price * i.qty)
        .reduce((a,b) => a+b, 0);
    res.json({ total });
});
```

Structural shapes: **guard** (auth check at top) and **map/reduce shape** (compute totals).

**Exercise**: *Predict â€” what does each snippet compute? Which structural shape (loop, recursion, guard) gave it away?*

### 4.3 Tests Are Beacons

- When reading unfamiliar code, a developer's primary challenge is **deducing the original author's intent**.
- Tests act as **explicit beacons** that illuminate this intent by providing **an executable, unambiguous specification** of how the production code *should* work.
- **Tip**: When reviewing code, **look at the tests first**. **Verify that the tests pass** on the current code before spending too much time reading them.

#### Code Examples â€” Tests Spell Out the Spec

*From the tests alone, describe a one-sentence spec for each function.*

**Python (pytest) â€” `apply` discount function**

```python
def test_no_discount():
    assert apply(100, 0) == 100

def test_half_off():
    assert apply(100, 50) == 50

def test_rejects_negative():
    with pytest.raises(
        ValueError):
        apply(100, -5)
```

Spec implied by tests: `apply(price, discount_percent)` returns the discounted price, treats `0` as no change, treats `50` as half off, and raises `ValueError` for negative discounts.

**React (Playwright) â€” Checkout button enablement**

```js
import { test, expect }
    from '@playwright/test';

test('checkout disables', async
    ({ page }) => {
    await page.goto('/cart');
    const btn = page.getByRole(
        'button', { name: /pay/i });
    await expect(btn).toBeDisabled();
    await page.getByLabel('Terms')
        .check();
    await expect(btn).toBeEnabled();
});
```

Spec implied by test: the Pay button is disabled until the user checks the Terms checkbox.

**Node.js (Jest + supertest) â€” Users endpoint**

```js
test('GET /users -> 200', async () => {
    const res = await request(app)
        .get('/users');
    expect(res.status).toBe(200);
    expect(res.body).toHaveLength(3);
});

test('unknown id -> 404', async () => {
    const res = await request(app)
        .get('/users/999');
    expect(res.status).toBe(404);
});
```

Spec implied by tests: `GET /users` returns 200 with a list of 3 users; `GET /users/<unknown_id>` returns 404.

**Exercise**: *From the tests alone, describe a one-sentence spec for each function.*

### 4.4 Assertions Are Beacons

- Zooming in from the file level to the statement level, the individual **assertions** within a test (or embedded within production code) act as **highly localized beacons**.
- **Check for pre- and post-conditions (and invariants)** specified in the code.
- **Tip**: **Add assertions in your own code** to make it easier for others to use top-down reading approaches.

#### Code Examples â€” Pre/Postconditions Pin Down Intent

*List an input that would crash each function. The assertions tell you â€” point to the line.*

**Python â€” `withdraw` with pre- and post-conditions**

```python
def withdraw(account, amount):
    # preconditions
    assert amount > 0
    assert account.balance >= amount

    account.balance -= amount

    # postcondition (invariant)
    assert account.balance >= 0
    return account.balance
```

Preconditions: `amount > 0`, `account.balance >= amount`. Postcondition / invariant: `account.balance >= 0`. Crashing input: `amount = 0` or negative, or `amount` greater than the balance.

**React â€” `useSlider` with invariants**

```jsx
function useSlider(min, max) {
    // precondition
    invariant(min < max);
    const [v, setV] = useState(min);
    const update = (n) => {
        invariant(
            n >= min && n <= max);
        setV(n);
    };
    return [v, update];
}
```

Precondition: `min < max`. Per-call invariant on `update`: `n >= min && n <= max`. Crashing input: instantiating with `min >= max`, or calling `update` with `n` outside `[min, max]`.

**Node.js â€” `transfer` with assertions**

```js
const assert = require('assert');
function transfer(from, to, amt) {
    // preconditions
    assert(from !== to);
    assert(amt > 0);
    assert(from.balance >= amt);
    from.balance -= amt;
    to.balance += amt;
    // postcondition
    assert(from.balance >= 0);
}
```

Preconditions: `from !== to`, `amt > 0`, `from.balance >= amt`. Postcondition: `from.balance >= 0`. Crashing inputs: `transfer(x, x, ...)`, `amt <= 0`, or `amt` greater than `from.balance`.

**Exercise**: *List an input that would crash each function. The assertions tell you â€” point to the line.*

### 4.5 Diagrams Are Beacons

- **UML Diagrams help build a top-down mental model of the system**.
- When reading a code base, **first look for diagrams**. Visual diagrams often help understanding important complex concepts more efficiently.
- **However**: Sometimes diagrams are **outdated**, which can lead to **incorrect hypotheses**.

The trade-off: diagrams compress structural information into something quickly digestible, but they are not automatically kept in sync with the code. Treat a diagram as a starting hypothesis to test against the source, not as a definitive specification.

---

## 5. Continuous Cycle of Hypothesis Testing

Top-down reading is not a one-shot guess â€” it is an iterative loop:

1. **Hypothesis Generation**: e.g., the developer assumes the system must have a **"database connection"** module.
2. **Beacon Hunting**: The developer **scans the code looking for beacons**, such as an **`SQL` library import**, a **`connectionString` variable**, or a **`db_connect()`** method.
3. **Verification or Rejection**:
   - If the anticipated beacons are **found**, the hypothesis is **verified and becomes a part of the mental model**.
   - If the beacons are **missing**, the hypothesis is **declined**, and the programmer **must adjust their assumptions**.

This loop runs continuously as the reader builds up an understanding of the codebase. Each verified hypothesis becomes part of the working mental model and seeds the next round of hypotheses (e.g., once "there is a DB module" is confirmed, the next hypothesis might be "queries are parameterized to prevent SQL injection").

---

## 6. Embrace "Opportunistic Processing"

- Experts **rarely use top-down processing in strict isolation**; instead, they act as **"opportunistic processors"** who **continuously toggle between top-down and bottom-up strategies**.
- **Tip**: Use **top-down methods** to quickly **navigate and build the big picture**. When you encounter a **highly complex, unfamiliar, or localized piece of logic** where your **hypotheses fail**, **seamlessly drop into a bottom-up approach** to understand that specific chunk, and then **immediately return to your top-down overview**.

The key insight: the two strategies are complementary, not competing. Top-down gives you global navigation; bottom-up handles the patches where the global model breaks down. Expert readers move fluidly between the two as the situation demands, treating bottom-up reading as a precision tool to use sparingly inside an overall top-down framework.

---

## Summary of the Tips Cluster (for Bottom-Up -> Top-Down Transition)

1. **Build Domain Knowledge** â€” read requirements, architecture docs, system summaries before the code.
2. **Practice Hypothesis-Driven Reading** â€” form a primary hypothesis, subdivide it, then read to validate/reject.
3. **Hunt for Beacons** â€” scan for the five beacon types (identifiers, code structures, tests, assertions, diagrams).
4. **Continuous Cycle of Hypothesis Testing** â€” Generate -> Beacon Hunt -> Verify or Reject -> adjust.
5. **Embrace Opportunistic Processing** â€” toggle between top-down and bottom-up as needed.


---

# Appendix: Lecture 17 Slides (raw extracted text)

The following is the full extracted text of Lecture 17. Preserved verbatim so no information is lost.

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
