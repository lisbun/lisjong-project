# Current focus

P1（門前の最初のPonをPassへ替えたときの行動価値）の本学習の設計。正本: [lisjong #365](https://github.com/lisbun/lisjong/issues/365)（設計のみ）。pilot: [Arena #555](https://github.com/lisbun/lisjong-arena/issues/555)（DESIGN-POSSIBLE、[レビュー](https://github.com/lisbun/lisjong/issues/362#issuecomment-6597953690)）。
配分: [#84](https://github.com/lisbun/lisjong-project/issues/84)、[#74](https://github.com/lisbun/lisjong-project/issues/74)。確認日: 2026-11-03 JST。
GitHub記録の照合。PC上のartifact・請求の独立再検証は行っていない。

# Now

- [lisjong #356](https://github.com/lisbun/lisjong/issues/356)は設計DESIGN-READYでcompleted。R1（リーチの初回だけダマ、以後Champion）を対象に、lisjong #357/#360、Arena #544/#548で実装、Arena #546でpilot（PILOT-COMPLETE）を実施。
- [Arena #550の結果](https://github.com/lisbun/lisjong/issues/356#issuecomment-6545223612): **NOT-PREDICTIVE**。凍結artifact `b765c5d7…e0d3`をTEST 44 seeds（546行）で1回評価し、E2のΔMSE −15.48、P5 −45.30で不通過。予測できないことの証明ではなく、強さの結果でもない。
- ユーザー判断で今回のR1実験（encoding v1・ridge λ=10・TRAIN/TEST各44 seeds・12時間）を終了。E4設計Arena #551はnot planned。R1系列は終了とせず、**再開条件付きで保留**（[決定](https://github.com/lisbun/lisjong-project/issues/84#issuecomment-6545612729)）。具体的な新仮説ができるまで作業を割り当てない。
- P1 pilot（Arena #555）: **DESIGN-POSSIBLE**。成立116 / 24 seeds、INVALID 0、取消率0、2 seed再実行一致、COMMITTED。c = 157.24秒 / seed、T* = 120、TEST分岐見込み580 ≥ 500。対の相関 −0.039・分散比1.037で、同じ山でも差のばらつきは減らなかった（R1の0.171と異なる）。分岐前時間の割合0.90（SUPPORT-OKのため案Cは非該当）。判定は学習開始の承認ではない。
- 限定候補の終了はpaijia v1・genbutsu v1・call-value v1・H5に続き、Learning側のR1で5件目。採用可能な強化候補はない。Championは維持。

# Next

1. 11月初の月次確認は#84の棚卸コメントへ記録済み。AWS月額・残額とRiichiLabの状況は未照合（11月以降の上限USD 20）。
2. lisjong #365でP1本学習を設計する（設計のみ）。主出口E2（定数予測に対するΔMSEを独立TESTで1回）、特徴・モデルはTRAIN生成前に固定、SELECTなし。K = 296（平均差の規模計算）をE2の必要seed数に使わない。
   - 60 seedsのブロックは約10,649秒で3時間上限までの余裕は約151秒。各ブロック開始時に固定seed数のまま速度を再確認し、超える見積もりなら開始せず停止する。実行機械・条件を固定する。
   - 実行承認は設計と予算の確認後の別判断。pilotのseed 553000..553023は開発用途のみ。
3. 生成費用の削減（記録行動の再生計測）は、pilotがNOT-VIABLE-SUPPORTで、D armの分岐前時間の割合（worker内測定のΣ分岐前 / ΣD arm全体）が7/10以上の場合だけ提案する。
4. R1を再開する場合は、TRAIN（547000..547043）と既使用の開発データだけを、分析項目をIssueへ事前固定して扱う。TEST（547100..547143）は調整に使わない。
5. lisjong #302の補助証拠、#285の推定研究、project #100の制御準備は独立管理。

# 判断の前提

- Champion維持。H5・paijia v1・genbutsu v1・call-value v1・R1を追加seed/条件緩和/TESTでの再fitで救済しない。
- 既使用DEVELOPMENT（H5 539000..539023、pilot 544000..544015、rehearsal 472000..472006）とR1のTRAIN/TESTを、後から未使用TESTへ改称しない。同一seedの全局/席/R/D派生を同一groupにする。
- 既存OFFLINE-EVALを再利用せず、旧#326・大規模NN・RLへ自動移行しない。project #57 / Arena #317 / lisjong #286はDEFERRED。
- #83の正式評価は候補固有の契約待ち。予測精度、判断改善、半荘棋力を区別する。
- 個人PCでの長時間実行は、負荷・電源条件で速度が変わる（Arena #550の最初の準備はNOT-STARTED-BUDGETで停止）。開始条件・上限はmax(monotonic, realtime)で判定する。
- native必須。pin変更時は同revision wheelと必要gateを確認する。
- AWS上限は11月以降USD 20（既支出込み・credit適用前）。残額未照合、課金実行は別判断。
- 通常merge/closeはAGENTSの継続承認。無人loopの条件は[HUMAN](HUMAN.md)を参照。
