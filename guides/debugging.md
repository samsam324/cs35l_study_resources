# Debugging â€” CS35L Study Guide

Source: Lecture 10 (debugging half) â€” *Debugging & Advanced Git* (Tobias DÃ¼rschmid).
Scope: This guide covers ONLY the debugging content from Lecture 10. The Advanced Git portion lives in `git.md`.

> **"Debugging is like being the detective in a crime movie where you are also the murderer."** â€” Filipe Fortes

---

## Opening Scenario: You See This Error. What Next?

```
mysql> CREATE DATABASE my_awesome_database
         -> CHARACTER SET utf8mb4
         -> COLLATE utf8mb4_unicode_ci;

ERROR 3680 (HY000): Failed to create schema directory
'my_awesome_database' (errno: 2 - No such file or directory)
```

**First instinct:** *Ask search engines (e.g., Google) and AI tools (e.g., Gemini, ChatGPT)*.

---

## Software Construction Practice: Search For The Error Message

### Problem
You see an error message from your **framework, library, or external service** (not your own code) that does not directly point you towards a solution.

### Context
As a human developer, you do not have infinite knowledge & experience. **Nobody expects you to understand every error message.**

### Solution
1. **Remove all project-specific identifiers** from the input & output.
2. **Copy & paste** input & output into a search engine / AI tool.
3. **Carefully study the results**, (cautiously) experiment with suggested solutions.
4. **(Only) after you cannot get any more useful information from online sources, ask your more experienced co-worker.**

---

## What Is Debugging?

> The systematic process of **finding & fixing faults** (aka. "bugs") in a program's source code.

The four debugging steps:

1. **Investigating symptoms to Reproduce the Bug**
2. **Locating the faulty code**
3. **Determining the root cause of the bug**
4. **Implementing and verifying a fix**

---

## Do Debugging Skills Matter?

- Software bugs cost **~60 billion USD / year**.
- **Validation (including debugging) can take up to 50â€“75% of development time.**
- When debugging, some developers are **three times as efficient as others**. (Hopefully you!)

---

## Bug Terminology â€” Fault vs. Error vs. Failure

### Fault
**The erroneous location in the code** (e.g., variable not initialized).

### Error
**An incorrect state during program execution** (e.g., variable has the wrong value).

### Failure
**Observed incorrect outside behavior** â€” when the Error reaches the System Boundary (e.g., system crashes, incorrect output displayed).

### Flow
```
Fault (in code) â”€â”€â–º Error (wrong runtime state) â”€â”€â–º (reaches system boundary) â”€â”€â–º Failure (observed bad behavior)
```

### Code Example â€” Circumference

```python
import sys
import math

def cal_circumference(radius):
    diameter = 2 * radius
    circumference = diameter * math.pi
    return circumference

def __main__():
    try:
        input_radius = sys.argv[1]
        C = cal_circumference(input_radius)
        print(f"The circumference of a circle "
              f"with radius {input_radius} is: {C}")
    except:
        print("An error occurred but there is no failure")

__main__()
```

- **Fault**: `input_radius = sys.argv[1]` is a string, not converted to a number.
- **Error**: `2 * radius` produces `'1010'` (string repetition), then `'1010' * math.pi` raises `TypeError`.
- **Failure (suppressed)**: The bare `except:` silently swallows the error and prints `"An error occurred but there is no failure"`.

**Discussion prompt:** *How can we prevent this error from becoming a failure?* (Answer: don't suppress the exception with a bare `except`; convert `input_radius` to `float`.)

### Xerox Scanner Case Study

Source: https://www.dkriesel.com/en/blog/2013/0802_xerox-workcentres_are_switching_written_numbers_when_scanning

Xerox WorkCentre scanners under certain settings **silently substituted digits** in scanned documents (e.g., scanning a price `6` as `8`). The lecture asks:

> *Are we looking at a fault, error, or failure?*

- **Fault**: Aggressive JBIG2 compression substitutes visually similar character patterns.
- **Error**: The internal scan representation contains wrong digits.
- **Failure**: The printed output shows the wrong numbers â€” visible to the user.

---

## Step 1: Reproduce the Bug

### Goal / Motivation
Reproduce the bug to be able to **observe it** and check whether you've fixed it.

### Discussion prompts
- *How to reproduce the bug?*
- *What should we ask the customer?*

### Reproduce the Problem Environment
- **Problem environment** = setting in which the problem occurs.
- Configuration of the machine: hardware, operating system, settings, run-time dependencies, software versions, ...
- Try to **re-create the problem environment on a different machine**.

### Reproduce the Problem History
- **Problem history** = steps necessary to re-create the problem.
- Record a sequence of:
  - **Data inputs**
  - **User interactions**
  - **Communications with other components**
- Additional variables: **timing, randomness, physical influences**, ...

> Reference: "Why Programs Fail â€” A Guide to Systematic Debugging" by Andreas Zeller (2009).

### If Possible: Write an Automated Bug Reproduction Test

#### Goal / Motivation
**Automated Tests allow you to check faster whether you've fixed the bug.**

#### Write a Bug Reproduction Test
**Automate the steps of reproducing the bug & checking if the bug is present.** Run your test several times during debugging.

#### Simplify Your Test Case
**Find out what is relevant & remove all apparently irrelevant steps** (e.g., details in the input that do not seem to be connected to the bug and that do not affect whether the failure is present or not).

---

## Step 2: Locate the Faulty Code

### Investigate Symptoms Using Logs

**Logging Libraries help recording symptoms:**
- **Inputs** (particularly unexpected ones)
- **State changes**
- **Communication with other components**

#### Python Logging Example

```python
import logging

# Create a log format
fmt = logging.Formatter(
    "%(name)s: %(asctime)s | %(levelname)s | "
    "%(filename)s:%(lineno)s | %(process)d >>> %(message)s"
)

logging.debug("A debug message")
logging.info("An info message")
logging.warning("A warning message")
logging.error("An error message")
logging.critical("A critical message")
```

The 5 log levels (low â†’ high severity): **DEBUG, INFO, WARNING, ERROR, CRITICAL**.

#### Example Output

```
2023-07-23 14:42:18,599 | INFO    | main.py:30 | 187901 >>> Server started listening on port 8080
2023-07-23 14:14:47,578 | WARNING | main.py:28 | 143936 >>> Disk space on drive '/var/log' is running low. Consider freeing up space
2023-07-23 14:14:47,578 | ERROR   | main.py:34 | 143936 >>> Failed to connect to database: 'my_db'
Traceback (most recent call last):
  File "/home/ayo/dev/betterstack/demo/python-logging/main.py", line 32, in <module>
    raise Exception("Failed to connect to database: 'my_db'")
Exception: Failed to connect to database: 'my_db'
```

### Use Visual Diagrams to Locate Bugs

Visual diagrams (e.g., a ROS computation graph) can reveal **what's missing** as a symptom:

- Symptoms: *No `sim_pose` messages*; *Many `fix` messages*.

By drawing the system as a graph, you spot which expected messages aren't flowing and trace upstream from there.

### Focus on the Most Likely Origins

#### Code Smells
**Bugs are more likely to be in poorly written code.** Get rid of code smells via **refactoring before debugging**.

#### Look for Common Bugs
- Uninitialized variables
- Unused values
- Unreachable code
- Memory leaks
- Interface misuse
- Null pointers
- Inconsistent types
- ...

#### Assertions
**Add assertions to your code to prevent errors from propagating.**

```python
assert input > 0
assert file != None
```

---

## Step 3: Determine the Root Cause

> **Your most valuable root cause analysis tool: [the rubber duck]**

### Rubber Duck Debugging Can Help You Find Bugs

#### The Curse of Knowledge
Having a mental model of your solution, **you read what you intended to write, not what you actually wrote.**

#### Explain Your Code to Your Duck
1. Place a rubber duck on your desk.
2. Explain to the duck **what your code is supposed to do, line by line**.
3. At some point, you will tell the duck what your code **should be doing next** & realize that **this is not what it actually does**.

Example monologue from the lecture:
> *"And then we call the function recursively until... oh wait! I forgot the base case! Thanks for listening, Duck."*

### Use Break Points for Debugging

#### Goal
You want to **observe the execution of the program step by step** and see variable values.

#### Break Point
**An intentional stopping point.**

#### How it works
Whenever the program execution reaches the break point, it stops and opens interactive control over the program to step through and watch.

> See: https://code.visualstudio.com/docs/debugtest/debugging

### Define Inputs & Run Configuration (VS Code `launch.json`)

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python Debugger: Current File",
      "type": "debugpy",
      "request": "launch",
      "args": ["10"],
      "program": "${file}",
      "console": "integratedTerminal"
    }
  ]
}
```

- `args` defines the inputs to pass when launching the program in debug mode.

### Observe Program Step by Step

VS Code Debug UI elements:
- **Step indicator** â€” shows where we are in the program execution.
- **Debug Console** â€” allows you to evaluate expressions in the scope of the program. Example:
  ```
  sys.argv[1]
  '10'
  input_radius
  NameError: name 'input_radius' is not defined  # before line 10 executes
  ```
- **Variables panel**:
  - **Locals** â€” current values of variables in the scope of the current statement (e.g., `input_radius = '10'`).
  - **Globals** â€” globals available.

Walkthrough using the circumference example:
1. Initial break at line 9 (`try:`): no locals yet.
2. After line 10 (`input_radius = sys.argv[1]`): **Locals: `input_radius = '10'`** (a string).
3. **Step into** the function `cal_circumference` at line 11: now we're inside the function, **Locals: `radius = '10'`**.
4. After line 4 (`diameter = 2 * radius`): **Locals: `radius = '10'`, `diameter = '1010'`** â€” string-repetition, not arithmetic. â† *The bug is here.*
5. Line 5 (`circumference = diameter * math.pi`): **Exception handling is triggered by multiplying a string with a float.**
6. Control returns to the `except` block at line 14 with **`input_radius = '10'`** still in scope.

New variables appear in the Variables panel **as they are assigned**.

### Conditional Break Points â€” Skip Directly to the Execution You Want to Observe

#### Goal
You want to observe the program **only if certain conditions are true** (e.g., the input is very large or the input contains a certain character, or the loop variable has reached 10).

#### Conditional Break Points
**Break points that trigger only when a given expression evaluates to true.**

#### How it works
**Right-click on the break point to edit it.** The expression can include functions and variables that exist in the code of this statement.

---

## Step 4: Implement and Verify a Fix

### Document the Bug Fix & Run Your Tests

#### Add Assertions
**After fixing the bug, add assertions to detect nearby bugs & run tests.**

#### Document Your Bug Fix
- In the code, **explain why your fix was necessary** in a comment.
- **Reference the bug in your git commit message.**
- **Document your solution in the bug report** to allow historical traceability.

#### After Fixing, Keep All New Tests for Regression Testing
**Regression Testing** = re-running existing tests after code changes to ensure that new updates, bug fixes, or features haven't introduced new problems into existing functionality.

The bug reproduction test from Step 1 becomes a permanent regression test guarding against the same bug reappearing.

---

## Quick-Reference Summary

| Step | Goal | Key Techniques |
|---|---|---|
| **0. Search the error message** | Avoid reinventing solutions | Strip project-specific identifiers; paste into Google / AI; experiment with suggestions; ask a coworker only as a last resort |
| **1. Reproduce the bug** | Make it observable & verifiable | Reproduce problem environment + history; write an automated bug reproduction test; simplify the test case |
| **2. Locate the faulty code** | Narrow scope | Logging (5 levels); visual diagrams; focus on code smells, common bug types, and assertions |
| **3. Determine the root cause** | Understand *why* | Rubber duck debugging; breakpoints (incl. conditional); step into/over/out; inspect Locals/Globals/Watch in VS Code |
| **4. Implement & verify the fix** | Confirm and prevent recurrence | Add assertions; commit message references bug; keep the bug reproduction test as a regression test |

---

## Terminology Glossary

- **Bug** â€” common name for a fault.
- **Fault** â€” the erroneous code itself.
- **Error** â€” the wrong runtime state caused by a fault.
- **Failure** â€” externally visible incorrect behavior caused by an error reaching the system boundary.
- **Problem environment** â€” the configuration in which the bug appears (OS, hardware, deps, settings).
- **Problem history** â€” the sequence of inputs/interactions/messages that triggers the bug.
- **Bug reproduction test** â€” an automated test that triggers the bug; reused as a regression test after the fix.
- **Regression testing** â€” re-running existing tests after changes to make sure nothing else broke.
- **Curse of Knowledge** â€” reading what you *meant* to write instead of what's actually there; rubber-duck explanation counters it.
- **Breakpoint** â€” intentional stopping point in the debugger.
- **Conditional breakpoint** â€” breakpoint that fires only when an expression is true.


---

# Appendix: Lecture 10 Slides (raw extracted text)

Full L10 extracted text. The advanced-git portion lives in git.md; the debugging portion below.

```text
CS 35L Software
Construction
Lecture 10 ­
Debugging & Git
                              detective in a crime movie where
Assistant Teaching Professor
Computer Science Department        you are also the murderer."
                                                              ­ Filipe Fortes
You see this Error. What do you do next?!
Talk to your Neighbor(s)!
mysql> CREATE DATABASE my_awesome_database
         -> CHARACTER SET utf8mb4
         -> COLLATE utf8mb4_unicode_ci;
ERROR 3680 (HY000): Failed to create schema directory
'my_awesome_database' (errno: 2 - No such file or directory)
                     Ask search engines (e.g., Google)
                    and AI tools (e.g., Gemini, ChatGPT)
Software Construction Practice:
Search For The Error Message
  Problem
You see an error message from your framework, library, or external service (not your
own code) that does not directly point you towards a solution.
  Context
As a human developer, you do not have infinite knowledge & experience. Nobody
expects you to understand every error message.
  Solution
1. Remove all project-specific identifiers from the input & output
2. Copy & paste input & output into search engine / AI tool
3. Carefully study the results, (cautiously) experiment with suggested solutions
4. (Only) after you cannot get any more useful information from online sources,
     ask your more experienced co-worker
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
· 1) Investigating symptoms to
  Reproduce the Bug
· 2) Locating the faulty code
· 3) Determining the root cause  "Debugging is like being the
  of the bug
· 4) Implementing and verifying  detective in a crime movie where
  a fix                               you are also the murderer."
                                                                     ­ Filipe Fortes
Do Debugging
Skills Matter?
· Software bugs cost  60 billion
  USD / year
· Validation (including debugging)
  can take up to 50-75% of
  development time
· When debugging, some
developers are three times as           "Debugging is like being the
efficient as others                 detective in a crime movie where
Hopefully you!                      you are also the murderer."
                                                                ­ Filipe Fortes
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
· 1) Investigating symptoms to
  Reproduce the Bug
· 2) Locating the faulty code
· 3) Determining the root cause  "Debugging is like being the
  of the bug
· 4) Implementing and verifying  detective in a crime movie where
  a fix                               you are also the murderer."
                                                                     ­ Filipe Fortes
Bug Terminology
  Fault                                             import sys
                                                    import math
The erroneous location in the code                  def cal_circumference(radius):
(e.g., variable not initialized)
                                                       diameter = 2 * radius
Error    Program execution                             circumference = diameter * math.pi
                                                       return circumference
An incorrect state during program execution
                                                    def __main__():
(e.g., variable has the wrong value)                   try:
                                                           input_radius = sys.argv[1]
         Error Reaches                                     C = cal_circumference(input_radius)
                                                           print(f"The circumference of a circle
Failure  System Boundary                                   with radius {input_radius} is: {C}")
                                                       except:
Observed incorrect outside behavior                        print("An error occurred
                                                                       but there is no failure")
(e.g., system crashes, incorrect output displayed)
How can we prevent this error from becoming a failure? __main__()
         CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git  7
         Tobias Dürschmid
How can we Debug these Xerox Scans?
Document:         Scan:
                  Are we looking at a
                  fault, error, or failure?
Source: https://www.dkriesel.com/en/blog/2013/0802_xerox-workcentres_are_switching_written_numbers_when_scanning
                                                         How to reproduce the bug?
Step 1: Reproduce the Bug What should we ask the
                   customer?
Goal / Motivation
Reproduce the bug to be able to observe it and check whether you've fixed it
Document:          Scan:
Source: https://www.dkriesel.com/en/blog/2013/0802_xerox-workcentres_are_switching_written_numbers_when_scanning
Reproduce the Bug & Write a Bug Report
   Reproduce the problem environment
· Problem environment = setting in which the problem occurs
· Configuration of the machine (e.g., hardware, operating system, settings,
   run-time dependencies, software versions, ...)
· Try to re-create the problem environment on a different machine
   Reproduce the problem history
· Problem history = steps necessary to re-create the problem
· Record a sequence of data inputs, user interactions, and
   communications with other components
· Additional variables: timing, randomness, physical influences, ...
See "Why Programs Fail ­ A Guide to Systematic Debugging" by Andreas Zeller 2009
If Possible, Write an Automated
Bug Reproduction Test
      Goal / Motivation
  Automated Tests allow you to check faster whether you've fixed the bug
      Write a Bug Reproduction Test
  Automate the steps of reproducing the bug & checking if the bug is present.
  Run your test several times during debugging
      Simplify your Test Case
  Find out what is relevant & remove all apparently irrelevant steps (e.g.,
  details in the input that do not seem to be connected to the bug and that do not
  affect whether the failure is present or not)
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
   · 1) Investigating symptoms to
     Reproduce the Bug
   · 2) Locating the faulty code
   · 3) Determining the root cause
     of the bug
   · 4) Implementing and verifying
     a fix
Investigate Symptoms                           import logging                                              Python Logging
Using Logs
                                               # Create a log format
· Logging Libraries help recording             fmt = logging.Formatter(
  symptoms
                                                      "%(name)s: %(asctime)s | %(levelname)s |
                                               %(filename)s:%(lineno)s | %(process)d >>>
                                               %(message)s"
                                               )
      · Inputs (particularly unexpected ones)  logging.debug("A debug message")
      · State changes                          logging.info("An info message")
      · Communication with other components    logging.warning("A warning message")
                                               logging.error("An error message")
Output                                         logging.critical("A critical message")
example: 2023-07-23 14:42:18,599 | INFO | main.py:30 | 187901 >>> Server started listening
on port 8080
example: 2023-07-23 14:14:47,578 | WARNING | main.py:28 | 143936 >>> Disk space on drive
'/var/log' is running low. Consider freeing up space
example: 2023-07-23 14:14:47,578 | ERROR | main.py:34 | 143936 >>> Failed to connect to
database: 'my_db'
Traceback (most recent call last):
File "/home/ayo/dev/betterstack/demo/python-logging/main.py", line 32, in <module>
raise Exception("Failed to connect to database: 'my_db'")
Exception: FailedCSto35LcoSnonftewcatretCoondsatrtuacbtiaons:eL:ec'tumrye_1d0b'­ Debugging & Advanced Git  14
              Tobias Dürschmid
Use Visual Diagrams to Locate Bugs
                                                                     Symptoms:
                                                                     · No
                                                                         sim_pose
                                                                         messages
                                                                     · Many fix
                                                                         messages
Focus on the Most Likely Origins
   Code Smells
Bugs are more likely to be in poorly written code.
Get rid of code smells via refactoring before debugging
   Look for common bugs
e.g., uninitialized variables, unused values, unreachable code, memory leaks,
interface misuse, null pointers, inconsistent types, ...
   Assertions
Add assertions to your code to prevent errors from propagating
(e.g., assert input > 0, assert file != null)
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
   · 1) Investigating symptoms to
     Reproduce the Bug
   · 2) Locating the faulty code
   · 3) Determining the root cause
     of the bug
   · 4) Implementing and verifying
     a fix
Your most valuable root
   cause analysis tool
Rubber Duck Debugging                         And then we call the function
Can Help you Find Bugs                        recursively until ... oh wait! I
  Curse of Knowledge                               forgot the base case!
                                               Thanks for listening, Duck
Having a mental model of your solution,
you read what you intended to write, not
what you actually wrote.
  Explain Your Code to Your Duck
· Place a rubber duck on your desk.
· Explain to the duck what your code is
   supposed to do, line by line.
· At some point, you will tell the duck what
   your code should be doing next & realize
   that this is not what it actually does.
Use Break Points                       1 import sys
For Debugging
                                       2 import math
  Goal
                                       3 def cal_circumference(radius):
You want to observe the execution
of the program step by step and see    4 diameter = 2 * radius
variable values
                                       5 circumference = diameter * math.pi
  Break Point
                                       6 return circumference
An intentional stopping point
                                       7
  How it works
                                       8 def __main__():
Whenever the program execution
reaches the break point, it stops and  9 try:
opens interactive control over the
program to step through and watch      10                   input_radius = sys.argv[1]
                                       11                   C = cal_circumference(input_radius)
                                       12                   print(f"The circumference of a circle
                                       13                   with radius {input_radius} is: {C}")
                                       14 except:
                                       15                   print("An error occurred
                                       16                   but there is no failure")
                                       17
                                       18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
   Define Inputs & Run                 1 import sys
   Configuration
                                       2 import math
                                       3 def cal_circumference(radius):
                                       4 diameter = 2 * radius
      Debug Configuration              5 circumference = diameter * math.pi
      in VS Code
                                       6 return circumference
                                       7
launch.json                            8 def __main__():
"version": "0.2.0",                    9 try:
"configurations": [                    10                   input_radius = sys.argv[1]
   {                                   11                   C = cal_circumference(input_radius)
      "name": "Python Debugger:        12                   print(f"The circumference of a circle
      Current File",                   13                   with radius {input_radius} is: {C}")
      "type": "debugpy",               14 except:
      "request": "launch",  inputs     15                   print("An error occurred
      "args": ["10"],
      "program": "${file}",            16                   but there is no failure")
      "console": "integratedTerminal"  17
   }                                   18 __main__()
]
See https://code.visualstudio.com/docs/debugtest/debugging
             CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git        21
             Tobias Dürschmid
Observe Program                         1 import sys
Step by Step
                                        2 import math
             Indicates where we are in
             the program execution      3 def cal_circumference(radius):
Allows you to evaluate                  4 diameter = 2 * radius
expressions in the
scope of the program                    5 circumference = diameter * math.pi
           Debug Console                6 return circumference
 sys.argv[1]
                                        7
     '10'
 input_radius                           8 def __main__():
      NameError: name                   9 try:
      'input_radius'
      is not defined                    10                  input_radius = sys.argv[1]
                                        11                  C = cal_circumference(input_radius)
                                        12                  print(f"The circumference of a circle
                                        13                  with radius {input_radius} is: {C}")
                                        14 except:
                                        15                  print("An error occurred
                                        16                  but there is no failure")
                                        17
                                        18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Observe Program            1 import sys
Step by Step
                           2 import math
     Variables
                           3 def cal_circumference(radius):
      Locals
                           4 diameter = 2 * radius
     input_radius = '10'
                           5 circumference = diameter * math.pi
      Globals
                           6 return circumference
Current values of
variables in the scope of  7
the current statement
                           8 def __main__():
                           9 try:
                           10                               input_radius = sys.argv[1]
                           11                               C = cal_circumference(input_radius)
                           12                               print(f"The circumference of a circle
                           13                               with radius {input_radius} is: {C}")
                           14 except:
                           15                               print("An error occurred
                           16                               but there is no failure")
                           17
                           18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Observe Program                 1 import sys
Step by Step
                                2 import math
    Variables
      Locals                    3 def cal_circumference(radius):
     radius = '10'              4 diameter = 2 * radius
      Globals
                                5 circumference = diameter * math.pi
     We have stepped into this
     Function                   6 return circumference
                                7
                                8 def __main__():
                                9 try:
                                10                          input_radius = sys.argv[1]
                                11                          C = cal_circumference(input_radius)
                                12                          print(f"The circumference of a circle
                                13                          with radius {input_radius} is: {C}")
                                14 except:
                                15                          print("An error occurred
                                16                          but there is no failure")
                                17
                                18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Observe Program          1 import sys
Step by Step
                         2 import math
     Variables
      Locals             3 def cal_circumference(radius):
     radius = '10'       4 diameter = 2 * radius
     diameter = '1010'
      Globals            5 circumference = diameter * math.pi
New variables appear as
they are assigned        6 return circumference
                         7
                         8 def __main__():
                         9 try:
                         10                                 input_radius = sys.argv[1]
                         11                                 C = cal_circumference(input_radius)
                         12                                 print(f"The circumference of a circle
                         13                                 with radius {input_radius} is: {C}")
                         14 except:
                         15                                 print("An error occurred
                         16                                 but there is no failure")
                         17
                         18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Observe Program                  1 import sys
Step by Step
                                 2 import math
    Variables
      Locals                     3 def cal_circumference(radius):
     input_radius = '10'         4 diameter = 2 * radius
      Globals
                                 5 circumference = diameter * math.pi
     Exception handling is
     triggered by multiplying a  6 return circumference
     string with a float
                                 7
                                 8 def __main__():
                                 9 try:
                                 10                         input_radius = sys.argv[1]
                                 11                         C = cal_circumference(input_radius)
                                 12                         print(f"The circumference of a circle
                                 13                         with radius {input_radius} is: {C}")
                                 14 except:
                                 15                         print("An error occurred
                                 16                         but there is no failure")
                                 17
                                 18 __main__()
See https://code.visualstudio.com/docs/debugtest/debugging
Conditional Break Points ­ Skip Directly to the
Execution You Want to Observer
    Goal
  You want to observe the program only if certain conditions are true (e.g., the
  input is very large or the input contains a certain character, or the loop variable
  has reached 10).
  Conditional Break Points
Break points that trigger only when a given expression evaluates to true.
  How it works
Right-click on the break point to edit it. The expression can include functions and
variables that exist in the code of this statement.
See https://code.visualstudio.com/docs/debugtest/debugging
What is Debugging?
· The systematic process of
  finding & fixing faults,
  (aka. "bugs") in a program's
  source code
   · 1) Investigating symptoms to
     Reproduce the Bug
   · 2) Locating the faulty code
   · 3) Determining the root cause
     of the bug
   · 4) Implementing and verifying
     a fix
Document The Bug Fix & Run Your Tests
    Add Assertions
After fixing the bug, add assertions to detect nearby bugs & run tests
   Document your Bug Fix
In the code, explain why your fix was necessary in a comment
Reference the bug in your git commit message
Document your solution in the bug report to allow historical traceability
   After Fixing, Keep all new Tests for Regression Testing
Regression Testing = re-running existing tests after code changes to ensure
that new updates, bug fixes, or features haven't introduced new problems into
existing functionality
CS 35L Software
Construction
Lecture 10 ­ Advanced
Git Techniques
Assistant Teaching Professor
Computer Science Department
Rapid Fire Quiz on Git
  What is HEAD?
The pointer to the currently selected commit / branch
  What is the staging area?
    f.
Code that is about to be committed in the next commit
  What is a branch?
A pointer to the most recent commit of a sequence of separately developed code
  What is a merge conflict?
The attempted merge of lines in the code that have been changed by both versions
  What makes a merge commit different?
It has two parents
Detached HEAD                    git checkout main  main HEAD
                                                    E main HEAD
    Non-Detached HEAD                ABCD
  HEAD is attached to a branch.  git commit
  Most Git commands, such as
  git commit, will move the          ABCD
  branch & HEAD.
  Detached HEAD                  git checkout D     main
                                                     HEAD
HEAD is attached to a commit,       ABCD
not a branch. Git commands,                         main
will NOT move the branch.        git commit         E HEAD
                                    ABCD
You are in 'detached HEAD' state. You can look around, make experimental changes and commit them, and you can
discard any commits you make in this state without impacting any branches by switching back to a branch.
Relative Commit Addresses
  Goal
You want to describe the parent commit (or grand parent commit, or great grand parent
commit) of a commit without having to get the commit ID from the log.
  How it works
BRANCH~n is the n-th commit before BRANCH.
feature~2 main~2           HEAD~1
A                 B        C       D
                                      HEAD main
                  f feature
                 See https://www.geeksforgeeks.org/git/git-cherry-pick/
Git Cherry-Pick
  Goal
You want to selectively "copy" one particular commit from one branch to another
branch without bringing over the entire history.
  How it works
git cherry-pick  calculates the difference (patch) for commit  and creates a new
commit that applies the same patch to the current base.
Note: This may result in merge conflicts if the patch cannot be applied directly.
           feature                           feature
                git cherry-pick                  
ABCD
                                  A B C D 
HEAD main                                                                          HEAD  main
              CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git               34
              Tobias Dürschmid
                                              See https://www.geeksforgeeks.org/git/git-stash/
Git Stash & Pop
  Problem
You want to checkout a branch or commit while you have uncommited changes
  Context
You might need your uncommited changes later
  Solution
Use git stash to move your changes to a separate stashing location
and use git stash pop to bring them back
  How it works
Git maintains a stack of stashes changes. Every time you call git stash it adds these
patches to the stashing stack. Pop removes them from the stack. git stash list
shows you the entire stack. git stash apply <id> applies a non-top stash patch
git blame ­ Which Commit                                        See https://www.geeksforgeeks.org/git/how-
Last Changed a Dubious Line?                                                                         to-use-git-blame/
  Goal
For a given line you want to identify why it exists / why it has been implemented the way
it has, or who has most recently changed this line.
  How it works                    git blame .\my_stack.py
git blame <filename> shows        ae721935 (TD 10-25 15:04 1) class MyStack:
you, for every line in the file:
· Commit id of the last commit    562ad4a6 (TD 10-25 15:06 2)   """
    that modified this line       562ad4a6 (TD 10-25 15:06 3)   A simple implementation of a Stack data str
· Author of that commit
· Timestamp of that commit        562ad4a6 (TD 10-25 15:06 4)
GitHub visualizes this very well  562ad4a6 (TD 10-25 15:06 5)   A Stack operates on the Last-In, First-Out
                                  562ad4a6 (TD 10-25 15:06 6)   the last element added to the stack is the
                                  562ad4a6 (TD 10-25 15:06 7)   """
                                  562ad4a6 (TD 10-25 15:06 8)
                                  6f33a948 (TD 10-25 18:58 9)   class PopOnEmptyStackException(Exception):
                                  6f33a948 (TD 10-25 18:58 10)       """
                                  6f33a948 (TD 10-25 18:58 11)       Exception raised when trying to call po
                                  6f33a948 (TD 10-25 18:58 12)       """
                                  6f33a948 (TD 10-25 18:58 13)       def __init__(self, message="Cannot pop
                                  6f33a948 (TD 10-25 18:58 14)            super().__init__(message)
                                  6f33a948 (TD 10-25 18:58 15)
                                  07768051 (TD 10-25 16:29 16)  def __init__(self):
                                  47c50cfe (TD 10-25 16:44 17)       """
git bisect ­ Which Commit                           See https://www.geeksforgeeks.org/git/git-bisect/
Introduced the Fault?
  Goal
Out of hundreds of commits, how to find the one that broke the tests?
How it works                                     A  A                         A  A
1. Identify a past commit c_good that worked     B  B                         B  B
2. Write a command test that tests whether a     C  C                         C  C
commit works and that can run on past
commits                                          D  D                         D  D
3. Run: git bisect start c_good                  E  E                         E  E
&& git bisect run test
4. Git will run a binary search and stop on the  F  F                         F  F
commit that broke the tests                      G  G                         G  G
         CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git        37
         Tobias Dürschmid
git revert ­ The Undo Button  See https://www.geeksforgeeks.org/git/how-
For Changes in the Past                     to-revert-a-commit-with-git-revert/
  Goal
You want to "undo" a commit in the history of your branch (e.g., it introduced a bug)
  How it works
git revert B calculates the difference (patch) for commit B and creates a new
commit that applies the inverse patch to the current base. B is still in the commit
history. The default commit message for the reverting commit is "Revert B".
Note: This may result in merge conflicts if the inverse patch cannot be applied directly.
This command is non-destructive (preserves the entire history).
ABCD  git revert B            A B C D -B
HEAD main                                                                          HEAD  main
              CS 35L Software Construction: Lecture 10 ­ Debugging & Advanced Git               38
              Tobias Dürschmid
git rebase ­i                                              See https://git-scm.com/book/en/v2/Git-
Rewrite Your Own History                                                        Tools-Rewriting-History
  Goal
You want to "undo" one or more commits in without introducing a revert commit.
   How it works                                         pick B "Let's see if they find my bug"
                                                        pick C "Harmless changes"
git rebase -i A rewrites the commit history             pick D "Implement magical feature"
starting from commit A. If changes are made (e.g.,
commits should be dropped or added within a break) git  # Rebase A..D onto A (3 commands)
creates new commits with the same patches. This         # Commands:
command is destructive (the result is a new commit      # p, pick <commit> = use commit
history which may require push --force to update)       # r, reword <commit> = use commit, but
                                                        edit the commit message
                                                        # b, break = stop here (continue rebase
                                                        later with 'git rebase --continue')
                                                        # d, drop <commit> = remove commit
                                                        [...]
ABCD              git rebase -i A                       A  C'        D'  Destructive
    HEAD main        Then drop B                                         New commits
                                                        HEAD main
git rebase ­i                                 See hTttapsm://gpit-esTcrmoion.clsog-mRe/wbworioittkihn/egn-H/vi2s/tGority-
Rewrite Your Own History                       time could destroy
    Data Loss Risk                                  everything!
  If you get confused, mess up a step, or
  accidentally drop a commit you needed, you
  could lose work. Always ensure your
  changes are committed (or stashed) before
  starting a rebase.
    The Safe Zone
  When working in teams, only use git
  rebase -i on commits that exist only on
  your local machine or on a feature branch
  that only you are working on.
                                                                                                                             See https://www.geeksforgeeks.org/git/rebasing-
                                                                                                                                                                       of-branches-in-git/
Simple Rebase ­ An Alternative to Merge
  Goal
You want to merge a feature branch but instead of a merge commit with two parents you
just want to "copy" the individual commits
   How it works
git rebase f combines the patches from all commits on the feature branch into one
instead of creating a merge commit with two parents
                   feature HEAD                                      ' ' '
ABCD                   git rebase  A B C D feature
                 main       main                                 HEAD
                                                    main
                                                                                                                             See https://www.geeksforgeeks.org/git/rebasing-
                                                                                                                                                                       of-branches-in-git/
Simple Rebase ­ An Alternative to Merge
  Goal
You want to merge a feature branch but instead of a merge commit with two parents you
just want to "copy" the individual commits
   How it works
git rebase f combines the patches from all commits on the feature branch into one
instead of creating a merge commit with two parents
                   feature                                                feature
                            ABC                                      
ABCD       git rebase                                                D' main
             feature                                                       HEAD
HEAD main                                                                                42
                                                                                                                                     See https://www.geeksforgeeks.org/git/git-
                                                                                                                                                                                     squash/
Squash Merge keeps your History Simple and Clean
  Goal
You want to merge a feature branch but instead of keeping all individual commit you want
just one single commit for the new feature
   How it works
git merge --squash combines the patches from all commits on the feature branch
into one instead of creating a merge commit with two parents
                 feature                                                 feature
ABCD                         ABCD                                    
                  git merge
    HEAD main       feature                HEAD                       
                   --squash                                           main
Git Submodules ­ Perfect for  See https://www.geeksforgeeks.org/git/git-
                                                                        submodule/
Large Projects with Multiple Repositories
   Goal
 You want to include external libraries or shared modules within your project while
 maintaining their history and keeping them separate from your main repository.
  How it works
Conceptually, git submodules keep a Git repository as a subdirectory of another Git
repository. Internally, they are represented as a file that points to the commit ID that
should be checked out in the submodule. You change the content of the submodule by
committing this file (which has the same name as the directory).
   Examples
https://github.com/cmu-rss-lab/rosdiscover-evaluation/tree/main/deps
https://github.com/cmu-rss-lab/rosdiscover-evaluation/tree/main/deps
https://github.com/UCLA-CS-35L/submodule_example_parent/
https://github.com/UCLA-CS-35L/submodule_example_parent/
Common Submodule Commands                              See https://www.geeksforgeeks.org/git/git-
(Yes, this is initially confusing)                                                               submodule/
  Add a Submodule
git submodule add <repo-url> <path>
Clones <repo-url> into <path> and starts keeping track of it as a submodule
  Clone a Repository that has Submodules
git clone --recursive <repo-url>
Clones <repo-url> and their transitive submodules and checks out the selected commit
  Reset Submodule Content to the Selected Commit
git submodule update
Clones all submodule repos (and their submodules) and checks out the selected commit
  Commit Content inside the Submodule
Commit the changes within the submodule folder & push
Change the directory to outside of the submodule, commit the submodule & push
Tutorials for this Lecture
· Advanced Git Tutorial
   · https://tobiasduerschmid.github.io/SEBook/tools/git-advanced-tutorial
   · Run rebase, bisect, and other commands in git with our grit graph visualization
· Debugging Tutorial
   · https://tobiasduerschmid.github.io/SEBook/tools/python-debugging
   · Use our time traveling debugger to step go back and forth in time
Please fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three key insights you learned about Debugging
& Git today. (Do not copy phrases from the lecture slides)
What is the difference between Testing & Debugging? Please describe the difference based
on an example.
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slide use images from Flaticon.com (Creators: Freepik)
Publish-Subscribe implements an N-to-M
Messaging Channel
    Problem
How to keep the state of cooperating components synchronized?
    Context
Components should be loosely coupled and reduce direct dependencies
Publisher            Usually identified  Message                     Subscriber
                     via string names     Queue
       publishes to
                           Topic         subscribes to
Publisher
                                         Message                     Subscriber
                                          Queue
Publish-Subscribe implements an N-to-M
Messaging Channel
Produces data or sends     How does this   Consumes data or
notifications about state   relate to the  listens to state
updates                      Observer?     updates
Publisher                                  Message                   Subscriber
                                            Queue
       publishes to
                           Topic  subscribes to
Publisher
                                           Message                   Subscriber
                                            Queue
Publish-Subscribe implements an N-to-M
Messaging Channel
pub = nh.advertise("topic_name");
...
                                 nh.subscribe("topic_name", 10, callback);
pub.publish(msg)
Publisher                   Message                                  Subscriber
                             Queue
       publishes to
                     Topic  subscribes to
Publisher
                            Message                                  Subscriber
                             Queue

```
