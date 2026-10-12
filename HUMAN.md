# Human attention

## Arena #455の比較先と実行場所

Priority: medium
Owner: lisbun（判断）、診断担当Agent（確定後の固定・集計）
Related: [Arena #455](https://github.com/lisbun/lisjong-arena/issues/455)「実行前に決めること」

Question:
1. 比較先は既存#170 corpusの3体すべてとし、bot別を併記する既定案で進めるか。3体合算を上位bot群の代表として読まない。上位botの選定規則に合うのはMortal-v4bだけである。
2. 実行は#251と同じcorpus保管環境内とし、raw log・materialize済み判断行を外へ出さない既定案で進めるか。AWS起動なし・費用$0見込みだが、実行担当が環境を確認する。

Default: 未回答なら診断実行を待つ。新しい取得・corpus移動・課金を開始せず、lisjong #254/#285/#267とArena #472の作業を継続する。
Why human: #455がこの2点をlisbunの判断事項として明示しており、比較populationとデータの実行境界に関わるため。棚卸しでは回答を推測しない。

## 自律強化ループの権限（ADR 0011）

Priority: medium
Owner: lisbun（判断）、ADR起草・実装担当Agent（確定後）
Related: [#98](https://github.com/lisbun/lisjong-project/issues/98)「決める項目」、親[#97](https://github.com/lisbun/lisjong-project/issues/97)

Question:
1. research-track leaderの自動更新を許可するか。Champion昇格は従来どおり別判断とするか。
2. 候補レーン（lisjongの候補用moduleとtestだけを変更し、CI・独立レビュー・情報境界チェックを通過したPR）の自動mergeを許可するか。
3. grid / optimizerで選んだ値と、LLMが開発評価の結果を見て変えた定数をLearning family扱いとするか（推奨）。結果を見ずに書いた規則はHeuristic familyとするか。
4. leader 5回交代ごとのC0比較、仮説系統ごとのS3試行上限、月予算（AWSは#84の$20内、LLM利用）、AWS用secret設定、停止スイッチの場所。

Default: 未回答なら自動merge・S3自動起動・leader自動更新を実装・有効化しない。#97のPhase 1（lisjong #296・#297、Arena #479・#480・#481）と、L1の準備（#99・#100の実装）は進めてよい。LLM sessionの定期稼働（L1の運用開始）は、項目4の予算が決まるまで始めない。
Why human: 各repoのAGENTS.mdはmergeにユーザーの明示承認を要求し、#44は自律loopに明示的なsafety / governance決定を求めている。課金・secret設定も個別承認の対象。

外部sourceの連絡待ちは全作業の前提にしない。
