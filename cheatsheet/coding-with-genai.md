# Coding with Generative AI Cheatsheet (candidate items)

## Learning objectives (L14)
- Explain how AI coding agents work
- Identify challenges & risks of AI-generated code
- Apply SE techniques to mitigate AI risks
- Apply prompt engineering techniques

## How LLMs work (simplified)
1. **Pre-training** — train on huge corpus of text/code; learns patterns
2. **Post-training** (RLHF / fine-tuning) — shape for instruction-following
3. **Inference** — given prompt, predict tokens probabilistically

## Core warning
> AI is an **amplifier, not a fixer**. It amplifies whatever skills you have.
- Good dev + AI = much faster
- Weak dev + AI = faster bugs, faster security holes

## "Vibe coding" critique
- Accepting AI output without understanding it
- You become unable to debug, extend, or own the code
- Long-term: skill atrophy (cognitive offloading)

## Security risks of AI code
- Cursor security study: notable rate of vulnerable code
- Copilot study: ~40% of AI-generated code has security issues
- Common: hardcoded secrets, SQL injection, weak crypto, missing input validation

## Mistakes AI commonly makes
- Hallucinates function/API names
- Outdated APIs (training cutoff)
- Plausible-looking but wrong logic
- Security antipatterns
- Inefficient algorithms
- Misses edge cases

## "AI as junior intern" analogy
Treat AI output like a junior dev's PR:
- Always review
- Test thoroughly
- Question design choices
- Don't merge without understanding

## Skill formation concern
- **Cognitive offloading**: AI does the work, you stop building the skill
- Implication: deliberately practice fundamentals; use AI for boilerplate, not foundations

## TDD with GenAI
- Write tests first (you, not AI)
- Have AI generate implementation
- Run tests → if green, accept; if red, iterate
- Tests serve as spec + guard

## Test-driven generation
- Give AI a failing test
- Ask it to write the function
- Verify the test passes
- Then ask for more tests/edge cases

## Prompt engineering patterns
| Component | Purpose |
|---|---|
| **Role** | "You are a senior Python developer..." |
| **Task** | What to do |
| **Context** | Relevant background, constraints |
| **Instructions** | Format, style, constraints |

## "Replace the TODO" pattern
```python
def calculate_tax(income):
    # TODO: implement based on US 2024 brackets
    pass
```
- Specific, scoped, easy for AI to fill
- Better than "write a tax calculator"

## Design decision prompts
- "Here's my context. Here's my problem. List 3 design alternatives with trade-offs."
- Use AI for ideation, not for the final pick

## Essential skills in AI era
- **Delegation** — what to assign to AI, what to do yourself
- **Verification** — testing, reviewing AI output
- **Requirements engineering** — clear specs → better AI output

## Workflow: when AI HELPS most
- Boilerplate (CRUD endpoints, config files)
- Translating between languages
- Writing tests
- Explaining unfamiliar code
- Generating documentation

## When AI HURTS most
- Critical security code
- Performance-sensitive algorithms (without verification)
- Domain-specific business logic
- Code requiring deep architectural understanding

## Red flags in AI output
- Functions that "look right" but use wrong API
- Comments that disagree with the code
- Hardcoded values where parameters should be
- Catch-all exception handlers (`except Exception: pass`)
- Missing input validation
- Missing error handling

## Tips for using AI safely
1. **Read every line** before accepting
2. **Run tests** before committing
3. **Ask AI to explain** its choice; verify the explanation
4. **Don't accept** unfamiliar APIs without checking docs
5. **Iterate** with feedback ("this doesn't handle empty input...")
6. **Use linting + security scanners** on AI output

## Quote (paraphrased lecture)
> "The dev who reads code carefully now becomes 10× more valuable. The dev who just hits Tab to accept gets replaced."

## Additional items (potentially missing)

### Key terminology
- **LLM** — Large Language Model
- **Tokens** — units of text the model processes (~4 chars each)
- **Context window** — how much text the model can see at once (e.g., 100K-1M tokens)
- **Hallucination** — model confidently produces false info
- **Prompt** — input you give the model
- **Completion** — model's output
- **Temperature** — randomness setting (0 = deterministic, 1+ = creative)

### Model phases
1. **Pre-training** — learn language patterns from huge corpus
2. **Post-training / RLHF** — Reinforcement Learning from Human Feedback to align with helpfulness
3. **Inference** — runtime: predict next token given context

### Hallucination types
- **Function/API hallucination** — invents a function that doesn't exist
- **Outdated info** — based on training cutoff date
- **Confident wrong logic** — code that compiles but doesn't work
- **False citations** — invented paper titles, URLs

### Why LLMs make mistakes
- Predict plausible-looking text, not verified facts
- Pattern-match rather than reason
- Training data may have wrong/outdated examples
- No real-world grounding (can't run/test what they suggest)

### Best prompt patterns
- **Role + Task + Context + Instructions** — most reliable structure
- Provide examples (few-shot prompting)
- Be specific about format ("output JSON with fields x, y")
- State constraints upfront ("Python 3.10+", "no external libs")
- Ask for reasoning (Chain-of-Thought): "explain step by step"

### Common useful prompt templates
- **Translate** — "Translate this Python to Java, preserving error handling..."
- **Explain** — "Explain this function as if I'm a junior dev..."
- **Refactor** — "Refactor this for readability without changing behavior..."
- **Generate tests** — "Write pytest tests covering normal + edge cases for..."
- **Debug** — "Here's the error and the code. Diagnose..."
- **Compare** — "Compare these 3 approaches in terms of perf/clarity..."

### "Replace the TODO" pattern (recap)
Better than vague asks — give the model:
- Surrounding code
- A clear TODO comment
- Function signature already defined
- Example tests

### TDD with AI (recap)
1. Write the failing test yourself
2. Ask AI to implement
3. Run test → green = accept
4. Ask for more edge case tests
5. Iterate

### Security issues common in AI code
- SQL injection (string concatenation)
- Hardcoded secrets / API keys
- Weak crypto (MD5 for passwords, ECB mode)
- Missing input validation
- Open CORS configuration
- Path traversal vulnerabilities
- Logging sensitive data

### Defending against AI security issues
- Run linters/scanners on AI output (Bandit, Semgrep, SonarQube)
- Always read security-relevant code yourself
- Test with attacker mindset
- Don't blindly trust "AI-generated security code"

### Skills that matter MORE in AI era
- **Code reading** (review AI output)
- **System design** (AI is bad at architecture)
- **Requirements engineering** (clear specs → better AI output)
- **Debugging** (AI confidently writes broken code)
- **Domain knowledge** (AI doesn't know your business)
- **Testing skills** (verify what AI produced)

### Skills at RISK of atrophying
- Basic syntax recall
- Boilerplate writing
- Algorithm implementation from memory
- Documentation lookups (AI summarizes them)
- → Counter: keep practicing fundamentals deliberately

### "Vibe coding" anti-pattern (recap)
- Accepting AI output without comprehending
- Cannot debug, extend, or own the code
- Codebase becomes "AI sludge" — you don't know what's in it
- Quality compounds downward

### Comparison: AI as junior dev
| | Junior dev | AI |
|---|---|---|
| Speed | Slow | Fast |
| Confidence | Often unsure | Always confident (incl. wrong) |
| Domain knowledge | Building | None (no specifics) |
| Learn over time | Yes | No (within one chat) |
| Catches own mistakes | Sometimes | Rarely |

→ Review like reviewing a junior's PR.

### Workflow patterns that work well
1. **Generate boilerplate** → review → commit
2. **Brainstorm alternatives** → pick one → implement yourself
3. **Generate tests for your code** → verify they're correct
4. **Translate between languages** → review semantics
5. **Explain unfamiliar code** → verify by reading docs
6. **Pair programming** — back-and-forth conversation

### Workflow patterns to AVOID
- "Just write the whole feature" (too much, hard to review)
- Accepting all suggestions in IDE blindly
- Using AI on critical security/payment code without review
- Letting AI choose architecture
- Trusting hallucinated APIs without verifying

### Common code-comp errors when reading AI code
- Misidentified API (looks plausible but doesn't exist)
- Off-by-one (AI confuses 0-indexed vs 1-indexed)
- Wrong async ordering
- Hallucinated imports
- Missing error cases
- Hardcoded "magic" values

### Lecture summary points
- AI amplifies your skills (good + bad)
- Pre-training → RLHF → inference
- ~40% of Copilot code has security issues (study)
- Treat AI as junior dev needing review
- TDD with AI keeps you in control
- Essential skills: delegation, verification, requirements engineering
