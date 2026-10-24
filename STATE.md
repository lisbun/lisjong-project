# Current focus

主仮説H5（リーチ/ダマ判断）の分岐測定の計画を固定する。
正本: [Arena #539](https://github.com/lisbun/lisjong-arena/issues/539)、[#84](https://github.com/lisbun/lisjong-project/issues/84)、[#74](https://github.com/lisbun/lisjong-project/issues/74)。確認日: 2026-10-25 JST。
GitHub記録を照合。選定の頻度根拠はArena #517の局単位記録の記述集計で、因果・棋力の根拠ではない。請求実額の独立再検証は行っていない。

# Now

- [Arena #539](https://github.com/lisbun/lisjong-arena/issues/539): H5の設計のみ。リーチ判断ごとにリーチ / ダマへ分岐し、同じ山・相手で局終了までの対の局収支差を測る。分岐方式・特徴・部分集合（最大3）・判定順序と数値・規模/環境を確定し、lisjong解析API / Arena分岐実行 / 実行Issueへ引き継ぐ。実装・実行・seed予約・課金は未実施。
- [#84](https://github.com/lisbun/lisjong-project/issues/84#issuecomment-6386424115): 選定根拠はChampionのリーチ宣言3.45回/sh・門前ダマのロン和了0件（#517）、RiichiLabリーチ率0.318（他家0.213、#441）。リーチ判断は`_choose_riichi`の常時リーチのまま未探索。直前のcall-value v1は[Arena #536](https://github.com/lisbun/lisjong-arena/issues/536#issuecomment-6386231064)でSTOP-LOW-ACTIVATION。
- [#74](https://github.com/lisbun/lisjong-project/issues/74): Learning主実験は未選定。#539の記録をリーチ/ダマの反実仮想labelとして再利用できる形にし、結果時にH/L配分を見直す。
- [#97](https://github.com/lisbun/lisjong-project/issues/97): 6件の残件は[準備文書](docs/autonomous-loop-readiness.md)。実候補列がないため#481の完走は進められない（H5はPROCEED後に接続を判断）。#480 / #100の実経路接続準備と#99は候補に依存せず進められる。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): 正式評価は候補固有の標本数・環境・時計/上限・費用の計画待ち。現在対象候補なし。
- [lisjong #302](https://github.com/lisbun/lisjong/issues/302): RiichiLab規則・補助証拠の残件。未確認環境への適用を保留。

# Next

1. Arena #539で計画を確定する。最初に、同じseedの再実行が元のaction列hashと一致する（局単位で分岐できる）ことを確認する。部分集合・閾値・停止条件は分岐結果を見る前に固定する。
2. 実装Issue（lisjong: 判断時点の特徴API・D armのリーチ抑止wrapper、Champion挙動は不変 / Arena: 分岐実行・記録・plan・strict verify）を起票し、実装後に実行Issueで環境・revision・wheel・gate・seed・保存先を束縛して一度だけ測る。PROCEED時だけ候補設計・開発比較へ進み、それ以外は理由を記録して#84 / #74で次の配分を決める。
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
