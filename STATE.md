# Current focus

L2aのLearning family昇格に続き、現行Heuristic対L2aのOverall正式比較が完了し、HeuristicをDIRECT Overall Championとしてユーザー承認により指定した。
新しい行動価値学習の主仮説は未選定で、今回の正式結果とは区別する。
確認日: 2026-11-23 JST（lisjong #384の結果と、推定研究の優先順位を追加。稼働・請求・artifactの独立再監査は行っていない）。

# Now

- Current pair: Heuristic = `placement-aware-speed-call`、Learning = exact L2a M1+R1、**Overall = `placement-aware-speed-call`（Heuristic / DIRECT）**。指定承認・正本は[project #69の2026-11-15記録](https://github.com/lisbun/lisjong-project/issues/69#issuecomment-6886584183)。[Policy status](https://github.com/lisbun/lisjong/blob/main/docs/policy-status.md)。
- [Arena #583](https://github.com/lisbun/lisjong-arena/issues/583) / [#588](https://github.com/lisbun/lisjong-arena/issues/588#issuecomment-6886499017) の直接比較は400/400 VALID、Heuristic superior（`D=L順位-H順位=+0.4775`、95% [+0.368,+0.587]）。正式seed 588100..588199 COMMITTED、初回lock-stage STOPの588000..588099 RETIRED。安全対策#588はcompleted。ローカルWSL / AWS費用0。追加seedや再実行は不要。
- [Arena #571](https://github.com/lisbun/lisjong-arena/issues/571)でHeuristic ChampionのRiichiLab参加を確認中。60分pilot（11/7、dev 22・baseline 20半荘）は運用PASS・最終請求は未照合。
  承認済みの2 Bot 12時間参加を11/10 06:34 JSTに1回起動した（[記録](https://github.com/lisbun/lisjong-arena/issues/571#issuecomment-6752076699)、監視付きUSD 4）。終了・回収・teardownの結果は未記録。棋力判定・正式評価・Overallには使わない。

- Heuristic候補の開発比較（AI作業環境、費用0）: [lisjong #375](https://github.com/lisbun/lisjong/issues/375)・[#377](https://github.com/lisbun/lisjong/issues/377)・[#379](https://github.com/lisbun/lisjong/issues/379)は完了。#379（東1局6,000局）でリーチ下1向聴の押し引き候補RRだけがSUPPORTED-DEVELOPMENT（+210点/分岐、95% [+93, +329]）。形式聴牌・通った牌は区間が0を含み、単独副露者の現物は平均が負。
  RRの半荘開発比較[Arena #591](https://github.com/lisbun/lisjong-arena/issues/591)（engine AABB、K = 500 block・3,000半荘、seed 591000..591499 COMMITTED）は平均 +0.85 pt・95% [−0.64, +2.35]・SD 17.1 ptで、区間が0を含みINCONCLUSIVE-DEVELOPMENT。Championは変更していない。
  RRのdora枚数を役込みの打点推定に替えた候補[lisjong #384](https://github.com/lisbun/lisjong/issues/384)（RRとの東1局対比較）は、SELECT最良の`C5200-ON-F3900`がCONFIRMで平均 −71.4点・95% [−248, +107]となり**NOT-SUPPORTED-DEVELOPMENT**（[結果](https://github.com/lisbun/lisjong/issues/384#issuecomment-6998090945)）。昇格させず、RRの扱いも変えない。

# Next

1. 候補RRの扱いを決める（未決、[#84の記録](https://github.com/lisbun/lisjong-project/issues/84)）。正式評価へ進む場合、今回の点推定どおりなら検出力80%に約3,150 block（約19,000半荘）が必要で、#83のgateも要る。追加seedでの救済はしない。
2. [#84](https://github.com/lisbun/lisjong-project/issues/84) / [#74](https://github.com/lisbun/lisjong-project/issues/74)、[lisjong #297](https://github.com/lisbun/lisjong/issues/297)で新仮説の比較材料を整理する。対象判断・頻度/support・既存結果と異なる根拠・収支経路・費用・停止条件を揃える。新案の採用は未決。
3. 推定研究は[lisjong #285](https://github.com/lisbun/lisjong/issues/285)（リーチ者の待ち推定v2）を優先する（2026-11-23ユーザー判断）。打牌行動尤度による他家待ち推定は研究候補としてDEFERRED、Issueは起票しない（[#84の記録](https://github.com/lisbun/lisjong-project/issues/84#issuecomment-6963636607)）。#285は強化仮説選定の一律の前提にしない。#286はDEFERRED。project #87の保守、#97/#99/#100の自律loop準備は強化の一般的な前提にしない。lisjong #302は完了済み。
4. #571の12時間参加の終了後、strict回収・teardown・費用を#571へ記録する。延長・追加run・長期rating観測は別承認。固定Policy期間の表示レートは[registry](docs/riichilab-strength-registry.md)（最終観測2026-10-07）へ未反映。
5. 11月のAWS実額・残額は未照合。記録上の承認上限はArena #564が累計3、#582が10、#571が2＋4（USD、上限であり実額ではない）。次の課金実行の前にCost Explorerと照合し、月次の判断は#84へ記録する。

# 判断の前提

- R1今回実験（Arena #550）・P1今回実験（#558）はNOT-PREDICTIVEで終了。R1系列は再開条件付き保留。P1系列/Learning全体の棄却ではない。新仮説なしに再学習・追加分析・E4を始めない。
- R1 TRAIN 547000..547043 / TEST 547100..547143、P1 TRAIN 556000..556119 / TEST 556200..556319は使用済み。TESTを調整や独立評価へ流用しない。既使用pilot/rehearsalも独立評価に使わない。
- L2aの模倣判定は不変、OFFLINE-EVAL再利用なし。旧#573の正式seedはRETIRED、#582はCOMMITTEDでOverall TESTに流用しない。Overall初回588000..588099はlock-stage STOPでRETIRED、正式PASSの588100..588199はCOMMITTEDで再使用しない。
- lisjong #379のseed 380000..385999、#377の377000..379399、#384のSELECT 386000..388399・CONFIRM 395000..400999は使用済み。#375 / #377 / #379 / #384の標本を合算しない。RRのSUPPORTEDを半荘の強さ・昇格の根拠にしない。
- H5・paijia v1・genbutsu v1・call-value v1・旧#326・#384を救済せず、大規模NN/RLへ自動移行しない。project #57 / Arena #317 / lisjong #286と打牌行動尤度の研究候補はDEFERRED。
- native必須。pin変更時は同revision wheelと必要gateを確認する。
- AWS月上限USD 20（既支出込み・credit適用前）。残額未照合、課金実行は別判断。PC使用・時間上限の承認は個別計画に限定する。
- 通常merge/closeはAGENTSの継続承認。無人loopは[HUMAN](HUMAN.md)、個別の依存は[BLOCKING](BLOCKING.md)を参照。
