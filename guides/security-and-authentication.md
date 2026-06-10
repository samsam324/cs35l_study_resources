# Security & Authentication â€” CS35L Study Guide

Source: Lecture 11 â€” Security & Authentication (Tobias DÃ¼rschmid).

---

## Why Should We Care About Security?

- Security breaches in 2024 were **up 75% year-over-year**, with organizations facing an average of **1876 attacks per quarter**.
  - Source: https://www.accenture.com/content/dam/accenture/final/accenture-com/document-3/State-of-Cybersecurity-report.pdf
- Data breaches cost **$4.4M each on average**.
  - Source: https://www.ibm.com/reports/data-breach

---

## The CIA Triad â€” The Three Main Security Attributes

### Confidentiality
Sensitive data must be accessed by authorized users only.

### Integrity
Sensitive data must be modifiable by authorized users only. The system maintains the **accuracy, consistency, and trustworthiness** of data over its entire lifecycle.

### Availability
Critical services must be available when needed by clients.

---

## Case Study: 2016 Hollywood Presbyterian Medical Center Ransomware

- **Ransomware** took the hospital offline.
- **$3.6M demanded** by attackers.
- **Malware locks systems by encrypting files** and demands ransom to obtain the decryption key.
- CIA attributes affected: **Availability** (primary), **Confidentiality** (data hostage), **Integrity** (encrypted data unusable).
- Source: https://www.csoonline.com/article/554745/ransomware-takes-hollywood-hospital-offline-36m-demanded-by-attackers.html

---

## Case Study: 2017 Equifax Data Breach

- Sensitive personal data of **148 million Americans** extracted.
- Data stolen: names, home addresses, phone numbers, dates of birth, **social security numbers**, and driver's license numbers.
- **Cost: $1.38 billion** total â€” settlements, regulatory fines, mandatory security improvements.
- CIA attributes affected: **Confidentiality** (primary).
- Source: https://www.breachsense.com/blog/equifax-data-breach/

---

## SQL Injection Attack

### Vulnerable Code

```python
name = get_user_input("username")
pass = get_user_input("userpassword")
sql = ('SELECT * FROM Users '
       'WHERE Name = "' + name + '" '
       'AND Pass = "' + pass + '"')
user = db.execute_query(sql)
login(user) if user else retry()
```

This query returns the user `Tobias` if his password is `password1234`. Otherwise returns nothing.

### Normal Usage

- Username: `Tobias`
- Password: `password1234`
- Resulting query:
  ```sql
  SELECT * FROM Users
  WHERE Name = "Tobias"
  AND Pass = "password1234"
  ```

### The SQL Injection Attack

- Username: `Tobias`
- Password: `" or ""="`
- Resulting query:
  ```sql
  SELECT * FROM Users
  WHERE Name = "Tobias"
  AND Pass = "" or ""=""
  ```
- `""=""` is **always true**. Hence, `Pass = "" or ""=""` is also **always true** â€” the attacker logs in without knowing the password.

### How Common Is SQL Injection?

**6.7% of all vulnerabilities** found in open-source projects are SQL Injection (2023).

Timeline:
- **1998** â€” SQLi attack was first described publicly.
- **2010â€“2021** â€” OWASP ranks SQLi **#1** web app security vulnerability.
- **2011** â€” Sony PlayStation breach compromises passwords & data of over **77 million users**.
- **2023** â€” SQLi in MOVEit transfer app affects **thousands of companies worldwide**.

> If SQL Injection is so common then it is probably very hard to fix, right? **RIGHT????**

(It's not. It's fully solved by parameterized queries â€” see below.)

Sources: https://www.w3schools.com/sql/sql_injection.asp & https://www.aikido.dev/blog/the-state-of-sql-injections

### Defense: Prepared Statements & Parameterized Queries

```python
name = get_user_input("username")
pass = get_user_input("userpassword")
sql = ('SELECT * FROM Users '
       'WHERE Name = @0 '
       'AND Pass = @1')
user = db.execute_query(sql, name, pass)
login(user) if user else retry()
```

- **Parameters referenced by their index.** The syntax varies by DBMS.
- The DBMS is **sanitizing the input parameters to prevent injection attacks**.

### XKCD #327 â€” "Little Bobby Tables"

The classic comic. Explanation:
- `');` ends the SQL query.
- `DROP TABLE Students` removes the entire student database table.
- `; --` ignores the remainder of the original query to avoid syntax errors.

Source: https://xkcd.com/327/

### Which CIA Attributes Are Affected by SQL Injection?

| Attribute | Effect | Frequency |
|---|---|---|
| **Confidentiality** | Attackers can **read sensitive data** from the database | Common |
| **Integrity** | Attackers can **modify, insert, or delete data** in the database | Common |
| **Availability** | Attacks can **disrupt database operations** (e.g., dropping tables, causing system crashes) | Rare in practice |

---

## Cross-Site Scripting (XSS) Attack â€” JavaScript Injection

### How XSS Works

1. An attacker **injects JS code into a trusted site** (e.g., as a comment on a social media post that contains `<script>alert(USC IS BETTER!!!)</script>`).
2. **Browser of the victim executes the JS code from a trusted site** (because it came from the trusted origin).
3. **JS code can access cookies, session tokens, or other sensitive information** retained by the browser and send it to the attacker.

### Diagram (described)

```
Attacker  --(injects JS into)-->  Trusted Site
                                       |
                                       v
                                   Victim's Browser
                                       |  (executes injected JS)
                                       v
                                   Sends sensitive data back to Attacker
```

### Which CIA Attributes Are Affected by XSS?

| Attribute | Effect |
|---|---|
| **Confidentiality** | Browsers of the victim execute the JS code from a trusted site â€” attacker can read sensitive data from the client & might gain access to privileged web sessions |
| **Integrity** | Attacker can modify webpage content & might submit unauthorized requests |

### Prominent Examples of XSS Attacks

#### 2010 Twitter Worm
- Twitter's parsing process failed to properly **sanitize or escape certain characters** (like `onmouseover`) in a tweet.
- Effects:
  - **Pranks** (e.g., displaying rainbow colored text or pop-up alerts).
  - **Redirecting users to malicious third-party websites**.

#### 2018 British Airways Data Breach
- Names, addresses, **credit card numbers**, and **card security codes** stolen from almost **250,000 customers**.
- **JS injected into the webpage re-directed users to a bogus site**.

Sources: https://www.theguardian.com/technology/blog/2010/sep/21/twitter-hack-explained-xss-javascript & https://en.wikipedia.org/wiki/British_Airways_data_breach

---

## Security Design Principle: Zero Trust & Trust Boundaries

- **Users and devices should not be trusted by default.**
- **Any input might be malicious, so sanitize all inputs.**

Question to ask: **Where to draw the trust boundary?**

Example layout:
- **Company Network**: Developer's Device, Server running the App, Database
- **Internet**: User, Boss's Laptop

Anything crossing the boundary must be sanitized.

---

## Security Through Obscurity â€” NOT Your Primary Approach

### Definition
**Security Through Obscurity** = concealing the details or mechanisms of a system to enhance its security.

> "This house is secure because nobody knows that the key is in the flowerpot."
> â€” Security Through Obscurity

### Weakness
The implementation details of the app can be discovered, which opens vulnerabilities. **Don't just hide the key in the flowerpot.**

---

## The Open Design Principle (Apply Carefully)

### Open Design Principle
Attackers **shouldn't be able to break into a system simply by understanding how it works**. Use **robust, public security mechanisms**.

### Public Scrutiny
**Full transparency** allows independent experts, researchers, and the security community to **examine, test, and critique the design** for potential vulnerabilities.

### Complementary Obscurity
**Hide implementation details & configuration details** (e.g., version of a particular framework, database, server configuration) to make it harder to exploit known vulnerabilities.

### When to Apply Each

| Approach | When |
|---|---|
| **Public Scrutiny** | When proposing a **new security approach or algorithm** |
| **Complementary Obscurity** | When implementing a **commercial system** by using existing, publicly scrutinized technology |

---

## Symmetric Encryption (e.g., AES) â€” Ensures Confidentiality

```
                    Secret (Key)
                        |
Cleartext  --Encryption--> Ciphertext --Decryption--> Cleartext
                        |
                  (same key both sides)
```

The **same secret key** is used for both encryption and decryption.

---

## Public-Key Cryptography (e.g., RSA)

### Goal
**How to encrypt & decrypt data without having to secretly exchange a key?**

### Mechanism
A **large random number** is fed into a **Key Generator**, which produces:

- **Public Key** â€” known to everyone.
- **Private Key** â€” known only to you.

### Sending a Message: Alice â†’ Bob

Alice encrypts with **Bob's Public Key**. Bob decrypts with **Bob's Private Key** (Bob's Secret).

```
                  Bob's Public Key                Bob's Private Key
                        |                                |
Cleartext  --Encryption--> Ciphertext --Decryption--> Cleartext
```

Only Bob can decrypt because only Bob has his private key.

---

## Digital Signatures

- **Encrypting a document with your private key allows everyone to read it** (by decrypting it with your public key).
- **Because only you have your private key**, you are the only one who could have written it.
- **Encrypting & decrypting the entire document is slow** â†’ encrypting just the **hash of the document** is faster.

```
                  Author's Private Key        Author's Public Key
                        |                              |
Document  --Signing-->  Signature  --Verification--> Verified
```

---

## Authentication is Not Trivial

After initial username/password login, how does the server know that **subsequent requests** come from an authenticated user?

```
Client â†’ [Username, Password] â†’ Server
Client â† [OK]                  â† Server

Client â†’ [Request]             â†’ Server  â† How does the server know this is authenticated?
Client â† [Reply]               â† Server
```

---

## Authentication Approach 1: NaÃ¯ve â€” Always Send Username & Password

**Don't do this!**

### Goal
How to prove to the server that the request comes from an authenticated user?

### Solution
Client sends username and password with **every request**. Server verifies them with every request.

### Consequences
- **Slow** (server needs to verify the password every time).
- **Insecure** (client needs to memorize the password in memory).

---

## Authentication Approach 2: Session-Based via Session Cookies

### Goal
How to prove to the server that the request comes from an authenticated user?

### Solution
After initial authentication, **server generates a session ID** that the client sends with every request.

```
Client â†’ [Username, Password] â†’ Server
Client â† [Session ID]         â† Server   (stored in Cookie)

Client â†’ [Request, cookie]    â†’ Server
Client â† [Reply]              â† Server

Client â†’ [Request, cookie]    â†’ Server
Client â† [Reply]              â† Server
```

### Details
- A new session ID for each session.
- Session ID is stored in a client-side cookie. Set the **HttpOnly flag**, which **prevents client-side JS from accessing the cookie** (mitigates XSS).

### Consequences
- **Fast** (no database operations needed to verify authentication).
- **Stateful** (the server needs to maintain a session ID with the corresponding ID).
- **Somewhat Secure** (session has short expiration time).
- **Still vulnerable to XSS attacks** (JS code can ride on the session â€” though HttpOnly mitigates direct cookie access).

---

## Authentication Approach 3: JSON Web Token (JWT)

### Goal
How to prove to the server that the request comes from an authenticated user?

### Solution
After initial authentication, **server generates a token (JWT)** that the client sends with every request.

```
Client â†’ [Username, Password] â†’ Server
Client â† [JWT]                â† Server

Client â†’ [Request, JWT]       â†’ Server
Client â† [Reply]              â† Server
```

### Details
- **JWT is an identifier of the user** (e.g., user ID) **signed by the server's private key**.
- Stored **client-side in memory**, in **cookies** (HttpOnly & SameSite), or **local storage**.

### Consequences
- **Fast** (no database operations needed to verify authentication).
- **Somewhat Secure** (JWT is signed by the server & has shorter expiration time than the password).
- **Vulnerable to XSS attacks** (since the JWT can be accessed via JS code â€” unless stored in HttpOnly cookies).
- **Statelessness** (the server does not need to store a session ID).

---

## A Security Plan Analyzes Four Aspects of the System

### 1. Security Model
**What are you defending?**

### 2. Threat Model
**Who might be attacking? What is the attacker trying to achieve?**

### 3. Attack Surface
**Which parts of the system are exposed to an attacker?**

### 4. Protection Mechanisms
**How do we prevent an attacker from compromising the system?**

---

## Building a Threat Model

Threat Model questions:
- **Knowledge** â€” What does the attacker know?
- **Actions** â€” What can the attacker do?
- **Resources** â€” How much effort can it spend?
- **Incentive** â€” Why does the attacker want to do this?

### Example: Wrong Threat Model & Larger Attack Surface

A keypad with only buttons for 9s and 1s assumes only those buttons can be pressed â€” but **foil can be pushed down to dial other buttons**, expanding the attack surface beyond what was modeled.

---

## Security Design Principle: Principle of Least Privilege

### Goal
How to reduce the **attack surface** and the **impact of a security breach** (i.e., blast radius)?

### Principle of Least Privilege
*(aka Principle of Minimal Privilege / Principle of Least Authority)*

> Every program and every privileged user of the system should operate using the **least set of privileges necessary** to complete the job.

### Implementation Ideas
- All user accounts / processes should run with as few privileges as possible.
- Launch applications with as few privileges as possible.

---

## Split Your Application into Separate Components & Apply Least Privilege

If an attacker compromises **one component**, the impact is reduced.

### Example Component Layout

| Component | Privileges |
|---|---|
| **Product Display Service** | Read-only access to the Products database table |
| **Email Notification Service** | Access to a specific email sending API (e.g., SendGrid, AWS SES) and no database access |
| **Image Upload Service** | Write access to a specific folder on the cloud storage bucket (e.g., `/uploads`) and no delete permission |
| **System Backup Service** | Read-only access to the file system or database. Write access only to the designated backup storage location |

---

## Exit Ticket Questions (for self-study)

1. Summarize **three key insights** you learned about Security & Authentication today (in your own words).
2. For each of the **three security attributes of the CIA triad**, describe one potential violation that would be problematic for your course project system.
3. Any unclear/confusing topics.


---

# Appendix: Lecture 11 Slides (raw extracted text)

The following is the full extracted text of Lecture 11. Preserved verbatim so no information is lost.

```text
CS 35L Software
Construction
Lecture 11 ­ Security
& Authentication
Assistant Teaching Professor
Computer Science Department
Why should we care about Security?
· Security breaches in 2024 were up 75% year-over-year, with
  organizations facing an average of 1876 attacks per quarter [1]
· Data breaches cost $4.4M each on average [2]
[1] https://www.accenture.com/content/dam/accenture/final/accenture-com/document-3/State-of-Cybersecurity-report.pdf
[2] https://www.ibm.com/reports/data-breach
The CIA Triad
The Three Main Security Attributes
   Confidentiality
 Sensitive data must be accessed by authorized users only
  Integrity
Sensitive data must be modifiable by authorized users only.
The system maintains the accuracy, consistency, and
trustworthiness of data over its entire lifecycle.
  Availability
Critical services must be available when needed by clients
Which Security Attribute(s) is/are affected?
                                   2016: Ransomware takes Hollywood
                                    Presbyterian Medical Center offline
                                            $3.6M demanded by attackers
                                                               Malware locks systems by encrypting files and
                                                                                   demanding ransom to obtain the
                                                                                                             decryption key
See https://www.csoonline.com/article/554745/ransomware-takes-hollywood-hospital-offline-36m-demanded-by-attackers.html
Which Security Attribute(s) is/are Affected?
     2017 Equifax Data Breach
  Sensitive personal data of 148 million Americans extracted
  (names, home addresses, phone numbers, dates of birth, social
  security numbers, and driver's license numbers)
  Cost: $1.38 billion total, including settlements, regulatory fines, and
  mandatory security improvements.
See https://www.breachsense.com/blog/equifax-data-breach/
SQL Injection           What does it do? Talk
Attack                  to your neighbor(s)
name = get_user_input("username");          How can we break into this
pass = get_user_input("userpassword"); system without knowing the
sql = ('SELECT * FROM Users '                 password of a user?
            'WHERE Name = "' + name + '" '  Talk to your neighbor(s)
            'AND Pass = "' + pass + '"')
user = db.execute_query(sql);                  Returns the user Tobias if his
login(user) if user else retry()               password is password1234.
Username: Tobias                  SELECT *     Otherwise returns nothing.
                                  FROM Users
Password: password1234            WHERE Name = "Tobias"
                                  AND Pass ="password1234"
Username: Tobias                  SELECT *                                    ""="" is always true.
Password: " or ""="               FROM Users                                   Hence, Pass = ""
                                  WHERE Name = "Tobias"                         or ""="" is also
                                  AND Pass = "" or ""=""                             always true
        CS 35L Software Construction: Lecture 11 ­ Security & Authentication                             6
        Tobias Dürschmid
SQL Injection (SQLi) is One of the
Most Common Software Vulnerabilities
· 6.7% of all vulnerabilities found in open-source projects are SQL Injection (2023)
  1998       2010-2021                2011                  2023
SQLi attack  OWASP ranks         Sony PlayStation        SQLi in MOVEit
 was first    SQLi #1 web      breach compromises      transfer app affects
described     app security      passwords & data of
  publicly    vulnerability    over 77 million users      thousands of
                                                      companies worldwide
If SQL Injection is so common then it is probably very hard to fix, right?
                               RIGHT????
See https://www.w3schools.com/sql/sql_injection.asp & https://www.aikido.dev/blog/the-state-of-sql-injections
             CS 35L Software Construction: Lecture 11 ­ Security & Authentication                              7
             Tobias Dürschmid
Use Prepared Statements and Parameterized
Queries to Protect against SQL Injection (SQLi)
· 6.7% of all vulnerabilities found in open-source projects are SQL Injection (2023)
  1998       2010-2021                2011                   2023
SQLi attack  OWASP ranks         Sony PlayStation        SQLi in MOVEit
 was first   SQLi #1 web       breach compromises      transfer app affects
described    app security       passwords & data of
  publicly   vulnerability     over 77 million users      thousands of
                                                      companies worldwide
name = get_user_input("username");
pass = get_user_input("userpassword"); Parameters referenced by their
sql = ('SELECT * FROM Users '       index. The syntax varies by DBMS
            'WHERE Name = @0 '             The DBMS is sanitizing the input
            'AND Pass = @1')                parameters to prevent injection
user = db.execute_query(sql, name, pass);                    attacks
login(user) if user else retry()
See https://www.w3schools.com/sql/sql_injection.asp & https://www.aikido.dev/blog/the-state-of-sql-injections
             CS 35L Software Construction: Lecture 11 ­ Security & Authentication                              8
             Tobias Dürschmid
Explain this Comic             '); ends the SQL query
                               DROP TABLE Students removes
                               the entire student database table
                               ; -- ignores the remainder of the
                               original query to avoid syntax errors
Source: https://xkcd.com/327/
Quiz: Which Security Attribute(s) is/are Affected
by SQL Injection Attacks?
       Confidentiality            Attacker                            Website
Attackers can read sensitive
   data from the database
             Integrity                     SQL
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
How Could We Try to Attack this Website?
          Tobias Dürschmid
          November 5 at 4 PM
UCLA students are by far the best students. Go Bruins!
David Smallberg, Sandra Batista, and 999 others                       5 comments
Like              Comment                                             Share
Paul Eggert
Facts
Write a comment ...
Cross-Site Scripting (XSS) Attack
(JavaScript Injection)
November 5 at 4 PM
UCLA students are by far the best students. Go Bruins!
David Smallberg, Sandra Batista, and 999 others                       5 comments
Like                Comment                                           Share
Paul Eggert
Facts
<script>alert(USC IS BETTER!!!)</script>
Cross-Site Scripting (XSS) Attack
(JavaScript Injection)
November 5 at 4 PM
UCLA students are by far the best students. Go Bruins!
David SmallbAerlge, Srtandra Batista, and 999 others                     6 comments
Like              USC CISomBmEenTtTER!!!                              Share
Attacker                           OK
Write a comment ...
Cross-Site Scripting (XSS) Attack
(JavaScript Injection)
· An attacker injects JS code into a trusted site
· Browser of the victim executes the JS code from a trusted site
· JS code can access cookies, session tokens, or other sensitive information
retained by the browser and send it to the attacker                             Victim
                            Send sensitive data
                                                                 JS Code
                            Trusted Site           Trusted Site
Attacker
          CS 35L Software Construction: Lecture 11 ­ Security & Authentication          14
          Tobias Dürschmid
Quiz: Which Security Attribute(s) is/are Affected
by SQL Injection Attacks?
· An attackerCionjnefcidtseJnStiacolidtye into a trusted site  Integrity
· ABtrtoawckseerrsocfathnerevaicdtismenesxieticvuetedsattahefroJmS code fArottmacaketrrussctaend msitoedify webpage
the client & might gain access to                              content & might submit
· JS copdreivcialengaecdcewsesbcsoitoekiceosn, tseenstsion tokens, or otuhnear usethnosritiizveedinrfeoqrmueastitosn
retained by the browser and send it to the attacker
                                                                                      Victim
                            Send sensitive data
                                                                             JS Code
                            Trusted Site                       Trusted Site
Attacker
          CS 35L Software Construction: Lecture 11 ­ Security & Authentication                                        15
          Tobias Dürschmid
Prominent Examples of XSS Attacks
   2010 Twitter Worm
Twitter's parsing process failed to properly sanitize or escape certain
characters (like onmouseover) in a tweet.
· Pranks (e.g., displaying rainbow colored text or pop-up alerts)
· Redirecting users to malicious third-party websites
   2018 British Airways Data Breach
Names, addresses, credit card numbers and card security codes stolen
from almost 250,000 customers
JS injected into the webpage re-directed users to a bogus site
See https://www.theguardian.com/technology/blog/2010/sep/21/twitter-hack-explained-xss-javascript &
https://en.wikipedia.org/wiki/British_Airways_data_breach
Security Design Principle:
Zero Trust Principle & Trust Boundaries
· Users and devices should not be trusted by default    Where to draw the trust
· Any input might be malicious, so sanitize all inputs           boundary?
      Company Network                                   Internet
                 Developer's Device                        User
Data  Server running the                                Boss's Laptop
Base            App
      CS 35L Software Construction: Lecture 11 ­ Security & Authentication       17
      Tobias Dürschmid
Security Through Obscurity
Should NOT be your Primary Approach to Security
   Security Through Obscurity                  "This house is secure
                                           because nobody knows that
Concealing the details or mechanisms of a  the key is in the flowerpot"
system to enhance its security
                                                  ­ Security Through Obscurity
   Weakness
The implementation
details of the app can
be discovered, which
opens vulnerabilities
 Don't just hide the
key in the flowerpot
Carefully apply
The Open Design Principle
   Security Through Obscurity
Concealing the details or mechanisms of a system to enhance its security
   Open Design Principle
Attackers shouldn't be able to break into a system simply by understanding
how it works. Use robust, public security mechanisms.
   Public Scrutiny                        Complementary Obscurity
Full transparency allows               Hide implementation details &
independent experts, researchers,      configuration details (e.g., version of a
and the security community to          particular framework, data base,
examine, test, and critique the        server configuration) to make it harder
design for potential vulnerabilities.  to exploit known vulnerabilities
When to Apply Public Scrutiny and
When to Apply Complementary Obscurity?
   Public Scrutiny                        Complementary Obscurity
Full transparency allows               Hide implementation details &
independent experts, researchers,      configuration details (e.g., version of a
and the security community to          particular framework, data base,
examine, test, and critique the        server configuration) to make it harder
design for potential vulnerabilities.  to exploit known vulnerabilities
· When proposing a New Security        · When implementing a commercial
  Approach or Algorithm                  system by using existing, publicly
                                         scrutinized technology
Symmetric Encryption (e.g., AES)
Ensures Confidentiality
Encryption                Decryption
                  Secret
Cleartext         (Ciphertext)                                        Cleartext
Public-Key Cryptography (e.g., RSA)
     Goal
  How to encrypt & decrypt data without having to secretly exchange a key?
Large Random Number
                   Key Generator
Public Key                        Private Key
Known to everyone                 Known only to you
To Send a Message to Bob, Alice Encrypts the
Message with Bob's Public Key
Bob's Public Key                Bob's Private Key
Encryption        Decryption
                  Bob's Secret
Cleartext         (Ciphertext)                                        Cleartext
Digital Signatures
· Encrypting a document with your private key allows everyone to read it (by
  decrypting it with your public key)
· Because only you have your private key,
  you are the only one who could have written it
· Encrypting & decrypting the entire document is slow
   · Encrypting just the hash of the document is faster
Signing             Verification
Author's               Author's
Private Key         Public Key
Authentication is Not Trivial
                               Client                                          Server
                               Username, Password
                                           OK
 How does the server know
that this request comes from
    an authenticated user?
                                                                      Request
                                                                       Reply
Naïve Approach: Always send username &
password?         Don't do this!
   Goal                                            Client             Server
How to prove to the server that the request comes  Username, Password
from an authenticated user?                                    OK
   Solution                                        Request, Username,
                                                          Password
Client sends username and password with every                Reply
request. Server verifies them with every request.
                                                   Request, Username,
   Consequences                                           Password
                                                             Reply
· Slow (server needs to verify the password every
   time)
· Insecure (Client needs to memorize the
   password in memory)
Session-Based Authentication
via Session Cookies
   Goal                                               Client                          Server
How to prove to the server that the request comes     Username, Password
from an authenticated user?                                  Session ID
   Solution                                              Cookie
                                                      Session ID
After initial authentication server generates a
session ID that the client sends with every request.          Request, cookie
Details                                                                        Reply
· A new session ID for each session                           Request, cookie
· Session ID is stored in a client-side cookie (set                  Reply
   HttpOnly flag, which prevents client-side JS               Request, cookie
   from accessing the cookie)                                        Reply
         CS 35L Software Construction: Lecture 11 ­ Security & Authentication                 27
         Tobias Dürschmid
Session-Based Authentication
via Session Cookies
Solution                                              Client                   Server
After initial authentication server generates a       Username, Password
session ID that the client sends with every request.          Session ID
   Consequences                                          Cookie
                                                      Session ID
· Fast (no database operations needed to verify
   authentication)                                            Request, cookie
· Stateful (the server needs to maintain a session                   Reply
   ID with the corresponding ID)                              Request, cookie
· Somewhat Secure (session has short expiration                      Reply
   time)
· Still vulnerable to XSS attacks (JS code can                Request, cookie
   ride on the session)                                              Reply
Authentication via JSON Web Token (JWT)
   Goal                                                Client                Server
How to prove to the server that the request comes      Username, Password
from an authenticated user?                                        JWT
   Solution                                                    Request, JWT
                                                                    Reply
After initial authentication server generates a token
(JWT) that the client sends with every request.
   Details                                                     Request, JWT
                                                                    Reply
· JWT is an identifier of the user (e.g., user ID)
   signed by the server's private key                          Request, JWT
                                                                    Reply
· Stored client-side in memory, in cookies
   (HTTPOnly & SameSite), or local storage
Authentication via JSON Web Token (JWT)
Solution                                               Client                Server
After initial authentication server generates a token  Username, Password
(JWT) that the client sends with every request.                       JWT
   Consequences                                                Request, JWT
                                                                    Reply
· Fast (no database operations needed to verify
   authentication)                                             Request, JWT
                                                                    Reply
· Somewhat Secure (JWT is signed by the server
   & has shorter expiration time than the password)            Request, JWT
                                                                    Reply
· Vulnerable to XSS attacks (Since the JWT can
   be accessed via JS code)
· Statelessness (the server does not need to store
   a session ID)
A Security Plan Analyzes Four Aspects of the
System
   Security Model
 What are you defending?
  Threat Model
Who might be attacking? What is the attacker trying to achieve?
  Attack Surface
Which parts of the system are exposed to an attacker?
  Protection Mechanisms
How do we prevent an attacker from compromising the system?
Build a Threat Model
   Threat Model
 Who might be attacking? What is the attacker trying to achieve?
 · Knowledge: What does the attacker know?
 · Actions: What can the attacker do?
 · Resources: How much effort can it spend?
 · Incentive: Why does the attacker want to do this?
What is the issue here?
           Wrong threat model
     Phone numbers with 9s and 1s
                can be dialed.
     Larger Attack Surface
Foil can be pushed down to dial
            other buttons
Security Design Principle:
Principle of Least Privilege
     Goal
  How to reduce the attack surface
  and the impact of a security breach (i.e., blast radius)?
     Principle of Least Privilege (aka Principle of Minimal Privilege / Principle of Least Authority)
  Every program and every privileged user of the system should operate using
  the least set of privileges necessary to complete the job.
     Implementation Ideas
  all user accounts / processes should run with as few privileges as possible &
  launch applications with as few privileges as possible
Split your Application into Separate Components
& Apply the Principle of Least Privilege
If an attacker compromises one component, the impact is reduced
Read-only access         Product     Email      Access to a specific
   to the Products       Display  Notification   email sending API
  database table.        Service                (e.g., SendGrid, AWS
                                    Service
                                                     SES) and no
                                                  database access.
  Write access to a      Image    System        Read-only access to the
specific folder on the   Upload   Backup        file system or database.
cloud storage bucket     Service  Service       Write access only to the
(e.g., /uploads) and no
 delete permission.                                 designated backup
                                                     storage location.
Please fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three key insights you learned about Security &
Authentication today. (Do not copy phrases from the lecture slides)
For each of the three security attributes of the CIA triad, describe one potential violation
of the attribute that would be problematic for your course project system.
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slides use images from Flaticon.com (Creators: Freepik, monkik, kliwir-art, md-tanvirul-haque, metami-septiana)

```
