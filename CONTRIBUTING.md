# Contributing to Mere

Mere への contribution、歓迎します。

## License (重要)

Mere は現在 **MIT License** で公開されています ([LICENSE](LICENSE) 参照)。

このプロジェクトは将来、**MIT OR Apache-2.0 dual license** へ移行する可能性が
あります。そのため、pull request / patch / commit を提出した時点で、
contributor は以下に同意したものと見なします:

1. あなたの contribution は **MIT License** のもとで配布される
2. 将来 Mere が Apache License 2.0 を追加して dual license 化する場合、
   あなたの contribution は **Apache License 2.0** のもとでも配布される

これは公開直後にプロジェクトの license 戦略を再検討する余地を確保するための
ものです。現状の利用者にとっては MIT License のみが effective です。

## 開発フロー

1. Fork して branch を切る (`git checkout -b your-feature`)
2. 変更を加える + テスト (`dune runtest`) を pass させる
3. Pull request を出す

PR には:
- 変更の動機 (どんな問題を解決するか / どんな機能を追加するか)
- 4 backend (interp + C + LLVM + Wasm) のうちどれに影響するか
- 新しいテストを追加した場合はその内容

を含めてください。

## バグ報告 / 機能要望

GitHub Issues でお知らせください。再現手順 + `dune exec ./bin/mere.exe --
--version` の出力があると助かります。

## 設計に関する議論

言語設計の OPEN_QUESTIONS や paper-validated な意思決定は別リポジトリで
管理されています (詳細は README 参照)。大きな設計変更を伴う提案は、まず
Issue で議論してから PR を出してください。

## コードスタイル

- OCaml 本体は dune の標準フォーマット (`dune fmt`)
- `.mere` example は既存ファイルのスタイルに合わせる (4 backend で
  diff = 0 を維持するためのテストが要る場合あり)
