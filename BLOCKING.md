# Blockers

確認日: 2026-11-11 JST。下記は対象作業だけを止める。

| 対象 | 不足しているもの | 続行できる作業 |
| --- | --- | --- |
| [Arena #583](https://github.com/lisbun/lisjong-arena/issues/583)のOverall正式比較 | [#588](https://github.com/lisbun/lisjong-arena/issues/588)の実行保護・失敗注入・環境適格性、WSL/AWS選択、時間/メモリ/容量/費用上限と実行計画の承認。二モデルlock #585・局記録 #587・development smokeは完了済み | [PR #589](https://github.com/lisbun/lisjong-arena/pull/589)の独立レビュー・merge、merge後のdevelopment再確認、計画の具体化。formal seed予約・対局はまだ行わない |
| #83のengine正式評価開始 | 候補固有の効果・分散・power・K・費用の事前計画、実行環境P3と時計/上限方式の判断。#517でJ1/N9は継続、J2の180秒は提案、約401秒の時計差は未解明。別pairの校正値を流用しない | 次仮説の選定と開発比較の設計。#472/#513/#514/#517の再実施は一般的な前提にしない |
| Heuristic Championの更新（候補RR） | 半荘での正式評価の`CANDIDATE SUPERIOR`とgovernance記録。lisjong #379は東1局の開発比較で、昇格の根拠にならない。半荘の開発比較[Arena #591](https://github.com/lisbun/lisjong-arena/issues/591)は計画案で未承認。正式評価は下の#83のgateも必要 | #591の計画承認後の実装・開発用seedでの記録経路確認。seed予約・本計測は承認後 |
| #481への実候補の接続 | 実候補の仕様・identityと適合する評価契約。Arena #480のlive割当・接続証拠。paijia v1（#522不通過）、genbutsu v1（#526）・call-value v1（Arena #536）は低頻度停止、H5（#542）はSTOP-NO-SIGNAL、R1（Arena #550）・P1（Arena #558）の今回実験はNOT-PREDICTIVEで終了。lisjong #375（形式聴牌）・#377はINCONCLUSIVE等で終了、#379はRRだけSUPPORTED-DEVELOPMENT（東1局のみ）。RR以外は既定候補にしない | 次仮説の比較・設計（project #84/#74）。Learning候補へ既存Heuristic契約を無条件流用しない |
| lisjong #286（DEFERRED） | 損失集中または予算配分による再開判断と、正例supportを得るpopulation設計 | リーチ者の待ち推定v2（#285）は独立に設計可能 |
| 外部sourceの用途拡大 | #86でHOLDの用途に対する根拠 | first-party sourceを使う次のLearning仮説の設計 |
| 次のAWS課金実行（[Arena #571](https://github.com/lisbun/lisjong-arena/issues/571)の延長・長期観測、#583でAWSを選ぶ場合） | 実行中の#571 12時間参加の終了・回収・teardown記録、11月のAWS実額と月上限USD 20の照合、個別承認 | WSLでの#588検証、費用を伴わない計画作成 |
| 自律loopの運用開始 | #98項目4〜6・予算等の未決判断（HUMAN.md）。通常開発mergeの継続承認では解除しない | #99/#100の準備実装、通常開発のレビュー・merge |
