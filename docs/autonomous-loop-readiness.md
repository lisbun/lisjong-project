# 自律ループの準備と運用開始条件

親: [#97](https://github.com/lisbun/lisjong-project/issues/97)。権限の正本は
[ADR 0011](decisions/0011-autonomous-strength-loop.md)。本書は未決値を承認しない。
実装済み・提案・実行証拠を分け、単発実験を無人loop完走と扱わない。

## 6件の完了境界

| Issue / owner | 今進める単位 | その後に必要な証拠 |
| --- | --- | --- |
| #97 / project | 依存・担当・完了証拠の索引 | 子Issueの完了、対象候補列の無人S0〜S2、L1の独立session記録 |
| #98 / project | 未決項目4〜6の具体案と既決条件の整合 | ユーザー判断、ADRへの確定値反映。文書mergeは運用承認ではない |
| #99 / projectと各実装owner | 下記の役割・handoff契約、既存skillの再利用 | skill実装、定期起動設定、L1の1サイクル。独立Reviewerはfresh sessionで実施 |
| #100 / Arena（実装）・project（方針） | 資源guardと停止理由の機械出力 | authoritative source取得、費用予約直列化、全開始経路への接続、通知、月次集約 |
| Arena #480 | S1/S2要求生成・予約後照合、formal容量検査 | live予約→lock→既知development実行→strict readback→commitの接続記録 |
| Arena #481 | 一つの探索空間・一つの開発経路の入出力を固定 | #297の候補列でS0〜S2を無人完走し、局単位記録と順位artifactをstrict readback |

具体的な実装進捗はIssue/PRを正本とする。Arenaの
[seed準備](https://github.com/lisbun/lisjong-arena/pull/529)と
[資源guard](https://github.com/lisbun/lisjong-arena/pull/530)は途中PR。
存在すること・単体test成功をmerge済みや運用済みと読み替えない。

## 依存順序と並行担当

1. Arena #480の予約契約と#100のguard判定部は並行実装できる。
2. lisjong #297の探索対象・identity・候補列、S0 fixture、S1の評価経路・指標・半減schedule、
   S2の別seed/別相手・再確認基準を事前固定する。終了したpaijia v1 / genbutsu v1を既定候補に戻さない。
3. Arena #481で既存の一つの開発比較経路に接続し、#480のlive bindingと#100のguardを各開始前に確認する。
4. 非LLMのS0〜S2が成立してから、#99の仮説→実装→独立レビューを接続する。
5. #98の未決条件と#83の候補固有正式gateが解消した後だけ、S3/自動昇格を別の実装単位で扱う。

2026-10-24の担当分離: lisjong #346はClaude Code、自律loop6件はChatGPT側。
loop準備からlisjongのRust selector・計算core・pin/wheelを変更しない。
#346の完了時には新revisionの配布確認とArena採用を別に照合する。
自律loopとRust移行の完成を、依存しない棋力改善の一律の前提にしない。

## #99: session間のhandoff契約

| 役割 | 読むもの | 残すもの / 許される変更 |
| --- | --- | --- |
| Analyst | STATE、確定評価artifact、局単位指標、診断の出典 | 損失集中と反例・不明点、最大3仮説の比較。数値は機械生成artifactへの参照 |
| Planner | Analystの根拠、現protocol、費用・seed・能力制約 | 仮説Issue。対象判断・変更path・期待効果・S0/S1/S2条件・S3標本数根拠または未到達・撤退条件を事前登録 |
| Implementer | 固定Issue、AGENTS、許可path、baseline revision | 候補module/testのPRと検証結果。既存`implement-issue` / `self-review` / `address-review` / `finalize-pr`を再利用 |
| Reviewer | Issue、PRのexact head、差分、CI、owner契約 | 情報境界・合法手・事前登録・所有境界のfinding。blocking/non-blocking、対象headを記録 |
| 決定的controller | reviewed candidate identities、固定protocol、live ledger、guard snapshot | S0〜S2実行とimmutable artifact・機械集計。LLMへ合否や数値の転記を委ねない |

ReviewerはImplementerの会話contextを引き継がない。Issue/PR/fixtureなどのdurable sourceは共有してよい。
この通常作業のセルフレビューを、L1の独立Reviewer実績として数えない。

handoffにはowner Issue URL、baseline SHA、candidate/parameter identity、変更許可path、
事前登録version、artifact identity/path、禁止操作、停止理由を残す。
sessionの未保存メモだけで再開しない。未知artifact/schema、出典欠落、baseline変化は停止し、推測で補完しない。

Routineの接続案は週初めのAnalyst/Planner、IssueごとのImplementer、PRごとの独立Reviewer。
現在は設定・有効化しない。週session/PR上限、費用取得方式、実際の起動サービスを確定してから接続する。
ここに書いた役割はskill実装・インストール済みの意味ではない。

## #481: 最小実行契約を固定するときの確認

- 候補identityはclass文字列だけでなくlisjong revision・パラメータ・artifact identityを含める。
  意味と候補生成はlisjong-owned。Arenaに候補生成器を複製しない。
- S0は#296の情報境界検査を再利用。合法手、例外、固定fixtureの決定性hash、処理時間上限を事前固定する。
- S1は同じC0との比較、候補数・各roundの予算・上位k・tie順・失敗時の扱いを実行前に固定。
  収支等の選別指標は対象決定後に固定し、既存formal判定式を開発用に上書きしない。
- S2は別seed Bと別の相手構成。Bの結果を見た調整後、同じBを独立再確認として使わない。
- 使用する局単位記録schema・集計母数・strict readerを一つの既存経路に固定する。
  勝率・放銃率・平均打点/失点を同じ実行から集計できることを開始前に確認する。
- 不正候補/失敗gameを黙って除外して順位を出さない。停止を結果の弱さと混同しない。
  実験失敗でもseedをFREEへ戻さない。再試行・resume・追加seedはv0に入れない。
- 順位レポートはDEVELOPMENT ONLY。S1/S2通過でChampionを更新しない。

#297未確定の段階で、架空の候補列・一般化したbackend registry・複数protocol controllerを先に作らない。
合成fixtureの配線検査と実候補の完走記録を区別する。

## #98: 残る判断の具体案（未承認）

以下はreview用の提案であり、採用値ではない。未回答時はADRの停止条件を維持する。

| 項目 | 初期案 | 限界・未決 |
| --- | --- | --- |
| 4 累積対策 | leader 5回交代ごとのC0比較を必須にする | baseline driftの点検であり、多重比較の偽陽性制御をこれだけで保証しない。S3の試行管理は別に必要 |
| 5 S3試行上限 | 仮説系統ごとにまず1回。結果を見た派生候補も同じ系統に含める | 追加試行は新たな判断と事前計画。名前/Issueの付替えで回数をリセットしない |
| 6 予算 | 既存AWS総額上限内でloop配分を明示、LLMも金額・計測方式・週session/PR上限を指定 | 現在残額・LLM配分は不明。契約済みサービス内の利用も無制限とは扱わない |
| 6 停止スイッチ | project mainの`LOOP_PAUSED`の存在を停止とする | 場所は未承認。取得失敗・古いsnapshotは停止、削除の取得成功だけでは他gateを免除しない |
| 6 secret | local S0〜S2の準備では新しいAWS secretを追加しない | AWSが必要になった時に対象と最小権限を別途判断 |
| #100 停滞 | 3サイクル連続で昇格なしなら新規開始を停止して見直す | 未承認の初期案。「3か月改善なし」の長期レビュー条件とは別。未確定値を実装defaultにしない |

本書のmergeで#98をcloseしない。実運用設定は承認記録へリンクし、candidate sessionから変更できない場所に置く。

## #100: integrationの完了証拠

pure guardへの合成入力testだけで完了とはしない。次をcontroller単位で確認する。

- pause中・source取得不能・期限切れ・予算未決・見積り超過・停滞・週上限の各ケースで、新しい実行を開始しない。
- authoritativeな費用snapshotはcredit適用前の総支出と未反映runを照合し、二重計上しない。
- 費用予約と開始を単一writerで直列化し、同時起動が同じ残予算を二重消費できない。
- STOPと80%警告を設定済み宛先へ届ける。未設定の通知を送信済みとしない。
- 月次ダイジェストはimmutable artifactから機械生成し、費用/試行/候補数/停止理由を再計算できる。
  LLM所見は別欄。校正・開発選別・正式強さを混同しない。

実運用開始前に、#98の判断、#99の起動構成、#100のintegration証拠を揃える。
