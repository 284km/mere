# contrib/regex — minimal regex matcher (Mere implementation)

An MVP of regex AST + backtracking matcher written in Mere. Namespaced as
`module Regex { ... }`. No NFA construction or Thompson's algorithm — straight
recursive matching.

## Files

| file | export | lines |
|---|---|---|
| `regex.mere` | `type regex` + `module Regex { parse_re: str -> regex; match_re: regex -> str -> bool }` | ~230 |
| `engine.mere` | More detailed engine prototype (top-level; module-wrapping is future work) | ~110 |

## Usage

```mere
import "contrib/regex/regex.mere";

let re = Regex.parse_re "^a.+z$" in
if Regex.match_re re "anything-then-z"
then print "matched"
else print "no match"
```

## Supported syntax (MVP)

| syntax | meaning |
|---|---|
| `c` | single-char literal (ASCII 1 byte) |
| `.` | any single char |
| `^` | start-of-line anchor |
| `$` | end-of-line anchor |
| `c*` | 0+ times (greedy) |
| `c+` | 1+ times (greedy) |
| `c?` | 0 or 1 time |
| `ab` | concatenation |

## Unsupported (issue-driven)

- groups `(...)`
- character classes `[a-z]`
- alternation `|`
- bounded quantifier `{n,m}`
- backreferences `\1`
- Unicode

## Position

Stage 2 contrib (incubation). See [contrib/README.md](../README.md).
Graduation target is `mere-regex` (separate repo, after pkg manager lands).
Not aiming for PCRE / RE2 compatibility from the start — growing it as a
"subset usable in everyday code".
