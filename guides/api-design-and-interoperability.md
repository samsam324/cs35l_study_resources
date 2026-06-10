# API Design & Interoperability

Source: CS 35L Lecture 17 (Tobias D√ºrschmid) ‚Äî API Design & Interoperability portion.

---

## 1. Motivating Story: Bob Books a Flight

A narrative motivating the need for interoperability:

1. **Bob wants to book flights so that he can go on vacation.**
2. **Bob's flight has been cancelled.**
3. **The airline agent tries to find an alternative flight.**
4. **Bob has been re-booked onto a flight with a different airline.** THE END.

### What makes this story difficult to implement?

- Bob sees **flights from hundreds of different airlines**.
- Bob can get **re-booked to a different airline**.
- **There needs to be a system that can interoperate with all airlines and all booking systems.**

---

## 2. The Global Distribution System (GDS)

The **Global Distribution System (GDS)** lets almost all airlines & booking systems interoperate.

> *Historical note: the invention of GDS is closer historically to the invention of flight by the Wright brothers than to today.*

### GDS Architecture (diagram)

Booking systems on one side, airlines on the other side, with GDS in the middle:

```
Booking System 1 ‚îÄ‚îÄ‚îê                          ‚îå‚îÄ‚îÄ Airline 1
                   ‚îÇ   Request Flights        ‚îÇ   Register Flights
                   ‚îú‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚ñ∫ GDS ‚óÑ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚îÄ‚î§   Sell Flights
                   ‚îÇ   Book Flights           ‚îÇ
Booking System 2 ‚îÄ‚îÄ‚îò                          ‚îú‚îÄ‚îÄ Airline 2
                                              ‚îÇ   Manage Passengers
                                              ‚îÇ   Manage Baggage
                                              ‚îî‚îÄ‚îÄ Airline 3
```

- **Booking systems ‚Üí GDS:** Request Flights, Book Flights
- **GDS ‚Üí Airlines:** Register Flights, Sell Flights, Manage Passengers, Manage Baggage

---

## 3. What is Interoperability?

**Interoperability** is the ability of multiple systems / components to exchange information AND to use the exchanged information (i.e., both being able to talk and being able to mean the same thing).

Two flavors of interoperability:

- **Syntactic interoperability** ‚Äî the systems agree on the format of the exchanged data (parameters, types, structure).
- **Semantic interoperability** ‚Äî the systems agree on the meaning / interpretation of the exchanged data (units, vocabulary, side effects).

Both are needed; **syntactic interoperability alone is not enough**.

---

## 4. Design Principle for Interoperability: Create Shared Interfaces / Data Formats

To make a set of systems interoperate, follow these three steps:

1. **List all data** that needs to be exchanged.
2. **Define an interface / data format** that supports all data.
3. **Implement serialization & deserialization.**

### Two qualities the shared interface must have

- **Build Abstractions** ‚Äî The interface / document format should **not expose any implementation details** of systems / components.
- **Ensure Language- and Platform-Independence** ‚Äî The format should be supported by all programming languages, operating systems, and devices.

---

## 5. Common Technique to Implement Shared Interfaces: REST APIs

**REST = REpresentational State Transfer.**

- REST is a **stateless protocol** (no server-side session) to exchange data in client-server systems via HTTP / HTTPS.
- HTTP verbs map to CRUD:
  - **POST** ‚Äî Creates a Resource
  - **GET** ‚Äî Reads a Resource
  - **PUT** ‚Äî Updates a Resource
  - **DELETE** ‚Äî Deletes a Resource
- **Naming convention for URLs**, based on resource identifiers.
  - Use `/users/123` instead of e.g., many variations of `/getUser?id=123`.
- **Resources are often described via XML, YAML, JSON, or HTML.**

### Example REST API: GDS API

Request: `POST /getFlights`

```json
‚Ä¶
"travelPreferences": {
    "flightType": "Direct",
    "maximumStopsQuantity": 1
},
"itineraryParts": [ {
    "departureAirportCode": "LAX",
    "arrivalAirportCode": "PIT",
    ‚Ä¶
}]
‚Ä¶
```

Response ‚Äî code **200**: *Successful response or application layer errors held in the response.*

Example response value (model):

```
"locationCode": "DFW",
"arrivalDateTime": "2019-02-20T05:50:00",
"departureDateTime": "2019-02-20T05:50:00",
"elapsedTime": 40,
"duration": 40,
"gmtOffset": 11,
"equipmentType": "73H"
```

---

## 6. Alternative Techniques to Implement Shared Interfaces (besides REST)

| Technique | Description |
|---|---|
| **RPC (Remote Procedure Call)** | Calling a function on a remote server as if it were local. Can be **stateful**. **Functions** and **actions** beyond CRUD. Good for **complex calculations**. Can use multiple document formats. |
| **SOAP (Simple Object Access Protocol)** | Can be stateful. More **complex**. Sometimes **slower**. Has more **security** features. Has built-in **error handling**. Good for **distributed enterprise environments**. Uses XML. |
| **GraphQL** | **Server-side schema defines types**, enabling checking of data structure conformance. Good for **large**, **complex**, and **interrelated** data sources. Uses JSON. |

---

## 7. Common Technique to Test Compatibility: XML / JSON / YAML Schema

- A schema **describes the structure** of a document.
- Schemas list **attributes**, their **possible values**, and **complex types** (e.g., sequences, recursive types) for a document.
- **Validation** of a document against a schema can be done **automatically** to **test compatibility at run-time**.

### Example JSON Schema

```json
"properties": {
    "departureAirportCode": {
        "type": "string",                   // Constraints on the Type of a Property
        "pattern": "^[A-Z]{3}$"             // RegEx!
    },
    "price": {
        "type": "number",
        "minimum": 0,                       // Constraints on the Values of a Property
        "exclusiveMinimum": true
    }
},
"required": ["departureAirportCode", "price"]   // Constraints on Document Structure
```

Three categories of constraints captured by a schema:

- **Constraints on the Type of a Property** (e.g., `"type": "string"`, `"pattern": "^[A-Z]{3}$"`)
- **Constraints on the Values of a Property** (e.g., `"minimum": 0`, `"exclusiveMinimum": true`)
- **Constraints on Document Structure** (e.g., `"required": [...]`)

---

## 8. Case Study: Mars Climate Orbiter ‚Äî Spacecraft Lost Due to Lack of Semantic Interoperability

A $193 million spacecraft was lost due to **lack of Semantic Interoperability** (shared interpretation of shared data).

| Side | Who | Behavior |
|---|---|---|
| **Flight System Software** | Developed by NASA JPL | **Expected commands in N (SI units)** |
| **Ground Software** | Supplied by Lockheed Martin (US-based sub-contractor) | **Sent commands in lbf (US Customary units)** |

Both sides shared the same **Command Interface** (syntax matched), but they interpreted the numeric values differently ‚Äî one assumed Newtons, the other assumed pound-force. The spacecraft was lost.

**Question:** How could they have prevented this bug?

### Lesson Learned: Syntactic Interoperability is Not Enough

> Data exchanged between systems / components must be **interpreted in the same way by all systems / components**.

Illustration: one side renders `<plant>` as a factory (industrial plant), the other renders `<plant>` as a flower (vegetation). Same tag, different meaning ‚Üí no semantic interoperability.

---

## 9. Design Principle for Interoperability: Define the Semantics of Shared Data

Three concrete tactics:

- **Document Interfaces and their Semantics.** E.g.:
  - What units are implied?
  - Does price include tax?
  - MM/DD/YY or DD/MM/YY?
  - What coordinate system is used as a reference frame?
- **Use Shared Dictionary of Items** or **Agree on Vocabulary**. E.g.:
  - "doughnut" or "donut"?
  - Âçï‰∏õËå∂ or ÂçïÊûûËå∂?
- **Write Integration Tests for the Systems.**

---

## 10. Interface Descriptions: Syntactic vs Semantic Views

| View | What it describes |
|---|---|
| **Syntactic View** | Describes document format, the actions that can be performed, their parameters, and outputs. |
| **Semantic View** | Describes the **purpose / meaning** of the resource / action: <br>‚Ä¢ **Side-effects:** Changes to the state of a resource or environment <br>‚Ä¢ **Usage restrictions:** Who can perform this action? <br>‚Ä¢ **Error Handling:** What errors can occur and why? <br>‚Ä¢ **Examples:** Examples of outputs for a given input |

### Example: (Simplified) Syntactic View of the GDS Flight Booking API

```
POST /createBooking {
    ‚Ä¢ flightNumber: str          // The flight number using the code
                                 //   of the operating airline
    ‚Ä¢ date: dateTime             // The date of the departing flight according to
                                 //   the time zone of the departure airport
    ‚Ä¢ seat: str                  // Seat number (same label as on the plane)
    ‚Ä¢ paymentReference: PaymentMethod
    ‚Ä¢ travelerName: str          // Full name of the person traveling in ASCII
}
```

### Example: Semantic View of GDS Booking

- **Purpose:** The airline confirms a requested booking.
- **Side-effects:** money is charged, seat is marked as sold, can be canceled within 1 hour.
- **Usage restrictions:** Authorized booking systems.
- **Errors:** Invalid Format, Unauthorized, too many requests, ‚Ä¶

Reference: https://developer.sabre.com/docs/rest_apis/trip/orders/booking_management

---

## 11. Guidelines on Interface Documentation

- Focus on how **elements interact** and their **externally visible effects**, not their implementation.
- To support **changeability** of the implementation, expose only **what is needed** to use the interface.
- Keep the documentation **minimal** and **use-case oriented** to increase **readability**.

### Mini-case: GDS Interface Documentation Pros and Cons

Two examples from the GDS docs:

- **AirportCode** (good ‚Äî semantic interoperability via shared dictionary):
  ```
  AirportCode: string
  pattern: ^[A-Z]{3}$
  example: DFW
  ```
  Documentation: *"The three-letter IATA code of the airport."* ‚Äî references the shared IATA dictionary, so meaning is unambiguous.

- **OnTimePerformance.level** (bad ‚Äî semantics of "level" are unclear):
  ```
  level: string
  pattern: ^[0-9]{1}$
  example: 1
  ```
  Documentation: *"The numeric value (0‚Äì9) associated with the level of on time performance of the flight."* ‚Äî what does each level mean? Is 0 best or worst? **Unclear semantics.**

**Lesson:** semantic interoperability via shared dictionary works (e.g., IATA codes); ad-hoc numeric "levels" without a defined dictionary leave room for misinterpretation.

---

## 12. Why Airlines Are Moving Away from GDS (Motivation for Change)

- GDS only lets you **book seats**.
- Airlines make more money **up-selling "priority boarding"**, or other **add-ons**.
- **Southwest doesn't have assigned seating.**
- Some airlines **have seats that can be transformed** into a sky couch, resulting in their unavailability in GDS.
- Airlines do not get valuable **customer data** via GDS.

These limitations of the shared interface push airlines toward alternative systems ‚Üí motivates the need for **extensible / changeable** interfaces.

---

## 13. Extensible Interfaces ‚Äî Making GDS More Changeable

Design tactic: **Extensible Interfaces** ‚Äî let the interface carry dynamic, open-ended data so new use cases don't require a new interface version.

Example proposal: add dynamically-defined add-ons to GDS offers.

- Offers can contain a **dynamically-defined add-ons** list.
- Each **Add-on** is a tuple `(price, name, description, id)`:
  - **price (int):** The price in cents (excl. tax) additionally charged when this add-on is selected.
  - **name (str):** The name of the add-on as shown to the user (in UTF-8).
  - **description (str):** A short description shown to the user in order to decide if they want to purchase the add-on (in UTF-8).
  - **id (str):** Unique identifier of this add-on starting with the flight number (in ASCII).
- A list of add-ons is added to a flight listing. The booking API needs to add an optional list of add-on `id`s to identify requested add-ons.

Trade-off: **"Harder to Implement"** ‚Äî this is more flexible but the interface and both sides' implementations become more complex.

---

## 14. Lesson Learned: Interoperability Often Conflicts with Changeability

- Shared interfaces and data formats mean changes have to be implemented **in all** cooperating systems.
- We **cannot localize** the interface change to a single system, so extensions / changes require a new version of the interface, **which breaks interoperability**.

---

## 15. How to Evaluate Practical Interoperability

These **Design-Time Measures** evaluate how likely a design for interoperability is to succeed in a practical setting. They can be used in **quality attribute specifications**.

Two measures, **often in conflict**:

| Measure | Rationale |
|---|---|
| **Measure the Effort to Implement the Interface in all Systems / Components** | A **more complex** interface / data format is **less likely to be adopted** by many systems due to **higher implementation cost**. |
| **Measure the Variability Allowed by the Interface / Format** | An interface / data format that supports **more use cases** and potential **extensions** is **more likely to be adopted**. |

The tension: simpler interface ‚Üí easier to adopt but less flexible; richer interface ‚Üí more flexible but harder to adopt. Practical interoperability balances these.

---

## 16. Design Pattern for Interoperability: Use Adapters to Connect Interfaces

(Briefly ‚Äî fuller treatment is in the design-patterns guide.)

- **Problem:** Two systems use **different interfaces** (e.g., different data formats, different protocols, ‚Ä¶).

  ```
  Your System ‚îÄ‚ñ∫ XML  ‚ö°  JSON ‚óÑ‚îÄ Their System
  ```

- **Solution:** Create an **adapter component** that **translates** between the two interfaces.

  ```
  Your System ‚îÄ‚ñ∫ Adapter ‚óÑ‚îÄ Their System
                  ‚îî‚îÄ‚îÄ XML-JSON Translation is localized here
  ```

- The translation logic is **localized** in the adapter ‚Äî so it does not pollute either system.

- **Question:** Does this only work for **Syntactic Interoperability**? (Adapters can also bridge semantic gaps ‚Äî e.g., a units-converting adapter between an N-based and lbf-based system ‚Äî though semantic adapters require the semantics to be well-documented on both sides to translate correctly.)

---

## Quick Reference: Key Terms

- **Interoperability:** systems exchange information AND use it.
- **Syntactic interoperability:** agreement on data format/types.
- **Semantic interoperability:** agreement on meaning/interpretation.
- **REST:** REpresentational State Transfer; stateless HTTP/HTTPS; POST/GET/PUT/DELETE.
- **RPC:** Remote Procedure Call; can be stateful; beyond CRUD.
- **SOAP:** Simple Object Access Protocol; XML; enterprise; built-in security/error handling.
- **GraphQL:** server-side schema; JSON; good for complex interrelated data.
- **Schema (XML/JSON/YAML):** describes document structure; enables automatic run-time validation.
- **Adapter pattern:** localized translation component between two different interfaces.
- **Extensible interface:** interface designed to accommodate future additions without breaking existing clients.
- **Design-time measures of interoperability:** implementation effort vs allowed variability (in conflict).


---

# Appendix: Lecture 17 Slides (raw extracted text)

The following is the full extracted text of Lecture 17. Preserved verbatim so no information is lost.

```text
CS 35L Software
Construction
L17 ≠ Code
Comprehension &
API Design
Assistant Teaching Professor
Computer Science Department
Why Comprehension Skills Matter
∑ Developers spend 58% of their time
 on understanding code
∑ A speedup in code understanding has a
 large impact on productivity
∑ With AI writing more and more code, developers might
spend even more time on reading code in the near future
Code Comprehension Approaches
"Bottom-Up" (Novices)                 "Top-Down" (Experts)
∑ Reading code line by line     ∑ Driven by a hypothesis of what the
    grouping statements into        code does
    logical "chunks" of higher
    abstraction                 ∑ Looking for "beacons" (key lines of
                                    code that reveal the core purpose
∑ Takes more time because           of a function or test the hypothesis)
    each statement needs to be
    analyzed                    ∑ First getting the big picture,
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
Beacons ≠ The Structural Signs
Guiding Your Through the Code
     Identifier Names Are Beacons
  The most frequent and most critical beacons are the names developers
  assign to variables, functions, and classes. Comprehension often depends
  domain information carried by identifier names.
  Tip: First look for the names of important classes, functions, and variables. If
  you don not know their meaning (e.g., due to lack of domain knowledge) look
  up their meaning first!
Identifier Names -- Code Examples
Spot the Domain in the Names ≠ What do the functiond do?
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
Beacons ≠ The Structural Signs
Guiding Your Through the Code
     Identifier Names Are Beacons
  The most frequent and most critical beacons are the names developers
  assign to variables, functions, and classes. Comprehension often depends
  domain information carried by identifier names.
  Tip: First look for the names of important classes, functions, and variables. If
  you don not know their meaning (e.g., due to lack of domain knowledge) look
  up their meaning first!
Think-Pair-Share: What are other beacons besides identifier names?
Beacons ≠ The Structural Signs
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
Beacons ≠ The Structural Signs
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
Beacons ≠ The Structural Signs
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
Beacons ≠ The Structural Signs
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
∑ REST (representational state transfer) is a stateless protocol (no server-side
  session) to exchange data in client-server systems via HTTP / HTTPS:
   ∑ POST ≠ Creates a Resource
   ∑ GET ≠ Reads a Resource
   ∑ PUT ≠ Updates a Resource
   ∑ DELETE ≠ Deletes a Resource
∑ Naming convention for URLs, based on resource identifiers
   ∑ "/users/123" instead of e.g., many variations of /getUser?id=123
∑ Resources are often described via XML, YAML, JSON, or HTML
Example REST API:            ...
GDS API                      "travelPreferences": {
∑ Request: POST /getFlights            "flightType": "Direct",
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
                  CS 35L Software Construction: Lecture 17 ≠ Code Comprehension & API Design  32
                  Tobias D¸rschmid
Common Technique to Test Compatibility:
XML / JSON / YAML Schema
∑ A schema describes the structure of document
∑ Schemas list attributes, their possible values, and complex types (e.g.,
  sequences, recursive types) for a document
∑ Validation of a document against a schema can be done automatically
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
             CS 35L Software Construction: Lecture 17 ≠ Code Comprehension & API Design  34
             Tobias D¸rschmid
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
∑ Document Interfaces and their Semantics
  (e.g., What units are implied? Does price include tax?
  MM/DD/YY or DD/MM/YY?
  What coordinate system is used a reference frame?)
∑ Use Shared Dictionary of Items or Agree on Vocabulary
  (e.g., doughnut or donut?  or?)
∑ Write Integration Tests for the Systems
Interface Descriptions
Syntactic Describe document format, the actions that can be
View      performed, their parameters, and outputs.
Semantic  Describe the purpose / meaning of the resource / action:
View          ∑ Side-effects: Changes to the state of a resource or
                 environment
              ∑ Usage restrictions: Who can perform this action?
              ∑ Error Handling: What errors can occur and why?
              ∑ Examples: Examples of outputs for a given input
      CS 35L Software Construction: Lecture 17 ≠ Code Comprehension & API Design  38
      Tobias D¸rschmid
(Simplified)
Syntactic View of the GDS Flight Booking API
POST /createBooking {       The flight number using the code
  ∑ flightNumber: str              of the operating airline
  ∑ date: dateTime
  ∑ seat: str          The date of the departing flight according to
                           the time zone of the departure airport
                       Seat number (same label as on the plane)
∑ paymentReference: PaymentMethod
∑ travelerName: str    Full name of the person traveling in ACSII
}
     In-Class Activity: Describe The Semantic
           View of GDS Flight Booking API
Example: Semantic View of GDS Booking
∑ Purpose: The airline confirms a requested booking
∑ Side-effects: money is charged, seat is marked as sold, can
  be canceled within 1 hour
∑ Usage restrictions: Authorized booking systems
∑ Errors: Invalid Format, Unauthorized, too many requests, ...
See here: https://developer.sabre.com/docs/rest_apis/trip/orders/booking_management
Guidelines on Interface Documentation
∑ Focus on how elements interact and their externally visible
 effects, not their implementation
∑ To support changeability of the implementation, expose only
 what is needed to use the interface
∑ Keep the documentation minimal and use-case oriented to
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
∑ GDS only lets you book seats
∑ Airlines make more money up-selling "priority boarding", or other add-ons
∑ Southwest doesn't have assigned seating
∑ Some airlines have seats that can be transformed into a sky couch, resulting in
  the unavailability in GDS
∑ Airlines do not get valuable customer data via GDS
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
    ∑ Offers can contain a dynamically-defined add-ons
    ∑ Add-on: (price, name, description, id)
        ∑ price(int): The price in cents (excl. tax) additionally charged when this add-on is
           selected
        ∑ name(str): The name of the add-on as shown to the user (in UTF-8)
        ∑ description(str): A short description shown to the user in order to decide if they want
           to purchase the add-on (in UTF-8)
        ∑ id(str): Unique identifier of this add-on starting with the flight number (in ASCII)
    ∑ A list of add-ons is added to a flight listing. The booking API needs to add an optional list of
       add-on ids to identify requested add-ons.
In-Class Activity: What Disadvantage does this have
                    over the existing GDS?
Lesson Learned: Interoperability
Often Conflicts with Changeability
∑ Shared interfaces and data format mean changes have to be
 implemented in all cooperating systems
∑ We cannot localize the interface change to a single system,
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
