# Reserved names — Mere top-level fn 名と C codegen の衝突

Mere の C codegen は **top-level fn を C 関数として直接 emit する**。 そのため、
top-level の `let` / `let rec` の binding 名が **macOS / Linux の libc / libm にすでに
存在する関数名** や **C 言語の予約語** と衝突すると、 codegen 段階では成功するが
`clang` / `gcc` で **compile error** になる。

interp / LLVM / Wasm の挙動は影響を受けない (= interp で動いた program が C
codegen で fail する) ため、 発見が遅れがち。 そこで **Phase 38.A3 の linter**
が parser 通過時点で warning を出す。 本ドキュメントはその全リストと回避策の
リファレンス。

> **TL;DR**: 下表のいずれかと衝突する名前を top-level に書くと、 codegen 時に
> warning + C compile error。 回避は **接頭辞 (`m_` / `mere_`) / 接尾辞 (`_` /
> `_v`) / 動詞句 (`run_*` / `*_list`)** のいずれか。

## 1. 衝突一覧 (約 110 名前)

実装は [`lib/pipeline.ml:42`](../lib/pipeline.ml) の `reserved_c_names` を
参照。 該当の名前を `let foo = ...` で定義すると Phase 38.A3 linter が
warning を出力する。

### 1.1 C 言語キーワード (約 30 個)

`short` / `long` / `int` / `char` / `float` / `double` / `signed` / `unsigned` /
`register` / `static` / `auto` / `extern` / `const` / `volatile` / `restrict` /
`inline` / `goto` / `return` / `break` / `continue` / `switch` / `case` /
`default` / `do` / `while` / `for` / `if` / `else` / `sizeof` / `typedef` /
`struct` / `union` / `enum` / `void`

最も hit しやすい: **`case`** (match arm の helper を書くと出やすい)、
**`default`** / **`type`** / **`return`** (DSL 風命名で出やすい)。

### 1.2 stdlib.h (libc 約 22 個)

`div` / `ldiv` / `exit` / `abort` / `atexit` / `atof` / `atoi` / `atol` /
`free` / `malloc` / `calloc` / `realloc` / `system` / `getenv` / `setenv` /
`putenv` / `unsetenv` / `rand` / `srand` / `abs` / `labs` / `qsort` /
`bsearch` / `mergesort`

最も hit しやすい: **`div`** (有理数 / 行列 / GCD 系で出やすい)、
**`rand`** (random 系 helper)、 **`abs`** (builtin がある絶対値関数を
shadowing したくなる)。

### 1.3 math.h (libm 約 17 個)

`pow` / `sqrt` / `sin` / `cos` / `tan` / `asin` / `acos` / `atan` / `atan2` /
`exp` / `log` / `log10` / `log2` / `ceil` / `floor` / `round` / `trunc` /
`fabs` / `fmod` / `hypot` / `sinh` / `cosh` / `tanh`

これらは Mere builtin としても同名で定義されている (見えるところでは `pow` /
`sqrt` 等)。 **user が同名 top-level を書くと shadow して衝突**。 回避は
`mere_pow` / `power_int` 等。

### 1.4 time.h (libc 約 9 個)

`time` / `clock` / `ctime` / `asctime` / `gmtime` / `localtime` / `mktime` /
`difftime` / `strftime`

最も hit しやすい: **`time`** (builtin がある時刻取得関数の shadow)。

### 1.5 POSIX I/O (約 18 個)

`read` / `write` / `open` / `close` / `lseek` / `stat` / `fstat` / `fopen` /
`fclose` / `fread` / `fwrite` / `fseek` / `ftell` / `rewind` / `printf` /
`scanf` / `fprintf` / `fscanf` / `sprintf` / `sscanf` / `puts` / `gets` /
`fputs` / `fgets` / `putchar` / `getchar`

最も hit しやすい: **`read`** / **`write`** (file I/O wrapper を書くと出る)、
**`printf`** (debug helper を書くと出る)。

### 1.6 misc libc (約 15 個)

`strlen` / `strcpy` / `strncpy` / `strcat` / `strncat` / `strcmp` / `strncmp` /
`strchr` / `strrchr` / `strstr` / `strdup` / `strerror` / `memcpy` / `memmove` /
`memset` / `memcmp` / `memchr` / **`main`**

`main` は特に注意 (実行 entry point として C が予約済)。 Mere の top-level
expression は自動的に `main` 関数を成すので、 user が `let main = ...` を
書くと **必ず衝突**。

## 2. 回避パターン (3 つ)

| パターン | 例 (衝突する名前 → 回避名) | 用途 |
|---|---|---|
| **接尾辞** | `case` → `case_` / `case_v` / `run_case`  | 1 文字付加で済む。 `case_v` は linter message が提示する例 (深い意味なし) |
| **接頭辞** | `div` → `divi` / `mere_div`、 `pow` → `power_int` / `m_pow` | 名前空間化のニュアンスが出る。 contrib lib は `mere_` 接頭辞推奨 |
| **動詞句** | `mergesort` → `sort_list`、 `pow` → `power_of` | 自然言語として最も読みやすい。 lib API には推奨 |

**推奨方針**:
- **個人 helper / 一時 fn**: 接尾辞 (`case_` / `_v`)
- **lib として公開する fn (contrib)**: 動詞句 (`run_case` / `power_of`)
- **module 化済 lib の内部 fn**: 接頭辞 (`m_div`) で module 系を示唆

## 3. 関連

- **linter 実装**: [`lib/pipeline.ml:42-82`](../lib/pipeline.ml) (Phase 38.A3)
- **patterns.md §5**: 摘要版 (本ドキュメントの圧縮)
- **language-reference.md**: Mere 言語予約語 (`let` / `fn` / `match` / `if` 等)
  は別 (parser 側で reject される、 binding 名にできない)

## 4. 将来拡張 (DEFERRED)

| stage | 内容 |
|---|---|
| A. 自動 rename suggestion | linter message が「`pow` だと `power`、 `case` だと `case_` を推奨」 のような名前固有 sugges を出す (現状は generic `_` / `m_` / `_v` 提案のみ) |
| B. namespace 化 | module 内なら同名 top-level も OK にする (現状は module rewrite 後の `M.case` も C 関数として emit されるので NG) |

これらは公開後 issue 駆動で。
