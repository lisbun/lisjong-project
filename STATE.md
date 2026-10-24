# Current focus

鳴き直後の採用閾値割れを対象に、一度限りのshadow計画を固定する。
正本: [#84](https://github.com/lisbun/lisjong-project/issues/84)、[lisjong #297](https://github.com/lisbun/lisjong/issues/297)。確認日: 2026-10-24 JST。
GitHub記録を照合。fixture・実験artifact・請求実額の独立再検証は行っていない。

# Now

- [Arena #533](https://github.com/lisbun/lisjong-arena/issues/533): shadowの設計。実際の打牌変更可能頻度と速度/保持価値の交換は未測定。候補Policy・S1/S2はまだない。
- [#74](https://github.com/lisbun/lisjong-project/issues/74): 次のLearning主実験は未選定。shadowの計画確定/結果時にH/L配分を見直す。
- [#97](https://github.com/lisbun/lisjong-project/issues/97): 6件の残件は[準備文書](docs/autonomous-loop-readiness.md)。seed準備・guard判定部から実経路への接続が残る。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): 正式評価は候補固有の標本数・環境・時計/上限・費用の計画待ち。
- [lisjong #302](https://github.com/lisbun/lisjong/issues/302): RiichiLab規則・補助証拠の残件。未確認環境への適用を保留。

# Next

1. Arena #533で対象条件・決定的代替打牌・計測母数・規模・速度損失/頻度の停止条件・revision/wheel・保存/verifyを固定。Policy意味論はlisjong所有を維持する。
2. 設計後にowner別の実装、必要な実行gateを満たして一度だけshadow。通過時だけ候補仕様/identity→固定開発比較→別seed/別相手の再確認。低頻度/不正時は拡張せず止める。
3. 実候補が揃ったらArena #480/#481にseed予約→lock→S0〜S2→strict readback→commitを接続。#100の停止制御も開始経路へ接続する。
4. 非LLM経路の成立後に#99の役割別session・独立レビューを接続。#98項目4〜6の未決判断は並行して整理し、定期起動/自動昇格を先行しない。
5. lisjong #285は独立した推定精度研究。#87の全面Python非依存化は後続に置き、主作業の前提にしない。

# 判断の前提

- Champion維持。手作りfixtureの反例は頻度・損失点・棋力の証拠ではない。終了候補の追加seedや条件緩和による救済をしない。
- LearningのOFFLINE-EVALを再利用せず、CALL再学習・大規模化・RLへ自動移行しない。
- native必須。lisjong pin更新時は同revision wheelと新gate証拠を確認する。WSLのPASSをAWSの証拠にしない。
- project #57 / Arena #317 / lisjong #286はDEFERRED。校正値を別pairへ単純流用しない。
- AWS上限は10月USD 30、11月以降USD 20（既支出込み・credit適用前）。新規Launch前に請求/未反映run照合と個別承認。seedはlive ledger照合。
- 通常merge/closeはAGENTSの継続承認。無人loopの運用条件はHUMANを参照。11月初に進捗・費用・H/L配分を見直す。
