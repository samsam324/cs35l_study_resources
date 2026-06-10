# People & Processes Cheatsheet (candidate items)

## Three process types (compare/contrast)
| | Plan-Driven (Waterfall) | Agile | Risk-Driven |
|---|---|---|---|
| Upfront design | Heavy | Tiny | Proportional to risk |
| Iteration | None (linear) | Short (1-4 weeks) | Mixed |
| Feedback | Late | Continuous | Risk-focused |
| Best for | High-risk, well-defined (NASA, medical) | Web apps, startups | Mid-range projects |

## Waterfall Model
Requirements → Design → Development → Testing → Deployment
- Each phase completed entirely before next
- **Challenges:** requirements change, late feedback, project delays

## Agile Manifesto (2001)
> While there is value in the items on the right, **we value the items on the left more.**

- **Individuals and interactions** over processes and tools
- **Working software** over comprehensive documentation
- **Customer collaboration** over contract negotiation
- **Responding to change** over following a plan

## Scrum Process
```
Customer Input
     ↓
Product Backlog ─→ Sprint Planning ─→ Sprint Backlog
                                          ↓
                                     SPRINT (2-3 weeks, locked)
                                          ↓
                                  Product Increment
                                          ↓
                                   Sprint Review
                                          ↓
                                  [loop back to Sprint Planning]

Daily Standup runs DAILY throughout
```

### Scrum events
- **Sprint Planning** — pick user stories for sprint
- **Sprint** — 2-3 weeks of uninterrupted dev (**backlog doesn't change**)
- **Daily Standup** — devs discuss progress (15 min)
- **Sprint Review** — validate increment with stakeholders
- **Sprint Retrospective** — improve process

### Scrum artifacts
- **Product Backlog** — all user stories
- **Sprint Backlog** — stories chosen for THIS sprint
- **Product Increment** — working software at end of sprint

## User Stories — INVEST criteria
- **I**ndependent — can be done in any order
- **N**egotiable — details can be discussed
- **V**aluable — delivers value to user
- **E**stimable — team can size it
- **S**mall — fits in one sprint
- **T**estable — has clear acceptance criteria

### User story template
```
As a [role],
I want to [do something]
so that [benefit].
```

## Planning Poker
- Estimate user story sizes
- Units are **imaginary effort points**, NOT time
- Each dev privately picks a number; reveal together
- Large estimate differences → debate (highest vs lowest)
- Common scale: 0, 1, 2, 3, 5, 8, 13, 20, 40, 100 (Fibonacci-ish)

## Bus Factor
> Number of people who can leave the project before it stalls.

- Low bus factor (1) = critical knowledge concentrated in one person
- High bus factor = knowledge well distributed
- Increase via: documentation, code review, pair programming

## Tiny Upfront Design — challenges
- Software hard to change
- Low quality attributes (perf, security)
- Small bus factor

## Risk-Driven Design (George Fairbanks)
- **Identify biggest risks** → focus design effort there
- Amount of risk determines amount of upfront design
- High cost of change → more upfront design
- Low cost of change → lean design

### Risks = decisions hard to change
- Programming languages
- Target platforms
- Component architectures / connectors
- Interfaces
- Quality attributes (perf, security, scalability)

### Risk Storming (Simon Brown) — 4 steps
1. **Model** — agree on system diagram
2. **Think** — independently identify risks (sticky notes)
3. **Share** — place stickies on diagram
4. **Review** — cluster + prioritize

## Changeability in Agile
- Good architecture allows LATE decisions
- Maximize decisions NOT yet made
- Use: Information Hiding, SOLID, Low Coupling, High Cohesion

## Feature backlog vs Technical Debt backlog
- **Feature backlog** — user stories (functional reqs)
- **Technical debt backlog** — refactoring, abstraction improvements
- Technical debt = short-term decisions making future changes costly

### Examples of tech debt
- Fix code smells (duplication, coupling)
- Improve documentation
- Architectural changes for scalability

### Integration approaches
- Mix debt issues into every sprint
- Dedicate one sprint per quarter to debt-only

## Ivory Tower Architects (anti-pattern)
- Not involved in actual construction
- Ignore developer input
- Produce elegant designs that don't work in practice
- **AVOID** — software construction is collaborative

## Rational vs Intuitive Decision Making
| | Rational | Intuitive (Naturalistic) |
|---|---|---|
| Approach | Explicit, logical, ranked alternatives | Unconscious "gut feeling" |
| Knowledge | Explicit only | Includes implicit experience |
| Speed | Slow | Fast |
| Bias risk | Anchoring, confirmation | Hard to justify |
| Best for | Well-structured, justification-needed | Time pressure, expertise, hard-to-define |
| Communication | Easy | Hard |

**Combine both.** Use rational for important / contested; intuitive for fast-moving work.

## Bounded Rationality
- Cognitive limits prevent considering ALL options
- We can't achieve truly optimal designs
- Designers often **retroactively rationalize** decisions
- Implication: accept "good enough" and iterate

## Doghouse vs Skyscraper analogy
| | Doghouse | Skyscraper |
|---|---|---|
| People | Few | Many |
| Process | Short | Long |
| Risk | Low | High |
| Upfront design | Less | More |
| Decision style | Intuitive | Rational |

→ Adjust your process to the **risk profile** of the project.

## Case studies (process by domain)
| Org | Process | Upfront design | Risks |
|---|---|---|---|
| **Facebook/Meta** | Agile, "Move fast & break things" | Small-medium | Usability, changeability, scalability |
| **Google** | Risk-driven with Design Docs | Medium (informal docs) | Performance, scale |
| **NASA** | Plan-driven, formal reviews | Heavy | Reliability, testability, "Failure is not an option" |
| **Startups** | Lean Agile, MVP-first | None to small | Time-to-market, extensibility |

## NASA review chain
System Definition → Preliminary Design → Critical Design → System Integration → Test Readiness → System Acceptance

## Google Design Doc (4 parts)
1. Context & Scope
2. Goals & Non-Goals
3. The Design
4. Alternatives + trade-offs

## Quick acronyms
- **INVEST** — Independent, Negotiable, Valuable, Estimable, Small, Testable (user stories)
- **MVP** — Minimum Viable Product
- **PO** — Product Owner
- **SM** — Scrum Master
- **CI/CD** — Continuous Integration / Deployment

## Additional items (potentially missing)

### Kanban (alternative to Scrum)
- Visualize workflow on a board (To Do | In Progress | Done)
- Limit WIP (Work In Progress) per column
- Pull system (take next item when you have capacity)
- Continuous flow (no sprints)
- Compared to Scrum: more flexible, less ceremony

### Scrum vs Kanban (quick)
| | Scrum | Kanban |
|---|---|---|
| Time | Sprints (2-3 wks) | Continuous |
| Roles | PO, SM, Team | Minimal |
| Cadence | Scheduled ceremonies | As-needed |
| Backlog | Re-prioritized per sprint | Continuous re-order |
| Use case | Feature dev, large teams | Support, ops, mixed work |

### Agile principles (full list, condensed)
The 12 principles behind the Agile Manifesto:
1. Customer satisfaction via early delivery
2. Welcome changing requirements
3. Deliver working software frequently
4. Business + devs work daily together
5. Build projects around motivated individuals
6. Face-to-face conversation (most effective)
7. Working software = primary measure of progress
8. Sustainable pace
9. Continuous attention to technical excellence
10. Simplicity (maximize work NOT done)
11. Self-organizing teams produce best architectures
12. Reflect and adjust

### Agile != "no planning"
- Agile plans iteratively
- Plans change based on feedback
- Doesn't mean ad-hoc/chaos

### Sprint cadence variations
- **1-week sprint** — fast feedback, lots of overhead
- **2-week sprint** — most common
- **3-week sprint** — moderate ceremony
- **4-week sprint** — long; risks feedback being late

### Definition of Ready (DoR)
Conditions a story must meet to enter a sprint:
- Acceptance criteria clear
- Sized (story points)
- Dependencies identified
- Mockups available (if UI)

### Definition of Done (DoD)
Conditions a story must meet to ship:
- Code written + reviewed
- Tests passing
- Documentation updated
- Deployed to staging
- Accepted by PO

### Velocity
- Story points completed per sprint
- Averaged over 3+ sprints for predictability
- Used to plan future sprints
- NOT a productivity metric per dev

### Backlog refinement (Grooming)
- Mid-sprint activity (not an official event)
- Break large stories into smaller ones
- Add details to upcoming stories
- Estimate sizes

### Retrospective (often overlooked)
- After Sprint Review
- "Start, Stop, Continue" or "Mad/Sad/Glad" formats
- Identifies process improvements
- Critical for continuous improvement

### User story examples
GOOD:
> As a returning customer, I want my saved address to auto-fill at checkout, so that I can pay quickly.

BAD (too big):
> As an admin, I want to manage all users.
(Split into: view list, search, edit, delete, role mgmt, etc.)

### Story point reality check
- Story points ≠ hours
- Capture COMPLEXITY + EFFORT + UNCERTAINTY
- Use relative sizing (this is twice as big as that)
- Common: Fibonacci-ish (1, 2, 3, 5, 8, 13)
- Anything 13+ → SPLIT IT

### Common Scrum anti-patterns
- "ScrumBut" — "we do Scrum, BUT we skip retrospectives"
- Sprint goal drift mid-sprint
- Stories not really "Done" at sprint end
- PO not available to answer questions
- Standups become status reports to managers
- Velocity used to compare individuals

### Tracking tools
- Jira, Linear, Asana, ClickUp
- GitHub Projects, GitLab boards
- Trello, Notion

### Estimation pitfalls
- Padding ("add 20% buffer")
- Anchoring (first estimate biases others)
- Optimism bias (devs systematically underestimate)
- Halo effect (last sprint went fast → assume this one will too)

### Case studies recap (from lecture)
- **Facebook/Meta** — "Move fast & break things"; agile; small upfront; risks (usability, changeability)
- **Google** — Design Docs first; risk-driven; informal docs
- **NASA** — plan-driven; formal review chain (SDR → PDR → CDR → SIR → TRR → SAR); heavy upfront
- **Startups** — lean; MVP-first; minimal upfront

### Process selection guidance
| Domain | Process |
|---|---|
| Spacecraft / medical | Plan-driven (NASA model) |
| Web app | Agile / Lean |
| Banking / regulated | Risk-driven with design docs |
| Research / prototype | Agile / Kanban |
| Maintenance / ops | Kanban |

### Bus factor improvement strategies
- Documentation
- Code review (knowledge transfer)
- Pair programming
- Internal tech talks
- Rotation of responsibilities

### Decision-making research (lecture cites)
- Considering 3+ design alternatives → better designs (Petre 2009)
- Rational decision-making → better outcomes for junior engineers (Tang 2008)
- Combining rational + intuitive best (Power & Wirfs-Brock 2019)

### Ivory Tower vs Embedded architect
- **Ivory Tower** — sits away, hands down decrees, doesn't know real codebase
- **Embedded** — works inside team, codes, listens, iterates
- Always prefer Embedded.

### When to use plan-driven (vs agile)
- High risk of failure (mission-critical)
- Hard to change after deploy (firmware, spacecraft)
- Regulatory requirements
- Single launch deadline
- Limited customer feedback opportunity

### When agile is preferable
- Requirements likely to change
- Customer feedback available
- Cost of change is low
- Web / mobile / SaaS
- Iterative deployment possible

### Common rituals quick ref
| Event | Frequency | Duration | Purpose |
|---|---|---|---|
| Daily standup | Daily | 15 min | Share progress + blockers |
| Sprint planning | Per sprint | 1-4 hr | Pick stories for sprint |
| Sprint review | End of sprint | 1-2 hr | Demo to stakeholders |
| Retrospective | End of sprint | 1-1.5 hr | Improve process |
| Grooming | Weekly | 1 hr | Refine upcoming stories |
