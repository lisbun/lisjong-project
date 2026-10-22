# Current focus

次の強化仮説を1件に絞る。現在地・配分・全Issue棚卸の正本は[#84](https://github.com/lisbun/lisjong-project/issues/84)。
確認日: 2026-10-23 JST。GitHub本文・コメント・状態を照合。実験artifact・請求実額の独立再検証は行っていない。

# Now

- [lisjong #297](https://github.com/lisbun/lisjong/issues/297): 次の探索対象を再選定。paijia v1は[Arena #522](https://github.com/lisbun/lisjong-arena/issues/522)でNOT-RECONFIRMED-DEVELOPMENT、追加対局・正式評価への進行を終了。
- [#74](https://github.com/lisbun/lisjong-project/issues/74): L2aとCALL特徴追加案の評価は完了。次のLearning主実験は未選定。模倣診断の追加を目的にせず、判断変更・期待収支・検証費用へ接続する。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): engine評価のfixture・record・protocol実装と[Arena #517](https://github.com/lisbun/lisjong-arena/issues/517)の限定校正は完了。候補固有の標本数・環境・上限の計画が残り、正式評価は未解禁。
- [lisjong #328](https://github.com/lisbun/lisjong/issues/328) / [#87](https://github.com/lisbun/lisjong-project/issues/87): Python重複計算の棚卸・仕様ベース検証への移行設計に着手可能。強化仮説選定と独立。
- [lisjong #302](https://github.com/lisbun/lisjong/issues/302): RiichiLab規則・実対局補助証拠の残件。未確認環境への適用を保留する。
- [#97](https://github.com/lisbun/lisjong-project/issues/97): 候補生成、Arena #480のseed手順、#481のS0〜S2が残る。#98項目4〜6はHUMAN判断待ち。

# Next（順序の提案）

1. #84 / lisjong #297 / #74で、既存証拠からHeuristic・Learningの仮説を比較し1件選ぶ。対象判断、損失の仕組み、介入、変更頻度、期待効果、検証費用、停止条件を明記してowner側の限定Issueへ渡す。
2. 選んだ仮説だけを実装・安価な開発比較へ進める。通過時に別seed・別相手で再確認し、その後に#83で候補固有の正式計画を判断する。
3. 並行してlisjong #328の移行範囲・代替検証・wheel対応を設計する。削除・pin変更・追加対局を先行しない。
4. lisjong #302の残る証拠を照合。Arena #480は手順を準備し、#481への実候補接続は#297の選定後。lisjong #285の推定v2は独立設計可能。
5. #98の未決事項を整理し、#99/#100の準備を進める。11月初の#84月次見直しで成果・消費資源・H/L配分を確認する。

# 判断の前提

- Champion維持。#522の不通過は効果不存在の証明ではない。#520とendpointが異なるため合算しない。旧候補の追加seed・条件緩和による救済はしない。
- LearningのCALL案はNOT-SUPPORTED-DEVELOPMENT。OFFLINE-EVALを再利用せず、追加CALL学習・大規模化・RLへ自動移行しない。
- #517はCALIBRATION COMPLETE; NOT A STRENGTH RESULT。別pairへのSD流用、AWS費用への単純換算をしない。
- project #57 / Arena #317 / lisjong #286はDEFERRED。外部source・自律loop・Rust移行の完成を強化実験の前提にしない。
- AWS上限は10月のみUSD 30、11月以降USD 20（既支出込み・credit適用前）。#506費用追記は完了。最新月総額・残額は未確認で、旧概算残額を実行判断に使わない。
- 新規Launchは請求反映と未反映runを照合して個別承認。seedはlive ledgerで照合する。
- 通常開発のmerge/closeはAGENTS.mdの継続承認に従う。自律loopの自動merge・S3起動・leader/Champion更新・LLM定期稼働は別gateで未解禁。
