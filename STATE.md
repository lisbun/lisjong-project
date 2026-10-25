# Current focus

主仮説H5（リーチ/ダマ判断）の一度限りの分岐測定と結果待ち。
正本: [Arena #542](https://github.com/lisbun/lisjong-arena/issues/542)、[#84](https://github.com/lisbun/lisjong-project/issues/84)、[#74](https://github.com/lisbun/lisjong-project/issues/74)。確認日: 2026-10-25 JST。
GitHub記録とユーザー承認を照合。PC上のartifact・請求実額を独立再検証していない。

# Now

- H5の設計[Arena #539](https://github.com/lisbun/lisjong-arena/issues/539)、解析API lisjong #352、分岐実装Arena #540は完了。赤5評価の修正lisjong #354 / PR #355とArena PR #543も反映済み。
- [#542修正版](https://github.com/lisbun/lisjong-arena/issues/542#issuecomment-6393357088): Arena a5b6098 / lisjong 84e9193 / features v2。新pin gate PASS、rehearsal-3 MATCH、23分岐INVALID 0、正式見積もり9,504.9秒でWITHIN-BUDGET。旧rehearsal-2のSTOP-INVALIDと保存物は保持。
- [#542本計測前記録](https://github.com/lisbun/lisjong-arena/issues/542#issuecomment-6393454629): seed照合・予約から本計測・verify・回収まで承認済み。539000..539023の全domain照合は重複なし。記録確認時点では予約readback・本計測開始/完了の証拠は未確認。最新状態はowner Issueを参照し、重複起動しない。
- [#74](https://github.com/lisbun/lisjong-project/issues/74): Learning主実験は未選定。H5記録の再利用設計は可能だが、学習・追加source生成は開始しない。
- [#97](https://github.com/lisbun/lisjong-project/issues/97): 実候補列は未確定。H5のPROCEEDは候補設計への入口であり、#481のS0〜S2完走や棋力改善ではない。
- [#98 / #100の準備](docs/autonomous-loop-readiness.md): 運用判断・実controller接続は残る。文書整備と合成入力の準備は並行可能。

# Next

1. #542の実行担当が固定planの本計測・固定2 seedの再実行verify・hash/回収・allocation状態を記録する。本実行＋verify＋回収4時間、失敗分を含む累積6時間。全工程に含めるsetup/回収の時間も分けて記録し、scope内時間だけを全時間と呼ばない。
2. 固定判定がPROCEEDなら該当部分集合の候補仕様と開発比較予算を新Issueで設計。それ以外は理由を記録して本versionを終了し、#84 / #74で配分を見直す。追加seed・閾値緩和で救済しない。
3. 並行: #100の開始前guard・実行中監視・停止後再開の接続証拠を整理。#98の未決値は補完せず、#99の定期起動・自動昇格は開始しない。
4. 11月初に進捗・費用・H/L配分を見直す。新規AWS Launch前には請求と未反映runを照合する。
5. lisjong #302のRiichiLab補助証拠、#285の独立推定研究は別管理。#87の全面Python非依存化は主作業の前提にしない。

# 判断の前提

- Champion維持。call-value v1は[Arena #536](https://github.com/lisbun/lisjong-arena/issues/536#issuecomment-6386231064)でSTOP-LOW-ACTIVATION、完了。paijia v1 / genbutsu v1も終了済み。
- #83の正式評価は候補固有のpower・標本数・環境・時計/上限・費用計画待ち。現在対象候補なし。
- LearningのOFFLINE-EVAL再利用、CALL再学習・大規模化・RLへの自動移行なし。project #57 / Arena #317 / lisjong #286はDEFERRED。
- native必須。pin更新時は同revision wheelと新gate証拠を確認し、WSLのPASSをAWSへ流用しない。
- AWS上限は10月USD 30、11月以降USD 20（既支出込み・credit適用前）。現在残額は未照合。
- 本計測と並行する作業は実行checkout・venv・pinを変更せず、追加対局・学習・重いtestを起動しない。
- 通常merge/closeはAGENTSの継続承認。無人loopの未決条件は[HUMAN](HUMAN.md)を参照。
