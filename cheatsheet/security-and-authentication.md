# Security & Authentication Cheatsheet (candidate items)

## Stats
- Security breaches up 75% YoY (2024); 1876 attacks/quarter avg
- Data breaches cost $4.4M each on average

## CIA Triad
- **C**onfidentiality — only authorized users can READ
- **I**ntegrity — only authorized users can MODIFY; data accurate & trustworthy
- **A**vailability — services available when needed

## Case studies (for exam recognition)
| Year | Event | CIA Affected |
|---|---|---|
| 2016 | Hollywood Presbyterian ransomware ($3.6M demanded) | A (primary), C, I |
| 2017 | Equifax data breach (148M Americans, $1.38B) | C |
| 2010 | Twitter XSS worm (onmouseover) | C, I |
| 2018 | British Airways breach (~250K customers, credit cards) | C, I |
| 2011 | Sony PlayStation breach (77M users) | C |

## SQL Injection (SQLi)
**Vulnerable code:**
```python
sql = "SELECT * FROM Users WHERE Name='" + name + "' AND Pass='" + pwd + "'"
```
**Attack:** password = `" or ""="` → query becomes always true → log in without password

**Defense: Prepared Statements / Parameterized Queries**
```python
sql = "SELECT * FROM Users WHERE Name=@0 AND Pass=@1"
db.execute_query(sql, name, pwd)    # DBMS sanitizes
```

**XKCD #327 ("Little Bobby Tables"):**
- `');` ends SQL query
- `DROP TABLE Students` deletes table
- `; --` ignores rest

**CIA attributes hit:**
- Confidentiality (read sensitive data) — common
- Integrity (modify/insert/delete data) — common
- Availability (drop tables, crash) — rare

## Cross-Site Scripting (XSS)
- Attacker injects JS into a trusted site (e.g., comment containing `<script>...</script>`)
- Browser of victim executes injected JS
- JS can steal cookies, session tokens
- **Defenses**: output encoding, Content Security Policy (CSP), HttpOnly cookies

## Zero Trust / Trust Boundaries
- Users and devices should NOT be trusted by default
- Any input MIGHT be malicious → sanitize all inputs
- Identify where the trust boundary lies (company net vs internet)

## Security Through Obscurity (NOT primary defense)
- "Key in flowerpot" — concealing implementation
- WEAKNESS: details can be discovered
- Use as **complementary** to robust mechanisms, not as primary

## Open Design Principle
- Robust public security > hidden mechanism
- **Public Scrutiny** — community can examine + critique (use when proposing NEW algorithm)
- **Complementary Obscurity** — hide config/version details (use atop publicly-vetted tech)

## Symmetric encryption (AES)
- Same secret key for encrypt + decrypt
- Both parties need the key → key exchange problem

## Public-key crypto (RSA)
- Public key (known to all) + Private key (only you)
- Encrypt with recipient's public key → only they can decrypt with their private

## Digital signatures
- Sign with YOUR private key → anyone can verify with YOUR public key
- Proves authenticity
- Usually sign a HASH of the document (faster)

## Authentication methods
| Method | How | Pros | Cons |
|---|---|---|---|
| Naive (send password every time) | Client sends user+pwd | Simple | Slow, insecure (pwd in memory) |
| Session cookie | Server issues session ID, client sends in cookie | Fast, somewhat secure | Stateful, XSS vulnerable |
| JWT (JSON Web Token) | Server issues signed token | Fast, stateless | XSS vulnerable |

## Cookie flags
- **HttpOnly** — prevents JS from accessing cookie (mitigates XSS theft)
- **SameSite** — restricts cross-site sending (mitigates CSRF)
- **Secure** — only sent over HTTPS

## JWT vs Session
| | Session | JWT |
|---|---|---|
| Server stores state | yes (session table) | no (stateless) |
| Storage | cookie | cookie / memory / localStorage |
| XSS vulnerable | yes (less if HttpOnly) | yes (if accessible to JS) |

## Security Plan (4 aspects)
1. **Security Model** — what are you defending?
2. **Threat Model** — who's attacking, why?
3. **Attack Surface** — what's exposed?
4. **Protection Mechanisms** — how do you defend?

## Threat Model questions
- **Knowledge** — what does the attacker know?
- **Actions** — what can they do?
- **Resources** — how much effort can they spend?
- **Incentive** — why?

## Principle of Least Privilege (POLP)
- Every program / user gets MINIMUM privileges needed
- Reduces attack surface AND blast radius of breach
- Example: separate services with narrow access
  - Display Service — read-only on Products table
  - Email Service — only SendGrid API access
  - Upload Service — write only to /uploads, no delete
  - Backup Service — read-only data + write only to backup location

## Component splitting
- Split monolithic app into components
- If one is compromised, attacker's reach is limited

## Common attacks summary
| Attack | Defense |
|---|---|
| SQL injection | Prepared statements |
| XSS | Output encoding, CSP, HttpOnly |
| CSRF | SameSite cookie, CSRF tokens |
| Ransomware | Backups, segmentation |
| MITM | TLS/HTTPS |
| Brute force | Rate limiting, MFA |

## SHA / hashing context
- Passwords stored as HASH (not plaintext)
- Use bcrypt, scrypt, Argon2 (NOT MD5/SHA1)
- Add salt to prevent rainbow table attacks

## Additional items (potentially missing)

### CSRF (Cross-Site Request Forgery)
- Attacker tricks user's browser into making request to a site they're logged into
- User's cookies sent automatically → action performed as user
- **Defense:**
  - CSRF tokens (random per-session token in form)
  - **SameSite cookie attribute** (`Strict` or `Lax`)
  - Check `Origin` / `Referer` header
- Different from XSS: CSRF doesn't run JS; just exploits cookie auth

### XSS vs CSRF (often confused)
| | XSS | CSRF |
|---|---|---|
| What | Inject JS into trusted site | Trick user into making request |
| Runs JS? | Yes | No |
| Defense | Sanitize output, CSP, HttpOnly | CSRF tokens, SameSite |

### MITM (Man-in-the-Middle)
- Attacker intercepts traffic between client and server
- Defense: **HTTPS / TLS** (encrypts traffic + authenticates server via cert)

### Hashing vs Encryption
| | Hashing | Encryption |
|---|---|---|
| Reversible | NO (one-way) | YES (with key) |
| Use for | Passwords, integrity | Confidentiality |
| Examples | SHA-256, bcrypt, Argon2 | AES, RSA |

### Password storage best practice
- Never store plain text
- Use **bcrypt, scrypt, or Argon2** (slow, adaptive)
- Don't use SHA / MD5 alone (too fast, brute-forceable)
- **Salt** each password (per-user random value) — prevents rainbow tables
- **Pepper** (optional, server-side secret added to all)

### Common hash algorithms
- **MD5** — broken; do NOT use for security
- **SHA-1** — deprecated; do NOT use
- **SHA-256, SHA-512** — fine for integrity, NOT passwords
- **bcrypt, scrypt, Argon2** — designed for passwords (slow)
- **HMAC** — keyed hash for message authentication

### TLS / HTTPS basics
- TLS = Transport Layer Security
- Provides:
  - **Confidentiality** (encryption)
  - **Integrity** (MAC / signature)
  - **Authentication** (server identity via certificate)
- Handshake: public-key for key exchange, then symmetric for data
- Certificates issued by **Certificate Authorities (CAs)**

### OAuth / OpenID Connect (brief)
- **OAuth 2.0** = delegated authorization ("let X access my Y on your behalf")
- **OpenID Connect** = layer on top for authentication ("who is this user")
- Used in "Sign in with Google/GitHub/Apple"
- You get an **access token** (and sometimes refresh token + ID token)

### Multi-factor authentication (MFA)
- Something you KNOW (password) + HAVE (phone) + ARE (biometric)
- **TOTP** (Time-based One-Time Password) — Google Authenticator, Authy
- **U2F / WebAuthn** — hardware key (YubiKey)
- **SMS** — least secure; SIM swap attacks

### Rate limiting
- Defense against brute force, scraping, DoS
- Common strategies:
  - Token bucket
  - Sliding window
  - Per-IP / per-user limits
- HTTP 429 Too Many Requests

### Input validation principles
- **Whitelist** valid inputs (not blacklist bad ones)
- **Validate at the boundary** (where input enters system)
- **Sanitize when interpreting** (HTML, SQL, shell)
- **Never trust** client-side validation alone

### Common OWASP Top 10 (rough list)
1. Broken Access Control
2. Cryptographic Failures
3. Injection (SQL, XSS, etc.)
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable & Outdated Components
7. ID & Auth Failures
8. Software & Data Integrity Failures
9. Logging & Monitoring Failures
10. Server-Side Request Forgery (SSRF)

### Secrets management
- NEVER commit secrets to git
- Use environment variables, vaults (HashiCorp Vault, AWS Secrets Manager)
- `.env` file (gitignored)
- Rotate secrets regularly

### Authorization vs Authentication
- **Authentication** = WHO are you? (login)
- **Authorization** = What are you ALLOWED to do? (permissions)
- Different concepts; often combined

### Common attacks summary table (review)
| Attack | Goal | Defense |
|---|---|---|
| SQL Injection | Read/modify DB | Prepared statements |
| XSS | Steal session, hijack | Sanitize output, CSP, HttpOnly |
| CSRF | Forge requests as user | CSRF tokens, SameSite |
| MITM | Intercept traffic | HTTPS/TLS |
| Ransomware | Encrypt + extort | Backups, segmentation |
| Brute force | Guess passwords | Rate limiting, MFA |
| Privilege escalation | Gain admin | Least privilege, audit |
| Phishing | Trick user | Awareness training, MFA |
