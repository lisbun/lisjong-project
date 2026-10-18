# Current focus

小規模Learning設計とengine正式評価の前提検証。現在地・順序提案の正本は[#84](https://github.com/lisbun/lisjong-project/issues/84)。
確認日: 2026-10-18 JST。Issue本文・コメントを照合。実験artifactの独立再計算は行っていない。

# Now

- [#74](https://github.com/lisbun/lisjong-project/issues/74): 学習の次作業は[lisjong #267](https://github.com/lisbun/lisjong/issues/267)のfirst-party C0小規模L2a設計。10/17の外部AI調査コメントは設計材料であり未決。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): [Arena #472](https://github.com/lisbun/lisjong-arena/issues/472)のfixture検証待ち。正式実行は未解禁。
- [lisjong #302](https://github.com/lisbun/lisjong/issues/302): 基盤実装済み。RiichiLab規則・環境別検証の残件を管理。候補の採否と基盤の正しさを分離する。
- [#97](https://github.com/lisbun/lisjong-project/issues/97): Phase 1の残りはlisjong #297、Arena #480/#481。#98項目4〜6と運用条件はHUMAN判断待ち。family分類はADR 0004/0011を維持。
- [lisjong #285](https://github.com/lisbun/lisjong/issues/285): リーチ者待ち推定v2。構造的待ちの精度向上を放銃確率の校正改善・棋力改善と同一視しない。
- [#87](https://github.com/lisbun/lisjong-project/issues/87): Rust次段階はartifact供給と検証付き入力境界を限定して扱う。

# Next（順序の提案）

1. lisjong #267の設計を完了する。教師・行動別support・合法手とlabel・envelope・オーラス・小規模モデル・全工程費用を固定し、実装/生成/学習は後続へ分ける。
2. Arena #472で精算・bridge/backend・固定C0のrevision差を確認する。その後に実装・校正を起票する。
3. lisjong #302のRiichiLab規則の根拠と実対局補助証拠を整理する。未確認のままなら適用を保留し、lisjong #267 / Arena #472は続行する。
4. lisjong #297の探索対象を1つ選定し、Arena #480の開発A/B割当手順から#481のS0〜S2へ接続する。対象選定と割当手順の準備は独立に可能。
5. lisjong #285、project #87・#99/#100は上記の後に限定着手。#98の未決項目はHUMANに従う。

# 判断の前提

- [Arena #503](https://github.com/lisbun/lisjong-arena/issues/503#issuecomment-6245291382)は採用条件未達・強さの優劣未確定で完了。追加評価は今は行わず、Championを維持する。基盤は保持する。
- engine domain 931000..931999および940000..940229は使用済みと報告されている。新しい予約ではlive ledgerを確認し、独立再確認・正式評価へ再利用しない。
- AWS月$20は既支出込み。Arena #503費用上限見積り2.68 USDだけから月残額を推定しない。
- project #57 / Arena #317 / lisjong #286はDEFERRED維持。Learningを外部source・自律loop・Heuristic候補の完了待ちにしない。
- 自動merge・S3自動起動・leader / Champion自動更新・LLM定期稼働は未解禁。課金・稼働bot変更は別判断。
