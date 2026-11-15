# Blockers

確認日: 2026-11-15 JST（Overall #583/#588の完了blockerを削除。他行は前回11/11の根拠を維持）。下記は対象作業だけを止める。

| 対象 | 不足しているもの | 続行できる作業 |
| --- | --- | --- |
| #83のengine正式評価開始 | 候補固有の効果・分散・power・K・費用の事前計画、実行環境P3と時計/上限方式の判断。#517でJ1/N9は継続、J2の180秒は提案、約401秒の時計差は未解明。別pairの校正値を流用しない | 次仮説の選定と開発比較の設計。#472/#513/#514/#517の再実施は一般的な前提にしない |
| Heuristic Championの更新（候補RR） | 半荘での正式評価の`CANDIDATE SUPERIOR`とgovernance記録。lisjong #379は東1局の開発比較で、昇格の根拠にならない。半荘の開発比較[Arena #591](https://github.com/lisbun/lisjong-arena/issues/591)はINCONCLUSIVE-DEVELOPMENT（+0.85 pt、95% [−0.64, +2.35]）で、優位は示せていない。正式評価は下の#83のgateも必要 | 正式評価へ進むか保留にするかの判断材料の整理（#84）。正式評価の計画・実行は承認後 |
| #481への実候補の接続 | 実候補の仕様・identityと適合する評価契約。Arena #480のlive割当・接続証拠。paijia v1（#522不通過）、genbutsu v1（#526）・call-value v1（Arena #536）は低頻度停止、H5（#542）はSTOP-NO-SIGNAL、R1（Arena #550）・P1（Arena #558）の今回実験はNOT-PREDICTIVEで終了。lisjong #375（形式聴牌）・#377はINCONCLUSIVE等で終了、#379はRRだけSUPPORTED-DEVELOPMENT（東1局のみ）。RR以外は既定候補にしない | 次仮説の比較・設計（project #84/#74）。Learning候補へ既存Heuristic契約を無条件流用しない |
| lisjong #286（DEFERRED） | 損失集中または予算配分による再開判断と、正例supportを得るpopulation設計 | リーチ者の待ち推定v2（#285）は独立に設計可能 |
| 外部sourceの用途拡大 | #86でHOLDの用途に対する根拠 | first-party sourceを使う次のLearning仮説の設計 |
| 次のAWS課金実行（[Arena #571](https://github.com/lisbun/lisjong-arena/issues/571)の延長・長期観測等） | 実行中の#571 12時間参加の終了・回収・teardown記録、11月のAWS実額と月上限USD 20の照合、個別承認 | WSL等での費用を伴わない開発・検証と計画作成 |
| 自律loopの運用開始 | #98項目4〜6・予算等の未決判断（HUMAN.md）。通常開発mergeの継続承認では解除しない | #99/#100の準備実装、通常開発のレビュー・merge |
