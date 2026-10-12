# ADR 0011: Autonomous strength loop decisions

## Status

Partially decided — 2026-10-12。項目1〜3はユーザー承認済み、4〜6は未決。自動merge・S3自動起動・leader / Champion自動更新の運用は未解禁。

正本: [project #98](https://github.com/lisbun/lisjong-project/issues/98)、親[project #97](https://github.com/lisbun/lisjong-project/issues/97)。項目1・2は[判断コメント](https://github.com/lisbun/lisjong-project/issues/98#issuecomment-6122880561)、項目3は同日20:49 JSTのユーザー判断を反映した#98本文による。本ADRの作成・mergeだけで未決項目を承認済みにしない。

## Context

候補生成から評価までの自律loopでは、Policyの提案・実装と評価規則の変更を分離し、正式評価を開発への適合から守る必要がある。同時に、手書き規則の閾値探索とLearned Policyを同じfamilyにすると、蒸留等の進捗と規則調整の進捗が混ざる。

## Confirmed decisions

### 1. 自動更新の範囲

research-track leaderとChampion昇格の自動更新を許可する。Champion昇格は、候補のfamilyに適合したS3の判定ラベル・判定式・標本数を実行前に固定し、ADR 0004のbaseline establishment / family promotionとADR 0005のOverall designationを区別して扱う。S1/S2は開発選別であり、Champion昇格の根拠にしない。稼働botの切替は引き続き別判断。

具体的な累積偽陽性対策・試行上限・予算が未決のため、自動更新はまだ有効化しない。

### 2. 候補レーンの自動merge

次の全条件を満たす候補PRに限り、mergeの条件付き事前承認を与える。

- 変更がlisjongの候補用moduleとそのtestに限られる。
- protocol・default Policy・Champion・catalogの既存entryに触れない。
- CIが通過し、実装とcontextを共有しない別sessionのレビューでblocking指摘がない。
- lisjong #296の情報境界チェックに合格する。

これは各ownerのAGENTS.mdが求める明示的なmerge承認を、限定したレーンに事前付与する決定である。一般の文書・評価基盤・設定変更への包括的な自動merge承認ではない。有効化前に各ownerのAGENTS.mdと実装がこの境界を表現していることを確認する。項目4〜6が未決の間は実装を進めてよいが、自動mergeを有効化しない。

### 3. Policy family

[ADR 0004](0004-champion-family-separation.md)の分類を改訂する。

| 構成 | 分類 |
| --- | --- |
| 手書き規則・固定閾値 | Heuristic |
| 手書き規則の閾値・重みをgrid / optimizer / 評価を見たLLMで調整 | Heuristic |
| 学習した待ち・危険度推定器を手書きの判断規則へ供給 | Heuristic＋学習推定器（familyはHeuristic） |
| 学習した行動・行動価値・rankingモデルが主たる判断を担う | Learning（NN・回帰・決定木等） |

学習した行動モデルを定数表へ変換してもLearningのまま。分類は名称・保存形式ではなく学習対象と推論での役割による。第3のHybrid familyは設けない。

本項は同日の先行判断「探索・評価後の定数変更はLearning」を置き換える。lisjong #297の手書き規則の閾値探索はHeuristic側、#267の蒸留はLearning側とし、自律loopは両方を支える。過去の固定済み実験の分類・結果は遡及変更しない。

分類と調整方法は別管理とする。調整方法・使用データ・候補identityを記録し、開発、別seed・別相手による再確認、未使用seedによる正式評価を分離する。結果を見て変更した候補に同じデータを独立検証として流用しない。Heuristicという分類は過適合対策の免除ではない。

## Pending decisions

以下は未回答であり、本ADRでは既定値を補わない。

4. leaderが5回交代するごとにC0とS3規模で比較することを必須にするか。
5. 仮説の系統ごとのS3試行上限。
6. 月予算（AWS月$20総額内の配分・LLM利用）、GitHub ActionsのAWS用secret設定、停止スイッチの場所。

最新の人の判断事項は[HUMAN.md](../../HUMAN.md)を参照。4〜6が決まるまで自動merge・S3自動起動・leader / Champion自動更新を有効化しない。Phase 1とLLM構成・安全装置の準備実装は進めてよい。LLM sessionの定期稼働は予算決定まで開始しない。

## Evaluation and ownership

- LLMは仮説・実装を担当し、数値・合否は不変artifactと機械判定を正本にする。候補sessionがprotocol・seed・判定規則を書き換えることは許可しない。
- S0〜S2とS3は別段階。Heuristicのengine正式評価移行は、fixture検証・実装・校正等の解禁条件を満たしてから利用する。分類変更だけで正式実行を解禁しない。Learning候補には適合する正式評価契約・比較先が必要。
- Policy・学習・推定の意味はlisjong、実行・seed・正式評価はArena、横断方針はprojectが所有する。学習推定器の情報境界とartifact管理は、利用先PolicyのfamilyによらずADR 0008を維持する。
- 課金・secret設定・repository settings・稼働bot切替への承認を今回の文書更新から拡張しない。

## Consequences and completion

閾値探索をHeuristicの改善として追跡し、Learningでは学習モデルが判断を担う能力の進捗を追跡できる。両familyで試行履歴・データ分離・正式評価の規律を維持する。

本ADRは#98の途中成果である。残る判断、S3/昇格の具体条件、各ownerの承認境界との整合が揃うまで#98をcloseしない。自律loopの実装・運用完了は#97と子Issueで確認する。
