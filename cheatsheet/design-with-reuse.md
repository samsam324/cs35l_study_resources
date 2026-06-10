# Design with Reuse Cheatsheet (candidate items)

## Why reuse?
1. **Higher productivity / faster time to market** — less to implement/test
2. **Higher software quality / fewer defects** — battle-tested code

## Vision vs reality
- **Vision** (McIlroy 1968): "mass-produced software components" — Lego-like composition
- **Reality** (Garlan 1995): "architectural mismatch" — modules don't fit cleanly; needs glue code

## Internal vs External reuse
| Type | Source | Examples |
|---|---|---|
| **Internal** | same team/org | product lines, shared modules |
| **External** | third party | npm packages, libraries, frameworks |

## External reuse — Python example
```bash
pip install requests
```
```python
import requests
response = requests.get("https://api.github.com")
response.status_code   # 200
response.json()
```

## Design principle: Pin dependency versions
- Use **Pipenv** + **Pipfile** for Python
- Use `package-lock.json` for npm
- Use `Cargo.lock` for Rust

**Pipfile example:**
```
[packages]
urllib3 = "<2.0.0"
docker = "==7.1.0"

[dev-packages]
pytest = "==5.4.2"

[requires]
python_version = "3.9"
```
- `pipenv install <pkg>` — install + record
- `pipenv run <prog>` — run inside virtual env

## Case studies
### Heartbleed (OpenSSL, 2014)
- Buffer over-read in Heartbeat extension
- Introduced Feb 2012, discovered April 2014
- 17% of secure web servers vulnerable on disclosure
- **Lesson:** reused packages can introduce vulnerabilities

### left-pad (npm, 2016)
- 11-line package; transitively used by React, Babel
- Author unpublished it on March 22, 2016 → broke web builds globally
- npm forcibly republished 2 hours later
- **Lesson:** avoid trivial-but-critical deps; analyze supply chain

### Ariane 5 (1996)
- Reused Ariane 4 inertial reference system
- 16-bit int couldn't hold Ariane 5's higher velocity
- Overflow → self-destruction in 37 sec
- $370M loss
- **Lesson:** software that worked in one context may not in another. Check IMPLICIT ASSUMPTIONS

## Design Principles for Reuse
1. **Keep dependency versions fixed** (Pipenv/lock files)
2. **Update dependencies** to receive security patches (be aware of side effects)
3. **Strive for fewer dependencies** — avoid trivial code, analyze supply chain
4. **Prefer popular & well-maintained packages** (but fit-to-context > popularity)
5. **External reuse is NOT one-time** — updates may break, packages may be abandoned

## Cost-Benefit Analysis (External Reuse)
**Costs (effort to adapt):**
- Integration effort (complexity, similarity of context)
- Finding the module
- Updating effort
- Limiting changeability

**Benefits (effort saved):**
- Implementation effort
- Testing effort
- Benefit of update propagation

## Library vs Framework
| | Library | Framework |
|---|---|---|
| Direction | You call IT | IT calls YOU |
| Control | You're in charge | "Hollywood Principle" / Inversion of Control |
| Flexibility | High | Lower (more decisions made for you) |
| Reverse-able | Easier | Harder |
| Example | `axios.get(...)` | Express's `app.get('/', cb)` |

> "Don't call us. We'll call you." — Hollywood Principle (frameworks)

## Internal reuse principles
- **Identify violated assumptions** — check docs and code for assumptions in reuse candidate
- **Test the reused module** in your new context
- Don't assume reused code is correct

## NASA reuse model
- **Tested in space → TRUSTED**
- **Tested on Earth → UNTRUSTED**
- Heavy internal reuse; almost no external reuse for critical software

## Decision-Making (Rational process)
1. Identify requirements (what's important?)
2. Think of many design alternatives (research: more options → better designs)
3. Evaluate alternatives against requirements
4. Consider trade-offs, make decision

## Design Docs (Google practice)
1. **Context & Scope** — background
2. **Goals & Non-Goals** — requirements + what to skip
3. **The Design** — selected solution (models, APIs, constraints)
4. **Alternatives** — what else was considered + trade-offs

> "Our job is not to produce code per se, but to solve problems." — Malte Ubl

## Quick decision questions
- Is this dependency well-maintained?
- Are version pinning + lock files in place?
- What's the bus factor of the package?
- Is my context similar enough to the original?
- Have I checked for implicit assumptions?

## Additional items (potentially missing)

### Semantic versioning (semver)
`MAJOR.MINOR.PATCH` (e.g., `2.4.1`)
- **MAJOR** — breaking changes
- **MINOR** — new features, backward compatible
- **PATCH** — bug fixes, backward compatible

### Version range syntax
- `^1.2.3` — compatible with 1.x (≥1.2.3, <2.0.0)
- `~1.2.3` — patch-compatible (≥1.2.3, <1.3.0)
- `>=1.2.3` — exact lower bound
- `1.2.3` — exact version
- `*` or `latest` — any (dangerous!)

### Open-source licenses (quick reference)
| License | Permissive? | Notes |
|---|---|---|
| MIT | Yes | Very permissive |
| Apache 2.0 | Yes | + patent grant |
| BSD (2/3-clause) | Yes | Similar to MIT |
| GPL v2/v3 | No (copyleft) | Derivatives must be GPL |
| LGPL | Weak copyleft | Lib-only |
| AGPL | Strong copyleft | Even network use triggers |
| Unlicense / CC0 | No restrictions | Public domain |
| Proprietary | Restricted | Read terms |

### License compatibility (rough)
- MIT, BSD, Apache → usable in most projects
- GPL → forces your project to be GPL if you link
- Check before adding deps to commercial software

### Dependency types
- **Runtime** — needed in production (`dependencies` in npm, `[packages]` in Pipfile)
- **Dev** — only for dev/testing (`devDependencies`, `[dev-packages]`)
- **Optional** — try to install but don't fail
- **Peer** — expects host project to install (frameworks)

### Dependency graphs
- `npm ls`, `pip show <pkg>`, `mvn dependency:tree`
- Transitive deps can be huge
- Vulnerabilities often in transitive deps

### Supply chain attacks
- **Typosquatting** — fake package with similar name (`requets` instead of `requests`)
- **Account compromise** — attacker takes over maintainer account
- **Malicious update** — pushed bad code (event-stream, eslint-scope)
- **Dependency confusion** — public package overrides private

### Defensive practices
- Pin exact versions in lockfile
- Audit dependencies (`npm audit`, `pip-audit`)
- Use Dependabot / Renovate to update controlled
- Verify package signatures where possible
- Avoid tiny packages (left-pad lesson)

### Lock files
- `package-lock.json` (npm)
- `yarn.lock`
- `Pipfile.lock`
- `poetry.lock`
- `Cargo.lock`
- COMMIT THESE to git

### Cost-Benefit reminder
**Costs:**
- Integration effort (does it fit?)
- Finding & evaluating
- Updating effort
- Limiting changeability
- Security risk

**Benefits:**
- Implementation effort saved
- Testing effort saved
- Update propagation
- Community support
- Battle-tested code

### Internal vs External principles
| | Internal | External |
|---|---|---|
| Cost | Identify implicit assumptions | Integration, finding |
| Risk | "Worked in different context?" | Maintenance, supply chain |
| Update | We control | We don't control |

### Library vs Framework (review)
| | Library | Framework |
|---|---|---|
| Control flow | You call it | It calls you (Hollywood) |
| Flexibility | High | Lower |
| Reverse cost | Low | High |
| Example | axios, lodash | Express, React, Django |

### Common reuse strategies
- **Source code copy** (vendoring) — full control, but no auto-updates
- **Package manager** — easy, but trust required
- **Git submodule** — pin to specific commit, manual update
- **Microservice** — wrap as service; deploy independently

### Code reviewability of reuse decisions
When choosing a dep, document:
- Why this package?
- What alternatives?
- Maintenance status (last commit?)
- License
- Trust level

### Case study recap
- **Heartbleed** (OpenSSL) — buffer over-read; 17% of web servers
- **left-pad** — 11-line npm package broke web builds globally
- **Ariane 5** — reused Ariane 4 code; 16-bit overflow; $370M loss
- Common theme: **assumptions** about reused code

### Update propagation (the upside)
- Security patches in deps reach you automatically
- New features arrive without rewriting
- Trade-off: occasional breaking changes
