# Current focus

H4 call-value shadow v1の終了を受け、HeuristicとLearningを並べて次の主仮説を選び直す。
正本: [#84](https://github.com/lisbun/lisjong-project/issues/84)、[#74](https://github.com/lisbun/lisjong-project/issues/74)、[lisjong #297](https://github.com/lisbun/lisjong/issues/297)。確認日: 2026-10-25 JST。
GitHub記録を照合。shadow artifact・fixture・請求実額の独立再検証は行っていない。

# Now

- [#84](https://github.com/lisbun/lisjong-project/issues/84) / [#74](https://github.com/lisbun/lisjong-project/issues/74): 主仮説は未選定。[Arena #536](https://github.com/lisbun/lisjong-arena/issues/536#issuecomment-6386231064)は**STOP-LOW-ACTIVATION**（主対象の変更1 / 96 seat-hanchans・1 seed、INVALID 0）でcall-value v1を終了。限定候補の終了はpaijia v1（#522）、genbutsu v1（#526）に続き3件目。
- [#97](https://github.com/lisbun/lisjong-project/issues/97): 6件の残件は[準備文書](docs/autonomous-loop-readiness.md)。実候補列がないため#481の完走は進められない。#480 / #100の実経路接続準備と#99は候補に依存せず進められる。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): 正式評価は候補固有の標本数・環境・時計/上限・費用の計画待ち。現在対象候補なし。
- [lisjong #302](https://github.com/lisbun/lisjong/issues/302): RiichiLab規則・補助証拠の残件。未確認環境への適用を保留。

# Next

1. #84でH/Lの次仮説候補を比較し、1つを選ぶ。候補ごとに、変更する判断の**到達頻度を既存の局単位記録・shadow記録から先に見積もり**、配分基準に届く見込みがない問いはfixture診断・shadow設計へ進めない。Learning案は学習量・source/label・support・費用・停止条件を明示する。
2. 選んだ仮説をowner repoの設計Issueへ固定する（頻度確認 → 候補仕様/identity → S1/S2予算）。結果を見た条件変更や終了済みv1の救済をしない。
3. 11月初（AWS上限がUSD 20へ戻る時点）に進捗・費用・H/L配分を見直す。新規Launch前に請求と未反映runを照合する。
4. 並行可: Arena #480のlive予約→lock→commit経路、#100のguard接続、#99のskill/session構成。定期起動・自動昇格は#98項目4〜6の判断まで先行しない。
5. lisjong #285は独立した推定精度研究。#87の全面Python非依存化は後続に置き、主作業の前提にしない。

# 判断の前提

- Champion維持。終了候補（paijia v1 / genbutsu v1 / call-value v1）の追加seed・条件緩和による救済をしない。低頻度停止から制約緩和の価値を推定しない。
- LearningのOFFLINE-EVALを再利用せず、CALL再学習・大規模化・RLへ自動移行しない。
- native必須。lisjong pin更新時は同revision wheelと新gate証拠を確認する。WSLのPASSをAWSの証拠にしない。
- project #57 / Arena #317 / lisjong #286はDEFERRED。校正値を別pairへ単純流用しない。
- AWS上限は10月USD 30、11月以降USD 20（既支出込み・credit適用前）。新規Launch前に請求/未反映run照合と個別承認。seedはlive ledger照合。
- 通常merge/closeはAGENTSの継続承認。無人loopの運用条件はHUMANを参照。
