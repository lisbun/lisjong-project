# Current focus

P1本学習は完了（NOT-PREDICTIVE）。次の主仮説は未選定。
配分・選定入口: [#84](https://github.com/lisbun/lisjong-project/issues/84)。Learningの順序・データ境界: [#74](https://github.com/lisbun/lisjong-project/issues/74)。
確認日: 2026-11-06 JST。GitHub記録の照合で、PC上のartifact・請求の独立再監査は行っていない。

# Now

- [Arena #558](https://github.com/lisbun/lisjong-arena/issues/558)は結果レビュー済み・completed。E2 ΔMSE −64.25（P5 −127.63、P95 −2.06）でNOT-PREDICTIVE。今回の10特徴・ridge λ=10はTESTでTRAIN平均より二乗誤差が大きかった。予測不能の証明でも棋力の結果でもない。
- TRAIN 120 seeds / 696行、TEST 120 seeds / 654行（分岐あり118 seeds）。fit・評価各1回、INVALID 0、verify・回収hash一致、両allocation COMMITTEDは[レビュー記録](https://github.com/lisbun/lisjong-arena/issues/558#issuecomment-6661359139)を参照。
- 収集46,472.4秒（約12.91時間）。初回STOP後、TEST結果を見る前の承認・[Arena PR #559](https://github.com/lisbun/lisjong-arena/pull/559)でブロック4時間・累積22時間へ変更してr2を完了。旧3時間/14時間は履歴。AWS利用なし。
- lisjong #362/#365の設計、#363/#366・Arena #553/#556の実装、Arena #555のpilotは完了済み。実装・設計待ちへ戻さない。
- R1今回実験（Arena #550）はNOT-PREDICTIVEで終了。系列は[ユーザー判断](https://github.com/lisbun/lisjong-project/issues/84#issuecomment-6545612729)どおり再開条件付き保留。具体的な新仮説ができるまで作業を割り当てない。
- 採用可能な新候補はない。Champion維持。P1はE4へ進めず、今回の終了をP1系列やLearning全体の棄却へ拡張しない。

# Next

1. #84 / #74、lisjong #297で次仮説の比較材料を整理する。最大3案に対象判断・頻度/support・既存結果と異なる根拠・収支への接続・費用・主出口・停止条件を揃え、選定後に1案を限定Issueへ渡す。新しい主実験は未選定。
2. P1の再fit・追加seed・TEST再評価、R1再開、TRAIN追加分析、生成費用削減を自動開始しない。新仮説や分析はscope・利用データ・項目を先に固定する。
3. AWS月額・残額とRiichiLabの稼働版・対局数・表示レートは未照合。10月月次記録は#84の11月1日コメント、11月のP1完了は#84本文を参照。
4. lisjong #302の補助証拠、#285の推定研究、project #87の保守、#97/#99/#100の自律loop準備は独立管理。一般的な前提にしない。

# 判断の前提

- H5・paijia v1・genbutsu v1・call-value v1・R1・P1の既存結果を追加seed/条件緩和/TEST調整で救済しない。
- R1 TRAIN 547000..547043 / TEST 547100..547143、P1 TRAIN 556000..556119 / TEST 556200..556319は使用済み。TESTを調整や新しい独立評価へ流用しない。
- H5 539000..539023、R1 pilot 544000..544015、P1 pilot 553000..553023、既使用rehearsalは開発/動作確認用途。同一seedの全局/席/R/D派生を同一groupにする。
- 既存OFFLINE-EVALを再利用せず、旧#326・大規模NN・RLへ自動移行しない。project #57 / Arena #317 / lisjong #286はDEFERRED。
- #83の正式評価は候補固有の契約待ち。予測精度、判断改善、半荘棋力、運用品質を区別する。
- native必須。pin変更時は同revision wheelと必要gateを確認する。
- AWS上限は月USD 20（既支出込み・credit適用前）。残額未照合、課金実行は別判断。個人PCの使用・時間上限は個別承認を後続へ一般化しない。
- 通常merge/closeはAGENTSの継続承認。無人loopの条件は[HUMAN](HUMAN.md)、個別の依存は[BLOCKING](BLOCKING.md)を参照。
