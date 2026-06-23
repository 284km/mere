# contrib/json — JSON parser / writer

Mere で書かれた JSON parse / serialize ライブラリ。 stdlib (`str_*` /
`is_digit` / `try_or` / `fail` / `StrBuf`) + 再帰 variant + pattern matching の
組み合わせで実装、 外部依存ゼロ。

## ファイル

| file | export | 行数 |
|---|---|---|
| `json.mere` | `type json = JNull \| JBool \| JNum \| JStr \| JArr \| JObj` + `parse_json: str -> json` | 約 180 行 |
| `writer.mere` | `to_json_str: json -> str` + `to_pretty_str: json -> str` | 約 130 行 |

## 使い方 (pkg manager 完成前)

```sh
# ユーザ project に copy paste
cp contrib/json/json.mere    my_project/
cp contrib/json/writer.mere  my_project/
```

各ファイル末尾の self-test ブロック (`case_v` で始まる demo / `let doc = …` 等)
は実 use 時に削除して良い。

`parse_json` と `to_json_str` を同じ project で使う場合は、 **`json.mere` の
`type json = …` 宣言を 1 箇所に統合する** 必要あり (Mere は同名 type の
multiple declaration を最新で上書きするので、 後勝ちで動くがソース merge が
推奨)。

## サポート範囲

- atoms: `null` / `true` / `false` / int (negative OK) / string
- composite: array / object
- escape: `\"` `\\` `\n` `\t` `\r` `\/` を `str_unescape` 経由で復元
- **非対応** (issue 駆動で拡張): float / unicode `\uXXXX` / exponential notation

## 既知の制約

- **`{` を含む文字列リテラル**: Phase 36 string interpolation の仕様で
  `"{"` が補間開始と解釈されるため、 `"\{"` で escape する必要あり
  (json.mere / writer.mere の demo は workaround 済)
- **C codegen で `case` という名前は予約語と衝突** (libc/C keyword) — 本 lib では
  `case_v` を使用

## 位置付け

stage 2 contrib (incubation)。 [contrib/README.md](../README.md) の lifecycle 参照。
公開 + pkg manager 完成後、 graduation 候補として別 repo `mere-json` に切り出す
計画 (internal design notes §3.1)。
