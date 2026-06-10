# CS 35L Lecture 14 â€“ Coding with GenAI

> "I suspect that machines to be programmed in our native tonguesâ€”be it Dutch, English, American, French, German, or Swahiliâ€”are as damned difficult to make as they would be to use." â€”Edsger W. Dijkstra (1979)

Instructor: Tobias DÃ¼rschmid, Assistant Teaching Professor, Computer Science Department.

---

## Learning Objectives

After this lecture you should be able to:

- **Explain how AI coding agents work**
- **Identify challenges & risks** that result from the use of AI coding agents
- Apply software engineering techniques to **mitigate the risks** of AI-generated code
- Apply **prompt engineering** techniques to **maximize the usefulness** of LLMs

---

## What is Different Now? (1960 vs 2026)

A historical comparison: in 1960, developers feared "These compilers are taking my job!" In 2026, developers fear "These LLMs are taking my job!" Three dimensions to compare:

### Productivity Gains
- **1960 (compilers)**: ~10X gain â€” a single developer can write the code for much larger software systems
- **2026 (LLMs)**: ~1.21X[1] â€“ 1.5X[2] gain

### Sound vs. Unsound Abstractions
- **Sound Abstractions (compilers)**: Results of a compiler are **predictable** since the abstractions are **procedural**.
- **Unsound Abstractions (LLMs)**: **Non-determinism** and **black-box** architecture of LLMs make the result **unpredictable**.

### Remaining Challenges
- Requirements, design & quality assurance still require **human intelligence** due to **essential complexity**.

---

## Industry Perspective: Thomas Dohmke (Former CEO of GitHub, @ashtom)

Quote from Dohmke's tweet (full tweet: https://x.com/ashtom/status/1952409910236291109; see https://ashtom.github.io/developers-reinvented; interview: https://www.youtube.com/watch?v=3WNukz5-Ch0):

> "**The evidence is clear: Either you embrace AI, or get out of this career.**
>
> Our latest field study with 22 developers who are integrating AI deeply into their workflows reveals a striking trend: those who persist beyond early skepticism emerge with dramatically higher ambition, technical fluency, and job satisfaction.
>
> **They're not writing less code â€“ they're enabling more complex, system-level work through orchestration.** [â€¦] Here's what we're seeing:
>
> - AI is on track to write 90% of code within the next 2â€“5 years.
> - Developers aren't worried. They're optimistic and realistic about the changes ahead.
> - **New skills matter now**, e.g. agent orchestration, iterative collaboration, and critical verification.
> - **Time savings? Sure. But the real shift is ambition. Developers are raising the ceiling, not just lowering the cost.**"

---

## How LLMs Work â€” "Statistical Parrots"

LLMs (Large Language Models) work like "Statistical Parrots." There are three phases to producing a useful coding LLM:

### 1. Pre-Training â€” Creating a Base Model / Foundation Model
- The model is trained on **vast amounts of code** to accurately **predict the most likely next token given all previous tokens**.
- Having **memorized** a large amount of code and **abstracted** key patterns in the code, the LLM can **reproduce variations** of the **most common coding tasks**.

### 2. Post-Training â€” Optimizing the Model for a Particular Use Case
- **Fine-tuning** the model on **labeled input-output pairs** (e.g., instructions + code solutions, such as LeetCode problems).
- Using **human feedback** to optimize the model using **reinforcement learning** (RLHF) â€” e.g., developers rank code outputs on correctness, efficiency, readability.

### 3. Inference â€” Prompting the Model for a Task
- Based on a **natural language prompt**, the LLM tries to produce the **most likely sequence of answer tokens** (usually in a **non-deterministic** way).

### Example Inference
Prompt: *"code the most efficient and shortest Python implementation of Fibonacci"*

Output:
```python
def fib_iter(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a
```

---

## Trusting a "Parrot Witness" / "Parrot Coder"

### Parrot Witness analogy (how trustworthy is a parrot?)
- Is **likely to repeat** what has been said many times before
- Might repeat phrases **out of context** or from a long time ago
- Might quote **factually-incorrect** statements
- Might **make up** statements
- **Useful in some cases, but not 100% trustworthy**

### Parrot Coder (aka "AI Coding Agent") â€” same warnings apply
- Is **likely to repeat** what has been coded many times before
- Might repeat **outdated** code or code from **different context**
- Might include **buggy** or **insecure** code
- Might **make up calls to non-existent code**
- **Useful in many cases, but not 100% trustworthy**

---

## Thinking / Reasoning Mode and Coding Agents

- **Thinking model** = the AI model spends **extra inference compute on intermediate reasoning** before answering.
  - This "appears" like the model is thinking / reasoning.
- **Coding agent** = an app/runtime around the model that gives it tools:
  - read files,
  - edit files,
  - run shell commands,
  - run tests,
  - inspect errors,
  - iterate.
- Coding agents can run in your IDE or as a standalone application.

---

## "Vibe Coding" â€” Are We There Yet?

- **Andrej Karpathy (Feb 2, 2025)**: *"There's a new kind of coding I call 'vibe coding', where you fully give in to the vibes, embrace exponentials, and forget that the code even exists. [â€¦]"*
- Vibe coder in interview study: *"Regulators don't care about your 'vibes'. Laws don't care about your feelings"* [1]
- Vibe coding might work for **low-risk, internal or hobby projects**. But it is **not yet feasible for commercial enterprise projects**.
- **Professional Software Developers Don't Vibe, They Control**.
  - *"I like coding alongside agents. Not vibe coding. But working **with**."* [2]

References:
- [1] V. Pimenova, S. Fakhoury, C. Bird, M.A. Storey, M. Endres. "Good vibrations? A qualitative study of co-creation, communication, flow, and trust in vibe coding". arXiv preprint (2025).
- [2] R. Huang, A. Reyna, S. Lerner, H. Xia, B. Hempel. "Professional Software Developers Don't Vibe, They Control: AI Agent Use for Coding in 2025". arXiv preprint (2025).

---

## Generative AI Works Like an "Amplifier", Not a "Fixer"

Sources: **2025 DORA report by Google** & E. Paradis, K. Grey, Q. Madison, D. Nam, A. Macvean, V. Meimand, N. Zhang, B. Ferrari-Church, S. Chandra. "How much does AI impact development speed? An enterprise-based randomized controlled trial." ICSE-SEIP '25.

### For high-performing teams
- With strong foundational practices (**automated testing, robust CI/CD pipelines, strong platform engineering**), **AI accelerates their success and throughput**.
- Clean architectures, good quality assurance processes **increase productivity**.
- **More experienced developers receive a larger speedup from AI**.

### For struggling teams
- AI **magnifies their dysfunction**. Generating code faster without the right guardrails just means generating **technical debt and bugs at an accelerated rate**.
- Technical debt and implicit documentation make it **harder for AI to modify the code**.

**Key idea**: AI doesn't replace skills; it amplifies whatever is already there.

---

# Common Challenges of Using Generative AI

## Challenge 1: AI Tools Result in Less Secure Code in Practice

- **AI-generated code is causing serious security vulnerabilities in 20% of companies** [1].
- Developers who had access to an AI assistant **wrote significantly less secure code** than those without access to an assistant **while being more likely to believe they wrote secure code** [2].
- Even when the LLM was **explicitly instructed to improve security**, it **still introduced new vulnerabilities**. **The more complex the code, the more vulnerabilities were added by AI** [3].
- **40% of code generated with GitHub Copilot contains security vulnerabilities** [4].

Why? Because **code in the training set is insecure**, and when corrected the LLM may still produce new (still incorrect) code.

References:
- [1] State of AI in Security & Development.
- [2] N. Perry et al. "Do Users Write More Insecure Code with AI Assistants" CHI 2023.
- [3] S. Shukla et al. "Security Degradation in Iterative AI Code Generation: A Systematic Analysis of the Paradox" ISTAS 2025.
- [4] H. Pearce, et al. "Asleep at the keyboard? Assessing the security of GitHub Copilot's Code Contributions." Com ACM 2025.

**Think-Pair-Share**: How do these results impact your approach to Software Engineering with GenAI? Discuss actionable insights!

---

## Challenge 2: Speed at the Cost of Quality (Cursor Study)

**Source**: H. He, C. Miller, S. Agarwal, C. KÃ¤stner, and B. Vasilescu. 2026. *Speed at the Cost of Quality: How Cursor AI Increases Short-Term Velocity and Long-Term Complexity in Open-Source Projects.* MSR '26.

### Study setup
- How does the use of AI affect open-source projects?
- **Study on 806 GitHub repos that used Cursor and 1380 similar non-Cursor repos**.

### Findings
- Adopting Cursor resulted in a **massive but temporary surge in development velocity** â€” **3xâ€“5x increase in lines of code added during the first one to two months**, but **disappeared shortly after**.
- **30.3% average increase in static analysis warnings** and a **41.6% increase in code complexity**.
- **Technical debt generated by rapid AI usage actively harms future productivity**.

The metaphor: "The Illusion of Speed (Development)" vs. "The Hidden Burden (Maintenance)" â€” velocity 1000x at first, but maintenance becomes im(possible) and technical debt becomes immense.

**Think-Pair-Share**: How do these results impact your approach to Software Engineering with GenAI? Discuss actionable insights!

---

## Challenge 3: The Illusion of AI Productivity

AI initially gives a solution that **looks "solid"** which makes us **feel very productive**.

| Days before OpenAI | Days after OpenAI |
| --- | --- |
| Developer coding: 2 hours | ChatGPT generates code: 5 minutes |
| Developer debugging: 6 hours | Developer debugging: 24 hours |

The "task progress" graph (by @forrestbrazeal): initial prompt â†’ tweaks â†’ "I'm so much more productive!" â†’ "I'm so productive!" â†’ "throw it all out and start over" â†’ "UGGGHH" â†’ "it's good enough" â€” meanwhile the without-AI baseline has steadily progressed past 100%.

---

## Challenge 4: How AI Impacts Skill Formation

**Source**: S. J. Hanwen, and A. Tamkin (Anthropic). *"How AI impacts skill formation."* arXiv (2026).

- **Methods**: Controlled experiment let **52 developers** code with a new library with/without AI and tests skill formation via questions and debugging tasks.
- **Results**: Participants who **relied on AI performed much worse** than participants who did not use AI when **debugging or explaining how to use the library**.

### Anti-Pattern: Cognitive Offloading
- Using AI for **"cognitive offloading"** (wholly or mostly relying on AI to solve the task) **minimizes learning effects**.

### Better Pattern: Conceptual Inquiry (Treating AI like a "TA")
- Ask only **conceptual questions** and rely on **improved understanding** to complete the task.
- Although this group encountered many errors, they also **independently resolved these errors**.
- On average, this mode was the **fastest among high-scoring patterns** and **second fastest overall** after the AI Delegation mode.

**Think-Pair-Share**: How do these results impact your approach to working with GenAI? Discuss actionable insights!

---

## Key Takeaway

> **In the hands of an expert, AI is a very powerful tool that creates efficiency.**
>
> **In the hands of a novice, AI creates the illusion of competence that creates more problems than it solves.**

---

# How to Use Generative AI The Right Way

## AI Pair Programming â€” The Most Patient Pair Programmer

Apply the **experience you've gained with Pair Programming** to AI.

### AI Pair Programming Mentality

**As the Driver (You write the code):**
- Ask AI to **critique** the code you wrote and ask for **suggestions** to improve it.

**As the Navigator (You let AI write the code):**
- Make sure you **understand all code**.
- **Critically review** AI-generated code.
- Ask AI to **explain** their code.

### Example review prompts to use
- "Scan this code for potential problems"
- "Evaluate this code and look for performance issues"
- "Explain the algorithm you used for this method"

---

## AI Is Your Knowledgeable Intern (Junior Developer)

> *AI is like a very knowledgeable intern who is not good at applying their knowledge to the right context. You should orchestrate their skills.*

### Iteratively Prompt AI with Concrete Steps
- **Define problem & high-level strategy**.
- **You pick the overall architecture, design patterns, how components interact, packages, and frameworks**.
- Before code generation, ask the AI to **explain its suggested approach** to critique & ask for specific refinements.

### Bad prompt (too vague)
- ~~"Implement a Monopoly board game"~~

### Better step-by-step prompts
1. *"Implement Monopoly with the player using the **state pattern** (InJail, Bankrupt, NormalTurn). **Before you write code, first show me a class diagram.**"*
2. *"Now implement & test this. Use **React & TypeScript**. Use **Playwright** for end-to-end tests and **Jest** for test doubles."*

---

## Example: Find the Mistakes Copilot Made

Prompt used: *"Implement monopoly with the player using the state pattern (InJail, Bankrupt, NormalTurn). Before you write code, first show me a class diagram."*

Copilot's response: *"Of course, here is your code. It's terrific, isn't it ??????"*

### Code Copilot Generated (with mistakes)
```python
class Player:
    def roll_dice(self) -> int:
        die1 = random.randint(1, 6)
        die2 = random.randint(1, 6)
        return die1 + die2

class InJailState:
    def take_turn(self, player: "Player") -> None:
        roll = player.roll_dice()

        # Simulate checking for doubles
        # (simplified: 50% chance)
        import random
        doubles = random.random() < 0.5
        if doubles:
            print(f"{player.name} rolled doubles
                  and is released from jail!")
```

### Mistakes
- `import random` **should be in the import section** (not inline inside the method).
- `doubles = random.random() < 0.5` â€” **Logical Error** (just simulates 50% chance instead of checking real doubles).
- `def roll_dice(self) -> int` **should return a pair to check if a double was rolled** (returning the sum loses information).

### After the AI Pair Programmer Fixed It
Prompt feedback: *"def roll_dice(self) -> int should return a pair to check if a double was rolled"*

Copilot: *"Oops, my bad. Here is the correct code"*

```python
class Player:
    def roll_dice(self) -> tuple[int, int]:
        die1 = random.randint(1, 6)
        die2 = random.randint(1, 6)
        return (die1, die2)

class InJailState:
    def take_turn(self, player: "Player") -> None:
        die1, die2 = player.roll_dice()
        roll = die1, die2

        print(f"{player.name} rolled a {die1} and
              {die2} (total: {roll})")
        if die1 == die2:
            print(f"{player.name} rolled doubles
                  and is released from jail!")
```

---

## Adopt These Rules for Best Results

- **Mandatory Code Review**: Adopt a strict rule: **"Always Review AI-Generated Code."** Every block of code an AI produces should be **scrutinized as if an unreliable teammate wrote it**.
- **The Explainability Rule**: **Never commit AI-generated code that you could not comfortably explain to a teammate**.

> *"Assume everything I do is subtly incorrect, buggy, and insecure"* â€” the AI
>
> *"I'll work around this, because I've learned reliable software engineering techniques"* â€” you

Source: Stephane H Maes. "The Gotchas of AI Coding and Vibe Coding. It's All About Support And Maintenance" April 2025.

---

## Lesson Learned: Always Double-Check AI Code

- A lot of the code in the training set has many **code smells**.
- **LLMs reproduce these code smells**.
- The LLM does **not understand** how the world works. It looks for **linguistic similarities and correlations** between the prompt and data in its training set.
- **This is why verifying LLM-generated code is soooo important!**

---

## How Can We Effectively Use Unreliable AI? â€” Test-Driven Development with GenAI

Make sure you can **verify the output** of genAI and that it **didn't break existing code**.

### TDD with GenAI Procedure
- **Write a test** (or more) just as if you were to implement the next feature yourself.
  - AI may help you with **test setup and infrastructure code**.
- **Prompt the AI to make the test pass**.
- **Refactor AI-written code** (important).

### Example prompt
> *"I am using TDD. Implement a **general** solution that passes test X. Do **not change the test**. Context: User Story Y"*

Reference: https://github.blog/ai-and-ml/github-copilot/github-for-beginners-test-driven-development-tdd-with-github-copilot/

---

## TDD with LLM-based Code Generation (Research)

**Source**: N. S. Mathews and M. Nagappan. *Test-Driven Development and LLM-based Code Generation.* ASE '24.

- Provide the LLM with **(a) a problem statement, and (b) a test suite**.
- Merely **providing the LLM with tests during generation improves correctness**.
- Adding a **remediation loop** that provided the LLM with the **stack trace of failed tests** increases correctness even more.

---

## Test-Driven Generation

Reference: https://dzone.com/articles/test-driven-generation

Five steps:

1. **Prompt the AI to generate tests** for a given specification.
2. **Carefully review the tests** to check if they are an adequate specification of the problem.
3. **Prompt AI to generate the implementation** for the generated tests.
4. **Prompt the AI to refactor the code**.
5. **Review the generated implementation to check for overfitting**.

---

## YOU Are the Supervisor, AI Is Your Junior Developer

> AI is like a very knowledgeable intern who is not good at applying their knowledge to the right context. You should **orchestrate their skills**.

Dialogue:
- AI: *"Sorry, I haven't taken CS130. So, I don't know how to design & test high-quality software. But I can write code very fast!"*
- You: *"No problem. I've taken CS130. I can guide you through the process."*

### Supervisor Mentality

- **You are fully responsible for all code** (including bugs & security vulnerabilities).
- It is your job to **read & verify** the correctness of AI-generated code.
- Always **test** all code written by AI.
- Always **refactor** low-quality code.
- Optimize important **quality attributes**.

---

## Which Coding Tasks Are Well-suited for AI?

**Source**: R. Huang, A. Reyna, S. Lerner, H. Xia, B. Hempel. *"Professional Software Developers Don't Vibe, They Control: AI Agent Use for Coding in 2025."* arXiv preprint (2025).

### Where AI shines
- Agents **accelerate straightforward, repetitive, scaffolding tasks**.
- Suitable for **following well-defined plans**.
- Suitable for **general SWE tasks**: writing tests, refactoring, documentation, simple debugging.
- Suitable for **prototyping and experimenting**.

### Where it stumbles
- Unsuitable for **business logic** and tasks requiring **domain knowledge**.
- Unsuitable for **complex, stateful tasks**.
- **Performance-critical** deployment, **high-stakes**, and **security-critical** situations.
- Anything requiring **real systems thinking** [and new architectures etc.].

### Ideal use case
> **Tasks that are tiresome to solve but easy to verify, and when the cost of an agent being wrong is low.**

---

# Prompt Engineering

## Prompt Engineering Patterns for Complex Prompts (Larger or Difficult Coding Tasks)

Four standard sections of a complex prompt:

### Introduction (Role)
Give the AI an imaginary 'role' to think of themselves in.
- E.g., *"**Role:** Act as a diligent **expert** software engineer. You have **expertise** in Python and design patterns, clean code, and testing."*
- Results in better responses, because the AI correlates words in the prompt with words mentioned in the training set. This way, it **"activates" specific knowledge and skills**.

### Concrete Task
Tell the AI **what** and **where** it should implement code.
- E.g., *"**Task:** Implement the `<feature>` in `<file>`"*

### Contextual Information
Provide the AI with enough information about what system this feature is used in, or what are important requirements for the feature.
- E.g., *"**Context:** This feature is part of this **user story** `<user story>`. We value **security & privacy** over **performance**."*

### Instructions
Explicitly list all steps.
- E.g., *"**Instructions:** (1) plan a clean design, then (2) generate the code to write the program, and then (3) write unit tests & end-to-end tests."*

---

## Prompt Engineering Tips

### Keep your AI prompt task Simple
- The **larger the amount of code** that genAI should produce in one go, the **more likely it is that the result is incorrect or low-quality**.
- AI has **limited context size** & **limited reasoning capabilities**.
- **Break up a task into smaller & simpler tasks** if possible:
  - *"Please only describe the design & your reasoning, don't implement code yet"*
  - *"Now please implement only `<feature 1>`, nothing else yet"*

### Be Specific
- Tell the AI all important **success criteria**.
- Fortunately, you've learned how to write good **Acceptance Criteria** already.

### Provide the exact context needed to solve the task (nothing more, nothing less)
- Addresses **context degradation**.

---

## Design Decision Prompts

- Besides coding, AI can help you make design decisions. You can ask it to **describe the pros and cons of different design alternatives**.
- **Tip**: Ask **general questions** about consequences of design rather than whether the decision would be ideal for your concrete scenario.

### Don't
> *"Should I use the State design pattern **for this class**?"*

- LLMs currently **do not understand design patterns** and **do not understand your context**.

### Better
> *"What are **common consequences** for using the State design pattern?"*

- LLMs are great at reproducing **general pros and cons**.
- Then **it's your job to make the decision**.

---

## Question-Prompts

- You can tell the AI to **ask you relevant questions** that then help the AI find the right solutions.
- E.g., *"[â€¦] Before you start your task, **ask me any questions** you have about important decisions, or requirements"*
- E.g., *"[â€¦] Before you start your task, **ask me ordered Yes / No questions** you needed to solve the task"*

---

## "Replace the TODO" Prompt Pattern

- **Key idea**: give the AI a **code skeleton** that allows **you to keep control over the design**.
- **Add `todo` notes with instructions** right in the code for the AI to replace.

### Example
```python
class Player:
    def roll_dice(self) -> tuple[int, int]:
        die1 = #TODO fill in random die roll
        die2 = #TODO fill in random die roll
        return (die1, die2)

class InJailState:
    def take_turn(self, player: "Player") -> None:
        #TODO fill in the in-jail behavior
```

### Prompt to AI
> *"Resolve all remaining todos by filling in the right code. **Ask me before making changes to other code**."*

---

# Essential Skills for Coding with AI

- **Delegation and agent orchestration**
  - Break down tasks, precisely articulate criteria, constraints, and context.
- **Verification and quality assurance** (Testing)
  - Review, test, and verify AI-generated code.
- **Product understanding, requirements engineering, and specification** (User Stories)
  - Apply systems thinking (looking at the product as a whole).
- **Architecture and systems design** (Up nextâ€¦)
  - Overall system architecture, design patterns, how different components interact.

> *"The future belongs to developers who can **model systems**, **anticipate edge cases**, and **translate ambiguity into structure** â€” skills that AI can't automate"*

Source: https://ashtom.github.io/developers-reinvented

---

## Exit Tickets (Bruin Learn)

**Question 1 (1 pt)**: In your own words, please summarize **three key insights** you learned about Software Engineering with Generative AI today. (Do not copy phrases from the lecture slides.)

**Question 2 (1 pt)**: For a particular coding task that you are currently working on for your course project: (1) Write an AI prompt that follows the best practices of this lecture that helps you solve this task. (2) Explain which best practices this prompt follows.

**Question 3**: Please leave any questions that you have about today's material and things that are still unclear or confusing to you (if none, simply write N/A).

---

## Recommended Resources

- **"Developers, Reinvented"** by Thomas Dohmke â€” https://ashtom.github.io/developers-reinvented
- **"GitHub CEO predicts the future of programming...(Full Interview)"** â€” https://www.youtube.com/watch?v=3WNukz5-Ch0
- **"Good Vibrations? A Qualitative Study of Co-Creation, Communication, Flow, and Trust in Vibe Coding"** by Veronica Pimenova et al. â€” https://arxiv.org/abs/2509.12491

---

*Credits: These slides use images generated with Gemini.*


---

# Appendix: Lecture 14 Slides (raw extracted text)

The following is the full extracted text of Lecture 14. Preserved verbatim so no information is lost.

```text
                              "I suspect that machines to be programmed in our native
                              tongues--be it Dutch, English, American, French,
                              German, or Swahili--are as damned difficult to make as
                              they would be to use."  --Edsger W. Dijkstra (1979)
CS 35L Software
Construction
Lecture 14 ­
Coding with GenAI
Assistant Teaching Professor
Computer Science Department
Learning Objectives of this Lecture
After this lecture you should be able to:
· Explain how AI coding agents work
· Identify challenges & risk that result from
  the use of AI coding agents
· Apply software engineering techniques to
  mitigate the risks of AI-generated code
· Apply prompt engineering techniques to
  maximize the usefulness of LLMs
                            CS 35L Software Construction: Lecture 14 ­ Coding with GenAI
                            Tobias Dürschmid
What do these Scenarios have in Common?
What is Different now? Talk to your Neighbor(s)
1960              Productivity Gains                                     2026
       10X        A single developer can write  ­111..2.551XXX[[12[1]]]
                  the code for much larger
                  software systems
      Sound Abstractions     Unsound Abstractions
      Results of a compiler  Non-determinism and
      are predictable since  black-box architecture
      the abstractions are   of LLMs make the result
      procedural             unpredictable
                    Remaining Challenges
                  Requirements, design &
                  quality assurance still
                  requires human intelligence
                  due to essential complexity
                                       Former CEO of GitHub
                                                             Full Tweet: https://x.com/ashtom/status/1952409910236291109
"The evidence is clear: Either you embrace AI, or get out of this career.
Our latest field study with 22 developers who are integrating AI deeply into their
workflows reveals a striking trend: those who persist beyond early skepticism emerge
with dramatically higher ambition, technical fluency, and job satisfaction.
They're not writing less code ­ they're enabling more complex, system-level work
through orchestration. [...] Here's what we're seeing:
· AI is on track to write 90% of code within the next 2­5 years.
· Developers aren't worried. They're optimistic and realistic about the changes ahead.
· New skills matter now, e.g. agent orchestration, iterative collaboration, and critical
verification.
· Time savings? Sure. But the real shift is ambition. Developers are raising the
ceiling, not just lowering the cost."  See https://ashtom.github.io/developers-reinvented
Watch this interview https://www.youtube.com/watch?v=3WNukz5-Ch0
LLMs (Large Language Models)                                                       def fib_iter(n):
Work like "Statistical Parrots"                                                       a, b = 0, 1
                                                                                      for _ in range(n):
  1. Pre-Training ­ Creating a base mode / foundation model                               a, b = b, a + b
                                                                                      return a
 The is model is trained on vast amounts of code to accurately predict the
 most likely next token given all previous tokens.
 Having memorized a large amount of code and abstracted key patterns in the
 code, the LLM can reproduce variations of the most common coding tasks.
 2. Post-Training ­ Optimizing the model for a particular use case                 code the most
                                                                                   efficient and
· Fine-tuning the model on labeled input-output pairs                              shortest Python
     (e.g., instructions + code solutions, such as LeetCode problems)              implementation
                                                                                   of Fibonacci
· Using human feedback to optimize the model using reinforcement learning
     (e.g., developers rank code outputs on correctness, efficiency, readability)
 3. Inference ­ Prompting the model for a task
Based on a natural language prompt, the LLM tries to produce the most likely
sequence of answer tokens (usually in a non-deterministic way)
How Much Would You Trust A                                                                Useful
Parrot Witness?                                                                           in some
                                                                                          cases, but
                                                    Is likely to repeat                   not 100%
                                                    what has been said                    trustworthy
                                                    many times before
                                                                                                                  6
                                                    Might repeat phrases
                                                    out of context or
                                                    from a long time ago
                                                    Might quote factually-
                                                    incorrect statements
                                                    Might make up
                                                    statements
                            CS 35L Software Construction: Lecture 14 ­ Coding with GenAI
                            Tobias Dürschmid
This is How Much You Should Trust a                                                       Useful
Parrot Coder (aka "AI Coding Agent")                                                      in many
                                                                                          cases, but
                              Is likely to repeat                                         not 100%
                              what has been coded                                         trustworthy
                              many times before
                                                                                                              7
                              Might repeat outdated
                              code or code from
                              different context
                              Might include buggy
                              or insecure code
                              Might make up calls
                              to non-existent code
                            CS 35L Software Construction: Lecture 14 ­ Coding with GenAI
                            Tobias Dürschmid
Thinking / Reasoning Mode Takes More Time
To answer more Intelligently
· Thinking model = the AI model spends extra inference compute on intermediate
  reasoning before answering
· This "appears" like the model is thinking / reasoning
· Coding agent = an app/runtime around the model that gives it tools:
· read files,                   Can Run in your IDE
· edit files,                   or as a standalone
· run shell commands,           application
· run tests,
· inspect errors,
· iterate.
              CS 35L Software Construction: Lecture 14 ­ Coding with GenAI      8
              Tobias Dürschmid
"Vibe Coding" ­ Are we there yet?
· Andrej Karpathy: "There's a new kind of coding I call "vibe coding", where you fully give in to
  the vibes, embrace exponentials, and forget that the code even exists. [...]" Feb 2, 2025
· Vibe coder in interview study: "Regulators don't care about your "vibes". Laws don't care
  about your feelings" [1]
· Vibe coding might work for low-risk, internal or hobby projects. But it is not yet feasible for
  commercial enterprise projects
    · Professional Software Developers Don't Vibe, They Control
        · "I like coding alongside agents. Not vibe coding. But working **with**." [2]
[1] V. Pimenova, S. Fakhoury, C. Bird, C., M.A. Storey, M. Endres. "Good vibrations? A qualitative study of co-creation, communication, flow, and trust in vibe coding". arXiv preprint (2025)
[2] R. Huang, A. Reyna, S. Lerner, H. Xia, B. Hempel. "Professional Software Developers Don't Vibe, They Control: AI Agent Use for Coding in 2025". arXiv preprint (2025)
Generative AI Works Like an "Amplifier"
Not a "Fixer"
· For high-performing teams with strong foundational practices (automated
  testing, robust CI/CD pipelines, strong platform engineering), AI accelerates
  their success and throughput
    · Clean architectures, good quality assurance processes increase productivity.
    · More experienced developers receive a larger speedup from AI
· For struggling teams, AI magnifies their dysfunction. Generating code
  faster without the right guardrails just means generating technical debt and
  bugs at an accelerated rate
    · Technical debt and implicit documentation makes it harder for AI to modify the code
Sources: 2025 DORA report by Google & E. Paradis, K. Grey, Q. Madison, D. Nam, A. Macvean, V. Meimand, N. Zhang, B.Ferrari-Church, S. Chandra. "How
much does AI impact development speed? An enterprise-based randomized controlled trial." ICSE-SEIP `25.
What Common
Challenges Arise
from the Use of
Generative AI?
Use of AI Tools Resulted In Code in the training
Less Secure Code in Practice set is insecure
· AI-generated code is causing serious security
  vulnerabilities in 20% of companies [1]
· Developers who had access to an AI assistant                                                                                I'm sorry for my mistake.
  wrote significantly less secure code than those                                                                              Here is some new (still
  without access to an assistant while being more                                                                              incorrect) code for you
  likely to believe they wrote secure code [2]
· Even when the LLM was explicitly instructed to
  improve security, it still introduced new
  vulnerabilities. The more complex the code, the
  more were vulnerabilities added by AI [3]
· 40% of code generated with GitHub Copilot
contains security vulnerabilities [4]
[1] State of AI in Security & Development.                                                                                    Think-Pair-Share: How do these
[2] N. Perry et al. "Do Users Write More Insecure Code with AI Assistants" CHI 2023
[3] S. Shukla et al. "Security Degradation in Iterative AI Code Generation: A Systematic Analysis of the Paradox" ISTAS 2025  results impact your approach to
[4] H. Pearce, et al. "Asleep at the keyboard? Assessing the security of Github Copilot's Code Contributions." Com ACM 2025
                                                                                                                              Discuss actionable insights!
Speed at the
Cost of Quality
· How does the use of AI affect
  open-source projects?
· Study on 806 GitHub repos that
  used Cursor and 1380 similar non-
  Cursor repos
· Adopting Cursor resulted in a
  massive but temporary surge in
  development velocity (3x-5x
  increase in lines of code added
  during the first one to two months
  but disappeared shortly after)
Source: H. He, C. Miller, S. Agarwal, C. Kästner, and B. Vasilescu. 2026. Speed at the Cost of Quality: How Cursor AI Increases Short-Term Velocity and Long-
Term Complexity in Open-Source Projects. MSR '26
Speed at the
Cost of Quality
· How does the use of AI affect
  open-source projects?
· Study on 806 GitHub repos that        Think-Pair-Share: How do these
  used Cursor and 1380 similar non-       results impact your approach to
  Cursor repos                          Software Engineering with GenAI?
                                          Discuss actionable insights!
· 30.3% average increase in static
  analysis warnings and a 41.6%
  increase in code complexity
· technical debt generated by rapid AI
usage actively harms future
productivity
Source: H. He, C. Miller, S. Agarwal, C. Kästner, and B. Vasilescu. 2026. Speed at the Cost of Quality: How Cursor AI Increases Short-Term Velocity and Long-
Term Complexity in Open-Source Projects. MSR '26
Be Aware of the Illusion
of AI Productivity
· AI initially gives a solution that looks
  "solid" which makes us feel very
  productive
Days before OpenAI   Days after OpenAI
Developer coding     ChatGPT generates
2 hours              code
                     5 minutes
Developer debugging  Developer debugging
6 hours              24 hours
                     CS 35L Software Construction: Lecture 14 ­ Coding with GenAI  15
                     Tobias Dürschmid
How AI Impacts Skill Formation                                                                    Think-Pair-Share: How do these
                                                                                                   results impact your approach to
                                                                                                          working with GenAI?
                                                                                                   Discuss actionable insights!
· Methods: Controlled experiment let 52 developers code with a new library
  with/without AI and tests skill formation via questions and debugging tasks
· Results: Participants who relied on AI performed much worse than participants
  who did not use AI when debugging or explaining how to use the library
   · Anti-Pattern: Using AI for "cognitive offloading" (wholly or mostly relying on
     AI to solve the task) minimizes learning effects
   · Better: Conceptual Inquiry (treating AI like a "TA") ­ ask only conceptual
     questions and reply on improved understanding to complete the task. Although
     this group encountered many errors, they also independently resolved these
     errors. On average, this mode was the fastest among high-scoring patterns and
     second fastest overall after the AI Delegation mode
Source: S. J. Hanwen, and A. Tamkin (Anthropic). "How AI impacts skill formation." arXiv (2026).
Key Takeaway
      In the hands of an expert, AI is a very powerful tool
                          that creates efficiency
In the hands of a novice, AI creates the illusion of
competence that creates more problems than it solves
How to Use
Generative AI
The Right Way?
Generative AI is The Most                      Scan this code for
Patient Pair Programmer                        potential problems
Now apply the experience you've gain with                 Evaluate this code and look
Pair Programming to AI!                                   for performance issues
   AI Pair Programming Mentality                 Explain the algorithm
                                               you used for this method
As the Driver (You write the code):
· Ask AI to critique the code you wrote
   and ask for suggestions to improve it
As the Navigator (You let AI write the code):
· Make sure you understand all code
· Critically review AI-generated code
· Ask AI to explain their code
Generative AI is Your                                  (1) Implement Monopoly with the player
Knowledgeable Intern                                  using the state pattern (InJail, Bankrupt,
                                                     NormalTurn). Before you write code, first
AI is like a very knowledgeable intern who is
not good at applying their knowledge to the                    show me a class diagram.
right context. You should orchestrate their skills.
Iteratively Prompt AI with Concrete Steps            (2) Now implement & test this. Use React &
 · Define problem & high-level strategy              TypeScript. Use Playwright for end-to-end
 · You pick the overall architecture, design
                                                             tests and Jest for test doubles.
     patterns, how components interact,
     packages, and frameworks                        Implement a Monopoly board game
 · Before code generation, ask the AI to
     explain its suggested approach to
     critique & ask for specific refinements
Find the Mistakes                                    Implement monopoly with the player using
Copilot Made                                              the state pattern (InJail, Bankrupt,
 class Player:                                       NormalTurn). Before you write code, first
                                                               show me a class diagram.
 def roll_dice(self) -> int:
     die1 = random.randint(1, 6)                                                           Of course, here
     die2 = random.randint(1, 6)                                                            is your code.
     return die1 + die2                                                                    It's terrific, isn't
                                                                                               it ??????
 class InJailState:
 def take_turn(self, player: "Player") -> None:
     roll = player.roll_dice()
# Simulate checking for doubles
# (simplified:  50%  chance)  Should be in import section
import random
doubles = random.random()     <  0.5  Logical Error
if doubles:
print(f"{player.name} rolled doubles                 def roll_dice(self) -> int should
and is released from jail!")                         return a pair to check if a double was rolled
The AI Pair Programmer                                       Implement monopoly with the player using
Successfully Fixed it!                                            the state pattern (InJail, Bankrupt,
 class Player:                                               NormalTurn). Before you write code, first
                                                                       show me a class diagram.
 def roll_dice(self) -> tuple[int, int]:
     die1 = random.randint(1, 6)                                                                 Oops, my bad.
     die2 = random.randint(1, 6)                                                                    Here is the
     return (die1, die2)                                                                           correct code
 class InJailState:
 def take_turn(self, player: "Player") -> None:
     die1, die2 = player.roll_dice()
     roll = die1, die2
print(f"{player.name} rolled a {die1} and
                                   {die2} (total: {roll})")
if die1 == die2:                                             def roll_dice(self) -> int should
   print(f"{player.name} rolled doubles                      return a pair to check if a double was rolled
                   and is released from jail!")
Adopt these Rules for                                                               Assume everything I do is subtly
Best Results                                                                         incorrect, buggy, and insecure
· Mandatory Code Review: Adopt a strict
rule: "Always Review AI-Generated Code"
Every block of code an AI produces should
be scrutinized as if an unreliable teammate
wrote it
· The Explainability Rule: Never commit AI-
generated code that you could not
comfortably explain to a teammate                                                    I'll work around this, because
                                                                                    I've learned reliable software
Source: Stephane H Maes. "The Gotchas of AI Coding and Vibe Coding. It's All About
Support And Maintenance" April 2025                                                    engineering techniques
          CS 35L Software Construction: Lecture 14 ­ Coding with GenAI                                               23
          Tobias Dürschmid
Lesson Learned: Always                             Assume everything I do is subtly
double-check AI Code                                incorrect, buggy, and insecure
· A lot of the code in the training set has many
  code smells.
   · LLMs reproduce these code smells.
· The LLM does not understand how the world
  works. It looks for linguistic similarities and
  correlations between the prompt and data in
  its training set
· This is why verifying LLM-generated code          I'll work around this, because
  is soooo important!                              I've learned reliable software
                                                      engineering techniques
How can we Effectively                                        Assume everything I do is subtly
Use Unreliable AI???                                           incorrect, buggy, and insecure
Make sure you can verify the output of genAI
and that it didn't break existing code.
Test-Driven Development with GenAI
· Write a test (or more) just as if you were
to implement the next feature yourself
· AI may help you with test setup and
infrastructure code
· Prompt the AI to make the test pass
  · Refactor AI-written code (important)                      I am using TDD. Implement a
                                                              general solution that passes
See https://github.blog/ai-and-ml/github-copilot/github-for-  test X. Do not change the test.
beginners-test-driven-development-tdd-with-github-copilot/
                                                                   Context: User Story Y
TDD with LLM-based                                               Assume everything I do is subtly
Code Generation                                                   incorrect, buggy, and insecure
· Provide the LLM with (a) a problem
  statement, and (b) a test suite
· Merely providing the LLM with tests
  during generation improves correctness
· Adding a remediation loop that provided
  the LLM with the stack trace of failed tests
  increases correctness even more
See: N. S. Mathews and M. Nagappan. Test-Driven Development and
LLM-based Code Generation. ASE '24
Test-Driven Generation                                  Assume everything I do is subtly
                                                         incorrect, buggy, and insecure
· 1. Prompt the AI to generate tests for a given
  specification
· 2. Carefully review the tests to check if they are
  an adequate specification of the problem
· 3. Prompt AI to generate the implementation for
  the generated tests
· 4. Prompt the AI to refactor the code
· 5. Review the generated implementation to check
  for overfitting
See: https://dzone.com/articles/test-driven-generation
YOU are the supervisor                               Sorry, I haven't taken CS130. So, I don't
AI is your junior developer                          know how to design & test high-quality
                                                      software. But I can write code very fast!
AI is like a very knowledgeable intern who is           No problem. I've taken CS130.
not good at applying their knowledge to the          I can guide you through the process.
right context. You should orchestrate their skills.
     Supervisor Mentality
  You are fully responsible for all code
  (including bugs & security vulnerabilities)
  · It is your job to read & verify the
      correctness of AI-generated code
  · Always test all code written by AI
  · Always refactor low-quality code
  · Optimize important quality attributes
Which Coding Tasks are Well-suited for AI?
Where AI shines:                              Where it stumbles:
· Agents Accelerate Straightforward,          · Unsuitable for Business Logic and Tasks
  Repetitive, Scaffolding Tasks                 Requiring Domain Knowledge
· Suitable for Following Well-Defined Plans   · Unsuitable for Complex, Stateful Tasks
· Suitable for General SWE Tasks: Writing     · Performance-Critical Deployment, High-
  Tests, Refactoring, Documentation, Simple     Stakes, and Security-Critical Situations
  Debugging
                                              · Anything requiring real systems thinking
· Suitable for Prototyping and Experimenting    [and new architectures etc].
Ideal use case: Tasks that are tiresome to solve but easy to verify
          and when the cost of an agent being wrong is low.
Source: R. Huang, A. Reyna, S. Lerner, H. Xia, B. Hempel. "Professional Software Developers Don't Vibe, They Control: AI Agent Use for Coding in 2025". arXiv preprint (2025)
Prompt Engineering Patterns for Complex
Prompts (Larger or Difficult Coding Tasks)
· Introduction: Give the AI an imaginary `role' to think of themselves in.
  E.g., "Role: Act as a diligent expert software engineer. You have expertise in Python and
  design patterns, clean code, and testing."
    · Results in better responses, because the AI correlates words in the prompt with words mentioned in
       the training set. This way, it "activates" specific knowledge and skills
· Concrete Task: Tell the AI what and where it should implement code.
  E.g., "Task: Implement the <feature> in <file>"
· Contextual Information: Provide the AI with enough information about what system this
  feature is used in, or what are important requirements for the feature.
  E.g., "Context: This feature is part of this user story <user story>. We value security &
  privacy over performance."
· Instructions: Explicitly list all steps. E.g., "Instructions: (1) plan a clean design, then (2)
  generate the code to write the program, and then (3) write unit tests & end-to-end tests."
Prompt Engineering Tips
· Keep your AI prompt task Simple
    · The larger the amount of code that genAI should produce in one go, the more likely it is that the result
       is incorrect or low-quality
         · AI has limited context size & limited reasoning capabilities
    · Break up a task into smaller & simpler tasks if possible
         · "Please only describe the design & your reasoning,
            don't implement code yet"
         · "Now please implement only <feature 1>, nothing else yet"
· Be Specific
    · Tell the AI all important success criteria
    · Fortunately, you've learned how to write good Acceptance Criteria already
· Provide the exact context needed to solve the task (nothing more, nothing less)
    · Addresses context degradation
Design Decision Prompts
· Besides Coding, AI can help you make design decisions. You can ask it to
  describe the pros and cons of different design alternatives
· Tip: Ask general question about consequences of design rather than whether the
  decision would be ideal for your concrete scenario:
· Don't: "Should I use the State design pattern for this class?"
    · LLMs currently do not understand design patterns and do not understand your context
· Better: "What are common consequences for using the State design
  pattern?"
    · LLMs are great at reproducing general pros and cons.
      Then it's your job to make the decision
Question-Prompts
· You can tell the AI to ask you relevant questions that then help the AI find the
  right solutions
   · E.g., "[...] Before you start your task, ask me any questions you have about
     important decisions, or requirements"
   · E.g., "[...] Before you start your task, ask me ordered Yes / No questions you
     needed to solve the task"
"Replace the TODO"
Prompt Pattern
· Key idea: give the AI a code skeleton that
  allows you to keep the control over the design
· Add todo notes with instructions right in the
  code for the AI to replace
class Player:                                     Resolve all remaining todos by filling
                                                      in the right code. Ask me before
def roll_dice(self) -> tuple[int, int]:                 making changes to other code
   die1 = #TODO fill in random die roll
   die2 = #TODO fill in random die roll
   return (die1, die2)
class InJailState:
def take_turn(self, player: "Player") -> None:
   #TODO fill in the in-jail behavior
Essential Skills for Coding with AI
· Delegation and agent orchestration
   · Break down tasks, precisely articulate criteria, constraints, and context
· Verification and quality assurance                    Testing
   · review, test, and verify AI-generated code
· Product understanding, requirements engineering, and specification
   · apply systems thinking (looking at the product as a whole)  User Stories
· Architecture and systems design Up next...
· overall system architecture, design patterns, how different components interact
"The future belongs to developers who can model systems, anticipate edge
cases, and translate ambiguity into structure--skills that AI can't automate"
Source: https://ashtom.github.io/developers-reinvented
Please fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three key insights you learned about Software
Engineering with Generative AI today. (Do not copy phrases from the lecture slides)
                       For a particular coding task that you are currently working on for your course project,
                       (1) Write an AI prompt that follows the best practices of this lecture that helps you solve
                       this task. (2) Explain which best practices this prompt follows.
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slides use images generated with Gemini
Recommended Resources
· "Developers, Reinvented" by Thomas Dohmke
   · https://ashtom.github.io/developers-reinvented
· "GitHub CEO predicts the future of programming...(Full Interview)"
   · https://www.youtube.com/watch?v=3WNukz5-Ch0
· "Good Vibrations? A Qualitative Study of Co-Creation, Communication,
  Flow, and Trust in Vibe Coding" by Veronica Pimenova et al.
   · https://arxiv.org/abs/2509.12491

```
