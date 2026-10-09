# Current focus

HandBeliefの副露者待ち推定、聴牌PUSH/FOLD設計、小規模L2a設計。
確認日: 2026-10-07 夜 JST。ライブ状態はリンク先の最新記録を優先する。

# Now

- [#84](https://github.com/lisbun/lisjong-project/issues/84): 全体目標・予算・月次判断。
- [lisjong #255](https://github.com/lisbun/lisjong/issues/255): 推定精度の親。次は[#259](https://github.com/lisbun/lisjong/issues/259)の副露者推定（[PR #266](https://github.com/lisbun/lisjong/pull/266)、open）。
- [lisjong #254](https://github.com/lisbun/lisjong/issues/254): 単独リーチ下の聴牌PUSH/FOLD設計。完成手の点数計算と和了・放銃の見込みを区別する。
- [#74](https://github.com/lisbun/lisjong-project/issues/74): 学習の順序。設計担当は[lisjong #267](https://github.com/lisbun/lisjong/issues/267)。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): engine移行仕様はArena #452（closed）で[文書化済み](https://github.com/lisbun/lisjong-arena/blob/main/docs/heuristic-candidate-engine-aabb-half.md)（PR #471）。次は[Arena #472](https://github.com/lisbun/lisjong-arena/issues/472)のfixture検証。実装・校正は未起票、正式実行は未解禁。
- [lisjong #262](https://github.com/lisbun/lisjong/issues/262): ロン合法確率の表現・フリテン追加fact・正解・baseline測定。
- [Arena #455](https://github.com/lisbun/lisjong-arena/issues/455): 危険局面の打牌不一致を記述。実行前の判断はHUMANを参照。
- [#87](https://github.com/lisbun/lisjong-project/issues/87): Rust移行の次段階はartifact供給と検証付き入力境界。

# Next

1. #259の事前登録を維持してPR #266・select結果を確認し、選択を固定して新seedのformal testへ進む。933000..933399は開発専用。#260は後続、#258は独立に設計できる。
2. #254で対象gate・PUSH/FOLDの共通単位/時間範囲・優先規則・不足する推定を固定する。
3. #267で教師・source・分割・行動別成立条件・全工程費用を固定し、実装/生成/学習を後続へ分ける。
4. Arena #472でengine新protocolの前提（RuleSet・最終得点・Champion bridge・Python/Rust）をfixtureで検証する。実装・校正はその結果を見てから起票する。あわせて#262の追加fact契約を各ownerで進める。未確定の契約を先行実装しない。

推定精度、判断の質、対局の強さ、運用品質を区別する。新しい課金実行・Champion変更は各ownerの手順で別判断する。
