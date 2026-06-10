# Regular Expressions Cheatsheet (candidate items)

## Anchors
- `^` start of line/string
- `$` end of line/string
- `\b` word boundary
- `\B` non-word-boundary
- `\A` start of string (no multiline override)
- `\Z` end of string

## Character classes
- `.` any char (except newline by default)
- `\d` digit `[0-9]`
- `\D` non-digit
- `\w` word char `[A-Za-z0-9_]`
- `\W` non-word char
- `\s` whitespace
- `\S` non-whitespace
- `[abc]` any of a, b, c
- `[^abc]` NOT a, b, c
- `[a-z]` range
- `[A-Za-z0-9_]` same as \w

## Quantifiers (greedy by default)
- `*` 0 or more
- `+` 1 or more
- `?` 0 or 1
- `{n}` exactly n
- `{n,}` n or more
- `{n,m}` between n and m

## Lazy versions (minimum match)
- `*?` `+?` `??` `{n,m}?`

## Groups
- `(abc)` capture group
- `(?:abc)` non-capturing group
- `(?P<name>abc)` named group (Python)
- `\1 \2` backreferences in pattern
- `\1` or `$1` in replacement (depends on tool)

## Alternation
- `a|b` a OR b
- `(cat|dog|bird)` matches any of them

## Lookarounds (zero-width)
- `(?=...)` lookahead positive
- `(?!...)` lookahead negative
- `(?<=...)` lookbehind positive
- `(?<!...)` lookbehind negative

## Flags
- `i` case-insensitive
- `g` global (find all)
- `m` multiline (`^`/`$` match each line)
- `s` dotall (`.` matches newline)
- `x` extended (whitespace ignored)

## Escapes (special chars to match literally)
- `\. \* \+ \? \( \) \[ \] \{ \} \| \\ \^ \$`

## Common patterns
- Email-ish: `\b[\w.+-]+@[\w-]+\.[\w.-]+\b`
- Phone (US): `\(?\d{3}\)?[-\s.]?\d{3}[-\s.]?\d{4}`
- URL-ish: `https?://[^\s]+`
- IPv4: `\b\d{1,3}(\.\d{1,3}){3}\b`
- Hex color: `#[0-9a-fA-F]{6}`
- Whitespace-only line: `^\s*$`
- Word: `\b\w+\b`

## Tool-specific quick differences
- `grep` — basic regex; use `-E` for extended (`+`, `?`, `|`, `()` unescaped)
- `sed` — basic regex; `-E` for extended
- Python `re` — full PCRE-ish; default behavior
- JavaScript — `/pat/flags`; no lookbehind in old engines

## Greedy vs lazy example
- `a.*b` on `axbyb` → matches `axbyb` (greedy)
- `a.*?b` on `axbyb` → matches `axb` (lazy)

## Capture group in replace
- Python: `re.sub(r'(\w+) (\w+)', r'\2 \1', 'John Smith')` → `Smith John`
- sed: `sed -E 's/(\w+) (\w+)/\2 \1/'`

## Common traps
- `.` doesn't match newline by default — use `s` flag for that
- `+`, `?`, `|`, `()` need escaping in basic regex (grep/sed without `-E`)
- `^` inside `[...]` means NEGATION; outside means start-of-line
- Greedy quantifiers can match more than you expect — use lazy `?` versions

## Additional items (potentially missing)

### POSIX character classes (use inside [...])
- `[:alpha:]` letters
- `[:digit:]` digits
- `[:alnum:]` alphanumeric
- `[:space:]` whitespace
- `[:upper:]` uppercase
- `[:lower:]` lowercase
- `[:punct:]` punctuation
- `[:xdigit:]` hex digit
- Usage: `[[:alpha:][:digit:]]`

### Replacement special chars
- `\1 \2 ...` capture group references (sed, Python re.sub)
- `$1 $2 ...` capture groups (JS, some sed dialects)
- `\g<name>` named group reference (Python re.sub)
- `&` (in sed replacement) = whole match
- `\n` newline (in replacement)
- `\` literal backslash

### Greedy vs Lazy
- `.*` greedy (matches as much as possible)
- `.*?` lazy (matches as little as possible)
- Example on `<b>x</b><b>y</b>`:
  - `<b>.*</b>` → matches whole string
  - `<b>.*?</b>` → matches `<b>x</b>`

### Quantifier reference
| Greedy | Lazy | Possessive | Meaning |
|---|---|---|---|
| `*` | `*?` | `*+` | 0 or more |
| `+` | `+?` | `++` | 1 or more |
| `?` | `??` | `?+` | 0 or 1 |
| `{n,m}` | `{n,m}?` | `{n,m}+` | between n and m |

### Common multi-line / dotall pattern
- To match across newlines: use `s` flag (or `re.DOTALL` in Python)
- `re.compile(r'<.*?>', re.DOTALL | re.MULTILINE)`

### Useful Python re functions
- `re.match(pat, str)` — match at START only
- `re.search(pat, str)` — first match anywhere
- `re.findall(pat, str)` — list of all matches
- `re.finditer(pat, str)` — iterator of match objects
- `re.sub(pat, repl, str)` — substitute
- `re.split(pat, str)` — split on pattern
- `re.compile(pat, flags)` — compile for reuse

### Common gotcha — escaping in different layers
- In a Python string, `\d` works but for full safety use **raw strings**: `r'\d+'`
- In shell `grep -E '\b\w+\b'` — quote!

### Patterns to memorize
| Need | Pattern |
|---|---|
| Whole word | `\b\w+\b` |
| Email | `[\w.+-]+@[\w-]+\.[\w.-]+` |
| Hex color | `#[0-9a-fA-F]{6}` |
| HTML tag | `<(\w+)[^>]*>.*?</\1>` (with `s` flag) |
| Float | `-?\d+\.?\d*` or `-?\d*\.?\d+` |
| IPv4 octet | `(25[0-5]|2[0-4]\d|[01]?\d\d?)` |
