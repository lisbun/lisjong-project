# Human attention

## 自律強化ループの残る権限項目（ADR 0011）

Priority: medium
Owner: lisbun（判断）、ADR起草・実装担当Agent（確定後）
Related: [#98](https://github.com/lisbun/lisjong-project/issues/98)「決める項目」4〜6、親[#97](https://github.com/lisbun/lisjong-project/issues/97)

項目1〜3は2026-10-12に回答済み（[記録](https://github.com/lisbun/lisjong-project/issues/98#issuecomment-6122880561)）。leaderとChampion昇格の自動更新、候補レーンの自動mergeを許可した。family分類は同日20:49 JSTの追加判断で変更済み（[#98本文](https://github.com/lisbun/lisjong-project/issues/98)、[ADR 0011](docs/decisions/0011-autonomous-strength-loop.md)）。手書き規則の閾値探索はHeuristic、主たる行動選択を学習モデルが担うPolicyはLearning。学習推定器を手書き規則へ供給する場合は「Heuristic＋学習推定器」とする。過去コメントの旧分類を現在の判断として使わない。

Question:

1. leaderが5回交代するごとに、C0とS3規模で比較することを必須にするか。
2. 仮説の系統ごとに、S3を何回まで試せるか。
3. 月予算（AWSは#84の通常月USD 20内の配分。2026年10月のみUSD 30、11月以降USD 20へ戻る、LLM利用）、GitHub ActionsへのAWS用secret設定、停止スイッチの場所（#100）。

Default: 未回答なら、自動merge・S3自動起動・leader / Championの自動更新を有効化しない（実装は進めてよい）。#97のPhase 1残件（lisjong #297、Arena #480・#481。lisjong #296・Arena #479は完了済み）と、L1の準備（#99・#100の実装）は進めてよい。LLM sessionの定期稼働（L1の運用開始）は、上の質問3（予算）が決まるまで始めない。
Why human: 偽陽性の累積の扱いと試行上限は、自動昇格の正しさに直結する。課金・secret設定は個別承認の対象。

外部sourceの連絡待ちは全作業の前提にしない。
