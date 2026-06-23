# contrib/ — incubating libraries

このディレクトリには **`examples/` より一段「lib」 寄り** の Mere コードを置く。
すなわち、 「単体で挙動を見る demo」 ではなく **「他の Mere program に組み込んで
使う前提の機能」**。

## 位置付け (3 段 lifecycle)

| stage | 場所 | 性質 |
|---|---|---|
| 1. example | `examples/foo.mere` | 単体実行で挙動を見る demo |
| **2. contrib (incubation)** | `contrib/foo/` | **lib 候補。 main repo に同居して core 改修と atomic に refactor 可能** |
| 3. 別 repo | `github.com/284km/mere-foo` | 独立 version / issues / PRs |

stage 2 → 3 の graduation 条件:
- Mere 本体に **pkg manager** が実装され、 `mere fetch` 経由で外部 dep を解決できる
- API が daily breaking でなくなる (= 1 ヶ月以上 signature 安定)
- 外部 consumer (Mere 以外で書かれた user code) が 1 つ以上存在する

## 使い方 (pkg manager 完成前)

Mere は **`module M { ... }` + `import "path";` を既に持っている** が、
contrib lib は **当面 module wrap せず top-level の `type` / `let` のまま**
で配布する。 理由は、 4 backend codegen で `match v with | M.JNull -> ...` の
ような **qualified constructor pattern が未対応** (DEFERRED §4.1) で、 module
wrap すると interp でしか pattern match が書けなくなるため。 詳細は
internal design notes §1。

そのため、 contrib の lib を使う方法は **copy-paste** か、 `import` でも構わない
(import すると top-level に splice される、 名前空間化はされない)。

```sh
# 例: JSON を使いたい
cp contrib/json/json.mere my_project/
# my_project/main.mere の先頭で type json と parse_json が available になる
```

ファイル単位で「先頭に concat する」 と prelude 同様に top-level let / type が
inject される。 名前衝突を避けるため、 contrib の lib は **prefix 付き命名規約**
(`json_parse / json_show / md_to_html / md_to_text`) を採用する。

## 現在の contrib lib

| lib | path | 機能 |
|---|---|---|
| **json** | `contrib/json/` | JSON parse + write (compact / pretty) |
| **markdown** | `contrib/markdown/` | Markdown 部分集合 → HTML / 平文 / TOC |

将来追加候補は `internal design notes` §3 参照。

## 設計判断の根拠

なぜ `examples/` から分けるかの詳細は internal design notes §3 を参照。
