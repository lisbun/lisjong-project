# Project agent workflow

このrepositoryは横断設計・方針・文書を扱う。実装は各owner repositoryで行う。

## Kickoff

1. このファイルと `STATE.md` を読む。
2. 今回の作業に関係する `BLOCKING.md` / `HUMAN.md` を読む。
3. STATEが直接参照するIssue / PRの最新本文・コメント・状態を確認する。
4. architecture等の追加文書・履歴は必要な範囲だけ読む。全履歴の無条件scanはしない。

## 正本と更新

- Issue / PRはタスク・結果・判断履歴の正本。本文の現在地と古い履歴を区別する。
- `docs/architecture.md` は横断設計、`docs/roadmap.md` は長期能力の正本。進捗を複製しない。
- STATEは現在の作業と次actionだけを通常50行以内で持ち、完了した項目を削除する。
- BLOCKINGは必要証拠・依存関係など、対象作業を進められない理由だけを持つ。一般TODOはIssueへ置く。
- HUMANは人の回答・作業が必要なものだけを持ち、理由と回答がない場合の動きを書く。通常の可逆な文書整理は確認待ちにしない。
- 観測・担当者報告・今回の独立検証を区別する。不明な値や進捗を補完しない。
- 進行中の比較は担当と事前登録を確認し、重複起動や結果を見た条件変更を行わない。
- 推定精度、判断の質、対局の強さ、運用品質を区別する。実装mergeだけで評価Issueをcloseしない。
- 文書の変更はPRでレビュー可能にする。mergeはユーザーの承認範囲に従う。
- 課金実行、稼働bot変更、外部source取得は文書整理から自動的に広げない。

## Handoff

結果と根拠をIssue / PRへ残し、STATEのNowと具体的なNextを更新する。
新しいblocker / human actionだけを対応ファイルへ記載し、解消済みの項目は削除する。
このファイルにsession logや完了履歴を蓄積しない。
