# Clean Code & Code Quality â€” CS 35L Study Guide

Source: Lecture 18 "Code Review & Code Quality" (Tobias Durschmid). This guide covers the **clean code / code quality** portion of the lecture. The code review research portion is in `code-review.md`.

---

## Learning Objectives

After this lecture, you should be able to:

- **Explain why clean code matters**
- **Evaluate** whether code is clean or unclean
- **Refactor** code to become cleaner

### Anchor Quote (Tony Hoare, Turing Award 1980)

> "There are two ways of constructing a software design: One way is to make it so simple that there are obviously no deficiencies and the other way is to make it so complicated that there are no obvious deficiencies. The first method is far more difficult."

---

## Recap: Information Hiding & Related Design Concepts

Clean code builds on top of design principles. Before diving in, the lecture revisits how three concepts relate.

### Information Hiding vs Separation of Concerns

Both share the same **goal: Complexity Reduction** and the same **solution: Modular Decomposition**. They differ in concreteness:

| Information Hiding | Separation of Concerns |
|--------------------|------------------------|
| Is more **concrete**. Tells you **which** "Concerns" should be separated (design decisions that are likely to change) | Is more **abstract**. Does not tell you **which** concerns you should consider to separate |
| Tells you to **"hide" the implementation** of concerns from other modules | Only tells you to **separate** them |

### Information Hiding vs SOLID

Both share the same **goal: Complexity Reduction** and the same **solution: Hiding Details**. They differ in scope and concreteness:

| Information Hiding | SOLID |
|--------------------|-------|
| More **abstract**. Does not tell you concrete steps how to implement information hiding | Is more **concrete**. A set of more concrete steps and rules that usually result in information hiding |
| Applies to **all** programming languages independent to their paradigm | Applies **only to object-oriented** programming languages, but **not** to functional programming |

### Where Clean Code Fits

Clean Code is the *practical, line-level* application of these higher-level design principles. It tells you concretely how to write the names, structure, comments, and contracts inside each module.

---

## Opinions on What Exactly Is Clean Code Vary

Several luminaries have defined clean code:

> "Clean code can be **read, and enhanced by a developer other than its original author**"
> â€” **Dave A. Thomas** (godfather of Eclipse IDE)

> "Clean code always looks like it was **written by someone who cares**"
> â€” **Michael Feathers** (author of *Working Effectively with Legacy Code*)

> "You know you are working on clean code when each routine you read **turns out to be pretty much what you expected**"
> â€” **Ward Cunningham** (inventor of the Wiki)

> "Clean code is simple and direct. Clean code **reads like well-written prose**. Clean code never obscures the designer's intent but rather is full of crisp abstractions and straightforward lines of control."
> â€” **Grady Booch** (co-inventor of UML)

### The Instructor's Definition

> "Clean code effectively **convinces the reader** that the code is **likely free of bugs** and gives contributors the **confidence** that everything will be fine. Clean code **never betrays** this trust."

This definition decomposes into two pillars:

**Likely Free of Bugs**
- Prioritizes **Simplicity** to reduce cognitive load
- Expresses the **design intent**

**Confidence for Contributions**
- The code is **robust** to change
- Violated assumptions are **caught early**

---

## Why Clean Code Matters

- **Code is read â‰ˆ 10â€“14 times more often than it is written.**
  - Investing time into making it easier to read **pays off**.
- **Developers spend 58% of their time on understanding code.**
  - A speedup in code understanding has a **large impact** on productivity.

### The "Broken Window" Effect

- When developers make changes to modules that have many **code smells**, they are more likely to **include code smells** in their code as well.
- **Technical debt propagates and compounds.**

References: `dev.to/wmattei/the-broken-windows-theory-in-software-development-why-you-must-fix-everything-now-fdp` and W. LevÃ©n et al. *"The broken windows theory applies to technical debt"*, Empirical Software Engineering 2024.

---

## Code Comprehension Approaches

Understanding how readers process code informs how we should write it.

### "Bottom Up" (Novices)
- Reading code **line by line**
- **Grouping** statements into logical "chunks" of higher abstraction
- Takes **more time** because each statement needs to be analyzed

### "Top Down" (Experts)
- Driven by a **hypothesis** of what the code does
- Looking for **"beacons"** (key lines of code that reveal the core purpose of a function or test the hypothesis)
- First getting the **big picture**, then **looking for details**

**Goal**: developers want to **move from Bottom Up to Top Down** as they gain expertise. Clean code supports both approaches â€” it gives novices clean chunks to group, and it gives experts crisp beacons.

---

# The Four Pillars of Clean Code

1. **Meaningful Naming**
2. **Simplified Structure**
3. **Purposeful Documentation**
4. **Design by Contract**

---

## Pillar 1: Meaningful Naming

### Example A vs B â€” Meaningless vs Misleading

```python
# Test Case
def test_apply_discount(self):
    self.assertAlmostEqual(apply_discount(50, 0.5), 25.0)

# Version A
def apply_discount(price: int, d: float): -> float | None
    discount_amount = price * d
    final_price = price - discount_amount
    return final_price

# Version B
def apply_discount(price: int, discount_percent: float): -> float | None
    discount_amount = price * discount_percent
    final_price = price - discount_amount
    return final_price
```

- **Version A** (`d`) â€” meaningless name, but harmless. **Meaningless names are still better than misleading names.**
- **Version B** (`discount_percent`) â€” **misleading**, since the name implies the unit is `%`, but the test passes `0.5` (i.e., a fraction, not 50%). Wrong hypothesis is induced in the reader.

### Example A vs B â€” Picking the Better Name with Correct Semantics

```python
# Test Case
def test_apply_discount(self):
    self.assertAlmostEqual(apply_discount(50, 50), 25.0)

# Version A
def apply_discount(price: int, d: float): -> float | None
    discount_amount = price * d / 100
    final_price = price - discount_amount
    return final_price

# Version B
def apply_discount(price: int, discount_percent: float): -> float | None
    discount_amount = price * discount_percent / 100
    final_price = price - discount_amount
    return final_price
```

Here Version B wins: **choose meaningful and precise parameter names** because the implementation now matches the implied semantics of `discount_percent`.

### Example A vs B â€” Conventions Beat Creativity

```javascript
// Version A
for (let i = 0, i < input_text.length, i++) {
    console.log(translateToGreek(text[i]));
}

// Version B
for (let index = 0, index < input_text.length, index++) {
    console.log(translateToGreek(text[index]));
}
```

**Version A** wins. **Choose commonly used names over creative inventions, especially when they are shorter.** Experienced developers can understand code more quickly if it follows **commonly seen patterns** or coding conventions. `i` is the universally-known loop index.

### Naming Priority Hierarchy

**Choose Meaningful & Precise Names for Variables, Parameters, Classes, and other Identifiers:**

- **Highest Priority: Wrong or misleading variable names are worse than none at all.**
  - E.g., if `discount_percent` is `[0, 1]` instead of `[0, 100]` it is **worse than** calling the variable `"v14"`.
    - `discount_percent` gives the developer a **wrong hypothesis**, so they **rely on it**.
    - `v14` gives them **no hypothesis at all**, so they **start thinking themselves**.
- **High Priority: Precise names are better than vague names.**
  - `discount_percent` is better than `discount`.
- **Medium Priority: Shorter is mostly better (unless it compromises meaning).**
  - Usually, **full words are better than one letter names**.
  - **Well-understood abbreviations** are better than long full words.

### Common Conventions

- **Class names should be nouns** (e.g., `Transfer` instead of `Transfering`).
- **Method names should be verbs** (e.g., `hasPermission()` instead of `permission()`, or `isActive()` instead of `active()`).
- Each community and programming language has different coding style conventions:
  - **Python uses `snake_case`**
  - **C++ uses `camelCase`**
- Become familiar with the coding style guidelines that are common in your language and domain.

---

## Pillar 2: Simplified Structure

### Example â€” Nested If/Else (High Cognitive Load)

```python
def calculate_pay_amount(self):
    if self.is_dead():
        result = self.dead_amount()
    else:
        if self.is_separated():
            result = self.separated_amount()
        else:
            if self.is_retired():
                result = self.retired_amount()
            else:
                result = self.normal_pay_amount()
    return result
```

The "normal" path is **highly nested** (i.e., inside multiple levels of conditions). The most common case is buried deepest.

### Refactor With Guard Clauses

```python
def calculaye_pay_amount(self):
    if self.is_dead():
        return self.dead_amount()
    if self.is_separated():
        return self.separated_amount()
    if self.is_retired():
        return self.retired_amount()
    return self.normal_pay_amount()
```

**A guard clause exits the function early after checking and handling an edge case.**

### Second Example â€” Nested Validation

```python
def apply_discount(price: int, discount_percent: float): -> float | None
    if price >= 0:
        if 0 <= discount_percent <= 100:
            discount_amount = price * (discount_percent / 100)
            final_price = price - discount_amount
            return final_price
        else:
            logger.error(f"Invalid discount (not 0<={discount_percent}<=100).")
            return None
    else:
        logger.error(f"Invalid price (not {price}>=0).")
        return None
```

The "normal" path is **highly nested** inside multiple levels of conditions.

### Refactored with Guard Clauses

```python
def apply_discount(price: int, discount_percent: float): -> float | None
    if price < 0:
        logger.error(f"Invalid price (not {price}>=0).")
        return None
    if not 0 <= discount_percent <= 100:
        logger.error(f"Invalid discount (not 0<={discount_percent}<=100).")
        return None
    discount_amount = price * (discount_percent / 100)
    final_price = price - discount_amount
    return final_price
```

### Minimizing Cognitive Load

- **The Research**: High complexity (deep nesting, many branches) **correlates strongly with bug rates** and **time-to-understand**. Human brains struggle to simulate deep decision trees.
- **Actionable Advice**:
  - **Reduce Nesting Level**: Whenever possible use lower nesting level. **Very high nesting level (15+) should be avoided.**
  - **Use Guard Clauses**: Instead of wrapping the entire function in an `if` block, **handle the negative case first and return early**.

Reference: J. Johnson et al. *"An Empirical Study Assessing Source Code Readability in Comprehension"*, ICSME 2019.

### Example â€” Long Function vs Extracted Helpers (Chunking)

**Version A â€” long, all logic inline:**

```python
def process_order_and_generate_invoice(order_data,
                                       customer_info, pricing_rules):
    if not order_data or not customer_info:
        raise ValueError("Missing order or customer data.")
    if 'items' not in order_data or not isinstance(order_data['items'], list):
        raise ValueError("Order must contain a list of items.")
    subtotal = 0
    for item in order_data['items']:
        base_price = pricing_rules.get(item['product_id'], 0)
        subtotal += base_price * item['quantity']
        if item['quantity'] > 10:
            subtotal -= (base_price * item['quantity']) * 0.05

    tax_rate = 0.0825  # Standard state tax
    if customer_info.get('is_tax_exempt', False):
        tax_amount = 0
    else:
        if customer_info.get('location') == 'Metropolis':
            tax_rate = 0.10
        tax_amount = subtotal * tax_rate

    total_amount = subtotal + tax_amount
    # [...]
```

**Version B â€” extracted methods:**

```python
def process_order_and_generate_invoice(order_data,
                                       customer_info, pricing_rules):
    _validate_order_data(order_data, customer_info)
    subtotal = _calculate_subtotal(order_data['items'], pricing_rules)
    tax_amount = _calculate_tax(subtotal, customer_info)
    total_amount = subtotal + tax_amount
    # [...]
```

Version B **requires less working memory to understand. Forms meaningful abstractions.**

### Write Code to Support "Chunking"

- **The Research**:
  - The human brain has a **limited "working memory"**.
  - We can only hold about **5 to 9 distinct pieces of information at once**.
- **Actionable Advice**:
  - **Limit Function Size**: A function should fit entirely on one screen. If a reader has to scroll, they lose the context stored in their working memory.
  - **Extract Methods**: Even if a block of code is only used once, extracting it into a function named `calculateTax()` allows the reader to **"chunk"** that logic into a single concept rather than tracking 10 lines of math.
    - **Find meaningful abstractions that do not require developers to look inside the function to understand what it does.**

---

## Pillar 3: Purposeful Documentation

### Example â€” Misleading Comment (Worst Case)

```python
# Version A
# Matches dates in DD-MM-YYYY or MM/DD/YYYY
pattern = re.compile(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b')
found_matches = pattern.findall(text)

# Version B
pattern = re.compile(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b')
found_matches = pattern.findall(text)
```

The comment in Version A is **incorrect** â€” the regex matches **emails**, not dates. Version B (no comment) is better.

- **Misleading or incorrect comments are harmful** because developers build an **incorrect hypothesis**, and continue to operate under this hypothesis for a long time.
- In practice, this often becomes an issue when the **code is updated without also updating the comments** so that the comment is outdated.

### Example â€” Comment That Explains *Why*

```python
# Version A
# Match arbitrary top-level domains to avoid updating when new TLDs are added
tld_pattern = re.compile(r'\b[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b')
found_matches = tld_pattern.findall(text)

# Version B
pattern = re.compile(r'\b[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b')
found_matches = pattern.findall(text)
```

Version A wins: the comment **helps developers understand *why* this decision was made**.

### Example â€” Useless Comment That Repeats the Code

```python
# Version A
def count_numbers(data):
    n = 0
    for current_value in data:
        if current_value % 2 == 0:
            n += 1  # increment n
    return n

# Version B
def count_numbers(data):
    n = 0
    for current_value in data:
        if current_value % 2 == 0:
            n += 1
    return n
```

The `# increment n` comment is **completely unnecessary. Only wastes time to read.** Version B wins.

### Example â€” Comment Explains Intent vs Self-Documenting Name

```python
# Version A
def count_numbers(data):
    n = 0
    for current_value in data:
        if current_value % 2 == 0:
            n += 1  # found even number

# Version B
def count_numbers(data):
    n = 0
    for current_value in data:
        if current_value % 2 == 0:
            n += 1

# Better Version
def count_even_numbers(data):
    n = 0
    for current_value in data:
        if current_value % 2 == 0:
            n += 1
```

Version A's comment is **fine, but there is an even better improvementâ€¦** â€” rename the function to `count_even_numbers` so the intent is **expressed in the code itself**, no comment needed.

### Focus Comments on *Intent* and *Non-Obvious Details*. Do Not Repeat *What* The Code Does

- **The Research**:
  - **Outdated or misleading comments are worse than no comments at all.**
  - Comments that **just describe what can be read in the code** does not have any impact.
  - Documenting the **purpose** of code and **why** it was written the way it was is **very helpful to developers**.
- **Actionable Advice**:
  - Use comments to express **what you cannot express in the code**.
  - Before writing a comment, **first try to refactor** the code to express this intent directly (e.g., renaming variables or methods, extract method, or introduce a helper constant).

---

## Pillar 4: Design by Contract

### Concept

Each function or operation defines:

- **Pre-Conditions**
  - Assumption on the inputs (or the state of the object).
- **Post-Conditions**
  - Guarantees on the output (or state of the object).

They are part of the **"visible contract"** of the module. **Users of the module can only assume these contracts. The Implementation is then hidden.**

(This mirrors the iceberg metaphor â€” the contract is the visible tip above the water; the implementation is the hidden mass beneath.)

### Making Assumptions Explicit With the Power of Assertions

**Python:**

```python
def apply_discount(price: int,
                   discount: float) -> float:
    assert 0 <= price, "Invalid price"
    assert 0 <= discount <= 100, "Invalid discount"
    return price * (1 - discount/100)
```

Turn off assertions with the **optimize interpreter flag**:
```
python -O my_program.py
```

**C/C++:**

```c
#include <assert.h>

double applyDiscount(int price,
                     double discount) {
    assert(0 <= price);
    assert(0 <= discount && discount <= 100);
    return price * (1 - discount/100);
}
```

Turn off assertions with the **`DNDEBUG` compiler flag**:
```
gcc -DNDEBUG main.c -o my_program
```

### Why Assertions Matter for Clean Code

- They **catch violated assumptions early** (fits the "Confidence for Contributions" half of clean code).
- They make the **contract executable**, not just documented â€” the assumption is checked at runtime in debug builds.
- They can be **disabled in production** (Python `-O`, C/C++ `-DNDEBUG`) so they don't cost performance once you trust the contract is upheld.

---

## "Likely Free of Bugs" â€” Prioritize Simplicity

Recall the instructor's definition: clean code **convinces the reader that the code is likely free of bugs**. The way to achieve this is by **prioritizing simplicity**:

- Simple code is easier to read top-down (experts) and bottom-up (novices).
- Simple code minimizes cognitive load (â‰¤ 5â€“9 chunks of working memory).
- Simple code expresses **design intent** clearly so the reader doesn't have to reverse-engineer it.
- Tony Hoare's quote applies: simple-with-obviously-no-deficiencies is the harder but correct path.

---

## The Boy/Girl Scout Rule

> "**Always Leave the ~~Campground~~ Module Cleaner Than You Found It**"

- Whenever you make changes to a module, **consider if there are possible improvements**.
- Before making the change, consider if there are **negative consequences** (did you miss something?).
- The point is to **continuously combat the "broken window" effect** â€” small cleanups each time prevent technical debt from compounding.

(Lecture humorously pairs this rule with the "Why? Why? Why? Oh *that's* why" meme â€” a reminder to actually understand why the existing code is the way it is before cleaning up.)

---

## Before You Refactor: Make Sure You Have Good Tests

- Refactoring is walking a tightrope.
- A **Comprehensive Test Suite** is your safety net beneath the tightrope.
- Without good tests, refactoring will silently change behavior â€” making the code "cleaner" while breaking it.

---

## Quick-Reference Summary

| Pillar | Key Principles |
|--------|----------------|
| **Meaningful Naming** | Misleading > meaningless. Precise > vague. Short (when clear). Class names = nouns. Method names = verbs. Follow language conventions (snake_case for Python, camelCase for C++). |
| **Simplified Structure** | Use **guard clauses** for early returns. Avoid nesting (â‰¥15 levels = banned). Extract methods to enable **chunking** (5â€“9 items working memory). Keep functions to one screen. |
| **Purposeful Documentation** | Comments explain **why**, not **what**. Outdated comment > worse than no comment. Refactor before commenting. Prefer **self-documenting code** (rename function, extract constant). |
| **Design by Contract** | Define **preconditions** and **postconditions**. Make them executable with `assert`. Python `-O` and C/C++ `-DNDEBUG` disable assertions in production. |

| Supporting Habit | What It Says |
|------------------|--------------|
| Boy/Girl Scout Rule | Leave the module cleaner than you found it. |
| Tests first | Get a comprehensive test suite **before** refactoring. |
| Beware Broken Windows | Code smells breed more code smells; technical debt compounds. |
| Read > Write | Code read 10â€“14Ã— more than written; 58% of dev time is on understanding. |


---

# Appendix: Lecture 18 Slides (raw extracted text)

The following is the full extracted text of Lecture 18. Preserved verbatim so no information is lost.

```text
CS 35L
Software
Construction
L18 Code Review
& Code Quality
Assistant Teaching Professor
Research on Modern Code Review
                                                                                                                                          F
· Expectation:                                                                                                                     t
· Code reviews finds Bugs                                                                                                          c
                                                                                                                                   c
                                                                                                                                   r
                                                                                                                                   a
                                                                                                                                   o
                                                                                                                                   A
                                                                                                                                   r
                                                                                                                                   f
                                                                                                                                   "
                                                                                                                                   c
                                                                                                                                   t
                                                                                                                                   p
EXPECTATIONS, OUTCOMES, AND CHALLENGES OF MODEFRiNgCuOreDE3.RDEVeIvEeWlo, pICeSrEs'2m01o3t,iBvAaCtiCoHnEsLfLoIrAcNoDdBeIRrDeview.  t
                                                                                                                                   d
Research on Modern Code Review
· Expectation:
   · Code reviews finds Bugs
· Reality:
   · Code review
     transfers knowledge,
     improves readability and
     maintainability of the code,
     and sometimes also finds bugs
                                 Figure 4. Frequency of comments by card sort category.
EXPECTATIONS, OUTCOMES, AND CHALLENGES OF MODERcNomCOmDeEntRsE) VIrEeWco, IrCdSeEd 20b1y3, BCAoCdCeHFElLoLwI A. NDFiBgIuRrDe 4 shows the
CS 35L Software   Construction:  LectcuarteeCg18oord­ieeCsoIodmfecpRoremovvmieemewne&tsnCtfsoo:udneTdQhteuharmloituoygsht  the card  sort.       w3ith
Why code review do not find bugs. How current code
review practices slows us down.
   · Top 3 categories are long term maintenance issues
       · Comments, naming, and styles (22%)
       · Organization of code (16%)
       · Alternative solutions for long term maintenance (9%)
   · Only 15% of comments provided by reviewers indicate a possible defect.
   · Long term maintainability issues are a much larger portion of comments (50%)
   · Only 33% of comments are deemed useful by the author.
                           Figure 5. Developers' responses in surveys of the amount
                                 of code understanding for code review outcomes.
CS 35L Software Constrruecvtiieown:iLnegctutriem1e8."­ CoOdne Rtehveiews&amCoedenQoutael,ityin the code review5
Why code review does not find bugs. How current code review
practices slows us down.
· ICSE 2015, Jacek Czerwonka et al. Microsoft
   · They mined code review tool usage data
   · CodeFlow is open 5 or 6 hours per developer per day.
   · 30 minutes of interaction time per developer per day.
   · Primary goals: Find defects, improve maintainability, share knowledge,
     broadcast progress.
Real-World Benefits of Code Reviews
· Code Review is the primary way to teach junior developers
· "Expectations for code review at Google do not center around problem solving.
  Reviewing was introduced at Google to ensure code readability and
  maintainability. Today's developers also perceive this educational aspect, in
  addition to maintaining norms, tracking history, gatekeeping, and accident
  prevention. Defect finding is welcomed but not the only focus"
· Code Review at google was introduced to "force developers to write code that
  other developers could understand"
Source: Sadowski, C., Söderberg, E., Church, L., Sipko, M., & Bacchelli, A.z Modern Code Review: a Case Study at
Google. International Conference on Software Engineering: Software Engineering in Practice (ICSE-SEIP 2018)
How to Maximize Code Review Effectiveness
· 1­2 Reviewers (more reviewers increase the bystander effect)
· Keep the review short
   · Review quality degrades after 1 hour of review
   · Keep pull requests roughly between 200 and 400 lines of code
   · Long pull requests with thousands of changes are not read in detail and
     often receive only superficial comments
   · Instead of bundling multiple changes into a single massive update, break them
     down into smaller, logically independent PRs that can be "stacked". Smaller
     PRs are reviewed more thoroughly, encounter fewer rebase conflicts, and are
     merged significantly faster
Takeaway from Modern Code Review Practices
· New reviewers learn fast but need at least 6-12 months to be productive as the
  rest of the team
· People do not usually find bugs and focus on maintainability instead
· We need to have "rigorous criteria" to find bugs
· Developers need to have "critical eyes" for reviewing code changes, thinking
  about corner cases.
· We believe identifying pre and post conditions is useful to help my students
  develop as "effective code reviewers for defect finding."
Learning Objectives                "There are two ways of constructing a software design: One
Of this Lecture
                                   way is to make it so simple that there are obviously no
After this lecture, you should be
able to:                           deficiencies and the other way is to make it so complicated
                                   that there are no obvious deficiencies. The first method is
                                   far more difficult."  ­ Tony Hoare (Turning Award 1980)
· Explain why clean code
  matters
· Evaluate whether code is clean
  or unclean
· Refactor code to become
  cleaner
 Recap of
Information
  Hiding &
  Related
   Design
 Concepts
                    CS 35L Software Construction: Lecture 18 ­ Code Review & Code Quality
                    Tobias Dürschmid
Compare
Information Hiding With Separation of Concerns
Information Hiding                        Separation of Concerns
Goal: Complexity Reduction
Solution: Modular Decomposition
     Is more concrete. Tells you which     Is more abstract. Does not tell you
"Concerns" should be separated (design    which concerns you should consider
    decisions that are likely to change)                    to separate
Tells you to "hide" the implementation    Only tells you to separate them
    of concerns from other modules
Compare
Information Hiding With SOLID
Information Hiding                          SOLID
Goal: Complexity Reduction
                  Solution: Hiding Details
 More abstract. Does not tell you         Is more concrete. A set of more
concrete steps how to implement       concrete steps and rules that usually
          information hiding                 result in information hiding
Applies to all programming languages     Applies only to object-oriented
     independent to their paradigm    programming languages, but not to
                                             functional programming
Learning Objectives                "There are two ways of constructing a software design: One
Of this Lecture
                                   way is to make it so simple that there are obviously no
After this lecture, you should be
able to:                           deficiencies and the other way is to make it so complicated
                                   that there are no obvious deficiencies. The first method is
                                   far more difficult."  ­ Tony Hoare (Turning Award 1980)
· Explain why clean code
  matters
· Evaluate whether code is clean
  or unclean
· Refactor code to become
  cleaner
Opinions on What Exactly is Clean Code Vary
"Clean code can be read, and         "Clean code always looks like it was
enhanced by a developer other        written by someone who cares"
than its original author" ­ Dave A.  ­ Michael Feathers (author of "Working
Thomas (godfather of Eclipse IDE)    Effectively with Legacy Code")
"You know you are working on clean   "Clean code is simple and direct. Clean
code when each routine you read      code reads like well-written prose.
turns out to be pretty much what     Clean code never obscures the
you expected"                        designer's intent but rather is full of
­ Ward Cunningham                    crisp abstractions and straightforward
(inventor of the Wiki)               lines of control."
                                     ­ Grady Booch (co-inventor of UML)
Opinions on What Exactly is Clean Code Vary
My Definition: "Clean code effectively convinces the reader that the code
is likely free of bugs and gives contributors the confidence that
everything will be fine. Clean code never betrays this trust."
  Likely Free of Bugs           Confidence for Contributions
· Prioritizes Simplicity to    · The code is robust to change
                               · Violated assumptions are
    reduce cognitive load
· Expresses the design intent      caught early
Why Clean Code Matters
· Code is read   -  times more often than it is written
· Investing time into make it easier to read pays off
· Developers spend 58% of their time
 on understanding code
· A speedup in code understanding has a
large impact on productivity
The "Broken Window" Effect
· When developers make changes made to
 modules that have many code smells, they
 are more likely to include code smells in
 their code as well
· Technical debt propagates and compounds
See https://dev.to/wmattei/the-broken-windows-theory-in-software-development-why-you-must-fix-everything-now-fdp
& W. Levén et al. "The broken windows theory applies to technical debt" Empirical Software Engineering 2024
Code Comprehension Approaches
"Bottom Up" (Novices)                 "Top Down" (Experts)
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
Four Pillars of Clean Code
Meaningful  Simplified     Purposeful  Design by
  Naming    Structure   Documentation   Contract
Meaningful
Naming
CLEAN CODE
& CODE QUALITY
                                                                                                                          21
Which Version is Better? A or B?
Test Case
def test_apply_discount(self):
       self.assertAlmostEqual(apply_discount(50, 0.5), 25.0)
Version A
def apply_discount(price: int, d: float): -> float | None
discount_amount = price * d            Meaningless names
final_price = price - discount_amount are still better than
return final_price                     misleading names
Version B
def apply_discount(price: int, discount_percent: float): -> float | None
discount_amount = price * discount_percent  Variable name is
final_price = price - discount_amount       misleading since it
return final_price                          implies that the unit is %
           CS 35L Software Construction: Lecture 18 ­ Code Review & Code Quality  22
           Tobias Dürschmid
Which Version is Better? A or B?
Test Case
def test_apply_discount(self):
       self.assertAlmostEqual(apply_discount(50, 50), 25.0)
Version A
def apply_discount(price: int, d: float): -> float | None
       discount_amount = price * d / 100
       final_price = price - discount_amount
       return final_price
Version B
def apply_discount(price: int, discount_percent: float): -> float | None
discount_amount = price * discount_percent / 100 Choose meaningful
final_price = price - discount_amount                        and precise
return final_price                                           parameter names
           CS 35L Software Construction: Lecture 18 ­ Code Review & Code Quality  23
           Tobias Dürschmid
Which Version is Better? A or B?
Version A  Choose commonly used names
                over creative inventions,
           especially when they are shorter
for (let i = 0, i < input_text.length, i++) {
           console.log(translateToGreek(text[i]));
}
Version B
for (let index = 0, index < input_text.length, index++) {
           console.log(translateToGreek(text[index]));
}
· Experienced developers can understand code more quickly if it follows
  commonly seen patterns or coding conventions
           CS 35L Software Construction: Lecture 18 ­ Code Review & Code Quality  24
           Tobias Dürschmid
Chose Meaningful & Precise Names for Variables,
Parameters, Classes, and other Identifiers
· Highest Priority: Wrong or misleading variable names are worse than none at al
   · E.g. if discount_percent is [0, 1] instead of [0,100] it is worse than called the
     variable "v14"
       · discount_percent gives the developer a wrong hypothesis, so they rely on it
       · v14 gives them no hypothesis at all, so they start thinking themselves
· High Priority: Precise names are better than vague names
   · discount_percent is better than discount
· Medium Priority: Shorter is mostly better (unless it compromises meaning)
   · Usually, full words are better than one letter names
   · Well-understood abbreviations are better than long full words
Common Conventions
· Class names should be nouns (e.g., "Transfer" instead of "Transfering"
· Method names should be verbs (e.g., "hasPermission()" instead of "permission()"
  or isActive() instead of "active()"
· Each community and programming language has different coding style
  conventions
   · Python uses snake_case
   · C++ uses camelCase
   · Become familiar with the coding style guidelines that are common in your
     language and domain
Simplified
Structure
CODE REVIEW
& CODE QUALITY
                                                                                                                          27
How can we Minimize the Cognitive Load
For this Function?
 Python
 def calculate_pay_amount(self):
         if self.is_dead():
                result = self.dead_amount()
         else:
                if self.is_separated():
                       result = self.separated_amount()
                else:
                       if self.is_retired():
                               result = self.retired_amount()
                       else:
                               result = self.normal_pay_amount()
       return result
The "normal" path is highly nested (i.e., inside multiple levels of conditions)
Use Guard Clauses To Simplify Functions
  Python
 def calculaye_pay_amount(self):
         if self.is_dead():
                return self.dead_amount()
         if self.is_separated():
                return self.separated_amount()
         if self.is_retired():
                return self.retired_amount()
         return self.normal_pay_amount()
A guard clause exits the function early after checking and handling an edge case.
How can we Minimize the Cognitive Load
For this Function?
 Python
 def apply_discount(price: int, discount_percent: float): -> float | None
         if price >= 0:
                if 0 <= discount_percent <= 100:
                       discount_amount = price * (discount_percent / 100)
                       final_price = price - discount_amount
                       return final_price
                else:
                       logger.error(f"Invalid discount (not 0<={discount_percent}<=100).")
                       return None
         else:
                logger.error(f"Invalid price (not {price}>=0).")
                return None
The "normal" path is highly nested (i.e., inside multiple levels of conditions)
Use Guard Clauses To Simplify Functions
  Python
  def apply_discount(price: int, discount_percent: float): -> float | None
         if price < 0:
                       logger.error(f"Invalid price (not {price}>=0).")
                       return None
         if not 0 <= discount_percent <= 100:
                       logger.error(f"Invalid discount (not 0<={discount_percent}<=100).")
                       return None
         discount_amount = price * (discount_percent / 100)
         final_price = price - discount_amount
         return final_price
A guard clause exits the function early after checking and handling an edge case.
Minimizing Cognitive Load
· The Research: High complexity (deep nesting, many branches) correlates
  strongly with bug rates and time-to-understand. Human brains struggle to
  simulate deep decision trees
· Actionable Advice:
   · Reduce Nesting Level: Whenever possible use lower nesting level. Very high
     nesting level (15+) should be avoided
   · Use Guard Clauses: Instead of wrapping the entire function in an if block,
     handle the negative case first and return early.
See J. Johnson et al. "An Empirical Study Assessing Source Code Readability in Comprehension". ICSME 2019
Which Version is Better? A or B?
Version A                                                      Version B
def process_order_and_generate_invoice(order_data,             def process_order_and_generate_invoice(order_data,
customer_info, pricing_rules):                                 customer_info, pricing_rules):
   if not order_data or not customer_info:                           _validate_order_data(order_data, customer_info)
      raise ValueError("Missing order or customer data.")
                                                                     subtotal = _calculate_subtotal(order_data['items'],
   if 'items' not in order_data or not                         pricing_rules)
isinstance(order_data['items'], list):
                                                                     tax_amount = _calculate_tax(subtotal, customer_info)
      raise ValueError("Order must contain a list of items.")
                                                                     total_amount = subtotal + tax_amount
   subtotal = 0
   for item in order_data['items']:                            [...]
      base_price = pricing_rules.get(item['product_id'], 0)    Requires less working memory
      subtotal += base_price * item['quantity']                       to understand. Forms
      if item['quantity'] > 10:
                                                                   meaningful abstractions
          subtotal -= (base_price * item['quantity']) * 0.05
   tax_rate = 0.0825 # Standard state tax
   if customer_info.get('is_tax_exempt', False):
      tax_amount = 0
   else:
      if customer_info.get('location') == 'Metropolis':
          tax_rate = 0.10
      tax_amount = subtotal * tax_rate
total_amount = subtotal + tax_amount
[...]
Write Code to Support "Chunking"
· The Research:
   · The human brain has a limited "working memory"
   · We can only hold about 5 to 9 distinct pieces of information at once
· Actionable Advice:
   · Limit Function Size: A function should fit entirely on one screen. If a reader
     has to scroll, they lose the context stored in their working memory.
   · Extract Methods: Even if a block of code is only used once, extracting it into a
     function named calculateTax() allows the reader to "chunk" that logic into a
     single concept rather than tracking 10 lines of math.
       · Find meaningful abstractions that do not require developers to look
         inside the function to understand what it does
Purposeful
Documentation
CODE REVIEW
& CODE QUALITY
                                                                                                                          35
Which Version is Better? A or B?
Version A                                    Version B
# Matches dates in DD-MM-YYYY or MM/DD/YYYY  pattern = re.compile(r'\b[A-Za-z0-9._%+-
pattern = re.compile(r'\b[A-Za-z0-9._%+-     ]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b')
]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b')         found_matches = pattern.findall(text)
found_matches = pattern.findall(text)
    Incorrect! The regex
machtes emails, not dates
· Misleading or incorrect comments are harmful because developers build an
  incorrect hypothesis, and continue to operate under this hypothesis for a long time
· In practice, this often becomes an issue when the code is updated without also
  updating the comments so that the comment is outdated
Which Version is Better? A or B?
Version A                                  Version B
# Match arbitrary top-level domains to     pattern = re.compile(r'\b[A-Za-z0-9.-
avoid updating when new TLDs are added     ]+\.[A-Z|a-z]{2,}\b') found_matches =
tld_pattern = re.compile(r'\b[A-Za-z0-9.-  pattern.findall(text)
]+\.[A-Z|a-z]{2,}\b') found_matches =
tld_pattern.findall(text)
       Helps developers
     understand why this
      decision was made
Which Version is Better? A or B?
Version A                          Version B
def count_numbers(data):           def count_numbers(data):
   n=0                                n=0
   for current_value in data:         for current_value in data:
       if current_value % 2 == 0:         if current_value % 2 == 0:
           n += 1 # increment n               n += 1
                                      return n
 return n
Completely unnecessary.
Only wastes time to read
Which Version is Better? A or B?
Version A                              Version B
def count_numbers(data):               def count_numbers(data):
   n=0                                    n=0
   for current_value in data:             for current_value in data:
       if current_value % 2 == 0:             if current_value % 2 == 0:
           n += 1 # found even number             n += 1
  fine, but there is an even
    better improvement...
Better Version
def count_even_numbers(data):
   n=0
   for current_value in data:
       if current_value % 2 == 0:
           n += 1
Focus Comments on Intent and Non-Obvious
Details. Do not Repeat What The Code Does
· The Research:
   · Outdated or misleading comments are worse than no comments at all
   · Comments that just describe what can be read in the code does not have any
     impact
   · Documenting the purpose of code and why it was written the way it was is
     very helpful to developers
· Actionable Advice:
   · Use comments to express what you cannot express in the code
   · Before writing a comment, first try to refactor the code to express this intent
     directly (e.g., renaming variables or methods, extract method, or introduce a
     helper constant)
Design By
Contract
CODE REVIEW
& CODE QUALITY
                                                                                                                          41
Design By Contract
Each function or operation defines:
· Pre-Conditions
   · Assumption on the inputs (or the state of the object)
· Post-Conditions
   · Guarantees on the output (or state of the object)
They are part of the "visible contract" of the module.
Users of the module can only assume these contracts.
The Implementation is then hidden
Making Assumptions Explicit
With the Power of Assertions
Python                                                        Turn off assertions with the optimize
                                                              interpreter flag:
def apply_discount(price: int,                                python -O my_program.py
                                  discount: float) -> float:
   assert 0 <= price, "Invalid price"
   assert 0 <= discount <= 100, "Invalid discount"
   return price * (1 ­ discount/100)
C/C++                                                    Turn off assertions with the DNDEBUG compiler flag:
                                                         gcc -DNDEBUG main.c -o my_program
#include <assert.h>
double applyDiscount(int price,
                                     double discount) {
   assert(0 <= price);
   assert(0 <= discount && discount <= 100);
   return price * (1 ­ discount/100);
}
Boy/Girl Scout Rule: "Always Leave the
Campground Module Cleaner Than You Found It"
· Whenever you make changes to a module,                                       While I'm
  consider if there are possible improvements                                here, let me
                                                                             remove this
· Before making the change, consider if there                                dependency
  are negative consequences (did you miss                                    and cleanup
  something?)
                                                                               the code
Why?                    Oh that's
Why?                    why
Why?
      CS 35L Software Construction: Lecture 18 ­ Code Review & Code Quality  44
      Tobias Dürschmid
Before you
Refactor
Make sure you
have Good Tests
Please fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three key insights you learned about Code
Review & Code Quality today. (Do not copy phrases from the lecture slides)
Refactor this piece of C++ Code and explain why your code is cleaner:
if(user!=nullptr){if(user->active()){if(user->permission("edit")){/* main logic */}}}
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slides use images generated with Gemini, from Flaticon.com (Creators: Freepik)

```
