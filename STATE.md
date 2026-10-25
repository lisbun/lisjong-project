# Current focus

限定的な行動価値学習を成立させるための設計。
正本: [lisjong #356](https://github.com/lisbun/lisjong/issues/356)、[#74](https://github.com/lisbun/lisjong-project/issues/74)、[#84](https://github.com/lisbun/lisjong-project/issues/84)。確認日: 2026-10-25 JST。
GitHub記録とユーザーの方向性承認を照合。PC上のartifact・請求の独立再検証は行っていない。

# Now

- #356を設計のみで起票。公開情報から行動間の価値差を学び、未使用データで判断改善を確かめる経路を主作業とする。対象・モデル・規模・実行条件はこれから固定する。
- [H5結果](https://github.com/lisbun/lisjong-arena/issues/542#issuecomment-6394749556): #542はSTOP-NO-SIGNALで完了。S1は11件で不足、S3は53件/21 seedsだが平均Δ/P10非正。286分岐INVALID 0、verifyと固定2 seed再実行一致、allocation COMMITTEDを記録上確認。
- H5のsourceは開発利用の候補。局内リーチ抑止wrapperの対の収支であり、期待値/最適行動の正解ではない。学習・新規source生成は開始していない。
- #297 / Arena #481は実候補待ち。新しいLearning候補のidentity・評価契約が決まるまで、Heuristic探索経路へ暗黙に接続しない。
- #98の運用判断と#99/#100の準備は並行可能。自律loopや全面Python非依存化の完成を#356の設計の前提にしない。

# Next

1. #356で対象1つ、行動・継続方策・報酬、player-safe入力、sourceとseed分割を固定する。初期案はリーチ判断の価値差だが、H5の救済として自動採用しない。
2. 最小モデル/比較先、ノイズ・support、必要独立seed数と費用、評価・停止条件を整理。未取得値は未取得とする。
3. DESIGN-READYならlisjongの学習/Policy実装とArenaの生成/評価を必要な単位で起票し、実行は別Issue。DESIGN-NOT-VIABLEなら根拠を記録して配分を見直す。
4. 11月初に進捗・費用・H/L配分を見直す。形式聴牌Heuristic診断は既定の次作業にしない。
5. lisjong #302の補助証拠、#285の推定研究、project #100の制御準備は独立管理。

# 判断の前提

- Champion維持。H5・paijia v1・genbutsu v1・call-value v1を追加seed/条件緩和で救済しない。
- H5の539000..539023とrehearsal seedは既使用DEVELOPMENT。後から未使用TESTへ改称しない。同一seedの全局/席/R/D派生を同一groupにする。
- 既存OFFLINE-EVALを再利用せず、旧#326・大規模NN・RLへ自動移行しない。project #57 / Arena #317 / lisjong #286はDEFERRED。
- #83の正式評価は候補固有の契約待ち。予測精度、判断改善、半荘棋力を区別する。
- native必須。pin変更時は同revision wheelと必要gateを確認する。
- AWS上限は10月USD 30、11月以降USD 20（既支出込み・credit適用前）。残額未照合、課金実行は別判断。
- 通常merge/closeはAGENTSの継続承認。無人loopの条件は[HUMAN](HUMAN.md)を参照。
