# Current focus

L2aのLearning family昇格は完了。現在はHeuristic対LearningのOverall比較準備。
新しい行動価値学習の主仮説は未選定で、既存L2aの評価経路とは区別する。
確認日: 2026-11-11 JST。GitHub記録の照合で、artifact・請求の独立再監査は行っていない。

# Now

- Current pair: Heuristic = placement-aware-speed-call、Learning = exact L2a M1+R1、Overall = **未確立**。
  正本: [#127完了記録](https://github.com/lisbun/lisjong-project/issues/127#issuecomment-6743073814)、[Policy status](https://github.com/lisbun/lisjong/blob/main/docs/policy-status.md)。
- [Arena #583](https://github.com/lisbun/lisjong-arena/issues/583)は準備中。二モデルlock #585・局記録 #587はmerge済み、development smoke/verify PASS。正式400半荘は未実行。
- 次の必須gateは[Arena #588](https://github.com/lisbun/lisjong-arena/issues/588)のcgroup / watchdog / 時間・容量上限 / 停止・隔離 / 失敗注入と実行環境の適格性。
  実装は[Arena PR #589](https://github.com/lisbun/lisjong-arena/pull/589)がopen（CI成功、development入力の失敗注入証拠あり）。独立レビューとmerge後のdevelopment再確認が残る。
- [Arena #571](https://github.com/lisbun/lisjong-arena/issues/571)でHeuristic ChampionのRiichiLab参加を確認中。60分pilot（11/7、dev 22・baseline 20半荘）は運用PASS・最終請求は未照合。
  承認済みの2 Bot 12時間参加を11/10 06:34 JSTに1回起動した（[記録](https://github.com/lisbun/lisjong-arena/issues/571#issuecomment-6752076699)、監視付きUSD 4）。終了・回収・teardownの結果は未記録。棋力判定・正式評価・Overallには使わない。

- Heuristic候補の開発比較（AI作業環境、費用0）: [lisjong #375](https://github.com/lisbun/lisjong/issues/375)・[#377](https://github.com/lisbun/lisjong/issues/377)・[#379](https://github.com/lisbun/lisjong/issues/379)は完了。#379（東1局6,000局）でリーチ下1向聴の押し引き候補RRだけがSUPPORTED-DEVELOPMENT（+210点/分岐、95% [+93, +329]）。形式聴牌・通った牌は区間が0を含み、単独副露者の現物は平均が負。
  東1局の1局収支だけの結果で、Championは変更していない。半荘での開発比較は[Arena #591](https://github.com/lisbun/lisjong-arena/issues/591)（計画案、未承認）。

# Next

1. PR #589のレビュー・mergeとmerge後のdevelopment再確認で#588の残gateを確認し、#583のWSL/AWS選択と資源・費用上限を含む具体的な実行計画を確定する。別承認後に新seed予約・formal lock・400半荘・verifyへ進む。
2. Arena #591の計画（K・環境・判定規則）の承認を得て、RRの半荘開発比較を1回行う。結果が良ければ正式評価の計画（#83のgate）へ進む。
3. [#84](https://github.com/lisbun/lisjong-project/issues/84) / [#74](https://github.com/lisbun/lisjong-project/issues/74)、[lisjong #297](https://github.com/lisbun/lisjong/issues/297)で新仮説の比較材料を整理する。対象判断・頻度/support・既存結果と異なる根拠・収支経路・費用・停止条件を揃える。新案の採用は未決。
4. lisjong #285の推定研究は独立管理、#286はDEFERRED。project #87の保守、#97/#99/#100の自律loop準備は強化の一般的な前提にしない。lisjong #302は完了済み。
5. #571の12時間参加の終了後、strict回収・teardown・費用を#571へ記録する。延長・追加run・長期rating観測は別承認。固定Policy期間の表示レートは[registry](docs/riichilab-strength-registry.md)（最終観測2026-10-07）へ未反映。
6. 11月のAWS実額・残額は未照合。記録上の承認上限はArena #564が累計3、#582が10、#571が2＋4（USD、上限であり実額ではない）。次の課金実行の前にCost Explorerと照合し、月次の判断は#84へ記録する。

# 判断の前提

- R1今回実験（Arena #550）・P1今回実験（#558）はNOT-PREDICTIVEで終了。R1系列は再開条件付き保留。P1系列/Learning全体の棄却ではない。新仮説なしに再学習・追加分析・E4を始めない。
- R1 TRAIN 547000..547043 / TEST 547100..547143、P1 TRAIN 556000..556119 / TEST 556200..556319は使用済み。TESTを調整や独立評価へ流用しない。既使用pilot/rehearsalも独立評価に使わない。
- L2aの模倣判定は不変、OFFLINE-EVAL再利用なし。旧#573の正式seedはRETIRED、#582はCOMMITTEDでOverall TESTに流用しない。
- lisjong #379のseed 380000..385999、#377の377000..379399は使用済み。#375 / #377 / #379の標本を合算しない。RRのSUPPORTEDを半荘の強さ・昇格の根拠にしない。
- H5・paijia v1・genbutsu v1・call-value v1・旧#326を救済せず、大規模NN/RLへ自動移行しない。project #57 / Arena #317 / lisjong #286はDEFERRED。
- native必須。pin変更時は同revision wheelと必要gateを確認する。
- AWS月上限USD 20（既支出込み・credit適用前）。残額未照合、課金実行は別判断。PC使用・時間上限の承認は個別計画に限定する。
- 通常merge/closeはAGENTSの継続承認。無人loopは[HUMAN](HUMAN.md)、個別の依存は[BLOCKING](BLOCKING.md)を参照。
