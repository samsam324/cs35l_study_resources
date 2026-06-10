# CS 35L Lecture 12 - Design With Reuse

Instructor: Tobias Durschmid (Assistant Teaching Professor, Computer Science Department, UCLA Samueli School of Engineering)

---

## Lecture Roadmap - Questions This Lecture Answers

- What are **advantages** of reusing existing modules?
- What **challenges** might arise from reusing existing modules?
- How to **decide** whether to reuse a module?
- How to **reduce the risk** of negative consequences of reuse?
- How to **make & document design decisions**?

---

## Why Reuse? (Instead of Re-Implementing)

Two primary benefits of software reuse:

### 1. Higher Productivity / Faster Time to Market
Reusing software can **speed up software development** because time for implementation and testing may be reduced.

### 2. Higher Software Quality / Fewer Defects
Reused software, which has been **tried and tested** in working systems, should be **more dependable** than new software, since most bugs have likely been found already by other users of the module.

> Reference: "What software reuse benefits have been transferred to the industry? A systematic mapping study" (Jose L. Barros-Justo et al. 2017)

---

## The Vision vs. Reality of Reuse

### The Vision of Reuse
**Creating New Software Mostly by Composing Existing Building Blocks.**

- Well-organized libraries of modules that are **highly compatible** with each other.
- Pre-built modules (Library 1, Library 2, ...) snap together cleanly like Lego bricks to form a System Design composed of stacked, interlocking modules.

> Reference: "Mass Produced Software Components" (Malcolm Douglas McIlroy 1968)

### The Reality of Reuse
**Modules are Partially Incompatible But Often Still Glued Together.**

- Cluttered libraries of modules that make **many undocumented assumptions**.
- In practice, modules from different libraries do not fit together cleanly; developers use **"glue" code** to force them to work together. The resulting system design is a misshapen collection of modules held together by glue.

> Reference: "Architectural Mismatch: Why Reuse Is (Still) So Hard" (David Garlan et al. 1995 and 2009)

---

## Reuse Must Be Approached Differently Depending on Its Source

| Internal Reuse | External Reuse |
| --- | --- |
| Code was written by the **same developer, team, or organization** that is reusing it. | Code was written by a **third party**. |
| Examples: product lines, component-based development process, ... | Examples: commercial off-the-shelf (COTS), open-source libraries, packages, frameworks. |

Each reuse type requires different design strategies (covered later).

---

# Part 1: How to Design with External Reuse?

## The Python Ecosystem Is Built on Reuse

Two key properties make the Python ecosystem reuse-friendly:

1. **Most commonly needed functionality is already implemented in a reusable way.**
2. **Low Entry Barrier:** Importing & starting to use reusable modules is easy.

Example install command:
```
$ pip install requests
```

Example usage in the Python REPL:
```python
>>> import requests
>>> response = requests.get("https://api.github.com")
>>> response.status_code
200
>>> response.json()
{'current_user_url': 'https://api.github.com/user', ...}
```

The Python ecosystem shows transitive dependency layers:
- `docker` depends on `requests`, which depends on `urllib3`.
- `pandas` depends on `numpy`.
- `tensorflow` depends on `scipy`, which depends on `numpy`.

---

## Example: Python Package Update Has API-Breaking Change

**Context:** No source code changes. Python's `docker` package imports the `requests` package and the `urllib3` package.

**Error Message:**
```
docker.errors.DockerException: Error while fetching server API version:
request() got an unexpected keyword argument 'chunked'
```

**Root Cause:** `urllib3 2.0.0` was just released, and it changed its API to be **incompatible with `requests`**.

Offending code inside the `requests` package:
```python
# in requests package
httplib_response = self._make_request(
    conn,
    method,
    url,
    timeout=timeout_obj,
    body=body,
    headers=headers,
    chunked=chunked,
)
```
The `chunked=chunked` keyword argument is the one urllib3 2.0.0 no longer accepts.

**Question raised:** How can we prevent this from happening?

---

## Design Principle: Keep Versions of Your Dependencies Fixed

- Most package managers allow you to **specify the versions of dependent packages** and install them in a **virtual environment locally to the project**.
- **For Python:** Use **Pipenv** & **Pipfiles**.

**Avoid (no version pinning, global install):**
```
$ pip install requests
$ python <program>
```

**Use instead (pinned, isolated in virtual env):**
```
$ pipenv install requests
$ pipenv run <program>
```

### Example Pipfile
```
[packages]
urllib3 = "<2.0.0"
docker = "==7.1.0"

[dev-packages]
pep8-naming = "==0.10.0"
mypy = "==0.910"
pytest = "==5.4.2"
tox = "==3.15.1"

[requires]
python_version = "3.9"
```

> See: https://pipenv.pypa.io/

---

## Case Study: Heartbleed Bug in OpenSSL

**Key Takeaway:** Reusable Packages can introduce **Security Vulnerabilities**.

OpenSSL sits between Component A and Component B, providing an **Encrypted Connection via SSL/TLS**. OpenSSL included an **insecure implementation of Heartbeat**, leading to a **buffer over-read**, **leaking memory data**.

### Timeline
| Date | Event |
| --- | --- |
| February 2012 | Bug **introduced** |
| 1 April 2014 | Bug **discovered** |
| 7 April 2014 | **Fixed version released** |
| 7 April 2014 | **17% of all secure web servers vulnerable** |
| 20 May 2014 | 1.5% of most popular TLS-enabled websites still vulnerable |
| January 2017 | 180k internet-connected devices still vulnerable |
| July 2019 | 91k devices still vulnerable |

**Discussion prompt:** "What can we learn from this bug? Talk to your Neighbor(s)."

---

## Design Principle: Update Your Dependencies To Receive Bug Fixes & Security Patches

- Reusing **well-maintained** modules can improve your software quality.
- Defects in popular modules are usually **fixed quickly**.
- **Regularly check** for security patches & bugfixes.
- Be aware of the **side effects** of updates (see the urllib3 example above - an update can also break things).

---

## Case Study: `left-pad` - A Simple and Highly Reused npm Package

`left-pad` adds characters in front of a string for alignment with just **11 lines of code**.

```javascript
module.exports = leftpad;
function leftpad (str, len, ch) {
  str = String(str);
  var i = -1;
  ch || ch = ' ';
  len = len - str.length;
  while (++i < len) {
    str = ch + str;
  }
  return str;
}
```

**Stats:**
- Stars on GitHub: **10**
- Weekly downloads: **~ 1M**

**Usage scope:** Transitively, used in big popular packages (e.g., **React**, **Babel**), which are used by most modern web apps (Facebook, Spotify, Netflix, etc.).

**Discussion prompt:** "What does it do? Talk to your neighbor(s). How could we refactor this code?"

---

## Case Study: How Reusing Just 11 Lines (`left-pad`) Broke the Entire Internet

- **March 22, 2016:** Due to a dispute with npm, the author of `left-pad` decides to **unpublish** all his packages (including left-pad).
- **Build processes for web apps across the internet broke** due to the missing package.

  Error seen everywhere:
  ```
  npm ERR! 404 'left-pad' is not in the npm registry
  ```
- Many developers did not even know that they were **transitively relying** on `left-pad`.
- **2 hours after the original `left-pad` package was removed, npm manually "un-unpublished" it.**

> References: https://www.davidhaney.io/npm-left-pad-have-we-forgotten-how-to-program/
> https://qz.com/646467/how-one-programmer-broke-the-internet-by-deleting-a-tiny-piece-of-code

### Discussion: Rules for Reusing Developers to Prevent This
Small-group discussion prompts:
- How should we decide **what to reuse**?
- How can we **minimize the risk** of reuse?

Companion example: `isArray` - another trivial package with **129 stars** and **~ 92 million weekly downloads**:
```javascript
return toString.call(arr) == '[object Array]';
```

---

## Design Principle for Design With Reuse: Strive for Fewer Package Dependencies

- **Avoid reusing trivial code**, especially from unreliable sources.
- **Carefully consider** adding new package dependencies:
  - Every dependency can break, or **stop being supported**.
  - Package dependencies can become a **security vulnerability** (e.g., the `eslint-scope` malicious update).
- **Analyze your supply chain** for weaknesses and risks.

> See: https://eslint.org/blog/2018/07/postmortem-for-malicious-package-publishes/

---

## Modules with Higher Maintenance Level & Popularity Are More Viable Reuse Candidates

Recap: "2 hours after the original `left-pad` package was removed, npm manually 'un-unpublished' it, **because it broke the entire internet**."

Evaluation criteria for reuse candidates:

- **Maintenance activity:** How actively does the development team **fix bugs** and update the module to support **new platforms**? Do they commit often?
- **Popularity:** Popular packages with many users are more likely to **resolve issues quickly** and have **better documentation**.
- However, **fit to your context is more important than popularity!**

---

## Lesson Learned: External Reuse Is Not a One-Time Investment!

(Key takeaway from the previous examples.)

- **Important updates** (e.g., fix security vulnerabilities) might come with **API-breaking changes** if you have skipped previous versions.
- **Poorly maintained packages** might require you to **abandon them later**.
- Relying too much on reused code **limits changeability** once you need more than what the library offers.

---

## Cost-Benefit Analysis for External Reuse

A scale balancing two sides:

### Effort to Adapt the Reusable Module (Costs)
- **Integration Effort** (Complexity, Similarity of Context)
- **Finding** the Module
- **Updating** Effort
- Limiting **Changeability**

### Effort Saved by Reusing the Module (Benefits)
- **Implementation** Effort (saved)
- **Testing** Effort (saved)
- Benefit of **Update Propagation**

> Reference: "Why reinventing the wheels? An empirical study on library reuse and re-implementation" (Xu et al. 2019)

---

## Practice Exercise: Should You Reuse These Packages?

**Context:** Building an **appointment scheduling system**. For each package, decide individually whether it is a good reuse candidate, with pros and cons.

### `python-constraint`
Provides a simple constraint satisfaction problem (CSP) solver in Python to identify a scheduling solution for multiple users.
- **Pro:** Reusing a **hard-to-implement** functionality of **medium size (1.3k LOC)**.
- **Con:** **Limits changeability** (what if we want priority scheduling instead of global optimization?).

### `icalendar`
Generates, parses, and manipulates iCalendar data to send invitations to users.
- **Pro:** Reusing a **large amount** of **hard-to-implement** functionality.
- **Pro:** **High changeability** due to **local change impact**.

---

# Part 2: How to Design with Internal Reuse?

## NASA Heavily Relies on Internal Reuse

- **Problem:** Creating appropriate integration & system-level **tests** for space craft software is **difficult on Earth**.
- **NASA's Solution:** Only **trust software that has worked in space**.

Trust model:
- **Tested in Space -> TRUSTED**
- **Tested on Earth -> UNTRUSTED**

---

## Case Study: The Ariane 5 Failure

A canonical case study in misapplied internal reuse.

### What Happened
- **Ariane 4 Flight Control System:**
  - Inertial Reference System used **Horizontal Velocity** stored as a **16 Bit Int**.
  - **Assumed lower velocity** (due to a **Performance Requirement**).
  - **Worked perfectly!**
- **Ariane 5 Flight Control System:**
  - **Reused the Ariane 4 Inertial Reference System**, including the same 16-bit int Horizontal Velocity.
  - However, Ariane 5 can **reach higher velocities than Ariane 4**.
  - The higher velocity triggered an **Overflow Error**.
  - This **Caused Self-Destruction**: **$370M loss in 37 seconds**.

> See: http://esamultimedia.esa.int/docs/esa-x-1819eng.pdf

### Discussion (Small-Group)
"Describe Reuse Rules that Avoid Failures Like This."

---

## Lesson Learned from Ariane 5

> **Software that Worked in one Context Might Not Work in Another Context.**

The official Inquiry Board report recommendation (R5):

> **R5 - Review all flight software (including embedded software), and in particular:**
> *Identify all implicit assumptions made by the code and its justification documents on the values of quantities provided by the equipment. Check these assumptions against the restrictions on use of the equipment.*

> See: https://www.esa.int/Newsroom/Press_Releases/Ariane_501_-_Presentation_of_Inquiry_Board_report

---

## Design Principle for Internal Reuse: Identify Violated Assumptions

- Check documentation and code to **identify assumptions** made by a reuse candidate.
- Check to make sure that reusable software was designed to operate reliably **under the conditions you want**.
- **Don't assume the code of the reuse candidate is correct; test it!**

---

## Cost-Benefit Analysis for Internal Reuse

### Effort to Adapt the Reusable Module (Costs)
- Identification of **Implicit Assumptions**
- Effort to **Create / Identify Reusable Modules**

### Effort Saved by Reusing the Module (Benefits)
- **Implementation** Effort (saved)
- **Testing** Effort (saved)
- Benefit of **Update Propagation**

---

## Library vs. Framework

### The "Hollywood Principle" of Inversion of Control
> "Don't call us. We'll call you." Frameworks primarily allow you to **define callbacks**.

| Library | Framework |
| --- | --- |
| Your Code -> calls -> Library | Framework -> calls -> Your Code |
| Your code makes **direct calls** to the library API to **use their functions now**. | Your code defines **callbacks** to be **called later** by the framework. |

### Library Example (Axios)
```javascript
const response = await axios.get('/user?ID=12345');
console.log(response);
```

### Framework Example (Express)
```javascript
app.get('/', (req, res) => {
  res.send('Hello World!')
})
```

### Trade-offs of Frameworks vs. Libraries
- A **framework makes more decisions for you** and gives you **less flexibility**.
- But it also **hides a lot of complexity**, which often means **you write less code**.
- Hence, decisions to use a framework are often **harder to reverse later**.

---

# Part 3: How to Make Design Decisions?

## Which Team Created a Better Design?

A pedagogical comparison:

- **Team A:** Produced **one** detailed design option.
- **Team B:** Produced **three** design options.
- **Team C:** Produced **five** design options.

The winning team (highlighted as the "better" design) is the one producing **more** alternatives - Team B beats Team A, and Team C beats Team B.

> Reference: Petre, Marian "Insights from Expert Software Design Practice" in European Software Engineering Conference and Symposium on The Foundations of Software Engineering (ESEC/FSE 2009)

---

## Lesson Learned: Think of Many Design Alternatives

- If you can think of a good design, try to **think of a better one**.
- Think broadly about a **diverse** range of solutions.
- Research has shown: When simply prompting designers to **consider other design alternatives**, designers with less experience create **better designs**. [1]

> [1] Tofan, Dan, Matthias Galster, and Paris Avgeriou. "Difficulty of architectural decisions-a survey with professional architects." European Conference on Software Architecture (ECSA 2013).

### Generate then Evaluate
The design process is a two-phase diamond:

- **Generate:** The purpose of idea **generation is to broaden up** (produce many candidate options: squares, circles, triangles, pluses, stars, etc.).
- **Evaluate:** **Narrowing down** of ideas is done in the **evaluation activity** (one option, e.g., the star, is selected).

---

## Delaying Decisions

- Identify design decisions **that need more information** or that are likely to change later.
- Attempt to **design your system without assuming a solution** for these difficult decisions.
- Keep a **list of delayed decisions** and keep track of what you need to resolve them.

---

## Design Exercise: Divide and Conquer

**Prompt:** Design an interplanetary messaging system for people living on **Earth** and **Mars** to communicate.

**What sub-problems would you need to solve?**

Sample answer:
- Problems that arise when messaging **on Earth**
- Added **infrastructure**
- **Networking** with largely varied distances
- **Different definition of a day**

(Imagery: Sun in the center, with Earth on the inner orbit and Mars on the outer orbit.)

---

## Lesson Learned: Solve Simpler Problems First

- When faced with a complex problem, experts **solve a simpler problem first**.
- Solution to the simpler problem might be incomplete but can be **extended later**.
- **Be aware** when the simpler problem is so **fundamentally different that solutions do not generalize**.

---

## Making Design Decisions - The "Rational" Decision Making Process

Has been found to **lead to better design** (especially effective for early-career software engineers). [2]

- **Step 1:** Identify your requirements. What is important?
- **Step 2:** Think of many design alternatives.
- **Step 3:** Evaluate how well your design alternatives help you implement the requirements from Step 1.
- **Step 4:** Consider the trade-offs and make a decision.

> [2] Tang, Antony, et al. "Design reasoning improves software design quality." International Conference on the Quality of Software Architectures. Springer. 2008.

---

## At Google: "Design Docs" Before Implementation

At Google, before the implementation of a new system, the developers write a **"Design Doc"**.

> "As **software engineers our job is** not to produce code per se, but rather **to solve problems**. Unstructured text, like in the form of a **design doc**, may be **the better tool for solving problems early in a project lifecycle**, as it may be more concise and easier to comprehend, and **communicates the problems and solutions at a higher level than code**." - Malte Ubl

### Goals of a Design Doc
- **Early identification of design issues** when making changes is still cheap.
- Achieving **consensus** around a design in the organization.
- **Transferring knowledge** from senior engineers into the organization.
- Form the basis of an organizational **memory around design decisions**.
- Acts as a summary artifact in the **technical portfolio** of the software designer(s).

> See: https://www.industrialempathy.com/posts/design-docs-at-google/

---

## "Design Docs" Consist of These Parts

Design Docs are practiced by: **Google, AWS, Azure, Kubernetes, Shopify**.

### 1. Context & Scope - bring the reader up to speed
**Background facts** that are relevant to understand the Design Doc.

### 2. Goals & Non-Goals - define the Scope
- **Goals:** Requirements & quality attributes.
- **Non-Goals:** What can be **ignored to simplify development**?

### 3. The Design - describes the selected solution
**Models & design descriptions** (e.g., context diagram, data model, API descriptions, pseudo-code, constraints, ...).

### 4. Alternatives - motivate why this decision was made
**Trade-offs** that each respective design makes and how those trade-offs led to the decision to select the design that is the primary topic of the document.

> See: https://www.industrialempathy.com/posts/design-docs-at-google/

---

## Exit Ticket Questions (Bruin Learn)

- **Question 1 (1 pt):** In your own words, please summarize **three design principles** you learned about Design With Reuse today. (Do not copy phrases from the lecture slides.)
- **Question 2 (1 pt):** Describe the **pros and cons** of using the **axios** library versus the **fetch** API for HTTP requests in Node.JS. **Decide which one is more appropriate for your course project and why.**
- **Question 3:** Please leave any questions that you have about today's material and things that are still unclear or confusing to you (if none, simply write N/A).

---

## Quick-Reference Summary of Design Principles

1. **Keep versions of your dependencies fixed** (use Pipenv/Pipfile; pin exact versions in a project-local virtual environment).
2. **Update your dependencies** to receive bug fixes & security patches - but be aware of side effects.
3. **Strive for fewer package dependencies** - avoid reusing trivial code; analyze your supply chain for risks.
4. Prefer modules with **higher maintenance level and popularity**, but fit-to-context outweighs popularity.
5. Remember: **External reuse is not a one-time investment** - updates, abandonment, and changeability limits are ongoing costs.
6. Perform a **Cost-Benefit Analysis** for both External and Internal reuse before committing.
7. For internal reuse: **Identify violated assumptions** - check that the conditions in the new context match those in the original context (Ariane 5 lesson).
8. **Think of many design alternatives** - Generate broadly, then Evaluate to narrow down.
9. **Delay decisions** that need more information; track them explicitly.
10. **Solve simpler problems first**, but watch for non-generalizable simplifications.
11. Use the **Rational Decision Making Process**: requirements -> alternatives -> evaluation -> trade-offs -> decision.
12. **Document design decisions** in a Design Doc (Context & Scope, Goals & Non-Goals, The Design, Alternatives).


---

# Appendix: Lecture 12 Slides (raw extracted text)

The following is the full extracted text of Lecture 12. Preserved verbatim so no information is lost.

```text
CS 35L Software
Construction
Lecture 12 ­ Design
With Reuse
Assistant Teaching Professor
Computer Science Department
This Lecture ­ Design with Reuse
· What are advantages of reusing existing modules?
· What challenges might arise from reusing existing modules?
· How to decide whether to reuse a module?
· How to reduce the risk of negative consequences of reuse?
· How to make & document design decisions?
Why Reuse? (Instead of Re-Implementing)
     Higher Productivity / Faster Time to Market
Reusing software can speed up software development because time for
implementation and testing may be reduced.
     Higher Software Quality / Fewer Defects
Reused software, which has been tried and tested in working systems, should
be more dependable than new software, since most bugs have likely been
found already by other users of the module.
See "What software reuse benefits have been transferred to the industry? A systematic mapping study" (José L. Barros-Justo et al. 2017)
The Vision of Reuse: Creating New Software
Mostly by Composing Existing Building Blocks
Library 1  Library 2                                                              System Design
                             Well-organized libraries of modules that are
                                  highly compatible with each other.
Read more in: "Mass Produced Software Components" (Malcolm Douglas McIlroy 1968)
           CS 35L Software Construction: Lecture 12 ­ Design with Reuse                          4
           Tobias Dürschmid
The Reality of Reuse: Modules are Partially
Incompatible But Often Still Glued Together
Library 1  Library 2
Module     Module                                            Module
           Module                                  Module
                             Cluttered libraries of modules that make
                               many undocumented assumptions
Read more in: "Architectural Mismatch: Why Reuse Is (Still) So Hard" (David Garlan et al. 1995 and 2009)
           CS 35L Software Construction: Lecture 12 ­ Design with Reuse                                   5
           Tobias Dürschmid
Reuse must be Approached Differently
Depending on its Source
         Internal Reuse                           External Reuse
Code was written by the same              Code was written by a third party.
developer, team, or organization          (e.g., commercial off-the-shelf, open-
that is reusing it (e.g., product lines,  source libraries, packages,
component-based development               frameworks)
process, ...)
How to
Design with
External Reuse?
Design With Reuse
CS35L Software Construction
                                                                                                                      7
The Python Ecosystem Is                                                                     Python
Built on Reuse                                                                            Ecosystem
 Most commonly needed functionality is                                                                          8
 already implemented in a reusable way
 Low Entry Barrier: Importing & starting to use
 reusable modules is easy: $ pip install requests
 >>> import requests
 >>> response = requests.get("https://api.github.com")
 >>> response.status_code
 200
 >>> response.json()
 {'current_user_url': 'https://api.github.com/user',
 ...}
                            CS 35L Software Construction: Lecture 12 ­ Design with Reuse
                            Tobias Dürschmid
Example: Python Package Update                                // in requests package
Has API-Breaking Change
                                                              httplib_response
 How can we prevent this from happening?                      = self._make_request(
  Context: No source code changes                                        conn,
  Python's docker package imports the requests                           method,
  package and the urllib3 package                                        url,
                                                                         timeout=timeout_obj,
                                                                         body=body,
                                                                   headers=headers,
                                                                   chunked=chunked,
                                                              )
Error Message: docker.errors.DockerException:
Error while fetching server API version: request()
got an unexpected keyword argument 'chunked'
Root Cause: urllib3 2.0.0 just released today! And it changed its API
to be incompatible with requests
Design Principle: Keep Versions of Your
Dependencies Fixed
· Most package managers allow you to             Example Pipfile
  specify the versions of dependent
  packages & install them in a virtual  [packages]
  environment locally to the project    urllib3 = "<2.0.0"
                                        docker = "==7.1.0"
· E.g., Python: Use Pipenv & Pipfiles
                                        [dev-packages]
                                        pep8-naming = "==0.10.0"
                                        mypy = "==0.910"
                                        pytest = "==5.4.2"
                                        tox = "==3.15.1"
                                        [requires]
                                        python_version = "3.9"
$ pip install requests                  $ pipenv install requests
$ python <program>                      $ pipenv run <program>
See more here: https://pipenv.pypa.io/
Heartbleed Bug Reusable Packages can
In OpenSSL                   introduce Security Vulnerabilities
What can we learn from this bug? Talk to your Neighbor(s)
             Encrypted Connection via SSL/TLS
Component A                                    Component B
Introduced in February 2012                           Included insecure
Discovered on 1 April 2014                     implementation of Heartbeat
Fixed version released on 7 April 2014         leading to a buffer over-read,
                                                     leaking memory data
7 April 2014 17% of all secure web servers vulnerable
20 May 2014 1.5% of most popular TLS-enabled websites still vulnerable
January 2017 180k internet-connected devices still vulnerable
July 2019    91k devices still vulnerable
           CS 35L Software Construction: Lecture 12 ­ Design with Reuse        11
           Tobias Dürschmid
Design Principle: Update Your Dependencies To
Receive Bug Fixes & Security Patches
· Reusing well-maintained modules
  can improve your software quality
· Defects in popular modules
  are usually fixed quickly
· Regularly check for security patches &
  bugfixes.
· Be aware of the side effects of updates
  (see previous example)
left-pad ­ A Simple and                                                                  Most Modern
Highly Reused npm Package                                                                  Web Apps
Wlefhta-tpaddoaedsdsit                module.exports = leftpad;                                                     13
                                      function leftpad (str, len, ch) {
dchoar?acTtearslkintforont of
ywa iosthturijnurgstfo11r  alignment     str = String(str);
                           lines of      var i = -1;
                                         ch || ch = ' ';
ncoedieg. hbor(s)                        len = len - str.length;
                                         while (++i < len) {
Transitively, used in
big popular                                  str = ch + str;
packages (e.g.,                          }
React, Bable),                           return str;
which are used by                     }
most web apps.                        Stars on GitHub: 10
                                      Weekly downloads:  1M left-pad
                                      How could we refactor this code?
                           CS 35L Software Construction: Lecture 12 ­ Design with Reuse
                           Tobias Dürschmid
left-pad ­ How Reusing Just                                                                    Most Modern
11 Lines Broke the Entire Internet                                                               Web Apps
 March 22, 2016: Due to a dispute with npm, the                                                      npm ERR! 404
 author of left-pad decides to unpublish all his                                                     'left-pad' is
 packages (including left-pad)                                                                       not in the
                                                                                                     npm registry
 Build processes for web apps across the internet
 broke due to the missing package                                                                                         14
 Many developers did not even know that they were
 transitively relying on left-pad
 2 hours after the original left-pad package was
 removed, npm manually "un-unpublished" it.
 Read more here: https://www.davidhaney.io/npm-left-pad-have-we-forgotten-how-to-program/ &
 https://qz.com/646467/how-one-programmer-broke-the-internet-by-deleting-a-tiny-piece-of-code
                            CS 35L Software Construction: Lecture 12 ­ Design with Reuse
                            Tobias Dürschmid
Learning from the left-pad story,                             Most Modern
Describe Rules for Reusing Developers                           Web Apps
that Prevent Issues like that
Small-Group           Talk to your
 Discussion            neighbor!
How should we decide  return toString.call(arr)
what to reuse?        == '[object Array]';
How can we minimize   Stars on GitHub: 129  isArray
the risk of reuse?
                      Weekly downloads:  92 million
Design Principle for Design With Reuse: Strive for
Fewer Package Dependencies
· Avoid reusing trivial code, especially from unreliable sources
· Carefully consider adding new package dependencies
   · Every dependency can break, or stop being supported
   · Package dependencies can become a security vulnerability
     (e.g., eslint-scope malicious update)
· Analyze your supply chain for weaknesses and risks
See https://eslint.org/blog/2018/07/postmortem-for-malicious-package-publishes/
Modules with higher Maintenance Level &
Popularity Are more Viable Reuse Candidates
 2 hours after the original left-pad package was removed, npm
 manually "un-unpublished" it, because it broke the entire internet
· How actively does the development team fix bugs and update the
  module to support new platforms? Do they commit often?
· Popular packages with many users are more likely to resolve issues
  quickly & have better documentation
· However, fit to your context is more important than popularity!
Lesson Learned: External Reuse
Is Not a One-Time Investment!
· Important updates (e.g., fix security vulnerabilities) might come
 with API-breaking changes if you have skipped previous
 versions.
· Poorly maintained packages might require you
 to abandon them later
· Relying too much on reused code limits changeability once you
 need more than what the library offers.
Cost-Benefit Analysis for External Reuse
Effort to adapt the      Effort saved by
reusable module          reusing the module
Integration Effort       Implementation Effort
(Complexity, Similarity  Testing Effort
of Context)
Finding the Module       Benefit of Update
Updating Effort          Propagation
Limiting Changeability
                         Read more here: Why reinventing the wheels? An
                         empirical study on library reuse and re-
                         implementation (Xu et al. 2019)
Should You Reuse these Packages?
Talk to Your Neighbor(s)
Context: Building an appointment scheduling system
Which of these packages are good reuse candidates? What are the pros
and cons of reusing them? Decide for each package individually.
python-constraint                                             icalendar
Provides a simple constraint                                  Generates, parses, and
satisfaction problem (CSP) solver in                          manipulates iCalendar data to
Python to identify a scheduling                               send invitations to users
solution for multiple users
     Reusing a hard-to-         Limits changeability (what        Reusing a large        High
implement functionality of  if we want priority scheduling    amount of hard-to-   changeability
 medium size (1.3k LOC)     instead of global optimization?)
                                                                   implement         due to local
                                                                   functionality   change impact
How to
Design with
Internal Reuse ?
Design With Reuse
CS35L Software Construction
                                                                                                                     21
NASA Heavily Relies on Internal Reuse
· Problem: Creating appropriate integration & system-level tests for space
  craft software is difficult on Earth
· NASA's Solution: Only trust software that has worked in space
TRUSTEDTested in Space  UNTRUSTEDTested on Earth
Ariane 5 Failure  Can reach higher velocities                                             $370M
                  than Ariane 4                                                           in 37s
Describe Reuse Rules that Assumed                                                                    23
Avoid Failures Like This lower velocity
Ariane 4 Flight Control System                      Due to
                                                Performance
  Worked                                        Requirement
Perfectly!
              Horizontal Velocity - 16 Bit Int
Ariane 5 Flight Control System
Caused Self-  Horizontal Velocity - 16 Bit Int  Overflow Error
 Destruction
See http://esamultimedia.esa.int/docs/esa-x-1819eng.pdf
                            CS 35L Software Construction: Lecture 12 ­ Design with Reuse
                            Tobias Dürschmid
Lesson Learned from Ariane 5:
Software that
Worked in one Context
Might Not Work in Another Context
See https://www.esa.int/Newsroom/Press_Releases/Ariane_501_-_Presentation_of_Inquiry_Board_report
Design Principle for Internal Reuse:
Identify Violated Assumptions
· Check documentation and code to identify assumptions
  made by a reuse candidate
· Check to make sure that reusable software
  was designed to operate reliably under the conditions you want
· Don't assume the code of the reuse candidate is correct; test it!
Cost-Benefit Analysis for Internal Reuse
Effort to adapt the  Effort saved by
reusable module      reusing the module
Identification of    Implementation Effort
Implicit             Testing Effort
Assumptions
Effort to Create /   Benefit of Update
Identify Reusable    Propagation
Modules
Library vs. Framework
    "Hollywood Principle" of Inversion of Control
"Don't call us. We'll call you". Frameworks primarily allow you to define callbacks
Your code makes      Your Code              Framework         Your code defines
direct calls to the                                            callbacks to be
library API to use            Calls                   Calls   called later by the
 their functions       Library              Your Code              framework.
         now
For example, in Axios:                      For example, in Express:
const response = await axios.get            app.get('/', (req, res) => {
                       ('/user?ID=12345');     res.send('Hello World!')
console.log(response);                      })
Library vs. Framework
    "Hollywood Principle" of Inversion of Control
"Don't call us. We'll call you". Frameworks primarily allow you to define callbacks
Your code makes      Your Code       Framework                Your code defines
direct calls to the                                            callbacks to be
library API to use            Calls            Calls          called later by the
 their functions       Library       Your Code                     framework.
         now
A framework makes more decisions for you and gives you less flexibility.
But it also hides a lot of complexity, which often means you write less code.
Hence, decisions to use a framework are often harder to reverse later.
How to
Make Design Decisions?
Design With Reuse
CS35L Software Construction
                                                                                                                     31
Which Team Created A Better Design?
Team A                        Team B
                              Produced three design options
Produced one detailed design
option
                              1                               2  3
Which Team Created A Better Design?
Team B        Team C
Produced three design options Produced five design options
              1          2              3
1       2  3
                      4              5
See Petre, Marian "Insights from Expert Software Design Practice" in: European Software Engineering Conference and
Symposium on The Foundations of Software Engineering (ESEC/FSE 2-09)
                    CS 35L Software Construction: Lecture 12 ­ Design with Reuse
                    Tobias Dürschmid
Lesson Learned:
Think of Many Design Alternatives
12                                 3  4  5
  · If you can think of a good design, try to think of a better one
  · Think broadly about a diverse range of solution
  · Research has shown: When simply prompting designers to consider
    other design alternatives, designers with less experience create
    better designs [1]
[1] Tofan, Dan, Matthias Galster, and Paris Avgeriou. "Difficulty of architectural decisions­a survey with professional architects."
European Conference on Software Architecture (ECSA 2013)
                               CS 35L Software Construction: Lecture 12 ­ Design with Reuse
                               Tobias Dürschmid
Lesson Learned:
Think of Many Design Alternatives
                             12                               3  4  5
Generate                     Evaluate
      The purpose of idea     Narrowing down of ideas is
generation is to broaden up  done in the evaluation activity
Delaying Decisions
· Identify design decisions that need more information or that
 are likely to change later
· Attempt to design your system without assuming a
 solution for these difficult decisions
· Keep a list of delayed decisions and keep track of what you
 need to resolve them
                            CS 35L Software Construction: Lecture 12 ­ Design with Reuse
                            Tobias Dürschmid
Design Exercise: Divide And Conquer
Design an interplanetary messaging system for people living on Earth and
Mars to communicate!
What sub-problems would you                                        Mars
need to solve?                              Earth             Sun
  Problems that arise when messaging on
   Earth, added infrastructure, networking
       with largely varied distances, and
            different definition of a day
Lesson Learned:
Solve Simpler Problems First
· When faced with a complex problem, experts solve a simpler problem first
· Solution to simpler problem might be incomplete but can be extended
  later
· Be aware when the simpler problem is so fundamentally different that
  solutions do not generalize
                            CS 35L Software Construction: Lecture 12 ­ Design with Reuse
                            Tobias Dürschmid
Making Design Decisions
(The "Rational" Decision Making Process)
        Has been found to lead to better design
        (especially effective for early-career software engineers) [2]
Step 1: Identify your requirements. That is important?
Step 2: Think of Many Design Alternatives
Step 3: Evaluate how well your Design Alternatives help you implement the
Requirements from Step 1
Step 4: Consider the Trade-offs and Make a Decision
[2] Tang, Antony, et al. "Design reasoning improves software design quality." International Conference on the Quality of Software
Architectures. Springer. 2008.
At Google, before the Implementation of a New
System, the Developers writes a "Design Doc"
  Goal
· Early identification of design issues when making changes is still cheap.
· Achieving consensus around a design in the organization.
· Transferring knowledge from senior engineers into the organization.
· Form the basis of an organizational memory around design decisions.
· Acts as a summary artifact in the technical portfolio of the software designer(s).
"As software engineers our job is not to produce code per se, but rather to solve
problems. Unstructured text, like in the form of a design doc, may be the better tool for
solving problems early in a project lifecycle, as it may be more concise and easier to
comprehend, and communicates the problems and solutions at a higher level than
See https://www.industrialempathy.com/posts/design-docs-at-google/  code." ­ Malte Ubl
Design Docs are practiced by:
"Design Docs" Consist of These Parts:
  Context & Scope bring the reader up to speed
Background facts that are relevant to understand the Design Doc
  Goals & Non-Goals define the Scope
Goals: Requirements & quality attributes
Non-Goals: What can be ignored to simplify development?
  The Design describes the selected solution
Models & design descriptions (e.g., context diagram, data model,
API descriptions, pseudo-code, constraints, ...)
Alternatives motivate why this decision was made
Trade-offs that each respective design makes and how those trade-offs led to
the decision to select the design that is the primary topic of the document.
See https://www.industrialempathy.com/posts/design-docs-at-google/
Please fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three design principles you learned about Design
With Reuse today. (Do not copy phrases from the lecture slides)
Describe the pros and cons of using the axios library versus the fetch API for HTTP requests
in Node.JS. Decide which one is more appropriate for your course project and why.
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slide use images from Flaticon.com (Creators: Freepik, surang, Eklip Studio)

```
