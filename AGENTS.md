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
- 文書の変更はPRでレビュー可能にする。merge / closeは下記の継続承認に従う。
- 課金実行、稼働bot変更、外部source取得は文書整理から自動的に広げない。

## AI間の連携

- AI間の相談・レビュー・意見交換は、対象のGitHub Issue / Pull Requestのコメントを正本とする。
  チャットで得た結論のうち、方針・設計・実験条件に関わるものは、Issue / PRへ記録してから
  次の作業へ進む
- 他のAIへ意見を求める場合は、本文を転記せず、Issue / PRのURLと読むべきコメントを示す
- 各AIの投稿は、どのAIの意見かが分かる形にする
- ユーザーが「方針が一致すれば進め、差異があれば意見をまとめる」形で条件付きに委任した場合、
  一致と判断した根拠（照合したIssue・記録・revision）をIssue / PRへ記録して進めてよい。
  差異がある場合は、意見をIssue / PRへ投稿してユーザーの判断を待つ

## merge・closeと完了後cleanup

- 通常開発のPR merge・Issue/PR closeは、以下のレビュー・検証条件を満たせばユーザーへの
  追加承認を不要とする（2026-10-22の継続承認、lisbun/lisjong-project#106）
- merge前にIssueのscope / acceptance criteriaと既存contractを確認し、implementation
  self-review、必要なCI、semantic reviewを完了する。CI成功だけでレビュー済みとしない
- 対象head SHA、レビュー結論、blocking指摘の解消、検証結果をPRへ記録する。
  修正後は変更範囲・既知finding・直接のregressionをfocused re-reviewする。
  security・情報境界・破壊的変更等では必要な追加レビューを行う
- blocking指摘がなく、当該headの必要CI / full suiteとbranch protection上の条件が満たされたら、
  原則squash mergeする。merge時はexpected head SHAを指定し、head変更時は差分とCIを再確認する。
  branch protectionやrequired reviewを迂回しない
- Issue全体の完了条件と証拠が揃った場合は、`Closes #...`による自動closeまたは手動の
  completed closeを行ってよい。実装mergeだけで実測・評価待ちのIssueをcloseしない。
  一部完了なら`Refs #...`を使う
- 重複・置換・中止等でIssue/PRをmergeせずcloseする場合も、根拠と引継ぎ先（あれば）を記録し、
  適切なclose理由を使えば追加承認不要。未解決のscope・方針判断を勝手に完了や中止としない
- 個別のユーザーによる保留、draft、具体的な未決判断がある場合は解消するまでmerge / closeしない。
  過去の一律の「merge承認待ち」だけは、この継続承認で置き換える
- 原則として変更担当AI（または明示的に引継ぎを受けたレビュー担当AI）が完了条件とcleanupを確認し、
  不要になった作業branchを削除する。長期branchや未保存の作業は削除しない
- merge commit、Issue状態、cleanup結果、残る作業を報告する

この継続承認は課金実行・実験開始・正式評価・稼働bot切替・secret/settings変更等の別承認やgateを変更しない。
STATE / HUMAN / ADR 0011の自律強化loopの運用開始条件は別判断であり、通常開発のmerge権限と区別する。

## 追加承認なしで進めてよい作業（継続承認）

- 次の作業は、記録を残すことを条件に、ユーザーへの追加承認を不要とする
  （2026-10-25の継続承認、lisbun/lisjong-project#118）
  - 既存Issue / PRへの分析・レビュー・意見のコメント投稿
  - 設計のみ・調査のみのIssueの本文更新。既存の決定、scope、対象外、停止条件、判定基準を
    変更せず、変更点と理由をコメントで記録する
  - 既存の保存物・既使用のdevelopmentデータに対する、事前に固定した分析。新しい対局・seed予約・
    課金・外部取得を伴わず、未使用の評価データ（OFFLINE-EVAL、未開封のformal test等）を開かず、
    結果を見て分析項目を変更しない。ユーザー管理環境で行う場合は、Issueに記載した時間・メモリの
    上限内に限る
  - 事前登録した実験計画の内部ステップ（gate → rehearsal → 本計測 → verify → 回収）の連続実行。
    ユーザーが計画全体を一度承認し、その計画のseed・予算・上限・停止条件・判定規則をIssueへ
    固定していることを条件とする。STOP・失敗・上限超過・不整合が出たらその場で止め、
    再試行や条件変更をせずに報告する
- 次は従来どおり、その都度ユーザーの承認を必要とする
  - 実験計画そのものの承認（上記の連続実行を始める前の1回）
  - 課金を伴う実行（AWS起動等）、予算の変更
  - 正式評価、Championの変更、稼働botの切替
  - 外部source・外部データの取得、外部への公開
  - secret・repository settings・branch protectionの変更
  - 方針の変更（ADR、lisbun/lisjong-project#74 / #84等の順序・目標の変更）
  - STOPした実験の再開、追加seedによる救済、判定基準の変更
- この継続承認は、自律強化loopの運用開始（lisbun/lisjong-project#98等）を含まない

## Handoff

結果と根拠をIssue / PRへ残し、STATEのNowと具体的なNextを更新する。
新しいblocker / human actionだけを対応ファイルへ記載し、解消済みの項目は削除する。
このファイルにsession logや完了履歴を蓄積しない。
