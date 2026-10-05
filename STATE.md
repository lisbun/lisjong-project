# Current focus

R1800に向けたH1の開発比較と、project側の方針・記録整備。
確認日: 2026-10-06 JST。ライブ状態はリンク先の最新記録を優先する。

# Now

- [#84](https://github.com/lisbun/lisjong-project/issues/84): 全体目標・予算・月次判断。
- [#74](https://github.com/lisbun/lisjong-project/issues/74): 学習の実行順序。小規模L2aの設計が次。
- [lisjong #249](https://github.com/lisbun/lisjong/issues/249): PR #250 merged。主比較900半荘＋別相手600半荘を事前登録済み。結果の公開記録は未確認。実行担当の集計を待つ。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): engine正式評価の移行条件を整理中。正式実行は未解禁。
- [#85](https://github.com/lisbun/lisjong-project/issues/85): 2botの条件付き200戦記録をregistryへ反映するPRのレビュー待ち。
- [#87](https://github.com/lisbun/lisjong-project/issues/87): Rust移行。次段階はartifact供給と検証付き入力境界。

# Next

1. projectの文書PRをレビューする。merge可否はHUMANを参照。
2. #249の完全な集計を受け、事前登録どおり主比較・別相手構成を判定し、#236/#84へ採否を反映する。再起動・閾値変更はしない。
3. #83のRuleSet差分表・bridge/backend整合の確認範囲をArenaで具体化する。
4. #86の利用条件と#442の校正を使い、#74でL2aの固定教師・データ分割・行動別成立条件・全工程費用を決める。新しい生成はまだ開始しない。
5. #81の次session trialで、必要な履歴だけから着手できたかと更新負担を記録する。
