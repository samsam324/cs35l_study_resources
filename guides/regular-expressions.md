This page is a reference guide for Regular Expression syntax, engine mechanics, and worked examples. It is designed to be consulted alongside or after the interactive tutorial — not as a replacement for hands-on practice.

Quick Reference
Literal Characters
a
Matches the exact character "a"
123
Matches the exact sequence "123"
HeLLo
Matches the exact (case-sensitive) sequence "HeLLo"
\.
Escaped dot — matches a literal "." (unescaped dot matches any character)
Character Classes
[abc]
A single character of: a, b, or c
[^abc]
Any character except: a, b, or c
[a-z]
Any character in range a-z
.
Any character except newline
\s
Whitespace
\S
Not whitespace
\d
Digit (0-9)
\D
Not digit
\w
Word character (a-z, A-Z, 0-9, _)
\W
Not word character
Quantifiers (Greedy)
a*
0 or more
a+
1 or more
a?
0 or 1 (optional)
a{n}
Exactly n times
a{n,}
n or more times
a{n,m}
Between n and m times
Quantifiers (Lazy)
a*?
0 or more, as few as possible
a+?
1 or more, as few as possible
Anchors & Boundaries
^
Start of string/line
$
End of string/line
\b
Word boundary
\B
Not a word boundary
Groups & Alternation
(...)
Group — treat as a single unit
(a|b)
Alternation — matches either a or b
(?<name>...)
Named group — access by name, not number
(?:...)
Non-capturing group
\1
Backreference to group 1
Lookarounds
(?=...)
Positive lookahead
(?!...)
Negative lookahead
(?<=...)
Positive lookbehind
(?<!...)
Negative lookbehind
Overview
The Core Purpose of RegEx
At its heart, RegEx solves three primary problems in software engineering:

Validation: Ensuring user input matches a required format (e.g., verifying an email address or checking if a password meets complexity rules).
Searching & Parsing: Finding specific substrings within a massive text document or extracting required data (e.g., scraping phone numbers from a website).
Substitution: Performing advanced search-and-replace operations (e.g., reformatting dates from YYYY-MM-DD to MM/DD/YYYY).
The Conceptual Power of Pattern Matching: What RegEx Actually Does
Before we dive into the specific symbols and syntax, we need to understand the fundamental shift in thinking required to use Regular Expressions.

When we normally search through text (like using Ctrl + F or Cmd + F in a word processor), we perform a Literal Search. If you search for the word cat, the computer looks for the exact character c, followed immediately by a, and then t.

However, real-world data is rarely that predictable. Regular Expressions allow you to perform a Structural Search. Instead of telling the computer exactly what characters to look for, you describe the shape, rules, and constraints of the text you want to find.

Let’s look at one simple and two complex examples to illustrate this conceptual leap.

The Simple Example: The “Cat” Problem
Imagine you are proofreading a document and want to find every instance of the animal “cat”.

If you do a literal search for cat, your text editor will highlight the “cat” in “The cat is sleeping”, but it will also highlight the “cat” in “catalog”, “education”, and “scatter”. Furthermore, a literal search for cat will completely miss the plural “cats” or the capitalized “Cat”.

Conceptually, a Regular Expression allows you to tell the computer:

“Find the letters C-A-T (ignoring uppercase or lowercase), but only if they form their own distinct word, and optionally allow an ‘s’ at the very end.” By defining the rules of the word rather than just the literal letters, RegEx eliminates the false positives (“catalog”) and captures the edge cases (“Cats”).

Complex Example 1: The Phone Number Problem
Suppose you are given a massive spreadsheet of user data and need to extract everyone’s phone number to move into a new database. The problem? The users typed their phone numbers however they wanted. You have:

123-456-7890
(123) 456-7890
123.456.7890
1234567890
A literal search is useless here. You cannot Ctrl + F for a phone number if you don’t already know what the phone number is!

With RegEx, you don’t search for the numbers themselves. Instead, you describe the concept of a North American phone number to the engine:

“Find a sequence of exactly 3 digits (which might optionally be wrapped in parentheses). This might be followed by a space, a dash, or a dot, but it might not. Then find exactly 3 more digits, followed by another optional space, dash, or dot. Finally, find exactly 4 digits.”

With one single Regular Expression, the engine will scan millions of lines of text and perfectly extract every phone number, regardless of how the user formatted it, while ignoring random strings of numbers like zip codes or serial numbers.

Complex Example 2: The Server Log Problem
Imagine you are a backend engineer, and your company’s website just crashed. You are staring at a server log file containing 500,000 lines of system events, timestamps, IP addresses, and status codes. You need to find out which specific IP addresses triggered a “Critical Timeout” error in the last hour.

The data looks like this: [2023-10-25 14:32:01] INFO - IP: 192.168.1.5 - Status: OK [2023-10-25 14:32:05] ERROR - IP: 10.0.4.19 - Status: Critical Timeout

You can’t just search for “Critical Timeout” because that won’t extract the IP address for you. You can’t search for the IP address because you don’t know who caused the error.

Conceptually, RegEx allows you to create a highly specific, multi-part extraction rule:

“Scan the document. First, find a timestamp that falls between 14:00:00 and 14:59:59. If you find that, keep looking on the same line. If you see the word ‘ERROR’, keep going. Find the letters ‘IP: ‘, and then permanently capture and save the mathematical pattern of an IP address (up to three digits, a dot, up to three digits, etc.). Finally, ensure the line ends with the exact phrase ‘Critical Timeout’. If all these conditions are met, hand me back the saved IP address.”

This is the true power of Regular Expressions. It transforms text searching from a rigid, literal matching game into a highly programmable, logic-driven data extraction pipeline.

The Anatomy of a Regular Expression
A regular expression is composed of two types of characters:

Literal Characters: Characters that match themselves exactly (e.g., the letter a matches the letter “a”).
Metacharacters: Special characters that have a unique meaning in the pattern engine (e.g., *, +, ^, $).
Let’s explore the most essential metacharacters and constructs.

Anchors: Controlling Position
Anchors do not match any actual characters; instead, they constrain a match based on its position in the string.

^ (Caret): Asserts the start of a string. ^Hello matches “Hello world” but not “Say Hello”.
$ (Dollar Sign): Asserts the end of a string. end$ matches “The end” but not “endless”.
By default ^ and $ match the start and end of the entire string. With the multiline flag (m in JavaScript / re.M in Python), they additionally match the start and end of each line within the string.

Practice this: Anchors exercises in the Interactive Tutorial

Character Classes: Matching Sets of Characters
Character classes (or sets) allow you to match any single character from a specified group.

[abc]: Matches either “a”, “b”, or “c”.
[a-z]: Matches any lowercase letter.
[A-Za-z0-9]: Matches any alphanumeric character.
[^0-9]: The caret inside the brackets means negation. This matches any character that is not a digit.
Practice this: Character Classes exercises in the Interactive Tutorial

Metacharacters
Because certain character sets are used so frequently, RegEx provides handy meta characters:

\d: Matches any digit. In ASCII-only engines (POSIX, JavaScript without the u flag), this is equivalent to [0-9]. In Python 3 (and other Unicode-aware engines), \d by default matches any Unicode digit (e.g., Devanagari ९); pass re.ASCII to restrict it to [0-9].
\w: Matches any “word” character. In ASCII-only engines this is [a-zA-Z0-9_]; in Unicode-aware engines (Python 3 by default) it also matches accented letters and characters from non-Latin scripts.
\s: Matches any whitespace character (spaces, tabs, line breaks).
. (Dot): The wildcard. Matches any single character except a newline (turn on the s/DOTALL flag to also match newlines). To match a literal dot, you must escape it with a backslash: \..
Practice this: Meta Characters exercises in the Interactive Tutorial

Quantifiers: Controlling Repetition
Quantifiers tell the RegEx engine how many times the preceding element is allowed to repeat.

* (Asterisk): Matches 0 or more times. (a* matches “”, “a”, “aa”, “aaa”)
+ (Plus): Matches 1 or more times. (a+ matches “a”, “aa”, but not “”)
? (Question Mark): Matches 0 or 1 time (makes the preceding element optional).
{n}: Matches exactly n times.
{n,m}: Matches between n and m times.
Practice this: Quantifiers exercises in the Interactive Tutorial

Real-World Examples
Let’s look at how we can combine these rules to solve practical problems.

Example A: Password Validation
Suppose we need to validate a password that must be at least 8 characters long and contain only letters and digits.

The Pattern: ^[a-zA-Z0-9]{8,}$

Breakdown:

^ : Start of the string.
[a-zA-Z0-9] : Allowed characters (any letter or number).
{8,} : The previous character class must appear 8 or more times.
$ : End of the string. (This ensures no special characters sneak in at the end).
Example B: Email Validation
Validating an email address perfectly according to the RFC standard is notoriously difficult, but a highly effective, standard RegEx looks like this:

The Pattern: ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$

Breakdown:

^[a-zA-Z0-9._%+-]+ : Starts with one or more alphanumeric characters, dots, underscores, percent signs, plus signs, or dashes (the username).
@ : A literal “@” symbol.
[a-zA-Z0-9.-]+ : The domain name (e.g., “ucla” or “google”).
\. : A literal dot (escaped).
[a-zA-Z]{2,}$ : The top-level domain (e.g., “edu” or “com”), consisting of 2 or more letters, extending to the end of the string.
Groups and Named Groups
Often, you don’t just want to know if a string matched; you want to extract specific parts of the string. This is done using Groups, denoted by parentheses ().

Groups
If you want to extract the domain from an email, you can wrap that section in parentheses: ^.+@(.+\.[a-zA-Z]{2,})$ The engine will save whatever matched inside the () into a numbered variable that you can access in your programming language.

Named Groups
When dealing with complex patterns, remembering group numbers gets confusing. Modern RegEx engines support Named Groups using the syntax (?<name>pattern) (or (?P<name>pattern) in Python).

Example: Parsing HTML Hex Colors Imagine you want to extract the Red, Green, and Blue values from a hex color string like #FF00A1:

The Pattern: #(?P<R>[0-9a-fA-F]{2})(?P<G>[0-9a-fA-F]{2})(?P<B>[0-9a-fA-F]{2})

Here, we define three named groups (R, G, and B). When this runs against #FF00A1, our code can cleanly extract:

Group “R”: FF
Group “G”: 00
Group “B”: A1
Seeing it in Action: Step-by-Step Worked Examples
Let’s put the theory of pattern pointers, bumping along, and backtracking into practice. Here is exactly how the RegEx engine steps through the three conceptual examples we discussed earlier.

Worked Example 1: The “Cat” Problem
The Goal: Find the distinct word “cat” or “cats” (case-insensitive), ignoring words where “cat” is just a substring. The Regex: \b[Cc][Aa][Tt][Ss]?\b (Note: \b is a “word boundary” anchor. It matches the invisible position between a word character and a non-word character, like a space or punctuation).

The Input String: "cats catalog cat"

Step-by-Step Execution:

Index 0 (c in “cats”):
The pattern pointer starts at \b. Since c is the start of a word (a transition from the start of the string to a word character), the \b assertion passes (zero characters consumed).
[Cc] matches c.
[Aa] matches a.
[Tt] matches t.
[Ss]? looks for an optional ‘s’. It finds s and matches it.
\b checks for a word boundary at the current position (between ‘s’ and the space). Because ‘s’ is a word character and the following space is a non-word character, the boundary assertion passes. Match successful!
Match 1 Saved: "cats"
Resuming at Index 4 (the space):
The engine resumes exactly where it left off to look for more matches.
\b matches the boundary. [Cc] fails against the space. The engine bumps along.
Index 5 (c in “catalog”):
\b matches. [Cc] matches c. [Aa] matches a. [Tt] matches t.
The string pointer is now positioned between the t and the a in “catalog”.
The pattern asks for [Ss]?. Is ‘a’ an ‘s’? No. Since the ‘s’ is optional (?), the engine says “That’s fine, I matched it 0 times”, and moves to the next pattern token.
The pattern asks for \b (a word boundary). The string pointer is currently between t (a word character) and a (another word character). Because there is no transition to a non-word character, the boundary assertion fails.
Match Fails! The engine drops everything, resets the pattern, and bumps along to the next letter.
Index 13 (c in “cat”):
The engine bumps along through “atalog “ until it hits the final word.
\b matches. [Cc] matches c. [Aa] matches a. [Tt] matches t.
[Ss]? looks for an ‘s’. The string is at the end. It matches 0 times.
\b looks for a boundary. The end of the string counts as a boundary. Match successful!
Match 2 Saved: "cat"
Worked Example 2: The Phone Number Problem
The Goal: Extract a uniquely formatted phone number from a string. The Regex: (\(\d{3}\)|\d{3})[- .]?\d{3}[- .]?\d{4}

The Input String: "Call (123) 456-7890 now"

Step-by-Step Execution:

The engine starts at C. The first alternative \(\d{3}\) needs a literal (, so C fails. The second alternative \d{3} needs a digit, so C also fails. Bump along.
It bumps along through “Call “ until it reaches index 5: (.
Index 5 (():
The engine tries the first alternative in the group: \(\d{3}\).
\( matches the (. (Consumed).
\d{3} matches 123. (Consumed).
\) matches the ). (Consumed).
[- .]? looks for an optional space, dash, or dot. It finds the space after the parenthesis and matches it. (Consumed).
\d{3} matches 456. (Consumed).
[- .]? finds the - and matches it. (Consumed).
\d{4} matches 7890. (Consumed).
The pattern is fully satisfied.
Match Saved: "(123) 456-7890"
Worked Example 3: The Server Log (with Backtracking)
The Goal: Extract the IP address from a specific error line. The Regex: ^.*ERROR.*IP: (?P<IP>\d{1,3}(\.\d{1,3}){3}).*Critical Timeout$ (Note: We use .* to skip over irrelevant parts of the log).

The Input String: [14:32:05] ERROR - IP: 10.0.4.19 - Status: Critical Timeout

Step-by-Step Execution:

Start of String: ^ asserts we are at the beginning.
The .*: The pattern token .* tells the engine to match everything. The engine consumes the entire string all the way to the end: [14:32:05] ERROR - IP: 10.0.4.19 - Status: Critical Timeout.
Hitting a Wall: The next pattern token is the literal word ERROR. But the string pointer is at the absolute end of the line. The match fails.
Backtracking: The engine steps the string pointer backward one character at a time. It gives back t, then u, then o… all the way back until it gives back the space right before the word ERROR.
Moving Forward: Now that the .* has settled for matching [14:32:05] , the engine moves to the next token.
ERROR matches ERROR.
The next .* consumes the rest of the string again.
It has to backtrack again until it finds IP: .
The Named Group: The engine enters the named group (?P<IP>...).
\d{1,3} matches 10.
(\.\d{1,3}){3} matches .0, then matches .4, then matches .19.
The engine saves the string "10.0.4.19" into a variable named “IP”.
The Final Stretch: The final .* consumes the rest of the string again, backtracking until it can match the literal phrase Critical Timeout.
$ asserts the end of the string.
Match Saved! The group “IP” successfully holds "10.0.4.19".
Advanced
Advanced Pattern Control: Greediness vs. Laziness
Once you understand the basics of matching characters and using quantifiers, you will inevitably run into scenarios where your regular expression matches too much text. To solve this problem, we use Lazy Quantifiers.

By default, regular expression quantifiers (*, +, {n,m}) are greedy. This means they will consume as many characters as mathematically possible while still allowing the overall pattern to match.

The Greedy Problem: Imagine you are trying to extract the text from inside an HTML tag: <div>Hello World</div>. You might write the pattern: <.*>

Because .* is greedy, the engine sees the first < and then the .* swallows the entire rest of the string. It then backtracks just enough to find the final > at the very end of the string. Instead of matching just <div>, your greedy regex matched the entire string: <div>Hello World</div>.

The Lazy Solution (Non-Greedy): To make a quantifier lazy (meaning it will match as few characters as possible), you simply append a question mark ? immediately after the quantifier.

*? : Matches 0 or more times, but as few times as possible.
+? : Matches 1 or more times, but as few times as possible.
If we change our pattern to <div>(.*?)</div>, the engine matches the tags and captures only the text inside. Running this against <div>Hello World</div> will successfully yield a match where the first group is exactly “Hello World”.

Advanced Pattern Control: Lookarounds
Sometimes you need to assert that a specific pattern exists (or doesn’t exist) immediately before or after your current position, but you don’t want to include those characters in your final match result. To solve this problem, we use Lookarounds.

Lookarounds are “zero-width assertions”. Like anchors (^ and $), they check a condition at a specific position, but they do not “consume” any characters. The engine’s pointer stays exactly where it is.

Positive and Negative Lookaheads
Lookaheads look forward in the string from the current position.

Positive Lookahead (?=...): Asserts that what immediately follows matches the pattern.
Negative Lookahead (?!...): Asserts that what immediately follows does not match the pattern.
Example: The Password Condition Lookaheads are the secret to writing complex password validators. Suppose a password must contain at least one number. You can use a positive lookahead at the very start of the string: ^(?=.*\d)[A-Za-z\d]{8,}$

^ asserts the position at the beginning of the string.
(?=.*\d) looks ahead through the string from the current position. If it finds a digit, the condition passes. Crucially, because lookaheads are zero-width, they do not consume characters. After the check passes, the engine’s string pointer resets back to the exact position where the lookahead started (which, in this specific case, is still the beginning of the string).
[A-Za-z\d]{8,}$ then evaluates the string normally from that starting position to ensure it consists of 8+ valid characters.
Positive and Negative Lookbehinds
Lookbehinds look backward in the string from the current position.

Positive Lookbehind (?<=...): Asserts that what immediately precedes matches the pattern.
Negative Lookbehind (?<!...): Asserts that what immediately precedes does not match the pattern.
Example: Extracting Prices Suppose you have the text: I paid $100 for the shoes and €80 for the jacket. You want to extract the number 100, but only if it is a price in dollars (preceded by a $).

If you use \$\d+, your match will be $100. But you only want the number itself! By using a positive lookbehind, you can check for the dollar sign without consuming it: (?<=\$)\d+

The engine reaches a position in the string.
It peeks backward to see if there is a $.
If true, it then attempts to match the \d+ portion. The match is exactly 100.
By mastering lazy quantifiers and lookarounds, you transition from simply searching for text to writing highly precise, surgical data-extraction algorithms!

How the RegEx Engine Finds All Matches: Under the Hood
To truly master Regular Expressions, it helps to understand exactly what the computer is doing behind the scenes. When you run a regex against a string, you are handing your pattern over to a RegEx Engine—a specialized piece of software (typically built using a theoretical concept called a Finite State Machine) that parses your text.

Here is the step-by-step breakdown of how the engine evaluates an input string to find every possible match.

The Two “Pointers”
Imagine the engine has two pointers (or fingers) tracing the text:

The Pattern Pointer: Points to the current character/token in your RegEx pattern.
The String Pointer: Points to the current character in your input text.
The engine always starts with both pointers at the very beginning (index 0) of their respective strings. It processes the text strictly from left to right.

Attempting a Match and “Consuming” Characters
The engine looks at the first token in your pattern and checks if it matches the character at the string pointer.

If it matches, the engine consumes that character. Both pointers move one step to the right.
If a quantifier like + or * is used, the engine will act greedily by default. It will consume as many matching characters as possible before moving to the next token in the pattern.
Hitting a Wall: Backtracking
What happens if the engine makes a choice (like matching a greedy .*), moves forward, and suddenly realizes the rest of the pattern doesn’t match? It doesn’t just give up.

Instead, the engine performs Backtracking. It remembers previous decision points—places where it could have made a different choice (like matching one fewer character). It physically moves the string pointer backwards step-by-step, trying alternative paths until it either finds a successful match for the entire pattern or exhausts all possibilities.

The “Bump-Along” (Failing and Retrying)
If the engine exhausts all possibilities at the current starting position and completely fails to find a match, it performs a “bump-along”.

It resets the pattern pointer to the beginning of your RegEx, advances the string pointer one character forward from where the last attempt began, and starts the entire process over again. It will continue this process, checking every single starting index of the string, until it finds a match or reaches the end of the text.

Finding All Matches (Global Search)
Usually, a RegEx engine stops the moment it finds the first valid match. However, if you instruct the engine to find all matches (usually done by appending a global modifier, like /g in JavaScript or using re.findall() in Python), the engine performs a specific sequence:

It finds the first successful match.
It saves that match to return to you.
It resumes the search starting at the exact character index where the previous match ended.
It repeats the evaluate-bump-match cycle until the string pointer reaches the absolute end of the input string.
An Example in Action: Let’s say you are searching for the pattern cat in the string "The cat and the catalog".

The engine starts at T. T is not c. It bumps along.
It eventually bumps along to the c in "cat". c matches c, a matches a, t matches t. Match #1 found!
The engine saves "cat" and moves its string pointer to the space immediately following it.
It continues bumping along until it hits the c in "catalog".
It matches c, a, and t. Match #2 found!
It resumes at the a in "catalog", bumps along to the end of the string, finds nothing else, and completes the search.
By mechanically stepping forward, backtracking when stuck, and resuming immediately after success, the engine guarantees no potential match is left behind!

Limitations of RegEx: The HTML Problem
As powerful as RegEx is, it has mathematical limitations. The “regular expressions” of formal language theory map cleanly to Finite Automata (state machines), which match exactly the regular languages. Most modern engines (PCRE, Python’s re, Java, JavaScript, Ruby, .NET) actually use backtracking NFA implementations that add features like backreferences and lookarounds — these go beyond pure finite automata, but at the cost of worst-case exponential matching time. DFA-based engines like RE2 and grep (without -P) stay closer to the theoretical foundation and guarantee linear-time matching.

Because Finite Automata have no “memory” to keep track of deeply nested structures, you cannot write a general regular expression to perfectly parse HTML or XML.

HTML allows for infinitely nested tags (e.g., <div><div><span></span></div></div>). A regular expression cannot inherently count opening and closing brackets to ensure they are perfectly balanced. Attempting to use RegEx to parse raw HTML often results in brittle code full of false positives and false negatives. For tree-like structures, you should always use a dedicated parser (like BeautifulSoup in Python or the DOM parser in JavaScript) instead of RegEx.

Conclusion
Regular Expressions might look intimidating, but they are incredibly logical once you break them down into their component parts. By mastering anchors, character classes, quantifiers, and groups, you can drastically reduce the amount of code you write for data validation and text manipulation. Start small, practice in online tools like Regex101, and slowly incorporate them into your daily software development workflow!


This hands-on tutorial will walk you through Regular Expressions step by step. Each section builds on the last. Complete exercises to unlock your progress. Don’t worry about memorizing everything — focus on understanding the patterns.

Regular expressions look intimidating at first — that’s completely normal. Even experienced developers regularly look up regex syntax. The key is to break patterns into small, logical pieces. By the end of this tutorial, you’ll be able to read and write patterns that would have looked like gibberish an hour ago. If you get stuck, that means you’re learning — every programmer has been exactly where you are.

Three exercise types appear throughout:

Build it (Parsons): drag and drop regex fragments into the correct order.
Write it (Free): type a regex from scratch.
Fix it (Fixer Upper): a broken regex is given — debug and repair it.
Your progress is saved in your browser automatically.

Literal Matching
The simplest regex is just the text you want to find. The pattern cat matches the exact characters c, a, t — in that order, wherever they appear. This means it matches inside words too: cat appears in “education” and “scatter”.

Key points:

RegEx is case-sensitive by default: cat does not match “Cat” or “CAT”.
The engine scans left-to-right, reporting every non-overlapping match.
Build Your First Pattern
Arrange the fragments to build a regex that matches print in the text.

Sample Text
The print function can print any value. To pretty-print JSON, use json.dumps. Don't forget to add a print statement for debugging — printf is different.
Choose fragments for the answer box. Press, tap, or drag a fragment to move it between the bank and the answer.
/
/g
Clear
Test Cases
● print this — contains "print"
● sprint away — "print" inside "sprint"
● no match here — no "print"
● a priori — has "pri" but not "print"
Check Answer
Skip →
Your Turn
Type a regex to match every occurrence of error in the text.

Sample Text
System log: error in module A. No error found in module B. Warning: terror alert is not an error. Error handling improved.
/
type your regex here
/g
Test Cases
● error occurred — contains "error"
● all clear — no "error"
● Error — capital E — should NOT match (case sensitive)
Check Answer
Skip →
Character Classes
A character class [...] matches any single character listed inside the brackets. For example, [aeiou] matches any one lowercase vowel.

You can also use ranges: [a-z] matches any lowercase letter, [0-9] matches any digit, and [A-Za-z] matches any letter regardless of case.

To negate a class, place ^ right after the opening bracket: [^a-z] matches any character that is not a lowercase letter — digits, punctuation, spaces, etc.

Build a Vowel Matcher
Arrange fragments to match any lowercase vowel.

Sample Text
Regular expressions give programmers superpowers for text manipulation and data extraction.
Choose fragments for the answer box. Press, tap, or drag a fragment to move it between the bank and the answer.
/
/g
Clear
Test Cases
● hello — has vowels
● xyz — no vowels
● HELLO — uppercase — should NOT match
Check Answer
Skip →
Not a Letter
Match every character that is not a letter. Use [^...] to negate.

Sample Text
Score: 42 points! Time remaining: 3:30. Player #7 wins $100.
/
type your regex here
/g
Test Cases
● 123 — all digits — should match
● abc — all letters — should NOT match
● hi! — has punctuation — should match
Check Answer
Skip →
Meta Characters
Writing out full character classes every time gets tedious. RegEx provides meta character escape sequences:

Meta Characters table
meta character	Meaning	Equivalent Class
\d	Any digit	[0-9]
\D	Any non-digit	[^0-9]
\w	Any “word” character	[a-zA-Z0-9_]
\W	Any non-word character	[^a-zA-Z0-9_]
\s	Any whitespace	[ \t\n\r\f]
\S	Any non-whitespace	[^ \t\n\r\f]
The dot . is a wildcard that matches any single character (except newline). Because the dot matches almost everything, it is powerful but easy to overuse. When you actually need to match a literal period, escape it: \.

Digit Detector
Match every individual digit.

Sample Text
Invoice #8842: 3 items at $15 each, total $45. Tax ID: 9021-XB. Ref code: A1B2C3.
/
type your regex here
/g
Test Cases
● Room 101 — contains digits
● hello — no digits
● 42! — digits with punctuation
Check Answer
Skip →
File Extensions
Match file extensions: a literal dot followed by one or more lowercase letters. The dot . is a wildcard — escape it as \. to match a real dot.

Sample Text
Files: report.pdf, data.csv, photo.jpg, README, notes.txt, archive.tar.gz, config.yaml
/
type your regex here
/g
Test Cases
● .txt — dot + letters
● txt — no dot — should NOT match
● .a — single-letter extension
Check Answer
Skip →
Anchors
Before reading this section, try the first exercise below. Use what you already know to write a regex that matches only if the entire string is digits. You’ll discover a gap in your toolkit — that’s the point!

So far every pattern matches anywhere inside a string. Anchors constrain where a match can occur without consuming characters:

Anchors table
Anchor	Meaning
^	Start of string (or line in multiline mode)
$	End of string (or line in multiline mode)
\b	Word boundary — the point between a “word” character (\w) and a “non-word” character (\W), or vice versa
Anchors are critical for validation. Without them, the pattern \d+ would match the 42 inside "hello42world". Adding anchors — ^\d+$ — ensures the entire string must be digits.

Word boundaries (\b) let you match whole words. \bgo\b matches the standalone word “go” but not “goal” or “cargo”.

The Challenge (Try Before You Learn!)
Can you write a regex that matches only if the entire string is digits? Try \d+ — does it work? It shouldn't pass all the tests. Read the section above to discover why, then fix your answer.

/
type your regex here
/g
Test Cases
● 12345 — all digits — should match
● abc — all letters — should NOT match
● 123abc — mixed — should NOT match
● "" — empty — should NOT match
Check Answer
Skip →
Build a Full-String Validator
Arrange fragments so the regex matches only if the entire string is digits. Use ^ (start) and $ (end) anchors.

Choose fragments for the answer box. Press, tap, or drag a fragment to move it between the bank and the answer.
/
/g
Clear
Test Cases
● 12345 — all digits
● 123abc — mixed — should NOT match
● "" — empty — should NOT match
● 007 — leading zeros OK
Check Answer
Skip →
Stand-Alone Words
Match only the standalone word go — not inside "goal" or "cargo". Use word boundaries \b.

Sample Text
Ready, set, go! The goal is to outperform the algorithm. Let's go before the cargo ship departs. Go ahead.
/
type your regex here
/g
Test Cases
● let's go — "go" as a word
● goal — "goal" — should NOT match
● cargo — "cargo" — should NOT match
How the Engine Processes \bgo\b
Word Boundary Matching
Regex:\bgo\b
String:go! goal cargo go
Press Step or Play to begin.
↺ Reset
← Back
Step →
▶ Play
0 / 9
Check Answer
Skip →
Fix the Username Validator
This regex is supposed to validate that a username is only alphanumeric characters. But it incorrectly accepts admin!@#. Fix it!

Broken regex (edit to fix):
/
[a-zA-Z0-9]+
/g
Why is it broken?
Test Cases
✓ admin — valid username
✓ user123 — alphanumeric
✗ admin!@# — special chars — should NOT match
✓ "" — empty — should NOT match
Check Answer
Skip →
Quantifiers
Quantifiers control how many times the preceding element must appear:

Quantifiers table
Quantifier	Meaning
*	Zero or more times
+	One or more times
?	Zero or one time (optional)
{n}	Exactly n times
{n,}	n or more times
{n,m}	Between n and m times
Common misconception: * vs +

Students frequently confuse these two. The key difference:

a*b matches b, ab, aab, aaab, … — the a is optional (zero or more).
a+b matches ab, aab, aaab, … — at least one a is required.
If you want “one or more”, reach for +. If you genuinely mean “zero or more”, use *. Getting this wrong is one of the most common sources of regex bugs.

ZIP Code Spotter
Match numbers that are exactly 5 digits — not shorter, not longer. Combine \b, \d, and {n}.

Sample Text
Locations: New York 10001, Los Angeles 90210, Chicago 60601, apt 42, serial 1234567, code 999.
/
type your regex here
/g
Test Cases
● 90210 — exactly 5 digits
● 123 — 3 digits — too short
● 1234567 — 7 digits — too long
Check Answer
Skip →
Star vs. Plus
Match strings that start with one or more a followed by a b. Notice: a*b would also match a lone b — but a+b requires at least one a.

Sample Text
Test: ab, aab, aaab, b, xb, aaa, aaaab.
/
type your regex here
/g
Test Cases
● ab — one a + b
● aaab — multiple a's + b
● b — lone b — should NOT match (need at least one a)
● aaa — no b — should NOT match
Check Answer
Skip →
Singular or Plural
Match both file and files — the trailing s should be optional.

Sample Text
Upload your file here. Multiple files are supported. The file manager shows all files in the current directory.
/
type your regex here
/g
Test Cases
● file — singular
● files — plural
● fil — too short — should NOT match
Check Answer
Skip →
Alternation & Combining
The pipe | works like a logical OR: cat|dog matches either “cat” or “dog”. Alternation has low precedence, so gray|grey matches the full words — you don’t need parentheses for simple cases.

When you combine multiple regex features, patterns become expressive:

gr[ae]y — character class for the spelling variant.
\d{2}:\d{2} — two digits, a colon, two digits (time format).
^(0[1-9]|1[0-2])/(0[1-9]|[12]\d|3[01])$ — a month/day format validator. (It accepts impossible combinations like 02/30 and 04/31; properly validating month-specific day limits — let alone leap years — is beyond what regex alone can express, and is one of the classic limits of regex pattern matching.)
Start simple and add complexity only when tests demand it.

Spelling Variants
Match both grey and gray.

Sample Text
The grey sky turned dark gray by evening. Is it grey or gray? The greyhound ran across the gravel path, its grey fur blending into gray fog.
/
type your regex here
/g
Test Cases
● grey — British spelling
● gray — American spelling
● gravy — "gravy" — should NOT match
Check Answer
Skip →
Time Format
Match times in HH:MM format — exactly two digits, a colon, two digits.

Sample Text
Schedule: standup at 09:30, lunch at 12:00, review at 15:45. Note: 9:5 is not valid format.
/
type your regex here
/g
Test Cases
● 09:30 — valid time
● 12:00 — noon
● 9:30 — single-digit hour — should NOT match
Check Answer
Skip →
Fix the Date Validator
This regex is supposed to validate dates in MM/DD format. But it accepts 99/99 and rejects nothing. Debug it — the month should be 01–12 and the day 01–31.

Broken regex (edit to fix):
/
\d{2}/\d{2}
/g
Why is it broken?
Test Cases
✓ 03/15 — valid date
✓ 12/25 — December 25
✗ 99/99 — invalid — should NOT match
✗ 00/15 — month 00 — should NOT match
Check Answer
Skip →
You’ve completed the basics! You now know how to match literal text, use character classes, metacharacters, anchors, quantifiers, and alternation.

Ready for more? Continue to the Advanced RegEx Tutorial to learn greedy vs. lazy matching, groups, lookaheads, and tackle integration challenges.

This is the second part of the Interactive RegEx Tutorial. If you haven’t completed the Basics Tutorial yet, start there first — the exercises here assume you’re comfortable with literal matching, character classes, metacharacters, anchors, quantifiers, and alternation.

Warm-Up Review
Before diving into advanced features, let’s make sure the basics are solid. These exercises combine concepts from the Basics tutorial. If any feel rusty, revisit the Basics.

Review: Standalone Numbers
Match standalone integers (one or more digits) that are NOT part of a word. Combine \b, \d, and +.

Sample Text
Scores: 42 points, code A7 rejected, player 3 scored 100, item99 skipped.
/
type your regex here
/g
Test Cases
● 42 — standalone number
● A7 — letter+digit — should NOT match whole thing
● 100 — another standalone number
Check Answer
Skip →
Review: Simple Email Shape
Match strings that look like simple email addresses: one or more word characters, an @, one or more word characters, a dot, then one or more letters. Validate the entire string.

/
type your regex here
/g
Test Cases
● user@example.com — valid email shape
● test@host.org — another valid shape
● no-at-sign.com — missing @ — should NOT match
● @host.com — missing username — should NOT match
Check Answer
Skip →
Review: Fix the Year Matcher
This regex should match exactly 4-digit years (like 2024) as standalone numbers. But it matches "20" inside "2024" and accepts "12345". Fix it.

Broken regex (edit to fix):
/
\d+
/g
Why is it broken?
Test Cases
✓ 2024 — 4-digit year
✓ 1999 — another year
✗ 12345 — 5 digits — should NOT match
✗ 99 — 2 digits — should NOT match
Check Answer
Skip →
Greedy vs. Lazy
By default, quantifiers are greedy — they match as much text as possible. This often surprises beginners.

Consider matching HTML tags with <.*> against the string <b>bold</b>:

Greedy <.*> matches <b>bold</b> — the entire string! The .* gobbles everything up, then backtracks just enough to find the last >.
Lazy <.*?> matches <b> and then </b> separately. Adding ? after the quantifier makes it match as little as possible.
The lazy versions: *?, +?, ??, {n,m}?

Use the step-through visualizer in the first exercise below to see exactly how the engine behaves differently in each mode.

Tag Trouble
Match each individual HTML tag (like <b> or </b>) — not the entire string. Try <.*> first to see the greedy problem, then add ? to make it lazy.

Sample Text
<b>bold</b> and <i>italic</i> text
/
type your regex here
/g
Test Cases
● <b> — opening tag
● </b> — closing tag
● <b>bold</b> — two separate tags, not one big match
● bold — plain text — should NOT match
How the Engine Processes <.*> vs <.*?>
Greedy: <.*>
Regex:<.*>
String:<b>bold</b>
Press Step or Play to begin.
↺ Reset
← Back
Step →
▶ Play
0 / 6
Lazy: <.*?>
Regex:<.*?>
String:<b>bold</b>
Press Step or Play to begin.
↺ Reset
← Back
Step →
▶ Play
0 / 6
Check Answer
Skip →
Quoted Strings
Match each individual double-quoted string separately. Use a lazy quantifier.

Sample Text
He said "hello" and she replied "goodbye" before they both whispered "see you later" softly.
/
type your regex here
/g
Test Cases
● "hello" — quoted string
● "hello" and "goodbye" — two quoted strings matched separately
● no quotes here — no quotes — should NOT match
Check Answer
Skip →
Groups & Named Groups
Parentheses (...) create a group — they treat multiple characters as a single unit for quantifiers. (na){2,} means “the sequence na repeated 2 or more times” — matching nana, nanana, etc. You can access what each group matched by index (e.g., match[1]).

Named groups let you label what each group matches instead of counting parentheses:

Groups & Named Groups table
Syntax	Meaning
(?<name>...)	Create a group called name
match.groups.name	Retrieve the matched value in code
For example, ^(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})$ matches a date and lets you access match.groups.year, match.groups.month, and match.groups.day directly — much clearer than match[1], match[2], match[3].

Repeated Syllables
Match the syllable na repeated 2 or more times in a row. Use a group (...) with a quantifier.

Sample Text
The crowd chanted: na nana nanana nananana! A banana has na but also nan. Just na alone is not enough.
/
type your regex here
/g
Test Cases
● nana — 2 repetitions
● nanana — 3 repetitions
● na — only 1 — should NOT match
Check Answer
Skip →
Fix the Repeater
This regex tries to match 3-letter airport codes (like LAX, JFK) but it incorrectly matches LA and ABCD. Fix it.

Broken regex (edit to fix):
/
[A-Z]+
/g
Why is it broken?
Test Cases
✓ LAX — 3 uppercase letters
✓ JFK — another valid code
✗ LA — only 2 letters — should NOT match
✗ ABCD — 4 letters — should NOT match
Check Answer
Skip →
Named Log Parser
Build a regex to parse log entries like [ERROR] 404 Not Found. Capture the level (e.g., ERROR) as (?<level>...) and the status code as (?<code>...).

Sample Text
[ERROR] 404 Not Found [WARN] 301 Moved [INFO] 200 OK
Choose fragments for the answer box. Press, tap, or drag a fragment to move it between the bank and the answer.
/
/g
Clear
Test Cases
● [ERROR] 404 Not Found — "level" captures "ERROR", "code" captures "404"
● [WARN] 301 Moved — "level" captures "WARN", "code" captures "301"
● ERROR 404 — missing brackets — should NOT match
Check Answer
Skip →
Named Email Parts
Match a simple email address and extract the username and domain using named groups: (?<user>...) and (?<domain>...). Username is one or more word characters, domain is one or more word characters, a dot, then one or more letters. Validate the entire string.

/
type your regex here
/g
Test Cases
● alice@example.com — "user" captures "alice", "domain" captures "example.com"
● bob@host.org — "user" captures "bob", "domain" captures "host.org"
● no-at-sign.com — missing @ — should NOT match
● @host.com — missing username — should NOT match
Check Answer
Skip →
Named Date Parts
Match dates in YYYY-MM-DD format using named groups: (?<year>...), (?<month>...), and (?<day>...). Year is 4 digits, month and day are each 2 digits, separated by hyphens. Validate the entire string.

/
type your regex here
/g
Test Cases
● 2026-04-01 — groups capture year, month, day
● 1999-12-31 — another valid date
● 26-04-01 — 2-digit year — should NOT match
● 2026/04/01 — wrong separator — should NOT match
Check Answer
Skip →
Lookaheads & Lookbehinds
Lookaround assertions check what comes before or after the current position without including it in the match. They are “zero-width” — they don’t consume characters.

Lookaheads & Lookbehinds table
Syntax	Name	Meaning
(?=...)	Positive lookahead	What follows must match ...
(?!...)	Negative lookahead	What follows must NOT match ...
(?<=...)	Positive lookbehind	What precedes must match ...
(?<!...)	Negative lookbehind	What precedes must NOT match ...
A classic use case: password validation. To require at least one digit AND one uppercase letter, you can chain lookaheads at the start: ^(?=.*\d)(?=.*[A-Z]).+$. Each lookahead checks a condition independently, and the .+ at the end actually consumes the string.

Lookbehinds are useful for extracting values after a known prefix — like capturing dollar amounts after a $ sign without including the $ itself.

Dollar Amounts
Match the numeric amount after a $ sign — but do NOT include the $ in the match. Hint: Use a positive lookbehind.

Sample Text
Prices: $25, €30, $100, £50, $7.99, ¥500, $0.50, and €12.50 have been on sale for 30 days.
/
type your regex here
/g
Test Cases
● $25 — match "25" only — not "$25"
● €30 — euro — should NOT match
● £50 — pound — should NOT match
Check Answer
Skip →
Password Check
Validate that the entire string has at least one digit and at least one uppercase letter. Hint: Use positive lookaheads

/
type your regex here
/g
Test Cases
● Hello1 — uppercase + digit
● hello1 — no uppercase — should NOT match
● HELLO — no digit — should NOT match
● H1 — minimal valid
Check Answer
Skip →
Putting It All Together
You’ve learned every major regex feature. The real skill is knowing which tools to combine for a given problem. These exercises don’t tell you which section to draw from — you’ll need to decide which combination of character classes, anchors, quantifiers, groups, and lookarounds to use.

This is where regex goes from “I can follow along” to “I can solve problems on my own”.

CSS Hex Color
Validate a CSS hex color code: a # followed by exactly 3 or 6 hex digits (0-9, a-f, A-F). The entire string must match. Combine: anchors, character classes, quantifiers, alternation, and grouping.

/
type your regex here
/g
Test Cases
● #FFF — 3-digit hex
● #1A2B3C — 6-digit hex
● #GGG — invalid hex chars — should NOT match
● #12 — only 2 digits — should NOT match
● 123456 — missing # — should NOT match
Check Answer
Skip →
Student ID Validator
Validate a student ID: exactly one uppercase letter followed by exactly 9 digits (e.g., A123456789). The entire string must match. Combine: anchors, character classes, and quantifiers.

/
type your regex here
/g
Test Cases
● A123456789 — valid ID
● B000000001 — another valid ID
● a123456789 — lowercase letter — should NOT match
● AB12345678 — two letters — should NOT match
● A12345 — too few digits — should NOT match
Check Answer
Skip →
Extract Prices (Dollar Only)
Match dollar amounts like $19.99 or $5 in a string. The match should include the $, one or more digits, and an optional decimal part (dot + exactly 2 digits). Combine: escaping, quantifiers, grouping, and the ? quantifier.

Sample Text
Items: $19.99 widget, $5 sticker, €12.50 imported, $100.00 premium, $0.99 candy, 50 cents.
/
type your regex here
/g
Test Cases
● $19.99 — dollars and cents
● $5 — whole dollars
● €12.50 — euro — should NOT match
● 50 — plain number — should NOT match
Check Answer
Skip →
Fix the Log Extractor
This regex tries to extract the word after "ERROR:" in log lines (e.g., "ERROR: timeout" should match "timeout"). But it matches the entire line instead. Fix it so it only matches the first word after "ERROR: ".

Broken regex (edit to fix):
/
ERROR: .+
/g
Why is it broken?
Test Cases
✗ ERROR: timeout — match "timeout" only — not "ERROR: timeout"
✗ ERROR: connection refused — match first word only
✓ INFO: all good — INFO line — should NOT match
✓ WARNING: low memory — WARNING line — should NOT match
Check Answer
Skip →