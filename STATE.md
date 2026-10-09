# Current focus

聴牌PUSH/FOLD設計、小規模L2a設計、engine新protocolの前提検証。
確認日: 2026-10-09 JST。ライブ状態はリンク先の最新記録を優先する。

# Now

- [#84](https://github.com/lisbun/lisjong-project/issues/84): 全体目標・予算・月次判断。
- [lisjong #285](https://github.com/lisbun/lisjong/issues/285): リーチ者の待ち推定v2（#245より精度向上、複数リーチ・観測者リーチへ提供拡大）。HandBelief推定精度の親#255はclose。門前非リーチは[#286](https://github.com/lisbun/lisjong/issues/286)でDEFERRED。
- [lisjong #254](https://github.com/lisbun/lisjong/issues/254): 単独リーチ下の聴牌PUSH/FOLD設計。完成手の点数計算と和了・放銃の見込みを区別する。
- [#74](https://github.com/lisbun/lisjong-project/issues/74): 学習の順序。設計担当は[lisjong #267](https://github.com/lisbun/lisjong/issues/267)。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): engine移行仕様はArena #452（closed）で[文書化済み](https://github.com/lisbun/lisjong-arena/blob/main/docs/heuristic-candidate-engine-aabb-half.md)（PR #471）。次は[Arena #472](https://github.com/lisbun/lisjong-arena/issues/472)のfixture検証。実装・校正は未起票、正式実行は未解禁。
- [Arena #455](https://github.com/lisbun/lisjong-arena/issues/455): 危険局面の打牌不一致を記述。実行前の判断はHUMANを参照。
- [#87](https://github.com/lisbun/lisjong-project/issues/87): Rust移行の次段階はartifact供給と検証付き入力境界。

# Next

1. #254で対象gate・PUSH/FOLDの共通単位/時間範囲・優先規則・不足する推定を固定する。放銃側の入力候補はlisjong #277のリーチ者ロン合法推定器（推定精度のみ確認済み、強さは未評価）。
2. #267で教師・source・分割・行動別成立条件・全工程費用を固定し、実装/生成/学習を後続へ分ける。
3. Arena #472でengine新protocolの前提（RuleSet・最終得点・Champion bridge・Python/Rust）をfixtureで検証する。実装・校正はその結果を見てから起票する。
4. #285で特徴量セット・対象条件（S1とそれ以外を別の主張）・合否を事前登録してからselectする。formal test用sourceを#277段階1の再測定と共用するかは生成前に決める。使用済みseed 933000..933399・934000..934199・936000..936399・937000..937199はformal testに再利用しない。

推定精度、判断の質、対局の強さ、運用品質を区別する。新しい課金実行・Champion変更は各ownerの手順で別判断する。
