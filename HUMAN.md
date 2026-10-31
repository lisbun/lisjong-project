# Human attention

## 次の主仮説とR1系列の扱い（月次見直し）

Priority: high
Owner: lisbun（判断）
Related: [#84](https://github.com/lisbun/lisjong-project/issues/84)、[#74](https://github.com/lisbun/lisjong-project/issues/74)、[lisjong #356](https://github.com/lisbun/lisjong/issues/356)

R1実験はArena #550でNOT-PREDICTIVEとなり、今回の設計での実験は終了済み（2026-11-01記録）。採用可能な強化候補はなく、主仮説は未選定。比較用の候補は#84の最新節に整理した（未承認の提案）。

Question:

1. R1系列全体を終了とするか、TRAINと既使用開発データに限った事前固定の分析だけを残すか。
2. 次の主仮説として、#84に挙げた候補のどれを設計対象にするか（または別案）。

Default: 未回答なら新しい対局・学習・seed予約・課金を始めない。既存記録だけを使う比較表の作成と、独立した保守・準備作業（#100、lisjong #302等）は進めてよい。
Why human: 主仮説の選定とH/L配分は#84の方針判断に当たり、継続承認の範囲外。

## 自律強化ループの残る権限項目（ADR 0011）

Priority: medium
Owner: lisbun（判断）、ADR起草・実装担当Agent（確定後）
Related: [#98](https://github.com/lisbun/lisjong-project/issues/98)「決める項目」4〜6、親[#97](https://github.com/lisbun/lisjong-project/issues/97)

項目1〜3は2026-10-12に回答済み（[記録](https://github.com/lisbun/lisjong-project/issues/98#issuecomment-6122880561)）。leaderとChampion昇格の自動更新、候補レーンの自動mergeを許可した。family分類は同日20:49 JSTの追加判断で変更済み（[#98本文](https://github.com/lisbun/lisjong-project/issues/98)、[ADR 0011](docs/decisions/0011-autonomous-strength-loop.md)）。手書き規則の閾値探索はHeuristic、主たる行動選択を学習モデルが担うPolicyはLearning。学習推定器を手書き規則へ供給する場合は「Heuristic＋学習推定器」とする。過去コメントの旧分類を現在の判断として使わない。

判断用の具体案: [自律ループ準備の未承認案](docs/autonomous-loop-readiness.md#98-残る判断の具体案未承認)。本案の文書mergeは運用承認ではない。開始前・実行中・停止後の接続案も同文書に整理済み。
追加で固定が必要な運用詳細: 費用/pause取得の鮮度、実行中pollと停止対象、週session/PR上限、停滞のサイクル定義とリセット条件。数値は未設定のままで、単発実験（Arena #542・#546・#550等）の承認を無人運用へ拡張しない。

Question:

1. leaderが5回交代するごとに、C0とS3規模で比較することを必須にするか。
2. 仮説の系統ごとに、S3を何回まで試せるか。
3. 月予算（AWSは#84の月USD 20内の配分、LLM利用）、GitHub ActionsへのAWS用secret設定、停止スイッチの場所（#100）。

Default: 未回答なら、自律loopの自動merge・S3自動起動・leader / Championの自動更新を有効化しない（実装は進めてよい）。#97のPhase 1残件（lisjong #297、Arena #480・#481。lisjong #296・Arena #479は完了済み）と、L1の準備（#99・#100の実装）は進めてよい。LLM sessionの定期稼働（L1の運用開始）は、上の質問3（予算）が決まるまで始めない。
Why human: 偽陽性の累積の扱いと試行上限は、自動昇格の正しさに直結する。課金・secret設定は個別承認の対象。

外部sourceの連絡待ちは全作業の前提にしない。

通常開発のPR merge・Issue closeは[AGENTS.md](AGENTS.md)と[#106](https://github.com/lisbun/lisjong-project/issues/106)の継続承認に従う。上の未決事項は無人loopの運用条件であり、通常の文書更新やレビュー済みPRを追加承認待ちにしない。
