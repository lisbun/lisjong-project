# Blockers

確認日: 2026-10-24 JST。下記は対象作業だけを止める。

| 対象 | 不足しているもの | 続行できる作業 |
| --- | --- | --- |
| #83のengine正式評価開始 | 候補固有の効果・分散・power・K・費用の事前計画、実行環境P3と時計/上限方式の判断。#517でJ1/N9は継続、J2の180秒は提案、約401秒の時計差は未解明。別pairの校正値を流用しない | 次の仮説選定と開発比較の設計。#472/#513/#514/#517の再実施は一般的な前提にしない |
| #481への実候補の接続 | lisjong #297での新しい探索対象とArena #480のlive割当・接続証拠（準備PR #529）。paijia v1は#522不通過、単独副露者genbutsu v1は#526低頻度停止。両者を既定候補にしない | 候補identity生成とseed手順の準備 |
| lisjong #286（DEFERRED） | 損失集中または予算配分による再開判断と、正例supportを得るpopulation設計 | リーチ者の待ち推定v2（#285）は独立に設計可能 |
| 外部sourceの用途拡大 | #86でHOLDの用途に対する根拠 | first-party sourceを使う次のLearning仮説の設計 |
| 確定安全牌のRiichiLab適用（lisjong #302） | server規則の確認と実対局補助検査の記録照合。「位置の不一致0件」「ロン違反0件」、engineとRiichiLabの証拠を分離する | 確認済み基盤を維持し、他の強化作業は続行 |
| 自律loopの運用開始 | #98項目4〜6・予算等の未決判断（HUMAN.md）。通常開発mergeの継続承認では解除しない | #99/#100の準備実装、通常開発のレビュー・merge |
| Rust必須化・Python計算削除（lisjong #328 / project #87） | D3準備はArena #528で完了。切替revisionのbundle保存/導入確認、Ubuntu CIのRust化と代替検証coverage。#342の準備完了は全revision/全環境の確認ではない | Claude Codeがlisjong #346を担当。自律loopと強化仮説の選定・限定開発は独立に続行 |
