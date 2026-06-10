# People and Processes (CS35L Study Guide)

Source material: Lecture 19 (L19_Process.pptx) and the opening process material from Lecture 3 (L3-PythonScripting.pdf), covering INVEST user-story criteria and adversarial thinking.

---

## Learning Objectives

By the end of this material, you should understand:

- The differences between **agile**, **plan-driven**, and **risk-driven** software development processes.
- The **human aspects** of software construction (decision making, collaboration, team structures, biases, the role of architects).
- How to **adjust the development process to domain-specific needs** (e.g., a startup vs. NASA vs. a social-media company vs. a large internet company).

---

## 1. The Waterfall Model (Plan-Driven Development)

The classic plan-driven process. Each phase is completed in full before the next phase starts:

1. **Requirements** — gather and document all customer requirements up front.
2. **Design** — produce a complete architectural and detailed design.
3. **Development** — implement the design.
4. **Testing** — verify and validate the implementation.
5. **Deployment** — release the software to the customer.

### Challenges of Waterfall

- **Requirements change** over the long lifecycle of the project, but the process resists change.
- **Late customer feedback**: customers only see working software near the end, so misunderstandings surface too late.
- **Delays**: each phase blocks the next, so any slippage cascades and the customer waits a long time before any value is delivered.

---

## 2. Key Idea: Iterative, Incremental Development

Instead of one long pipeline, work proceeds in **short iterations / sprints** (typically **1–4 weeks**) drawn from a **feature backlog**. Each iteration produces a usable product increment, allowing continuous customer feedback and adaptation.

---

## 3. The Agile Manifesto (2001)

Reference: https://agilemanifesto.org/

The Agile Manifesto values:

- **Individuals and interactions** over processes and tools
- **Working software** over comprehensive documentation
- **Customer collaboration** over contract negotiation
- **Responding to change** over following a plan

> "While there is value in the items on the right, we value the items on the left more."

---

## 4. The Scrum Process

Scrum is a specific agile framework. Its main artifacts and events:

- **Product Backlog** — the ordered list of all user stories (features the customer wants).
- **Customer Input** — feeds into and reshapes the product backlog.
- **Sprint Planning** — at the start of each sprint, the team selects user stories from the product backlog for one sprint.
- **Sprint Backlog** — the chosen user stories committed to for this sprint.
- **Sprint** — 2–3 weeks of un-interrupted development; **the sprint backlog does not change during the sprint**.
- **Product Increment** — the working software produced at the end of the sprint.
- **Sprint Review** — validation of the finished increment (with the customer/stakeholders).
- **Daily Standup** — developers meet daily to discuss progress and obstacles.

### Planning Poker

A technique used during sprint planning to estimate the **size** of user stories:

- Estimates are given in **imaginary units of effort** — **not** units of time.
- Each developer privately picks a value, then everyone reveals at once.
- **Large differences in estimates trigger a debate**: the high and low estimators explain their reasoning, surfacing hidden assumptions and risks.

---

## 5. Design-Related Principles Behind the Agile Manifesto

- **Working software is the primary measure of progress.**
- **Continuous attention to technical excellence and good design enhances agility.**
- **The best architectures emerge from self-organizing teams.**
- **Design is part of every iteration**, not a one-time initial phase.
- **No single architect or top-down design** — design is a team-wide, ongoing activity.

---

## 6. Tiny Upfront Design: Challenges

Doing very little upfront design has real costs:

- **Software becomes hard to change** because no coherent structure was planned.
- **Low quality attributes** (e.g., performance, scalability, security) — these are hard to retrofit.
- **Small bus factor**.

### Bus Factor

The **number of people who can leave the project before the project stalls**. A small bus factor means knowledge is concentrated in too few heads; a large bus factor means knowledge and skills are well distributed.

---

## 7. Risk-Driven Design (George Fairbanks)

A middle ground between waterfall and pure agile:

- **Identify the biggest risks** to the project early.
- **Focus design effort on those risks.**
- **The amount of risk determines the amount of upfront design** — high risk warrants more upfront design; low risk warrants less.

### Risks Are Decisions That Are Hard to Change

Examples of "risky" (hard-to-change) decisions:

- **Programming languages**
- **Target platforms** (OS, hardware, browsers, devices)
- **Component architectures and connectors**
- **Interfaces** (APIs, protocols, data formats)
- **Quality attributes** (performance, security, scalability, reliability, etc.)

### Risk Storming (Simon Brown)

Reference: https://riskstorming.com/

A four-step workshop technique for identifying risks:

1. **Model** — agree on a model/diagram of the system being designed.
2. **Think** — each participant independently identifies risks (often on sticky notes).
3. **Share** — participants place their risks on the model and explain them.
4. **Review** — collectively review, cluster, and prioritize the risks.

### Identify and Mitigate the Highest-Priority Risks

- Make **big design decisions early**; **defer small decisions**.
- **High cost of change → more upfront design.**
- **Low cost of change → leaner design**, decide later.

---

## 8. Changeability in Agile Projects

A good architecture enables **late decisions** — it is designed so that decisions can be delayed without major rework.

- **Maximize the number of decisions not yet made.**
- Apply principles that preserve flexibility:
  - **Information Hiding**
  - **SOLID principles**
  - **Low Coupling**
  - **High Cohesion**

---

## 9. Feature Backlog vs. Technical Debt Backlog

Two parallel backlogs:

- **Feature Backlog** — **user stories** describing functional requirements (what the customer wants).
- **Technical Debt Backlog** — design improvements achieved through **refactoring** and the introduction of better **abstractions**.

### Technical Debt

**Short-term decisions that make future changes costly.** Like financial debt, it accumulates "interest" — the longer you delay paying it down, the more it costs you.

### Examples of Technical Debt Issues

- **Fix code smells** — duplicate code, high coupling, low cohesion, complex interfaces.
- **Improve documentation.**
- **Architectural changes** to address performance or scalability problems.

### Integrating Technical Debt in Agile

- An **architect role** is responsible for maintaining the technical-debt backlog.
- Either **include debt issues in every sprint** (a fixed share of capacity), or **dedicate an entire sprint to debt** periodically.

---

## 10. Ivory Tower Architects (Anti-Pattern)

Architects who:

- Are **not involved in construction** (do not write or read code).
- **Ignore input from the rest of the team.**
- Produce designs that are **elegant in theory but only work in theory** — not on the actual system.

### Software Construction Is a Collaborative Activity

To avoid the ivory tower:

- **Encourage design alternatives** — invite multiple proposals.
- **Consult domain experts.**
- **Include developers** in design discussions.
- **Stay in touch with the codebase** — architects should still read and write code.

---

## 11. Rational vs. Intuitive Decision Making

Two complementary modes of making design decisions.

### Rational Decision Making

- **Explicit identification, evaluation, and ranking** of options using logical reasoning.
- Helps **gain explicit knowledge** about the problem and the trade-offs.
- **Guides non-experts** through a problem they don't yet have intuition about.
- Can only access **explicit knowledge** — knowledge that has been written down or can be articulated.
- **Hard to communicate** in its raw form (long chains of reasoning).
- **Challenging for groups** to perform together (slow, lots of debate).
- **Prone to cognitive biases**, such as:
  - **Anchoring** — over-relying on the first piece of information.
  - **Confirmation bias** — favoring evidence that supports an existing belief.

### Intuitive (Naturalistic) Decision Making

- Unconscious "**gut feeling**" judgments.
- Can access **all implicit knowledge** (experience accumulated by the decision maker).
- **Helps experts** make better decisions.
- **Faster** than rational analysis.
- **Hard to communicate or justify** — the decision maker often can't explain why.

### Combining Both Processes — Appropriate Context

Use **intuitive** decision making when:

- There is **time pressure**.
- The decision makers are **experienced**.
- There is a **lack of information**.
- The problem is **hard to define**.
- There is **uncertainty**.
- A "**good-enough**" decision is sufficient.

Use **rational** decision making when:

- **Justification is needed** (e.g., to stakeholders, reviewers, regulators).
- The problem is **well-structured**.
- An **optimal decision** is needed.

### Bounded Rationality

- **Human rationality is limited by cognitive capabilities.**
- We **cannot consider all options** for any non-trivial decision.
- **Designers retroactively rationalize** decisions that were in fact made intuitively — i.e., we invent justifications after the fact.

---

## 12. Doghouse vs. Skyscraper (Design Process Analogy)

The scale of the artifact determines how much process is appropriate:

| Aspect | Doghouse | Skyscraper |
|---|---|---|
| People involved | Fewer | Many |
| Process length | Short | Long |
| Risk | Lower | Higher |
| Upfront design | Less | More |
| Decision style | More intuitive | More rational |

### Lesson Learned

**Adjust the design process to the specific domain.** No single process (agile, plan-driven, or risk-driven) is right for every project — match the process to the size, risks, and constraints of the work.

---

## 13. Case Study: Facebook / Meta

- Famous quote (Zuckerberg): "**Go fast and break things.**"
- Domain: **web-based social media**.
- **Small-to-medium upfront design.**
- Main **risks**: **usability, changeability, scalability**.
- **Agile**, with **perpetual development** and **frequent releases**.
- Relies on **testing and peer review** instead of formal design review.

---

## 14. Case Study: Google

- **Design docs are written before major decisions.**
- **Self-organized, autonomous teams.**
- Decisions are reached via **rational persuasion plus data**.
- **The Tech Lead approves** designs.
- **Informal design docs** include:
  - Goals
  - Non-goals
  - Context diagrams
  - Interface descriptions
  - Data models
  - Alternatives considered
  - Justification for the chosen approach
- Overall approach: **risk-driven**.

---

## 15. Case Study: NASA Spacecraft Software

- Famous quote (Gene Kranz): "**Failure is not an option.**"
- **Lots of upfront design.**
- **Many models and formal reviews.**
- **Plan-driven** lifecycle with a fixed sequence of formal reviews:
  1. **System Definition Review**
  2. **Preliminary Design Review**
  3. **Critical Design Review**
  4. **System Integration Review**
  5. **Test Readiness Review**
  6. **System Acceptance Review**
- Produces **detailed design documents**.
- Main **risks**: **robustness, testability**.
- **Single launch date** — there is no opportunity to "ship a patch" once the spacecraft is en route.
- **Avoids intuitive decisions** — favors rational, documented decisions.
- **Formal V&V** (verification and validation).
- **External reuse is uncommon** — e.g., NASA rebuilt its own Linux kernel rather than reuse an external one.
- **Internal reuse is common.**

---

## 16. Case Study: Startups

- Famous quote: "**Fake it until you make it.**"
- Process: **lean and agile**.
- **MVP (Minimum Viable Product)** is built via **rapid prototyping**.
- **Reuse heavily** to move fast.
- Main **risks**: **extensibility, time-to-market**.
- **After reaching MVP / break-even**, the team should **pay more attention to clean code and architecture**.
- **Avoid over-engineering** early — but plan to pay down debt once the product is validated.

---

## 17. INVEST Criteria for User Stories (from L3)

A good user story satisfies **INVEST**:

- **I — Independent**: the story can be developed and delivered independently of other stories (minimizing scheduling and ordering constraints).
- **N — Negotiable**: the story is not a fixed contract; details can be discussed and refined with the customer.
- **V — Valuable**: the story delivers clear value to the customer or user.
- **E — Estimable**: the team can estimate the size/effort of the story (otherwise it cannot be planned into a sprint).
- **S — Small**: the story is small enough to fit comfortably within one iteration/sprint.
- **T — Testable**: the story has clear acceptance criteria so it can be verified when "done."

---

## 18. Adversarial Thinking (from L3)

**Adversarial thinking** is the practice of deliberately taking the perspective of an attacker, misuser, or hostile environment when designing and evaluating software:

- Ask "**how could this be broken, abused, or misused?**" rather than only "how is this supposed to work?"
- Used to expose missing requirements, security holes, edge cases, and assumptions that an honest user would never trigger but a hostile one will.
- Complements normal user-story thinking (which focuses on cooperative, well-intentioned users) by adding **misuse cases** and **threat scenarios**.

---

## Quick Reference Summary

- **Plan-driven (Waterfall)**: full requirements → full design → build → test → deploy. Good when requirements are stable and risks demand heavy upfront design (e.g., NASA).
- **Agile (Scrum)**: iterate in short sprints from a product backlog; embrace change; working software is the measure of progress.
- **Risk-driven (Fairbanks)**: amount of upfront design is proportional to risk; defer low-risk, low-cost-of-change decisions; lock down high-risk, hard-to-change ones early.
- **Match the process to the domain**: doghouse vs. skyscraper; Facebook vs. Google vs. NASA vs. startup.
- **People matter**: avoid ivory-tower architects; design collaboratively; balance rational and intuitive decision making; respect bounded rationality and cognitive biases.
- **Manage technical debt explicitly** alongside features.
- **Good user stories are INVEST**; complement them with **adversarial thinking** to surface misuse and abuse cases.


---

# Appendix: Lecture 19 Slides (raw extracted text)

The following is the full extracted text of Lecture 19. Preserved verbatim so no information is lost.

```text
=== SLIDE 1 ===
Tobias Dürschmid
Assistant Teaching Professor
Computer Science Department
CS 35L Software Construction
L18 – People &amp; 
Processes
=== SLIDE 2 ===
2
Learning Objectives
Of this Lecture
After this lecture you 
should be able to:
Explain the 
differences
 between 
agile
, 
plan-driven
, and 
risk-driven
 software construction
Describe the
 human aspects 
of software construction
Explain &amp; justify how to 
adjust
 the software construction 
process 
to 
domain-specific needs
=== SLIDE 3 ===
Peoples &amp; Processes
Agile, Plan-driven, and
Risk-Driven Software 
Construction
=== SLIDE 4 ===
In the Past
Big Upfront 
Design
 was very 
Common
Requirements
Design
Development
Testing
Deployment
Plan-Driven
Waterfall Model
 Requirements might 
change
 during development
 Customer 
feedback
 is considered very 
late
 in the process 
 Projects were 
delayed
 very often
Each activity is 
completed entirely 
before the next one 
What Challenges might Arise from this Process?
=== SLIDE 5 ===
5
Key Idea: Iterative, Incremental Development
       
How can we get 
continuous feedback 
from customers to identify 
missing &amp; misunderstood requirements?
       
Short, 
time-boxed periods
 (usually 1-4 weeks) where a specific set of work (a set of user stories, i.e., “
Feature Backlog
”) is completed and ready for review.
After each iteration, the 
increment is demonstrated to the customer
, feedback is collected, and new user stories are written from these insights 
Challenge: How to Involve Feedback in Software Construction?
Solution: Sprints / Iterations
=== SLIDE 6 ===
Alternative to Waterfall: 
Agile Manifesto
See 
https://agilemanifesto.org/
  
We are uncovering better ways of developing software by doing it and helping others do it. Through this work 
we have come to value
:
         
Individuals and interactions
 
over processes and tools
Working software
 
over comprehensive documentation
Customer collaboration
 
over contract negotiation
Responding to change 
over following a plan
         
That is, while there is 
value in the items on the right
, 
we 
value the items on the left more
.
2001
What implications does this have on software construction? 
What role should software design play in agile projects?
The important part
=== SLIDE 7 ===
7
The Scrum Process
Product 
Backlog
User Stories
User Stories
User Stories
Customer 
Input
Daily Standup
(Developers meet daily to discuss their progress)
Sprint Planning
(selects user stories from product backlog for one sprint
Sprint 
Backlog
Stories
User Stories
Sprint 
(2-3 Weeks of un-interrupted software development. Backlog does not change)
Product Increment
Sprint Review
(validation of the finished increment)
See 
https://www.atlassian.com/agile/scrum
 
=== SLIDE 8 ===
8
Planning Poker Helps 
Estimating User Story Sizes 
Numbers
represent
 
effort
 in 
imaginary
 
units
, 
not time
Large estimate
differences
trigger
a 
debate
 between 
highest
 and 
lowest
Part of 
Sprint Planning
=== SLIDE 9 ===
9
The Scrum 
Process
Product 
Backlog
User Stories
User Stories
User Stories
Customer 
Input
Daily Standup
(Developers meet daily to discuss their progress)
Sprint Planning
(selects user stories from product backlog for one sprint
Sprint Backlog
Stories
User Stories
Sprint 
(2-3 Weeks of un-interrupted software development. Backlog does not change)
Product Increment
Sprint Review
(validation of the finished increment)
See 
https://www.atlassian.com/agile/scrum
 
How do the INVEST criteria of user stories support Agile Development? Talk to your Neighbor!
=== SLIDE 10 ===
Design-related Principles 
behind the Agile Manifesto
Working software 
is the primary measure of progress. 
[…]
Continuous attention 
to technical excellence 
and 
good design enhances agility
. 
[…]
The best architectures, requirements, and designs 
emerge from 
self-organizing teams
. 
See 
https://agilemanifesto.org/principles.html
 
The quality of the design only matters if it is 
observable
Design is not an initial phase but part of 
every iteration
There is no single architect or top-down design
=== SLIDE 11 ===
Today 
No 
/ 
Tiny Upfront Design
Is Common
Agile
Development
Design
Development
Testing
Deployment
Requirements
Release
 Software is 
hard to change
 Improving Quality Attributes 
 
is hard
 Small 
Bus Factor
(i.e., number of people who can leave a project (“hit by a bus”) before the project stalls. Measures shared knowledge &amp; documentation)
Review
Read more here: 
Dikert, Kim, Maria 
Paasivaara
, and Casper 
Lassenius
. "Challenges and success factors for large-scale agile transformations: A systematic literature review." 
Journal of Systems and Software
 (2016)
What Challenges might Arise from this Process?
=== SLIDE 12 ===
What should we do instead? 
Tiny Upfront 
Design
Big Upfront 
Design
Risk-Driven
Design
Requirements
Design
Development
Testing
Deployment
Design
Development
Testing
Deployment
Requirements
Release
Review
=== SLIDE 13 ===
Risk-Driven Design
Read more here: Fairbanks, George. 
Just enough software architecture: a risk-driven approach
. Marshall &amp; Brainerd, 2010.
Identify 
biggest risks 
of the software 
and focus design on these risks.
The amount of risk involved in the project 
determines the 
amount of upfront design.
=== SLIDE 14 ===
Risks are Decisions that are 
hard to change
Programming Languages 
Target Platforms
Component Architectures &amp; Connectors
Interfaces
Quality Attributes
Example Risks
=== SLIDE 15 ===
What Risks are Most Important in These Domains?  
Online Shops
Games
Medical Software
Reliability
Usability
Performance
Security
Privacy
Robustness
Platforms
Changeability
Testability
=== SLIDE 16 ===
Collaborative Risk Identification Technique:
Risk Storming
Read more here: 
https://riskstorming.com/
																	      by Simon Brown										 
       
Model your software design as diagrams
Step 1: Model
Identify the risks silently on post-its
Step 2: Think
       
Add post-its to the diagram
Step 3: Share
       
Discuss risks and summarize
Step 4: Review
Colors represent
 priorities
=== SLIDE 17 ===
Identify and Mitigate Highest-Priority Risks
Make big design decisions early 
Defer all small-scale decisions until later.
High
 cost of change 
 
upfront design
Low cost of change 
 lean design
 
=== SLIDE 18 ===
Responding to change 
over following a plan
– Agile Manifesto 
Changeability in Agile Projects
A good architecture allows you to 
make decisions late 
A good architect maximizes the 
number of decisions 
not made
Information Hiding
SOLID Principles
Low Coupling
High Cohesion
=== SLIDE 19 ===
Feature Backlog vs 
Technical Debt Backlog 
User stories capture functional requirements in the feature backlog 
Maintain 
a technical debt backlog 
with issues 
that improve software design 
by refactoring &amp; building abstractions
Technical debt is the result of short-term-oriented decisions that make future changes more costly or impractical. 
Read more here: 
https://agilewaters.com/technical-debt-and-product-backlog/
 
=== SLIDE 20 ===
Examples of Technical Debt Issues
Fix 
code smells
 (e.g., duplicate code, high coupling, low cohesion, complex interfaces, …)
Improve
 documentation 
Architectural changes 
to support performance, scalability, …
=== SLIDE 21 ===
Integrating Technical Debt Issues in Agile Processes
Having a special role of an 
architect
 who 
maintains the technical debt backlog
 can be good option
Either include some technical 
debt issues in every sprint
, or dedicating 
one sprint to only reducing technical debt
=== SLIDE 22 ===
People &amp; Processes
How to Consider the 
Human Aspect of 
Software Engineering?
=== SLIDE 23 ===
Don’t Design In an 
Isolated 
Ivory Tower
I’m worried how the decisions made up there will 
affect my work
Read more here: 
https://techcommunity.microsoft.com/t5/azure-architecture-blog/armchair-architects-architects-vs-the-ivory-tower/ba-p/3711703
 
DON’T 
ENTER
I work in 
isolation
 and 
pass down my wisdom 
to the developers who will implement 
my design 
Ivory Tower Architects are:
Not involved in the activities 
of software construction
Ignoring input from other team members
  
Ivory Tower Designs are:
Elegant, beautiful, well-documented
Only work in theory 
=== SLIDE 24 ===
Lesson Learned: 
Software Construction
is a Collaborative Activity
 Encourage other group members to present 
design alternatives
 Consult 
domain experts 
to take advantage of their 
experience
 Include 
developers 
in important discussions to ensure 
realism
 of design
 Stay in touch with the current state of the 
codebase 
Read more here: 
Smrithi Rekha V, 
Muccini
, Henry. "Group decision-making in software architecture: A study on industrial practices." Information and software technology 101 (2018): 51-63.
=== SLIDE 25 ===
Rational Vs. Intuitive Decision Making
Intuitive Decision Making
(unconscious decisions 
relying on “gut feeling”)
Rational Decision Making
(explicitly identifying, evaluating, 
and ranking design options via logical reasoning)
Documentation of rationale helps 
revisiting 
decisions
Helps
 
gain
 
explicit
 
knowledge
 
and 
experience
Guides
 
non-experts
 
to
 
better design
 
[1]
Can access 
all
 
implicit
 
knowledge
 
and
 
experience
 
Helps experts 
with many years 
of experience to make
 
better decisions
Can lead to 
faster
 
decision making
Hard to 
communicate
 
/ justify it
Challenging for 
group decision making
Prone to 
cognitive biases
(e.g., anchoring, confirmation bias, …) 
Can access 
only explicit knowledge
[1] 
Tang, Antony, et al. "Design reasoning improves software design quality." 
International Conference on the Quality of Software Architectures
. Springer. 2008.
explicit knowledge
implicit knowledge
=== SLIDE 26 ===
Lesson Learned: 
Combine Both Processes
Intuitive Decision Making
(unconscious decisions 
relying on “gut feeling”)
aka. 
Naturalistic Decision Making
Rational Decision Making
(explicitly identifying, evaluating, 
and ranking design options via logical reasoning)
Read more here: 
Power, Ken, and Rebecca 
Wirfs
-Brock. "An exploratory study of naturalistic decision making in complex software architecture environments." 
European Conference on Software Architecture
  2019. 
and 
Tang, Antony, et al. "Human aspects in software architecture decision making." 
2017 IEEE International Conference on Software Architecture (ICSA)
.
and 
Pretorius, 
Carianne
, et al. "Combined intuition and rationality increases software feature novelty for female software designers." 
IEEE Software
 38.2 (2020): 64-69. 
Appropriate Context: 
Time pressure
Experienced decision makers
Lack of information
Hard-to-define problem 
Uncertainty
“Good-enough” is sufficient
Appropriate Context: 
Justification is Needed
Well-structured problem
Optimal decision is needed
=== SLIDE 27 ===
Bounded Rationality
The rationality of our design decisions is 
limited by our cognitive capabilities 
Realistically, we 
cannot consider all possible
 
design options to achieve an optimal design
Designers often 
retroactively rationalize 
decisions 
Read more here: 
Tang, Antony, et al. "Human aspects in software architecture decision making." 
2017 IEEE International Conference on Software Architecture (ICSA)
.
and 
Tang, Antony, and Hans van Vliet. "Software designers satisfice." 
Software Architecture: 9th European Conference, ECSA 2015,
How do these insights impact our approach to software construction?
=== SLIDE 28 ===
People &amp; Processes
How to 
Adjust the Design Process
To Domain-Specific Needs?
=== SLIDE 29 ===
How does the 
Design Process Differ 
for 
Doghouses
 and 
Skyscrapers
?
Many People 
involved
Higher Risk
 of Failure
Lower Risk 
of Failure
Higher Percentage of 
Intuitive Decision Making
Higher Percentage of 
Rational Decision Making
Fewer People 
involved
Short Process
of Construction
Long Process
of Construction
Less
Upfront
Design 
&amp; Models
More
Upfront
Design 
&amp; Models
Read more here: 
Software Architecture in Practice 3
rd
 Edition Chapter 15 
 
How do these insights apply to software engineering?
=== SLIDE 30 ===
 
More Upfront Design
Detailed Design Documents
Rigorous Design Evaluation 
Lesson Learned: 
Adjust the Design Process to the Specific Domain
Some Upfront Design 
Focusing on Highest Risks 
Designing while Coding
Higher Risk Domains
Lower Risk Domains
=== SLIDE 31 ===
“Go fast and break things” 
– Mark Zuckerberg
(CEO of Facebook / Meta)
Web-based Social Media Apps
     
    
Small to Medium
. Mostly limited to technology choices, client-server interfaces, component structures, and data models
       
Usability
: Easily change UI
Changeability
: Easily add new features
Scalability
: Support growth of userbase
       
Agile Process
: Iterative development. “Perpetual development” (i.e., no predefined final objective). Frequent releases. Testing and peer review instead of design review. Responding 
to usage metrics, public opinion, and competitors. 
In practice, a large portion of decisions are made intuitively, due to rapid development cycle. 
We recommend to deliberately think of hard-to-change design decisions! 
For more details see: 
Feitelson, 
Dror
 G., Eitan 
Frachtenberg
, and Kent L. Beck. "Development and Deployment at Facebook." 
IEEE Internet Computing
 17.4 (2013): 8-17.
Amount of Upfront Design
Risks
Process Changes
=== SLIDE 32 ===
Case Study: Design Decision Making 
at
Decisions are made by 
self-organized, autonomous teams 
based on rational persuasion and data. Tech Lead approves the design.
Decision Makers
Risk-Driven
: Creating design docs 
before
 implementing 
major decisions
. Discussion &amp; review mostly via comments. Some teams have weekly design review meetings.
Design Process
Read more here: 
https://www.industrialempathy.com/posts/design-docs-at-google/
 and 
https://open.lib.umn.edu/organizationalbehavior/chapter/11-1-decision-making-culture-the-case-of-google
   
Informal Design Docs 
(goals, non-goals, context diagrams, interface descriptions, 
datamodels
, alternative options, and justification for chosen design).
Design Artifacts
=== SLIDE 33 ===
       
Plan-Driven Process
: Limited benefits of full-cycle iterations, due to single launch date.
Avoiding intuitive decision making
 to extensively document &amp; review design decisions. 
Formal validation &amp; verification of important components due to high cost of failure. 
External reuse is very un-common, due to extreme reliability requirements. NASA even re-built their own Linux kernel. Internal reuse is very common. 
Spacecraft Software
     
    
A lot! 
Many models &amp; formal design 
reviews. Mission-critical elements are analyzed very rigorously. 
       
Robustness
: Operate reliably in uncertain environments without human interference
Testability
: Detecting faults on Earth is hard
“Failure is not an option” 
– Gene Kranz 
(NASA Flight Director of Apollo 13)
For more details see: 
Markosian, Lawrence Z., et al. "Program model checking using Design-for-Verification: NASA flight software case study." 2007 IEEE Aerospace Conference. IEEE, 2007.
Amount of Upfront Design
Risks
Process Changes
=== SLIDE 34 ===
Case Study: Design Decision Making 
at
       
Project managers develop, record, and maintain software design documents are reviewed based on 
detailed checklists.
Decision Makers
       
Plan-Driven
: System Definition Review -&gt; Preliminary Design Rev. -&gt; Critical Design Rev. -&gt; System Integration Rev. -&gt; Test Readiness Rev. -&gt; System Acceptance Rev.  
Design Process
Read more here: 
https://swehb.nasa.gov/display/SWEHBVD/SWE-058+-+Detailed+Design
 
       
Detailed Design Documents
 (very long documents outlining every aspect of the structure, behavior, and quality attributes of the design)
Design Artifacts
=== SLIDE 35 ===
=== SLIDE 36 ===
 
Lean &amp; Agile Process
: Rapid prototyping &amp; taking shortcuts to quickly get to the 
minimum viable product (MVP)
. Relying as much on reuse as possible can speed up development.  
After reaching the MVP and/or breaking-even point, paying more attention to clean code and clean architecture supports future growth, onboarding of new developers, extensibility, and scalability to build a robust foundation for long-term success. But: 
Avoid over-engineering
Software Startups 
     
    
None to Small. 
Most design happens implicitly while coding or after first release. Decisions are driven by short-term needs.
Amount of Upfront Design
       
Extensibility
: Quickly respond to new customer needs
Time-to-Market
: Quickly start breaking even 
Risks
Process Changes
“Fake it until  you make it”
For more details see: 
Tegegne, 
Esubalew
 
Workineh
, 
Pertti
 
Seppänen
, and Muhammad 
Ovais
 Ahmad. "Software development methodologies and practices in start‐ups." IET Software 13.6 (2019)
=== SLIDE 37 ===
Please fill out your Exit Tickets on Bruin Learn!
Credits: These slide use images from 
Flaticon.com
 (Creators: 
Freepik
, itim2101, 
Imaginationlol
, HAJICON, 
Smashicons
, 
Eucalyp
)
SVGRepo
 (
Cretors
: 
Twemoji
 Emojis),  and the Noun Project (look up by 
Syawaluddin
, and Designer by Amethyst (CC BY 3.0))
In your own words, please summarize 
three key insights 
you learned about 
Plan-Driven, Agile, and Risk-driven Processes 
today. (Do not copy phrases from the lecture slides)
Please leave any questions that you have about today’s material and things that are still unclear or confusing to you (if none, simply write N/A)
How does what you learned today about the 
human aspect 
of software development impact team-work in a project like your course project? Please describe 
actionable insights 
you would like to 
apply during your next team project
.
37

```


---

# Appendix: Lecture 20 Slides — Course Summary (raw extracted text)

The full L20 Summary lecture (Tobias Dürschmid) recaps every topic. Appended here in people-and-processes.md since it includes substantial Scrum/process recap content.

```text
CS 35L Software
Construction
L20 � Summary
Assistant Teaching Professor
Computer Science Department
Your Skills Are Needed!                                Attention                               You are the only ones
Come Well Prepared!                                     Bruins!                                  left with software
                                                       We need                                  construction skills
  We                                     Thanos has    your help
need to                                    written a                                                                             2
write a                                   Makefile!        with
                                                      "Operation
 good                                    What does
 user                                     it do????       Final
story!!!                                                 Exam"
                                         How to fix
                                         our git???
We need to     We need to                How can TDD
refactor this       find                   help us???
   code to     mistakes in                    This is too much code! We
 defend the                              Constnruecetidona: Lteocptu-rdeo2w0 �nSruemamdaeryr!
  universe!    CthSe3s5eL USoMftwLare
               Todbiaiags rDa�mrscshmid
Now You Should be Able to:
� Design, Implement, and Test Medium-Size Software
� Collaborate with other students to build a system that is
 larger than what you could build on your own
A Good Software Engineer has a Great
Understanding of Existing Tools
      PyTest                                        Debugger
JS C
                     Shell Scripts                         Make
                                                                             4
A Great Software Engineer Follows a
High-Quality Design Process
Which software design practices have you learned
     in this class that increase software quality?
                  Talk to your Neighbor(s)!
You've learned Concepts & Technology
    Design           Implement                            Test
Interoperability   Shell Scripts                    PyTest
Security           Git                              Test-Driven
Networking         GNU Make                         Development
Data Management    Debugging                        Integration
                   Clean Code                       Tests
Modeling
Reuse
Design Principles
Design Patterns
How to make Design Decisions (And how to write
them down in your Individual Reports)
1. Describe the Problem
2. List all design options
   (models, interface descriptions, ...)
3. Evaluate the design options to identify how well you solve
   your problem
4. Make a decision based on the priorities of your problem
   and justify your choice
Modeling
   Which diagrams are
useful for which aspects
      of the software?
Talk to your Neighbor(s)!
Different Views Visualize                                                                                                                     Cover of the
Different Aspects of Software
                                                                   Read more here: Documenting textbook
                                                                   Software Architectures: Views  'Documenting
                                                                                                  Software
                                                                   and Beyond by Clements et al. Architectures:
                                                                                                                                              Views and Beyond'
                                                                   (available in UCLA digital library) by Clements et al.,
                                                                                                  Second Edition.
Code View /                           Data View           Run-Time View /     Behavioral View
Module View                                               Component-
                                      Describes           Connector View      Describes
Describes Structure                   Structure of Data                       Interactions
of Code                                                   Describes how Run-
                                                 N        Time Component
             Shape                                        are Connected
                                                Course
                -color: int                        study  Client              client_1:  server:
    +setColor(r: int, g: int, b:int)
                                                                              Client LibraryServer
    Rectangle                         Student             Library
                                          UID             Server
        -width: int
       -length: int
 +setWidth(width: int)
+setHeight(height: int)
TDD
             What is TDD?
     What are the TDD Rules?
Why are these Rules important?
     Talk to your Neighbor(s)!
     CS 35L Software Construction: Lecture 20 � Summary  10
     Tobias D�rschmid
Test-Driven Development (TDD)                                          Red            Green
Puts Testing First
Originally proposed by Kent Beck                                            Refactor
(who later went on to work for Facebook/Meta)
 Red      For your new requirement write a small test that fails,
Green     and perhaps doesn't even compile at first
          Make the test pass with minimal coding effort,
          potentially using simplifying shortcuts in the process
Refactor  Make the design more elegant, cleaner, and potentially
          faster while not changing the functionality of the program
More in on this in "Test Driven Development: By Example" by Kent Beck
Information Hiding
What is Information Hiding?
     How can we achieve
     Information Hiding?
  Talk to your Neighbor(s)!
The Information Hiding Principle
     Information Hiding Principle
  "We propose [...] that one begins with a list of difficult design
  decisions or design decisions which are likely to change.
  Each module is then designed to hide such a decision
  from the others." � David Parnas, 1972 (4 years after "Software Crisis" NATO conference)
One of the most important concepts in
              computer science
David L. Parnas "On the criteria to be used in decomposing systems into modules" Communications of the ACM 1972
Modules should have Fixed Interfaces to
Allow for Variations in the Implementation
The interface of a          A stable contract that
module is what       describes WHAT the module does
should be visible
The implementation
of a module is what
should be hidden
                     This is HOW the module fulfills its
                     contract. It can be changed freely
                       without affecting the rest of the
                      system, as long as the interface
                               remains the same.
Information Hiding & TDD?
How does TDD relate to information hiding?
 Does it increase or decrease modularity?
             Talk to your Neighbor(s)!
Clean Code
  What is Clean Code?
  How do we make our
        Code Clean?
Talk to your Neighbor(s)!
Four Pillars of Clean Code
Meaningful  Simplified     Purposeful               Design-by-
  Naming    Structure   Documentation                Contract
Debugging
   What is Debugging?
Which steps are needed?
Talk to your Neighbor(s)!
What is Debugging?               How do the other
                                       software
� The systematic process of
  finding & fixing faults,          construction
  (aka. "bugs") in a program's   techniques help
  source code                      you with this?
� 1) Investigating symptoms to
  Reproduce the Bug
� 2) Locating the faulty code
� 3) Determining the root cause  "Debugging is like being the
  of the bug
� 4) Implementing and verifying  detective in a crime movie where
  a fix                               you are also the murderer."
                                                    � Filipe Fortes
Programming Languages
For which use cases would you use
              C / Python / JS?
       Talk to your Neighbor(s)!
In C You are the Boss
but Also the Janitor
 You are the Boss
 You have full control
 You are the Janitor
 You have to do everything yourself
Software Process
   What is Agile software development?
When should I use Agile over plan-driven?
             Talk to your Neighbor(s)!
The Scrum                          How do the INVEST criteria of user stories support
Process                                 Agile Development? Talk to your Neighbor!
   Customer                                                       Daily Standup
      Input
                                                              (Developers meet daily to discuss their progress)
Product                               Sprint         Sprint            Sprint             Product
Backlog                             Planning        Backlog                              Increment
                                                                    (2-3 Weeks of un-
SSUStUotUsotrsoeriseeirreeirsesrs    (selects user    SUtsoerires  interrupted software                       23
                                     stories from    Stories
                                   product backlog                     development.
                                    for one sprint                  Backlog does not
                                                                           change)
                                       Sprint Review
                                               (validation of the finished increment)
See https://www.atlassian.com/agile/scrum
                          CS 35L Software Construction: Lecture 20 � Summary
                          Tobias D�rschmid
How does the Design Process Differ for
Doghouses and Skyscrapers?
How do these insights apply to software engineering?
Fewer People Short Process                                                Long Process       Many
                                                                                            People
involved       of Construction                                             of Construction
                                                                                            involved
Lower Risk                         Less                                     More
                                 Upfront                                  Upfront           Higher Risk
   of Failure                    Design                                   Design
                                                                                               of Failure
                                 & Models                                 & Models
Higher Percentage of                                                      Higher Percentage of
Intuitive Decision Making                                                 Rational Decision Making
Read more here: Software Architecture in Practice 3rd Edition Chapter 15
               CS 35L Software Construction: Lecture 20 � Summary
               Tobias D�rschmid
Reuse
   Which Design Principles
For Reuse have you learned?
   Talk to your Neighbor(s)!
       CS 35L Software Construction: Lecture 20 � Summary  25
       Tobias D�rschmid
Cost-Benefit Analysis for External Reuse
Effort to adapt the                                                           Effort saved by
reusable module                                                               reusing the module
Integration Effort                                                            Implementation Effort
(Complexity, Similarity                                                       Testing Effort
of Context)
Finding the Module                                                            Benefit of Update
                                                                              Propagation
Updating Effort
                                                                              Read more here: Why reinventing the wheels? An
Limiting Changeability                                                        empirical study on library reuse and re-
                                                                              implementation (Xu et al. 2019)
                          CS 35L Software Construction: Lecture 20 � Summary
                          Tobias D�rschmid                                                                             26
Ariane 5 Failure           Can reach higher velocities                          $370M
                           than Ariane 4                                        in 37s
Describe Reuse Rules that
Avoid Failures Like This      Assumed                                                      27
                           lower velocity
Ariane 4 Flight Control System                      Due to
                                                Performance
  Worked                                        Requirement
Perfectly!
              Horizontal Velocity - 16 Bit Int
Ariane 5 Flight Control System
Caused Self-  Horizontal Velocity - 16 Bit Int  Overflow Error
 Destruction
See http://esamultimedia.esa.int/docs/esa-x-1819eng.pdf
                            CS 35L Software Construction: Lecture 20 � Summary
                            Tobias D�rschmid
Gen AI
    How should we use
Generative AI in software
       Construction?
Talk to your Neighbor(s)!
        CS 35L Software Construction: Lecture 20 � Summary  28
        Tobias D�rschmid
Security
Which Attacks are most
          common?
   How can we defend
       against them?
Talk to your Neighbor(s)!
          CS 35L Software Construction: Lecture 20 � Summary  29
          Tobias D�rschmid
Which Security Attribute(s) is/are affected?
                                   2016: Ransomware takes Hollywood
                                    Presbyterian Medical Center offline
                                            $3.6M demanded by attackers
                                                               Malware locks systems by encrypting files and
                                                                                   demanding ransom to obtain the
                                                                                                             decryption key
See https://www.csoonline.com/article/554745/ransomware-takes-hollywood-hospital-offline-36m-demanded-by-attackers.html
Explain this Comic             '); ends the SQL query
                               DROP TABLE Students removes
                               the entire student data base table
                               ; -- ignores the remainder of the to
                               original query to avoid syntax errors
Source: https://xkcd.com/327/
Quiz: Which Security Attribute(s) is/are Affected
by SQL Injection Attacks?
       Confidentiality            Attacker                     Website
Attackers can read sensitive
   data from the database
             Integrity                              SQL
                                                    Injection
Attackers can modify, insert, or
   delete data in the database
name = get_user_input("username");         Rare in practice
pass = get_user_input("userpassword");
sql = ('SELECT * FROM Users '                         Availability
            'WHERE Name = @0 '              Attacks can disrupt database
            'AND Pass = @1')                  operations (e.g., dropping
user = db.execute_query(sql, name, pass);
login(user) if user else retry()           tables, causing system crashes)
You started with Magic Skills!
You can "summon" magical programs simply by typing code
You can make the computer listen to your commands
  CS 35L Has Transformed You Into A
True Software Construction Superhero!
   BRUINS         Wednesday
ASSEMBLE!          3pm-6pm
We Need your Help
Come Well-Prepared CS 35L Software Construction: Lecture 20 � Summary  35
Reminder: Please fill out the Course Evaluations
For Grading Credit in this Course
Link: https://go.blueja.io/c5HucLjXZ0G73cdkPAlbzg
Please fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three key insights you learned in this course.
(Do not copy phrases from the lecture slides)
Which of the software construction techniques you learned in this class are most effective
for increasing the bus factor of a project? Please explain why.
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slide use images from Flaticon.com (Creators: Freepik, itim2101, Imaginationlol, HAJICON, Smashicons, Eucalyp)
SVGRepo (Cretors: Twemoji Emojis), and the Noun Project (look up by Syawaluddin, and Designer by Amethyst (CC BY 3.0))

```
