# Code Review Cheatsheet (candidate items)

## L18 title
"Code Review & Code Quality" — code review part below; clean code in separate guide.

## Modern Code Review research

### Microsoft (Czerwonka et al., ICSE 2015)
- Studied CodeFlow tool at Microsoft
- Open 5-6 hours/dev/day, but only ~30 min of actual interaction
- Primary goals (in practice):
  - Find defects
  - Improve maintainability
  - Share knowledge
  - Broadcast progress

### Google (Sadowski et al., ICSE-SEIP 2018)
- "Code review at Google was introduced to FORCE developers to write code that other developers could understand"
- Primary purpose: **teach junior developers**
- Expectations don't center around problem solving

## Expectation vs Reality
| | Expectation | Reality |
|---|---|---|
| Main goal | Find bugs | Knowledge transfer + maintainability + sometimes bugs |
| Frequency | Many bugs caught | Few bugs caught |
| Real value | Defect detection | Readability, maintainability, mentorship |

## Why code reviews don't find many bugs
- Developers reviewing don't deeply understand the code
- Limited time per review
- Lack of context

## Real benefits of code reviews
1. **Knowledge transfer** — junior devs learn from senior
2. **Maintainability** — code stays understandable
3. **Defect detection** — sometimes, not always
4. **Shared ownership** — multiple people know the code
5. **Mentorship** — primary way Google teaches juniors

## Best Practices to Maximize Effectiveness

### Reviewer count
- **1-2 reviewers is optimal**
- More = diminishing returns (people defer to each other)

### PR size
- **Short PRs: ~200-400 LOC**
- Larger PRs get rubber-stamped (effective review impossible)
- If big: split into stacked PRs

### Stacked PRs
- Chain related changes
- Each PR builds on the previous
- Each is reviewable independently

## For the AUTHOR
- Write small, focused PRs (one logical change)
- Self-review first (run through your own diff)
- Write a clear PR description: WHAT + WHY
- Reference the issue / bug being fixed
- Add tests
- Run the linter / CI before requesting review

## For the REVIEWER
- Read with intent to understand
- Look for:
  - Correctness
  - Readability / clarity
  - Tests
  - Edge cases
  - Code smells
- Be specific in feedback — point to lines
- Distinguish required changes vs suggestions
- Be kind — code review is about helping, not gatekeeping

## What to look for in a code review
- Does it work? (logic correct)
- Is it tested?
- Is it readable?
- Are there code smells?
- Are edge cases handled?
- Security implications?
- Performance concerns?
- Documentation?
- Naming?
- Consistency with codebase style?

## Common smells flagged in review
- Duplicate code
- Long methods / large classes
- Magic numbers
- Insufficient tests
- Poor naming
- Missing error handling
- Suppressed exceptions
- Hard-coded values that should be config

## Review feedback style
- ❌ "This is wrong"
- ✅ "Consider X because Y. Have you thought about edge case Z?"
- Tag severity: `[blocker]`, `[suggestion]`, `[nit]`, `[question]`

## Anti-patterns to avoid
- Rubber-stamp approval (LGTM without reading)
- Bikeshedding (arguing over trivial style)
- Personal attacks vs critiquing code
- Massive PRs that can't be reviewed properly
- Ignoring CI failures

## Stats (rough)
- Effective review limit: ~200-400 LOC per session
- Diminishing returns after ~60 minutes of review
- Most defects in first 30 min of review

## Code review ties to Clean Code (next guide)
- Clean code is review-able
- 4 Pillars: Meaningful Naming, Simplified Structure, Purposeful Documentation, Design by Contract
- Reviewers check these pillars

## Quick checklist for "is this PR mergeable"
- [ ] Tests pass
- [ ] New code has tests
- [ ] Documentation updated (if API changed)
- [ ] No obvious bugs
- [ ] Readable
- [ ] No code smells (or justified)
- [ ] Security implications considered
- [ ] No accidental scope creep

## Additional items (potentially missing)

### Review checklist (extended)
- [ ] Does it solve the problem? (correctness)
- [ ] Tests added?
- [ ] Edge cases handled? (empty, null, max, min, unicode)
- [ ] Error handling sufficient?
- [ ] Security implications? (input validation, secrets in logs)
- [ ] Performance reasonable?
- [ ] Naming clear?
- [ ] Comments useful?
- [ ] Documentation updated?
- [ ] Backward compatibility maintained?
- [ ] Any code smells?
- [ ] Follows project conventions?
- [ ] Logging appropriate?
- [ ] Resource cleanup (files, connections)?
- [ ] Concurrency safe (if relevant)?

### Reviewer feedback levels
| Tag | Meaning |
|---|---|
| `[blocker]` | Must fix before merge |
| `[required]` | Should fix before merge |
| `[suggestion]` | Consider, but not blocking |
| `[nit]` | Trivial / style preference |
| `[question]` | Asking to understand, not necessarily change |
| `[praise]` | Calling out good work |

### Best review practices (academic findings)
- Limit to **200-400 LOC per review session** (effective)
- **60 min max** per session (focus drops)
- Review **same day** if possible (PR rots)
- Don't be the **bottleneck** — review quickly even if not deep
- **Pair on big changes** when 1 reviewer feels stuck

### Common review situations
- "I disagree" → propose alternative + reasoning, not just "no"
- "This is wrong" → cite specific behavior + suggest
- "Why this approach?" → ask BEFORE assuming bad
- "LGTM" → only after actually reading

### Author best practices
- Self-review your diff before requesting review
- Write a clear PR description (WHAT + WHY)
- Reference the issue/bug
- Add screenshots for UI changes
- Tag specific reviewers (don't broadcast)
- Respond to comments — don't just push and ignore
- Use `git rebase` to keep history clean
- Don't take feedback personally

### When the author and reviewer disagree
- Try to understand the OTHER perspective
- Escalate to senior dev if stuck
- Default to "ship something now; iterate"
- Avoid bike-shedding (arguing trivial things)

### PR template example
```markdown
## Why
[Why is this change needed?]

## What
[What does this PR do?]

## How tested
[How did you test? Manual? CI?]

## Screenshots
[If UI changes]

## Related
[Issue link]
```

### Stacked PRs concept
- Each PR builds on previous (chain)
- Smaller individual reviews
- Often: PR1 (refactor) → PR2 (feature using new structure)

### Anti-patterns
- 1000-line PRs (impossible to review properly)
- "LGTM, didn't read" rubber-stamp
- "Reviewer ego" — gatekeeping over teaching
- Author defensiveness about feedback
- Forgetting to re-review after changes pushed
- Bike-shedding trivial issues

### Code review research summary (lecture)
| Source | Finding |
|---|---|
| Microsoft (Czerwonka 2015) | CodeFlow open 5-6 hr/day, 30 min interaction. Goals: defects, maintainability, knowledge, progress |
| Google (Sadowski 2018) | Introduced to FORCE understandable code. Primary purpose: teach juniors |

### Code review for AI-generated code
- AI output should ALWAYS be reviewed
- Treat AI like a junior dev: ask "why this approach?"
- Verify imports/APIs are real (hallucination risk)
- Test thoroughly (AI may miss edge cases)

### Tools commonly used
- GitHub PRs / Reviews
- GitLab MRs
- Gerrit (Google-style)
- Phabricator (legacy)
- Reviewable.io
- Linters integrated (eslint, pylint) — auto-flag basic issues
- CI must pass before merge

### Common defects code review actually catches
- Naming issues
- Missing error handling
- Test gaps
- Code smell creep
- Style inconsistency
- Knowledge gaps (junior misunderstanding API)

### Defects code review usually MISSES
- Concurrency bugs
- Performance regressions
- Security issues (need specialized review)
- Complex logical bugs
- Integration issues
