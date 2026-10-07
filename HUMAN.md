# Human attention

## 文書更新PRのレビュー

Priority: medium
Related: project #81 / #83、[PR #89](https://github.com/lisbun/lisjong-project/pull/89)

Question: PR #89の修正後差分をレビューし、merge可否を判断する。
Default: PRをopenに保ち、Arena #452の仕様整理とlisjong #254の設計を進める。
Why human: mergeは既存のユーザー承認運用に従う。

## 上位botデータの取得と受渡し

Priority: medium
Owner: lisbun（取得・受渡し）、分析担当Agent（受領後の照合・集計）
Related: [Arena #441最新手順](https://github.com/lisbun/lisjong-arena/issues/441#issuecomment-6018456067)、[project #85](https://github.com/lisbun/lisjong-project/issues/85)

Question: ユーザー管理AWSの既存Arena環境で、PR #451を含むrevisionを記録し、最新手順の`select`→`fetch`を同一UTC日に続けて実行する。`selection/`、`windows/bot-*/`、自己除外した`sha256sums.txt`、実行Arena revisionを非公開の共有先へ渡す。途中で失敗した場合は停止し、その表示を共有する。
Default: 回収物の受領・検証までは上位bot比較と#85の観測値記録を待つ。取得済みと推測せず、既存partial記録を保持し、#254・#452・L2aの設計を続ける。
Why human: 現行の取得手順はユーザー管理AWSでの操作と回収物の受渡しをlisbunが担当するため。取得toolのmergeだけではデータ取得・検証は完了しない。

外部sourceの連絡待ちは全作業の前提にしない。
