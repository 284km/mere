# contrib/test — minimal unit test framework

`assert_eq` / `assert_true` / `assert_false` + suite カウンタ + 失敗ログ集約。
`show` builtin の多相性を使い、 int / str / bool / variant / tuple すべて
同じ API で比較できる。 `module Test { ... }` で名前空間化。

## ファイル

| file | export | 行数 |
|---|---|---|
| `test.mere` | `module Test { new_suite, assert_eq, assert_true, assert_false, summary, exit_status }` | 約 90 行 |

## 使い方

```mere
import "contrib/test/test.mere";

let s = Test.new_suite () in
let _ = Test.assert_eq s "1 + 1 = 2"   2 (1 + 1) in
let _ = Test.assert_eq s "concat"      "hi" ("h" ++ "i") in
let _ = Test.assert_true s "5 > 3"     (5 > 3) in
let _ = Test.summary s in
Test.exit_status s   // 0 if all pass, 1 if any fail
```

出力例:
```
  ok | 1 + 1 = 2
  ok | concat
  ok | 5 > 3
FAIL | wrong value
--- 3/4 passed ---
FAIL | wrong value
  expected: 42
  actual:   41
```

## API

| fn | signature | 用途 |
|---|---|---|
| `new_suite` | `unit -> suite` | suite 初期化 (counters Map + StrBuf log) |
| `assert_eq` | `suite -> str -> 'a -> 'a -> unit` | `show x == show y` で比較 |
| `assert_true` | `suite -> str -> bool -> unit` | cond が true なら pass |
| `assert_false` | `suite -> str -> bool -> unit` | cond が false なら pass |
| `summary` | `suite -> unit` | pass/fail/total を print |
| `exit_status` | `suite -> int` | all pass = 0、 fail あり = 1 (CI 用) |

## backend サポート

| backend | 状態 |
|---|---|
| interp | ✓ |
| C | ✓ |
| LLVM | ✓ |
| Wasm | ✓ (Phase 43 で `ty_tag` に Map case を追加し、 closure env / tuple slot で Map を carrier として扱えるように) |

## 限定事項 (MVP)

- 多 instantiation: 同じ test program 内で `assert_eq` を異なる型 (int + str + bool) で複数回呼ぶのは OK (Phase 43 で DEFERRED §1.7 を fix、 chained poly inst も追従するように)
- test の grouping (describe/context) なし
- parallel execution なし (Mere に thread がない)
- setup/teardown hook なし
- `fail` を catch しない (user code が `fail` すると runner も落ちる) — 将来 `try_or` wrap を入れる

## 位置付け

stage 2 contrib (incubation)。 [contrib/README.md](../README.md) 参照。
graduation 先は `mere-test` (別 repo、 pkg manager 完成後)。 OCaml の Alcotest
や Rust の `cargo test` のような統合は将来扱い。

将来構想:
- snapshot test (`assert_matches_snapshot`)
- property-based test (QuickCheck 風)
- benchmark mode (`time` builtin で計測)
