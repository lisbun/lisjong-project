# Current focus

次の強化仮説を1件に絞る。現在地・配分・全Issue棚卸の正本は[#84](https://github.com/lisbun/lisjong-project/issues/84)の2026-10-24棚卸節。
確認日: 2026-10-24 JST。GitHub本文・コメント・状態を照合。実験artifact・請求実額の独立再検証は行っていない。

# Now

- [lisjong #297](https://github.com/lisbun/lisjong/issues/297) / [#74](https://github.com/lisbun/lisjong-project/issues/74): 次のHeuristic/Learning仮説を比較して1案へ。採用待ちの候補・次のLearning主実験は現在なし。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): 正式評価は未解禁。通過候補固有の標本数・環境・時計/上限・費用の計画が残る。
- [lisjong #328](https://github.com/lisbun/lisjong/issues/328) / [#87](https://github.com/lisbun/lisjong-project/issues/87): D3はArena #528で完了。lisjong #346のRust必須化/限定削除はClaude Code担当、その後に配布確認・Arena採用。
- [lisjong #302](https://github.com/lisbun/lisjong/issues/302): RiichiLab規則・実対局補助証拠の残件。未確認環境への適用を保留する。
- [#97](https://github.com/lisbun/lisjong-project/issues/97): ChatGPTが#97〜#100・Arena #480/#481の6件を並行担当。seed準備はArena PR #529、資源guardはPR #530。merge/検証状況は各PRを正本とする。[役割と残gate](docs/autonomous-loop-readiness.md)。

# Next

1. #297 / #74で最大3案を、対象判断・介入・発動頻度の根拠/不明点・攻撃損失・収支経路・検証費用・停止条件で比較する。#84で1案へ絞り、owner側の限定Issueへ渡す。根拠不足なら不足1点を解く診断を設計する。
2. 新案の局面test→頻度/実行品質→固定開発比較→別seed/別相手の再確認。通過時だけ#83の正式計画へ進む。旧候補の条件緩和を既定路線にしない。
3. Claude Codeの#346と競合しないArena/projectで自律loopを進める。#529/#530のreview/CI後、#297対象固定→#481の一経路接続→#99の順。#98未決値・定期起動・課金を自動承認しない。
4. #302の残る証拠を照合。Arena #480は手順準備、#481の実候補接続は対象固定後。lisjong #285は独立研究として設計可能。
5. 次仮説の設計確定時・最初のgate停止/結果時・11月初の月次確認でH/L配分と消費資源を見直す。#99/#100の準備は可能、無人運用はHUMANの条件を満たしてから。

# 判断の前提

- Champion維持。直近の候補停止と証拠は#84を参照。終了候補の追加seed・条件緩和による救済をせず、使用済みseedを独立確認へ流用しない。
- LearningのOFFLINE-EVALを再利用せず、追加CALL学習・大規模化・RLへ自動移行しない。
- 校正は強さの証拠ではない。別pairへのSD流用、AWS費用への単純換算をしない。
- project #57 / Arena #317 / lisjong #286はDEFERRED。外部source・自律loop・Rust移行の完成を強化実験の前提にしない。
- AWS上限は10月のみUSD 30、11月以降USD 20（既支出込み・credit適用前）。最新月総額・残額は未確認。
- 新規Launchは請求反映と未反映runを照合して個別承認。seedはlive ledgerで照合する。
- 通常開発のmerge/closeはAGENTS.mdの継続承認に従う。自律loopの自動merge・S3起動・leader/Champion更新・LLM定期稼働は別gateで未解禁。
