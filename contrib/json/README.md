# contrib/json — JSON parser / writer

JSON parse / serialize library written in Mere. Built on stdlib (`str_*` /
`is_digit` / `try_or` / `fail` / `StrBuf`) + recursive variant + pattern
matching with zero external dependencies.

## Files

| file | export | lines |
|---|---|---|
| `json.mere` | `module Json { type json = JNull \| JBool \| JNum \| JStr \| JArr \| JObj; parse_json: str -> json }` | ~180 |
| `writer.mere` | `type json` (top-level) + `module JsonWriter { to_json_str, to_pretty_str }` | ~135 |

## Usage (before pkg manager lands)

```mere
// Bring in via import (works since Phase 9.5)
import "contrib/json/json.mere";

let v = Json.parse_json "[1, 2, 3]" in
match v with
| Json.JArr xs -> ...
| Json.JNull -> ...
| _ -> ...
```

Or **copy-paste** into a project:

```sh
cp contrib/json/json.mere    my_project/
cp contrib/json/writer.mere  my_project/
```

The self-test block at the end of each file (`run_case` demo, `let doc = …`,
etc.) may be removed in real use.

`writer.mere` was wrapped in `module JsonWriter { ... }` in Phase 43. However,
`type json` is kept **outside the module** (it can't coexist with the parser's
`module Json { type json = ... }` inside a single file, but each file works
independently). To round-trip parser + writer in one program, the user must
avoid `type json` collision (using either parser or writer only is the expected
mode for now).

## Coverage

- atoms: `null` / `true` / `false` / int (negative OK) / string
- composite: array / object
- escape: `\"` `\\` `\n` `\t` `\r` `\/` decoded via `str_unescape`
- **Unsupported** (extension driven by issues): float / unicode `\uXXXX` / exponential notation

## Known gotchas

- **String literal containing `{`**: Phase 36 string interpolation treats `"{"`
  as an interpolation start, so escape it with `"\{"` (the json.mere /
  writer.mere demos already apply this workaround)
- **The name `case` collides with a C reserved word** in C codegen
  (libc/C keyword) — this lib renames its own test helper to `run_case`. See
  [docs/reserved-names.md](../../docs/reserved-names.md) for the full reserved-name list.

## Position

Stage 2 contrib (incubation). See lifecycle in [contrib/README.md](../README.md).
After public release + pkg manager lands, graduation target is the separate
repo `mere-json` (internal design notes §3.1).
