# ADR 0004: Champion family separation

## Status

Accepted. Family classification amended on 2026-10-12 by [project #98](https://github.com/lisbun/lisjong-project/issues/98), superseding the previous parameter-origin-only definition. See [ADR 0011](0011-autonomous-strength-loop.md).

## Context

lisjongではこれまで、主にheuristic / mechanism baseのPolicyを改善し、Arena上で比較評価してきた。今後learned policyの研究を進めるにあたり、HeuristicとLearningを単一のChampion系統だけで管理すると問題が生じる。

Learning系統は初期段階でHeuristic系統に及ばない期間が長くなり得る。単一Championだけを持つと、その期間の内部改善、すなわち「Learning系統そのものが前進しているか」を追跡しにくい。

一方で、最終的な強さの主張はarchitectureに依存しない共通evidenceで比較したい。familyごとに有利なmetricや条件を選べる状態では、「learned policyがhandcrafted policyを上回った」というclaimが再現可能な意味を持たない。

つまり、family内部の改善追跡と、family間のarchitecture-independentな強さ比較は、要求が異なる別のevaluation concernである。

さらに、`Champion` designationがまだ存在しない状態は正常であり、それを埋めるためにdesignationを緩めるべきではない。片方のfamilyだけがChampionを持つ状態で、そのChampionを自動的に全体最強として扱うと、cross-family evidenceなしにOverall claimが成立してしまう。

## Decision

### Champion families

正式なChampion familyは次の2つとする。

```text
Heuristic Champion
Learning Champion
```

`Heuristic Champion` は、主たる行動選択を手書き規則・明示的なアルゴリズムで行うPolicyのうち、Heuristic family内で最強と認定されたものを指す。規則の閾値・重みをgrid / optimizerや開発評価を見たLLMで調整しても、このfamilyとする。

`Learning Champion` は、主たる行動選択（打牌・行動価値・押し引き）をデータから学習したモデルが担うPolicyのうち、Learning family内で最強と認定されたものを指す。NNに限定せず、回帰・決定木等を含む。

### Family classification boundary

分類の軸は主たる行動選択の作り方であり、調整方法・model fileの有無・保存形式とは別に記録する。

| 行動選択の構成 | family / 表記 |
| --- | --- |
| 手書き規則、固定した閾値・重み | Heuristic |
| 手書き規則の閾値・重みをgrid / optimizer / LLMで調整 | Heuristic。調整方法・使用データを記録 |
| 学習した待ち・危険度推定器を手書きの判断規則へ供給 | Heuristic＋学習推定器（familyはHeuristic） |
| 学習モデルが行動・行動価値・候補rankingを決定 | Learning |
| heuristic候補生成＋学習した行動scoring / reranking | Learning |

例えば手書きの `score = 3.7 * ukeire + 1.2 * dora - 2.8 * risk` の重みを開発比較で調整する場合はHeuristic。行動価値や行動rankingを目的変数として回帰モデルを学習する場合は、同じ線形式でもLearningとして扱う。学習した行動モデルを定数表へ変換してもLearningのままとする。境界が曖昧なら学習対象・目的関数・推論での役割を候補Issueへ明記して、評価前に分類を固定する。

学習部品のdiagnostic / logging専用利用はPolicy familyを変えない。独立した `Hybrid Champion` は設けない。「Heuristic＋学習推定器」は構成の説明であり第3のfamilyではない。推定器自体の学習・artifact・情報境界の責務はADR 0008に従う。

### Classification and evaluation provenance

Heuristicでも探索・評価後の調整による過適合は起こり得る。調整方法、使用データ、候補identityを記録し、開発用、別seed・別相手構成による再確認、未使用seedによる正式評価を分離する。結果を見て変更した候補に、同じデータを独立検証として流用しない。family名を理由に評価規律を緩めない。

この改訂は過去の固定済み実験のfamily・artifact・判定・Champion記録を遡及変更しない。新候補の分類を評価前に固定し、既存protocolの適用条件・昇格条件は別途満たす。

### Overall Champion is a derived designation

`Overall Champion` は第3のChampion familyではなく、projectのcurrent canonical strength-evaluation protocolによるcross-family evidence、またはADR 0005で定義するwinning-family successionによって保持される派生designationとする。

`Overall` はあらゆるmetric / opponent / environmentに対する普遍的な最強を意味しない。

### `not established` is a valid state

family Championと `Overall Champion` の双方について、`not established` を正式な状態として許容する。

原則として次を同一視しない。

```text
family Champion
        !=
Overall Champion
```

片方のfamilyのChampionが未確立であることだけを理由に、他方のfamily ChampionをOverall Championへ自動昇格させない。`Overall Champion` の初回確立、winning familyの変更、またはcurrent losing-family Championの変更後の再確立には、比較可能なfamily Championが存在し、cross-family formal evaluation evidenceが得られていることを要求する。current Overall-winning familyがvalidなfamily-internal promotionで自身のChampionを置き換え、opposing family Championが不変である場合だけ、ADR 0005のlifecycle ruleに従ってOverall designationをsuccessorへ継承できる。

Championがまだ存在しないfamilyでは、通常の `candidate vs current Champion` promotionを実行できない。初代Championには、通常promotionとは別のbaseline establishmentを必要とする。baseline establishmentでは少なくともevaluable artifact、reproducible provenance、valid inference / policy contract、必要なsanity checkを確認できることを想定する。

### Family promotion and Overall determination are separate events

family内promotionとcross-family Overall determinationは別のevaluation eventとして扱う。

```text
new heuristic candidate vs current Heuristic Champion
new learning candidate  vs current Learning Champion
        (family-internal promotion)

Heuristic Champion vs Learning Champion
        (cross-family Overall determination)
```

family内promotionでは、family固有のdevelopment gate / diagnostic / training validationを利用してよい。cross-family比較では、両familyへ共通して適用可能なformal protocolを使用し、architectureごとにseed、opponent、scoring、criterionを有利・不利に変更しない。

family内promotionとcross-family evidenceが別eventであることは、winning-family successionを禁止することを意味しない。current Overall-winning familyのexact Championがvalidなfamily-internal promotionでsuccessorへ置き換わり、opposing family Championがdirect basisから変わっていない場合は、ADR 0005に従いproject-governance上のOverall designationを継承できる。この継承はstatistical transitivityの主張ではない。

### Research-track leaders

Learning family内では、研究方式ごとの進歩を追跡するためにresearch-track leader、例えば `BC Research Leader` や `Offline Q Research Leader` を持つことを許容する。

research-track leaderは正式なChampion familyではない。track leaderの選定にはtrack固有のdevelopment evidenceを使用してよいが、`Learning Champion` の決定・昇格はtrack固有metricだけでは行わず、候補となるtrack leaderへ共通に適用できるinteractive strength evidenceを使用する。

新しいlearning paradigmが追加されても、自動的に新しい正式Champion familyを作らない。research-track leaderを追加するのは、そのtrackが独立した研究loopとして継続的に比較する価値を持つ場合に限る。

## Repository responsibility

- `lisjong-project`: Champion family、Overall designation、research-track leaderのcross-repository semanticsとpromotion boundaryを所有する。
- `lisjong-arena`: Policy / game evaluationのconcrete protocol、seed / seat rotation、sample size、metric、artifact、cross-family comparison実行を所有する。
- `lisjong`: Championとして評価される対象であるstable Policy / AI-side contractと、そのPolicyのidentity / current roleを所有する。family classificationはPolicyの由来に対する判定であり、`lisjong` のcontract自体をfamilyごとに分岐させない。

現在どのPolicyがどのChampion designationを保持しているかを表現するcanonical registry / metadata placementは、本ADRでは決定しない。evaluation protocol / artifact / measurement evidenceは `lisjong-arena` が正本として所有し、stable Policy identityとcurrent roleは既存の `lisjong` ownershipに従う。Champion designationの永続的なregistry / metadata placementは後続Issueで決定する。

具体的なthreshold、game / seed数、evaluation頻度、evaluation artifact schemaも本ADRへ固定せず、Arena側の正本と該当Issue / PRを正本とする。

## Consequences

- Learning系統がHeuristic Championより弱い期間でも、Learning family内の改善を継続的に評価できる。
- `learned policyがhandcrafted / mechanism-based policyをいつ正式に上回ったか` を、共通protocol上のcross-family eventとして再現可能に記録できる。
- 片方のfamilyだけが強い状態でOverall claimが自動成立しなくなり、`Overall Champion: not established` が正常な状態になる。
- 既存の強いheuristic Policyやexperiment-localなlearning checkpointは、このdecisionの成立だけでは初代Championにならない。designation付与にはbaseline establishmentが別途必要になる。
- Policy familyとparameter調整のprovenanceを分けて管理し、Heuristicでも開発・再確認・正式評価のデータ分離を維持する。
- learning paradigmが増えてもChampion familyは増えず、research-track leaderとして追跡することになる。
- Champion registry、promotion automation、cross-family evaluation runner、統計protocolは本decisionでは導入せず、必要になった時点で個別Issueとして扱う。
