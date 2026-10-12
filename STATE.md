# Current focus

自律強化ループPhase 1の前提部品、engine新protocolの前提検証、小規模L2a設計。
確認日: 2026-10-12 JST。ライブ状態はリンク先の最新記録を優先する。

# Now

- [#84](https://github.com/lisbun/lisjong-project/issues/84): 全体目標・予算・月次判断。H2（[lisjong #254](https://github.com/lisbun/lisjong/issues/254)）は2026-10-12に開発比較不通過でclose。Championは維持し、H3/H4は未起票。
- [#97](https://github.com/lisbun/lisjong-project/issues/97): 自律強化ループ。ADR 0011（[#98](https://github.com/lisbun/lisjong-project/issues/98)）は項目1〜3が回答済み（自動merge・leader / Championの自動更新を許可）、項目4〜6はHUMAN判断待ち。familyは[ADR 0004](docs/decisions/0004-champion-family-separation.md)改訂に従い、閾値探索はHeuristic、主たる行動選択を学習モデルが担うPolicyはLearning。S0の情報境界チェック（[lisjong #296](https://github.com/lisbun/lisjong/issues/296)）はPR #298で完了。Phase 1の残りは[Arena #479](https://github.com/lisbun/lisjong-arena/issues/479)・#480・#481、lisjong #297。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): engine移行仕様はArena #452（closed）で[文書化済み](https://github.com/lisbun/lisjong-arena/blob/main/docs/heuristic-candidate-engine-aabb-half.md)（PR #471）。次は[Arena #472](https://github.com/lisbun/lisjong-arena/issues/472)のfixture検証。実装・校正は未起票、正式実行は未解禁。
- [#74](https://github.com/lisbun/lisjong-project/issues/74): 学習の順序。設計担当は[lisjong #267](https://github.com/lisbun/lisjong/issues/267)（10/7から未着手）。
- [lisjong #285](https://github.com/lisbun/lisjong/issues/285): リーチ者の待ち推定v2。H2候補の再開条件の1つ（`p` の校正改善）。門前非リーチは[#286](https://github.com/lisbun/lisjong/issues/286)でDEFERRED。
- [Arena #455](https://github.com/lisbun/lisjong-arena/issues/455): 危険局面の打牌不一致を記述。比較先（#170の3体・bot別併記）と実行場所（lisbunのlocal）は確定済み。実行はlocalで行う。
- [#87](https://github.com/lisbun/lisjong-project/issues/87): Rust移行の次段階はartifact供給と検証付き入力境界。

# Next

- #98の残る項目4〜6を決め、[ADR 0011](docs/decisions/0011-autonomous-strength-loop.md)の未決部分と各ownerの運用条件を確定する。自動merge・S3自動起動・leader / Champion自動更新は未解禁。Phase 1と準備実装は進めてよい。LLM定期稼働は予算決定待ち。

1. Arena #479でloop sessionからprotocol・seed・判定規則のfileを守るguardを追加する。S0の情報境界チェックはlisjong `lisjong.candidate_boundary`（#296）を使う。
2. Arena #472でengine新protocolの前提（RuleSet・最終得点・Champion bridge・Python/Rust）をfixtureで検証する。実装・校正はその結果を見てから起票する。
3. #267で教師・source・分割・行動別成立条件・全工程費用を固定し、実装/生成/学習を後続へ分ける。
4. lisjong #297の探索対象を選び直す。#254の結論（打牌を替える頻度0.12〜0.14回/席・半荘、2,700半荘でも区間が0を含む）から、聴牌PUSH/FOLDの閾値は単独の探索空間としてS1で差を検出しにくい。
5. Arena #480の前提: engine domainの開発seed 931000..931999（Arena #385、RETIRED）は#290で使い切った。以後の開発比較・S1/S2は新しいallocationが要る。
6. #285で特徴量セット・対象条件・合否を事前登録してからselectする。使用済みseed 931000..931999・933000..933399・934000..934199・936000..936399・937000..937199はformal testに再利用しない。

推定精度、判断の質、対局の強さ、運用品質を区別する。新しい課金実行・Champion変更は各ownerの手順で別判断する。
