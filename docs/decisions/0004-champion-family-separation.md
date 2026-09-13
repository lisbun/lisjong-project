# ADR 0004: Champion family separation

## Status

Accepted.

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

`Heuristic Champion` は、action selection / rankingへ影響するdecision logicとparameterがexplicit human-authored rule、explicit constant、exact algorithmic derivationで定義され、empirical training / fittingへ依存しないPolicyのうち、Heuristic family内で最強と認定されたものを指す。

`Learning Champion` は、action selection / rankingへ影響するparameterの少なくとも一部がobservation、generated sample、rollout result、self-play、gameplay record、optimization / fitting等のempirical sourceから推定・最適化されたPolicyのうち、Learning family内で最強と認定されたものを指す。

### Family classification boundary

family判定はimplementationの見た目ではなく、action decisionへ影響するparameterの由来で行う。

```text
human-authored constant
    -> Heuristic

empirically fitted / optimized constant
    -> Learning
```

例えば次のscoring式は、係数を設計上のexplicit constantとして人間が定めた場合はHeuristic familyに含められる。

```python
score = 3.7 * ukeire + 1.2 * dora - 2.8 * risk
```

同じ式でも、係数をgameplay dataやoptimizerからfittingして得た場合はLearning familyとする。formula-basedな外観、model fileの有無、実行時のnetwork推論の有無はfamily判定基準にしない。

学習済みcomponentをdiagnostic / loggingのみに使用し、action decisionを変更し得ない場合はLearning familyとみなさない。

heuristic candidate generationとlearned scoring / rerankingを組み合わせたPolicyは、learned componentが最終action decisionを変更し得る場合にLearning familyとする。当面、独立した `Hybrid Champion` は設けない。

### Overall Champion is a derived designation

`Overall Champion` は第3のChampion familyではなく、projectのcurrent canonical strength-evaluation protocolにおいてcross-family evidenceにより最強と確認されたfamily Championを示す派生designationとする。

`Overall` はあらゆるmetric / opponent / environmentに対する普遍的な最強を意味しない。

### `not established` is a valid state

family Championと `Overall Champion` の双方について、`not established` を正式な状態として許容する。

原則として次を同一視しない。

```text
family Champion
        !=
Overall Champion
```

片方のfamilyのChampionが未確立であることだけを理由に、他方のfamily ChampionをOverall Championへ自動昇格させない。`Overall Champion` を新規確立・変更する場合は、比較可能なfamily Championが存在し、cross-family formal evaluation evidenceが得られていることを要求する。

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

### Research-track leaders

Learning family内では、研究方式ごとの進歩を追跡するためにresearch-track leader、例えば `BC Research Leader` や `Offline Q Research Leader` を持つことを許容する。

research-track leaderは正式なChampion familyではない。track leaderの選定にはtrack固有のdevelopment evidenceを使用してよいが、`Learning Champion` の決定・昇格はtrack固有metricだけでは行わず、候補となるtrack leaderへ共通に適用できるinteractive strength evidenceを使用する。

新しいlearning paradigmが追加されても、自動的に新しい正式Champion familyを作らない。research-track leaderを追加するのは、そのtrackが独立した研究loopとして継続的に比較する価値を持つ場合に限る。

## Repository responsibility

- `lisjong-project`: Champion family、Overall designation、research-track leaderのcross-repository semanticsとpromotion boundaryを所有する。
- `lisjong-arena`: Policy / game evaluationのconcrete protocol、seed / seat rotation、sample size、metric、artifact、cross-family comparison実行を所有する。
- `lisjong`: Championとして評価される対象であるstable Policy / AI-side contractと、そのPolicyのidentity / current roleを所有する。family classificationはPolicyの由来に対する判定であり、`lisjong` のcontract自体をfamilyごとに分岐させない。

現在どのPolicyがどのChampion designationを保持しているかを表現するcanonical registry / metadata placementは、本ADRでは決定しない。evaluation protocol / artifact / measurement evidenceは `lisjong-arena` が正本として所有し、stable Policy identityとcurrent roleは既存の `lisjong` ownershipに従う。Champion designationの永続的なregistry / metadata placementは後続Issueで決定する。

具体的なthreshold、game / seed数、evaluation頻度、artifact schemaも本ADRへ固定せず、Arena側の正本と該当Issue / PRを正本とする。

## Consequences

- Learning系統がHeuristic Championより弱い期間でも、Learning family内の改善を継続的に評価できる。
- `learned policyがhandcrafted / mechanism-based policyをいつ正式に上回ったか` を、共通protocol上のcross-family eventとして再現可能に記録できる。
- 片方のfamilyだけが強い状態でOverall claimが自動成立しなくなり、`Overall Champion: not established` が正常な状態になる。
- 既存の強いheuristic Policyやexperiment-localなlearning checkpointは、このdecisionの成立だけでは初代Championにならない。designation付与にはbaseline establishmentが別途必要になる。
- 見た目がheuristicなscoring式でも、係数の由来次第でfamily分類が変わるため、parameter provenanceを記録する必要がある。
- learning paradigmが増えてもChampion familyは増えず、research-track leaderとして追跡することになる。
- Champion registry、promotion automation、cross-family evaluation runner、統計protocolは本decisionでは導入せず、必要になった時点で個別Issueとして扱う。
