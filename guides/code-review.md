# Code Review (CS35L Lecture 18 â€” Code Review & Code Quality)

> *"There are two ways of constructing a software design: One way is to make it so simple that there are obviously no deficiencies and the other way is to make it so complicated that there are no obvious deficiencies. The first method is far more difficult."* â€” **Tony Hoare (Turing Award 1980)**

This guide focuses on the **Code Review** portion of L18. The companion guide `clean-code.md` covers the Four Pillars (Meaningful Naming, Simplified Structure, Purposeful Documentation, Design by Contract). The two topics overlap thematically â€” clean code matters precisely because reviewers (and future maintainers) must read it â€” but this guide leans into the empirical research on modern code review, the gap between expectation and reality, and the practical effectiveness levers.

---

## Learning Objectives

After this lecture, you should be able to:

- **Explain why clean code matters**
- **Evaluate** whether code is clean or unclean
- **Refactor** code to become cleaner

(These are the lecture-wide objectives. Although phrased around clean code, they directly drive code review: a reviewer's primary job is evaluating cleanliness and pushing the author toward refactorings.)

---

## Research on Modern Code Review

The lecture grounds code review in three large empirical studies. These are the citations you should know:

- **Bacchelli & Bird, ICSE 2013** â€” *"Expectations, Outcomes, and Challenges of Modern Code Review."*
- **Czerwonka et al., ICSE 2015** â€” Microsoft CodeFlow study.
- **Sadowski, SÃ¶derberg, Church, Sipko, & Bacchelli, ICSE-SEIP 2018** â€” *"Modern Code Review: a Case Study at Google."*

### Bacchelli & Bird (ICSE 2013): Developers' Motivations

**Ranked Motivations from Developers** (Figure 3 of the paper), in descending order of how often developers cited each as a Top / Second / Third motivation:

1. **Finding Defects** (the most frequently cited motivation)
2. **Code Improvement**
3. **Alternative Solutions**
4. **Knowledge Transfer**
5. **Team Awareness**
6. **Improve Dev Process**
7. **Avoid Build Breaks**
8. **Share Code Ownership**
9. **Track Rationale**
10. **Team Assessment**

So developers *believe* they review code primarily to find bugs.

### Expectation vs. Reality

- **Expectation:** Code reviews find bugs.
- **Reality:** Code review **transfers knowledge, improves readability and maintainability of the code, and sometimes also finds bugs.**

The mismatch is the central insight of modern code review research.

### Frequency of Comments by Card Sort Category (Figure 4)

When Bacchelli & Bird actually card-sorted the comments left in CodeFlow reviews, the distribution looked nothing like the developers' stated motivations:

- **Code Improvements** â€” the most frequent category, with **165 (29%) comments**, described as "code improvements" in detail
- **Understanding** â€” roughly 22% (second most frequent)
- **Social Communication** â€” ~15%
- **Defects** â€” ~14%
- **External Impact** â€” ~5%
- **Testing** â€” ~5%
- **Review Tool** â€” ~3%
- **Knowledge Transfer** â€” ~2%
- **Misc** â€” ~6%

Defects are only the **fourth** most common category of comment â€” *behind* code improvements, understanding, and social communication.

### Level of Understanding Needed (Figure 5)

The paper also surveyed developers about how much code understanding each outcome required. Two outcomes demand the most understanding:

- **Finding Defects** â€” needed *High* or *Complete* understanding
- **Alternative Solutions** â€” needed *High* or *Complete* understanding

Other outcomes (Share Code Ownership, Knowledge Transfer, Team Assessment, Code Improvement, Improve Dev Process, Team Awareness, Track Rationale, Avoid Build Breaks) can be done with *Low* or *None* level of understanding.

**Implication:** Defect finding requires deep understanding â€” but reviewers don't have time for that.

---

## Why Code Review Does Not Find Bugs / How Current Code Review Practices Slow Us Down

The lecture has two slides under this title. Both are critical.

### Slide 1 â€” Distribution of Review Comments (from Bacchelli & Bird):

- **Top 3 categories are long-term maintenance issues:**
  - **Comments, naming, and styles (22%)**
  - **Organization of code (16%)**
  - **Alternative solutions for long-term maintenance (9%)**
- **Only 15% of comments provided by reviewers indicate a possible defect.**
- **Long-term maintainability issues are a much larger portion of comments (50%).**
- **Only 33% of comments are deemed useful by the author.**

Takeaway: two-thirds of reviewer comments are not useful to the author, and only 15% of comments concern defects.

### Slide 2 â€” Czerwonka et al. (Microsoft, ICSE 2015):

- They **mined code review tool usage data**.
- **CodeFlow is open 5 or 6 hours per developer per day.** (the review tool stays open most of the workday)
- **30 minutes of interaction time per developer per day.** (actual hands-on time is small)
- **Primary goals:** Find defects, improve maintainability, share knowledge, broadcast progress.

The contrast between "5-6 hours open" and "30 minutes of interaction" is the core data point: reviewers spend much less *engaged* time with each PR than the wall-clock suggests, which is part of why bugs slip through.

---

## Real-World Benefits of Code Reviews (Sadowski et al., Google, ICSE-SEIP 2018)

- **Code Review is the primary way to teach junior developers.**

- Direct quote from the Google study:

  > *"Expectations for code review at Google do not center around problem solving. Reviewing was introduced at Google to **ensure code readability and maintainability**. Today's developers also perceive this educational aspect, in addition to maintaining norms, tracking history, gatekeeping, and accident prevention. **Defect finding is welcomed but not the only focus.**"*

- Code Review at Google was introduced to *"force developers to write code that other developers could understand."*

**Key non-defect-finding purposes Google attributes to code review:**

- Ensuring **code readability and maintainability**
- **Educational aspect** (teaching juniors)
- **Maintaining norms**
- **Tracking history**
- **Gatekeeping**
- **Accident prevention**

**Source citation (memorize for the exam):** Sadowski, C., SÃ¶derberg, E., Church, L., Sipko, M., & Bacchelli, A. *Modern Code Review: a Case Study at Google.* International Conference on Software Engineering: Software Engineering in Practice (ICSE-SEIP 2018).

---

## How to Maximize Code Review Effectiveness

Three practical levers, all backed by the research:

### 1. 1â€“2 Reviewers

- **More reviewers increase the bystander effect** â€” each reviewer feels less personal responsibility because "someone else will catch it."
- Optimal is **one or two** assigned reviewers; more produces diminishing returns and worse reviews, not better ones.

### 2. Keep the Review Short

- **Review quality degrades after 1 hour of review.** After that point, reviewers are skimming, not understanding.
- **Keep pull requests roughly between 200 and 400 lines of code.**
- **Long pull requests with thousands of changes are not read in detail and often receive only superficial comments.** A 5000-line PR will get rubber-stamped or get nitpicks only â€” not deep defect analysis.

### 3. Stack Smaller PRs Instead of Bundling

- Instead of bundling multiple changes into a single massive update, **break them down into smaller, logically independent PRs that can be "stacked"**.
- **Stacked PRs**: a chain of related PRs, each building on the previous, each independently reviewable.
- Benefits of smaller, stacked PRs:
  - **Reviewed more thoroughly**
  - **Encounter fewer rebase conflicts**
  - **Merged significantly faster**

---

## Takeaway from Modern Code Review Practices

Five summary points from the lecture's "Takeaway" slide:

1. **New reviewers learn fast but need at least 6â€“12 months to be productive as the rest of the team.** Becoming an effective reviewer is a multi-month process tied to learning the codebase.
2. **People do not usually find bugs and focus on maintainability instead.** This is the empirical reality.
3. **We need to have "rigorous criteria" to find bugs.** Casual reading won't surface them.
4. **Developers need to have "critical eyes" for reviewing code changes, thinking about corner cases.**
5. **Identifying pre- and post-conditions is useful to help students develop as "effective code reviewers for defect finding."** (This is the lecturer's pedagogical bridge to Design by Contract â€” covered in `clean-code.md`.)

---

## Why Clean Code Matters (Why Reviewability Matters)

The lecture motivates clean code with reader-centric statistics. These directly justify code review:

- Code is **read â‰ˆ 10â€“14 times more often than it is written.** Investing time into making code easier to read pays off.
- **Developers spend 58% of their time on understanding code.** A speedup in code understanding has a large impact on productivity.

If you accept these numbers, code review is the discipline that prevents read-hostile code from entering the codebase in the first place.

### The "Broken Window" Effect

- When developers make changes to modules that have many **code smells**, they are **more likely to include code smells in their code as well**.
- **Technical debt propagates and compounds.**
- Sources: *https://dev.to/wmattei/the-broken-windows-theory-in-software-development-why-you-must-fix-everything-now-fdp* and W. LevÃ©n et al., *"The broken windows theory applies to technical debt,"* Empirical Software Engineering 2024.

This is why reviewers should resist letting "just one more" smell slip through â€” each one increases the likelihood of further smells.

---

## Code Comprehension: Bottom-Up vs. Top-Down (Reviewer Cognition)

How reviewers actually read code matters for designing review-friendly code:

### "Bottom Up" (Novices)

- Reading code **line by line**
- **Grouping** statements into logical "chunks" of higher abstraction
- Takes **more time** because each statement needs to be analyzed

### "Top Down" (Experts)

- Driven by a **hypothesis** of what the code does
- Looking for **"beacons"** (key lines of code that reveal the core purpose of a function or test the hypothesis)
- First getting the **big picture**, then **looking for details**

**Move from here to there:** novices grow into experts, shifting from bottom-up to top-down. Code that supports beacon-based reading (clear names, single-responsibility functions, obvious entry points) is easier to review.

---

## What Reviewers Actually Look For (the Four Pillars â€” pointer)

The lecture organizes the reviewer's checklist around **Four Pillars of Clean Code**:

1. **Meaningful Naming**
2. **Simplified Structure**
3. **Purposeful Documentation**
4. **Design by Contract**

> These four pillars are covered in detail in `clean-code.md`. For code review, the practical point is that these pillars define *what reviewers comment on*. Empirically (Bacchelli & Bird), 22% of comments target comments/naming/styles, 16% target organization of code (structure), and 9% target alternative solutions â€” together they dominate the review feedback distribution.

---

## Boy/Girl Scout Rule for Reviewers and Authors

> **"Always Leave the Campground Module Cleaner Than You Found It."**

(The lecture strikes through "Campground" and replaces it with "Module.")

- Whenever you make changes to a module, **consider if there are possible improvements**.
- **Before making the change, consider if there are negative consequences (did you miss something?)**

The cartoon caption: *"While I'm here, let me remove this dependency and cleanup the code."*

The flip side, illustrated with the *"Why? Why? Why? â€” Oh that's why"* meme, is that not every cleanup is safe â€” code often exists for reasons that aren't obvious. **Investigate before you delete.** This is itself a code review heuristic: when a reviewer asks "why is this here?", the author should be able to answer.

### Before You Refactor, Make Sure You Have Good Tests

Refactoring â€” whether prompted by your own scout-rule instinct or by a reviewer's comment â€” is only safe with a **Comprehensive Test Suite**: `test_user_creation`, `test_invalid_password`, `test_payment_succeeds`, `test_database_connection`, `test_api_response`, `test_database`, etc. Without tests, refactoring is a guess.

---

## Cross-References to Clean Code Principles (Reviewer Checklist)

Reviewers should evaluate code against the Four Pillars. The full treatments live in `clean-code.md`; below is the brief reviewer-facing checklist with the lecture's specific examples.

### Naming (review for):

- Wrong/misleading names (e.g., `discount_percent` holding a value in `[0,1]` instead of `[0,100]` â€” **worse** than calling it `v14`, because it gives the reader a wrong hypothesis they then rely on).
- Vague vs. precise (`discount_percent` is better than `discount`).
- Shorter is *mostly* better (full words usually beat one-letter names; well-understood abbreviations beat long full words).
- Commonly used names over creative inventions (e.g., `i` as a loop index is fine â€” experienced developers expect it).
- Class names = nouns (`Transfer`, not `Transfering`). Method names = verbs (`hasPermission()`, not `permission()`; `isActive()` not `active()`).
- Language conventions: Python `snake_case`, C++ `camelCase`.

### Structure (review for):

- **Deep nesting** â€” high complexity correlates strongly with bug rates and time-to-understand. Avoid nesting level 15+. Reference: J. Johnson et al., *"An Empirical Study Assessing Source Code Readability in Comprehension,"* ICSME 2019.
- **Guard clauses** instead of wrapping the entire function in nested `if` blocks â€” handle the negative/edge case first and return early.
- **Chunking and limited function size** â€” human working memory holds only 5â€“9 distinct pieces. A function should fit on one screen; if the reader has to scroll, they lose the context in working memory. Extract methods to let readers chunk.

### Documentation (review for):

- Misleading/outdated comments (e.g., a comment claims a regex matches dates when it actually matches emails â€” **worse than no comment**).
- Comments that just restate the code (`n += 1  # increment n` adds nothing).
- Prefer renaming/refactoring over commenting (e.g., `count_even_numbers` is better than `count_numbers` with a `# found even number` comment).
- **Good** comments explain *intent* and *why* (e.g., `# Match arbitrary top-level domains to avoid updating when new TLDs are added`).

### Design by Contract (review for):

- Explicit **pre-conditions** (assumptions on inputs / object state) and **post-conditions** (guarantees on outputs / object state) â€” these form the **"visible contract"** of the module while the implementation stays hidden.
- Use **assertions** to make assumptions explicit:
  - Python: `assert 0 <= price, "Invalid price"` â€” turn off with `python -O my_program.py`.
  - C/C++: `#include <assert.h>`, `assert(0 <= price);` â€” turn off with `gcc -DNDEBUG main.c -o my_program`.

---

## Quick-Reference Reviewer Mental Model

Putting the empirical findings together into the mental model an effective CS35L code reviewer should carry:

1. **Default reality:** Most of your comments will be about maintainability (naming, structure, alternatives) â€” that's fine and that's where you'll add the most value. Only ~15% of comments cite defects.
2. **To find bugs you must apply rigorous criteria** â€” read line-by-line, develop and test hypotheses, examine corner cases, identify pre/post-conditions. Defect-finding requires *High* or *Complete* understanding.
3. **Keep the PR shape reviewable:** 200â€“400 LOC, 1â€“2 reviewers, â‰¤1 hour of review. Stack PRs instead of bundling. Author and reviewer share responsibility for this.
4. **Use code review as a teaching mechanism** â€” it's the primary way junior developers learn. New reviewers themselves need 6â€“12 months before they're as productive as senior teammates.
5. **Code is read 10â€“14Ã— more than written**; developers spend 58% of time understanding code. Every review comment that makes future readers faster has compounding value.
6. **Resist broken windows** â€” every smell you let through makes future smells more likely.
7. **Scout rule both ways:** improve what you touch, but check that the existing code isn't there for a reason you don't yet see.

---

## Citations to Remember

- Bacchelli, A., & Bird, C. *Expectations, Outcomes, and Challenges of Modern Code Review.* ICSE 2013. â€” source of the motivations chart, comment-category card sort, and understanding-level survey.
- Czerwonka, J., et al. (Microsoft). ICSE 2015. â€” CodeFlow usage mining: 5â€“6 hours open/day, 30 min interaction; goals = find defects, improve maintainability, share knowledge, broadcast progress.
- Sadowski, C., SÃ¶derberg, E., Church, L., Sipko, M., & Bacchelli, A. *Modern Code Review: a Case Study at Google.* ICSE-SEIP 2018. â€” "force developers to write code that other developers could understand"; defect finding is welcomed but not the only focus; primary way to teach juniors.
- Johnson, J., et al. *An Empirical Study Assessing Source Code Readability in Comprehension.* ICSME 2019. â€” complexity correlates with bugs and time-to-understand.
- LevÃ©n, W., et al. *The broken windows theory applies to technical debt.* Empirical Software Engineering 2024.


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
