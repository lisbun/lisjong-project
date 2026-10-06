# Current focus

R1800に向けたH2設計と、engine正式評価への移行条件の具体化。
確認日: 2026-10-07 JST。ライブ状態はリンク先の最新記録を優先する。

# Now

- [#84](https://github.com/lisbun/lisjong-project/issues/84): 全体目標・予算・月次判断。
- [lisjong #254](https://github.com/lisbun/lisjong/issues/254): 単独リーチ下の聴牌PUSH/FOLD設計。#249は両候補不採用で完了。閾値調整で再開しない。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): 移行仕様の担当は[Arena #452](https://github.com/lisbun/lisjong-arena/issues/452)。正式実行は未解禁。
- [#85](https://github.com/lisbun/lisjong-project/issues/85): 条件付きregistry反映済み。上位botの観測日時付き表示レートが残る。
- [#74](https://github.com/lisbun/lisjong-project/issues/74): first-party C0による小規模L2aの設計。
- [#87](https://github.com/lisbun/lisjong-project/issues/87): Rust移行の次段階はartifact供給と検証付き入力境界。

# Next

1. #254で聴牌gate・PUSH/FOLDの共通単位/時間範囲・優先規則を固定する。比較の再起動や新規対局は設計から自動的に行わない。
2. Arena #452で差分fixture、protocol/保存物仕様、bridge/backend確認計画を固定し、実装・校正を別工程へ渡す。
3. #85の上位bot観測は[Arena #441最新手順](https://github.com/lisbun/lisjong-arena/issues/441#issuecomment-6018456067)を使う。PR #451のselect/fetchを同一UTC日に実施する工程で、結果の取得済みを推定しない。
4. #74で#86の利用条件と#442の校正を使い、教師・分割・行動別成立条件・全工程費用を固定する。
5. この更新PRをレビューし、#81の試行記録と更新負担から完了判断する。
