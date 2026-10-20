# Current focus

L2a完了後の限定診断設計とengine正式評価の前提検証。現在地・順序提案の正本は[#84](https://github.com/lisbun/lisjong-project/issues/84)。
確認日: 2026-10-20 JST。Issue本文・コメント・状態・merge済みPRを照合。実験artifact・請求実額の独立再検証は行っていない。

# Now

- [#74](https://github.com/lisbun/lisjong-project/issues/74): L2aは設計・学習・評価・動作確認まで完了。打牌/Chi NOT ESTABLISHED、Pon PARTIAL。次は新しい限定診断の設計。旧#267の設計待ちは解消。
- [#83](https://github.com/lisbun/lisjong-project/issues/83): [Arena #472](https://github.com/lisbun/lisjong-arena/issues/472)のfixture検証待ち。正式実行は未解禁。
- [lisjong #302](https://github.com/lisbun/lisjong/issues/302): RiichiLab規則・実対局補助証拠の残件。基盤の正しさと候補の採否を分離する。
- [#97](https://github.com/lisbun/lisjong-project/issues/97): lisjong #297、Arena #480/#481が残る。#98項目4〜6と運用条件はHUMAN判断待ち。
- [Arena #506](https://github.com/lisbun/lisjong-arena/issues/506): source引継ぎ済み、費用追記待ち。学習の技術的blockerではない。

# Next（順序の提案）

1. #74でL2a後続診断の範囲を固定し、lisjongへ新しい子Issueを切り出す。第一候補は門前ChiのPass/Chi選択とChi候補間選択の分解、第二候補は門前1〜2向聴の打牌不一致。原因は未確定。
2. Arena #472で精算・bridge/backend・固定C0のrevision差を確認する。その後に実装・校正を起票する。固定revisionと現在pinを区別する。
3. lisjong #302の規則根拠と補助証拠を照合する。未確認なら適用を保留し、1・2は続行する。
4. lisjong #297の探索対象を1つ選定し、Arena #480の開発A/B割当手順から#481のS0〜S2へ接続する。
5. lisjong #285の待ち推定v2、project #87のRust次段階を限定して扱う。#99/#100は準備可能だが、運用は#98の未決判断に従う。

# 判断の前提

- L2aの[評価](https://github.com/lisbun/lisjong/issues/318#issuecomment-6270387381)と[16半荘動作確認PASS](https://github.com/lisbun/lisjong-arena/issues/507#issuecomment-6278840709)を区別する。Champion維持。#318内の救済やOFFLINE-EVALの再実行をせず、改善検証は新仮説・新seed・新事前登録。
- Arena #503は採用条件未達で完了。追加評価は今は行わない。project #57 / Arena #317 / lisjong #286はDEFERRED維持。
- 新しいseed予約はlive ledgerで照合する。使用済みの開発・評価seedを独立再確認や正式評価へ再利用しない。
- AWS上限は2026年10月のみUSD 30、11月以降USD 20（既支出込み・credit適用前）。#506まで約22.7 + #318推計3.43 = 月見込み約26.1、概算残額約3.9。請求・その他利用は未照合で確定残額ではない。
- 新規Launch前に請求反映と未反映runのCollect記録を照合し、個別承認を得る。月上限変更は起動承認を兼ねない。
- 自動merge・S3自動起動・leader / Champion自動更新・LLM定期稼働は未解禁。Learningを外部source・自律loop・Heuristic候補の完了待ちにしない。
