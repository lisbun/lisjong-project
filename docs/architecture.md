# Architecture

## 目的

本書は lisjong ecosystem 全体のrepository責務、repository間の依存方向、横断的な責務境界を定める正本です。

個別repository内部のmodule構成、公開API、実装詳細はそれぞれの `docs/architecture.md` を正本とします。
現在の作業内容・進捗・完了状態はGitHub Issues / PRsを正本とし、本書には重複して記録しません。

historicalなrepository boundary decisionは `docs/decisions/` のADRへ残しますが、現在のtarget architectureは本書を正本とします。

## Repository responsibilities

### `lisjong`

`lisjong` は、観測可能な麻雀状態からActionを選ぶ **AI decision core** と、production / stableなAI-side semanticsを担当します。

主な責務:

- Policy / AI戦略
- `DecisionContext` / `PolicyInput` / Policy contract
- `InternalAction` 等のAI-side contract
- 実行時に利用するPolicy / AI configurationの選択
- Policy返却値の合法性・semantic identity validation
- 向聴数、受け入れ枚数、牌効率、lookahead
- remaining tile information、HandBelief等のhidden-state inference semantics
- 打点・offensive value、守備・defensive risk、押し引き、utility-aware decision等のstable AI semantics
- Policy内部analysis schema / meaning
- production / public Learned Policyの意味契約
- stable feature / inference / estimator contract
- componentのcorrectness / physical-validity semantics

lisjongの中心的な境界は概念上次とします。

```text
DecisionContext / PolicyInput
        |
        v
      Policy
        |
        v
InternalAction
```

RiichiEnv / RiichiLab等の外部environment固有型、WebSocket、credential、matchmaking、retry / reconnect、continuous participation、game record persistence等のexecution concernをPolicy contractへ持ち込みません。

`lisjong` は麻雀ルールそのものを進行するgame engineでも、外部環境へのstandalone runner / clientの長期的な所有者でも、複数trialの評価計画・集計を行うrepositoryでもありません。

また、研究用dataset / trainer / checkpointが存在するだけで、それを `lisjong` のstable contractへ自動昇格させません。production / stable semanticsへのpromotionは明示的に判断します。

### `lisjong-engine`

`lisjong-engine` は、与えられたActionに従って日本式リーチ麻雀を正しく進行し、対局結果を生成する **first-party game execution substrate** を担当します。

主な責務:

- 牌、手牌、山、河、副露等のドメインモデル
- game / round / turn状態管理
- 合法手判定・合法手生成
- 和了、役、符、点数
- 鳴き、立直、流局、本場、供託、連荘
- 東風戦・半荘等のgame / match進行
- 最終点数・順位
- `RuleSet`
- deterministicなseed管理
- player-visible observation / public action boundary
- external selectorから利用可能なdeterministic execution boundary
- engine固有component / rule correctnessのvalidation

`lisjong-engine` はPolicy、AI戦略、学習、human-facing UI / input handling、外部AI hosting、RiichiEnv / RiichiLab固有integration、evaluation orchestrationを持ちません。

誰がActionを選ぶか、どのように人間へ表示・入力させるか、複数trialをどう評価するかはconsumer側の責務です。

### `lisjong-arena`

`lisjong-arena` は、lisjongをconcrete environmentで実行・観測し、bounded research candidateを再現可能に生成・診断し、そのcandidate / Policyをcontrolled conditionで比較・評価するarena基盤を担当します。

Arena内部では、少なくとも次の **3責務** を分離します。

```text
Execution / Observation
    what happened
        |
        v
objective execution data
        |
        +------------------------------+
        |                              |
        v                              v
Experiment-local Research          Evaluation
bounded dataset / training        matchup / seeds / rotation
analysis / model artifact         metrics / artifact / provenance
        |                              ^
        v                              |
research candidate -------------------+
```

重要な境界:

```text
Arena may own experiment-local ML
!=
Arena owns stable AI semantics
```

#### Execution / observation

主な責務:

- environment-specific integration
- RiichiLab client / Adapter等のexternal participation能力
- local / external execution runner
- matchmaking participation
- repeated / continuous participation
- retry / reconnect / backoff等のexecution resilience
- execution profile / credential source resolution
- protocol trace
- raw game record / objective execution eventの取得
- environmentへ実際に送信・適用したActionの記録
- external representationからlisjong-owned Policy contractへのprojection
- `InternalAction`からexternal legal Actionへのmapping / revalidation

このlayerは研究仮説やPolicy performance metric、comparison protocol、Arena固有seed / seat rotation semanticsを知らなくても成立する構造とします。

AI判断ロジックや麻雀ruleを再実装しません。

#### Experiment-local research / ML

Arenaは、**bounded research questionを検証するために必要なpurpose-specific dataset / training / model / analysis implementation** を所有できます。

主な例:

- experiment-local player-safe feature / tensor representation
- purpose-specific dataset schema / split / manifest
- data generation / materialization harness
- bounded training harness
- fixed experiment model architecture / loss / optimizer configuration
- checkpoint / result / diagnostic artifact
- offline failure diagnosis
- experiment-local Learned Policy adapter
- experiment固有のclassification / decision rule

この責務は、次の条件を満たす場合に限定します。

```text
bounded research question
+ explicit provenance / reproducibility
+ purpose-specific schema
+ clear promotion boundary
+ no silent production adoption
```

Arenaのexperiment-local researchが利用するshanten / HandBelief / value等の**stable meaning**は、必要に応じて `lisjong` owned semanticsをreuseします。Arena都合で同じdomain semanticsを別定義しません。

一方、experiment-local metric、dataset split、training protocol、checkpoint selection、bounded calibration study等は、そのexperimentのevidence contractとしてArenaが所有できます。

#### Evaluation

主な責務:

- Policy / agentのmatchup定義
- evaluation用seed集合
- deterministicなseat rotation
- round / game等のevaluation scopeに応じた実行計画
- Policy / agent assignmentの記録
- raw result収集
- evaluation artifact / provenance
- 評価scopeに応じたmetrics
- 再現可能なcomparison protocol
- statistical comparison
- external benchmark / report
- evaluation目的のexternal competitor integration / orchestration

Evaluationはexecution / observation能力をconsumerとして利用できます。
Experiment-local Researchが生成したcandidateを評価できますが、評価結果を見てtraining conditionを暗黙に変更しません。

逆方向にcomparison semanticsをexecution / observationへ漏らしません。

### Experiment-local research promotion boundary

研究用implementationは、最初からproduction-quality generic frameworkへ昇格させません。

```text
bounded experiment
    |
    v
experiment-local implementation
    |
    v
result / evidence
    |
    +--> negative / inconclusive
    |       keep as historical experiment record
    |
    `--> repeatedly useful / promoted principle
            |
            v
       owner repository review
            |
            +--> remain Arena experiment infrastructure
            `--> formalize as lisjong stable AI contract
```

次はそれぞれ別物です。

```text
experiment-local model
!= canonical production model architecture

experiment-local feature schema
!= stable PolicyInput / production feature contract

experiment-local checkpoint
!= production Policy

experiment result
!= stable public API
```

promotion時には少なくとも次を確認します。

- semanticsが特定experimentを超えてstableか
- production / multiple consumerで必要か
- Arena evidence concernとAI decision concernのどちらがownerとして自然か
- experiment-local identityをそのままstable contractへ流用してよいか
- versioning / compatibility / breaking-change policyを新たに定義すべきか

「研究で使えた」だけではpromotion理由にしません。

### `lisjong-play`

`lisjong-play` は、first-party `lisjong-engine` を利用するHuman Play / presentation consumerを担当します。

主な責務:

- human seat assignment
- human-facing state / action presentation
- human input
- action selection UX
- confirmation / interaction
- CLI / GUI presentation
- Human Playに必要なminimum session orchestration
- spectator / replay等のread-oriented presentation where concrete requirements justify it

Human decisionは `lisjong-engine` のplayer-safe public boundaryを直接利用します。

```text
SeatObservation
+
ActionDescriptor[]
        |
        v
Human selector
        |
        v
selected ActionDescriptor
```

Human choiceを `PolicyInput` / `DecisionContext` / `InternalAction` / `execute_policy()` へ通しません。
game / round / turn state、legal actions、reaction priority、scoring / settlement、round / match progression、terminal conditionsは引き続き `lisjong-engine` が所有します。

AI seatを含むHuman Playでは `lisjong` Policyをconsumer側から利用します。必要なbridgeを既存ownerからreuseし、presentation都合でAI-side semanticsやengine semanticsを複製しません。

## Execution paths

Arena execution / observationは、external environmentだけでなく、利用可能なfirst-party engineを含むconcrete execution pathへlisjong Policyを接続できます。

ただし、複数pathが存在することだけを理由にproject-wide generic backend abstractionを先行設計しません。

### First-party game execution

`lisjong-engine` は、ecosystem自身が完全に制御できるfirst-party execution pathを提供します。

```text
execution / integration consumer
          |
          +----> external game environment
          |
          +----> lisjong-engine
                     first-party execution
```

RiichiEnv等は高速simulation、external ecosystem interoperability、独立実装との比較等に利用できます。
`lisjong-engine` はdeterministic reproduction、controlled scenario、Human Play等のconsumer requirementへ利用できます。

どちらか一方をproject-wideな唯一のexecution backendとして固定しません。

privileged engine-owned truthをoffline validationで利用する場合も、Policy-visible stateへ逆流させません。

### Policy-vs-Policy development evaluation

```text
lisjong-arena evaluation
        |
        v
Arena execution / observation
        |
        v
selected game environment
        |
        v
lisjong Policy
```

selected game environmentはexternal environmentでも `lisjong-engine` のfirst-party pathでも構いません。

### Research execution

Experiment-local Researchは、必要な場合にArena execution / observationやdurable raw evidenceを利用してdataset / diagnostic artifactを構成できます。

```text
execution / observation
        |
        v
player-safe / objective raw evidence
        |
        v
experiment-local materialization / training / diagnosis
```

training-only ground truthやomniscient labelを使う場合、serving inputと明確に分離します。

### Live / standalone participation

RiichiLab等へlisjongを参加させるproject-owned entry pointはArena execution / observationが担当します。

接続・session lifecycle・matchmaking・retry / reconnect・continuous participation・protocol trace等はAI decision coreから分離します。

### Mixed-agent external benchmark

Mortal等のexternal competitorを含むbenchmarkでは、Arenaが選択したOSS execution environment等をorchestrateしてよいものとします。

```text
                  lisjong-arena
                       |
              execution environment
                   /         \
                  v           v
          lisjong seat   external competitor
```

この場合でも、ArenaがlisjongのPolicy判断ロジックや麻雀ルールを複製してよいことを意味しません。

## Dependency direction

現在許可するfirst-party dependency directionは次です。

```text
lisjong-arena -> lisjong
lisjong-arena -> lisjong-engine
lisjong -> lisjong-engine
lisjong-play -> lisjong-engine
lisjong-play -> lisjong
lisjong-play -> lisjong-arena

lisjong-engine -X-> lisjong
lisjong-engine -X-> lisjong-play
lisjong -X-> lisjong-arena
```

`lisjong-arena -> lisjong` は、stable Policy / feature / belief / value semanticsをconsumerとして利用する方向です。Arenaのexperiment-local research codeが存在しても、`lisjong -> lisjong-arena` の逆依存を作りません。

`lisjong-arena -> lisjong-engine` はfirst-party execution backend利用のために許可します。Arena固有のevaluation / training semanticsをengineへ持ち込みません。

`lisjong-play -> lisjong-arena` は、Arena-owned bridge等をconcrete consumerとしてreuseする場合に許可しますが、Arena evaluation semanticsをHuman Playへ持ち込むことを意味しません。別のnon-Arena consumerも同じbridgeを必要とする等のconcrete needが生じた場合にplacement / extractionを再評価します。

`lisjong-project` はdocumentation / coordination repositoryでありruntime dependency graphには含めません。

## External ecosystem boundary

成熟したOSSや外部実装は、reference、backend、benchmark、toolingとして積極的に評価・利用します。

external benchmark、simulation、game execution、protocol interoperability等に必要な能力を成熟したOSSが既に提供している場合は、それを優先的に評価し、同等機能をecosystem内で無目的に重複実装しません。

ただし、project-wide stable contract、Policy semantics、repository responsibility、project固有artifact contract、cross-repository dependency directionはlisjong ecosystem側で所有します。

外部OSS固有の型・API・内部設計を上位contractへ直接漏らしません。

correctness validationで複数実装を比較する場合は、実装系譜・algorithm・backend等が十分に独立していることを確認します。同一backendを薄くwrapした複数実装を独立referenceとして数えません。

複数実装のagreementは強いevidenceになり得ますがproofとは扱いません。差異が発生した場合も多数決をoracleとせず、semantic difference、rule configuration、bug、unsupported case等を調査します。

性能最適化はcorrectness、independent validation、regression protectionの後に行います。Python実装であることだけを理由にnative backendへ移行せず、実測されたbottleneckとsemantic compatibilityを確認してから最適化を判断します。

## Evaluation and research ownership

component quality、research evidence、Policy decision quality、game performanceは異なるclaimとして扱います。

```text
Stable component semantics / production correctness
    -> component owning repository (`lisjong` / `lisjong-engine`)

Experiment-local training / measurement / diagnosis
    -> lisjong-arena experiment-local research

Policy / game performance evaluation
    -> lisjong-arena evaluation

External benchmark for evaluation
    -> lisjong-arena evaluation

Live / standalone participation
    -> lisjong-arena execution / observation
```

例えばHandBeliefについて:

```text
HandBeliefのstable field / meaning / physical semantics
    -> lisjong

bounded dataset / training / MAE / calibration study / artifact
    -> Arena experiment-local research

HandBelief-aware Policyが対局上強くなったか
    -> Arena evaluation
```

同様に、Learned Policyのexperiment-local model / checkpointはArenaで研究できても、production Policy semanticsへ自動昇格しません。

prediction improvement、decision improvement、game-strength improvementを同一claimとして扱いません。

評価scopeは対象に対してminimum sufficientなものを選びます。cheap diagnostic / round-level development evaluationから始め、placementやlong-horizon objectiveが重要な段階でhanchan等のgame-level validationへ拡張できます。

具体的なseed / seat rotation / sample size / variance / confidence interval / paired comparison等はproject-wide architectureでは固定せず、Arena側のpurpose-specific contractへ委ねます。

## Champion governance

Policy強さのcanonical designationは、Policy architectureのfamilyごとに分離して管理します。

```text
Champion families
├─ Heuristic Champion
└─ Learning Champion

Overall Champion
    = cross-family formal evidenceに基づくderived designation
    = 独立したChampion familyではない
    = not establishedを許容する

Research-track leader
    != Champion family
```

### Family classification boundary

familyは、action selection / rankingへ影響し得るdecision logicとparameterの由来で判定します。

```text
explicit human-authored rule / constant / exact algorithmic derivation
    -> Heuristic

observation / sample / rollout / gameplay record / optimization / fittingから
得たparameterがaction decisionへ影響し得る
    -> Learning
```

formula-basedに見えるPolicyでも、action decisionへ影響するparameterがempirical data / optimization / fittingから得られている場合はLearning familyとして扱います。逆に、学習済みcomponentをdiagnostic / logging専用に使い、action decisionを変更し得ない場合はLearning familyとみなしません。

heuristicとlearned componentを組み合わせたPolicyは、learned componentが最終action decisionを変更し得る場合にLearning familyとして扱い、独立したHybrid familyを設けません。

### Promotion boundary

family内promotionとcross-family Overall determinationは別のevaluation eventです。

```text
family-internal promotion
        !=
cross-family Overall determination
```

family内promotionはそのfamilyの評価loopとして独立に進められます。Overall Champion designationの新規確立・変更には、比較可能なfamily Championが存在し、両familyへ共通して適用可能なformal strength-evaluation protocolによるevidenceが得られていることを要求します。architectureごとに有利・不利の異なるOverall criterionを使いません。

いずれのdesignationも `not established` を正式な状態として許容します。あるfamilyのChampionが未確立であることだけを理由に、他方のfamily ChampionをOverall Championへ自動昇格させません。

```text
family Champion
        !=
Overall Champion
```

### Research-track leader

Learning family内では、BC / Offline Q等のresearch trackごとにleaderを追跡できます。research-track leaderは正式なChampion familyではなく、track固有のdevelopment evidenceで選定してかまいません。一方、Learning Champion自体の決定は、track固有metricだけでは行わず、trackをまたいで共通に適用できるstrength evidenceを使用します。新しいlearning paradigmが増えても、自動的に新しいChampion familyを作りません。

具体的なthreshold、sample size / seed、evaluation頻度、evaluation artifact schema、promotion automationはproject-wide architectureで固定せず、Arena側のpurpose-specific contractと該当Issueへ委ねます。Champion designationのcanonical registry / metadata placementは未決定であり、後続Issueで扱います。判断の背景は [ADR 0004: Champion family separation](decisions/0004-champion-family-separation.md) を参照してください。

## Execution data, AI improvement, and Visualization / Analysis boundary

external environmentから得るraw execution dataと、それを研究・評価・可視化へ利用する意味付けを分離します。

```text
first-party engine / external environment / live integration
                       |
                       v
          Arena execution / observation
                       |
               raw execution data
                 /          \
                v            v
       Arena evaluation   Arena experiment-local research
                                |
                                v
                         research evidence / candidate
                                |
                                +----> evaluation
                                |
                                `----> promotion review
                                         |
                                         v
                                   lisjong stable contract

Policy decision / analysis data --------+
Arena result / provenance / artifact ---+--> analysis / viewer consumer
```

Arena raw execution dataには、例えば次を含められます。

- raw game record
- protocol trace
- objective execution event
- seat-visible observation record
- environmentへ実際に送信・適用したAction
- game result

一方、shanten / ukeire値、HandBelief、候補評価、選択理由等のAI内部analysisをexecution recordへ暗黙に混在させません。必要な場合は別channel / typed payloadとして扱い、そのstable semanticsはproducer ownerに残します。

### Visibility / secret boundary

```text
runtime credential / Authorization information
        -X-> trace / game record / evaluation artifact

privileged offline / ground-truth data
        -X-> online Policy input

privileged execution observation
        -X-> Policy decision path
```

- token、Authorization header等のsecretをtrace / game record / artifactへ保存しない
- offline researchで利用可能なhidden ground truthをonline Policy inputへ逆流させない
- privileged observer informationをPolicy-visible stateへ追加しない
- execution / observationの追加がPolicy選択へ干渉しない境界を維持する

Visualization / Analysisは、対局状況・牌譜・Policy意思決定過程・evaluation / research resultを観察、再生、分析するread-oriented consumer能力として扱います。

project-wide canonical event schemaを先行要件とせず、concrete consumer requirementから必要なadapter / normalization boundaryを抽出します。

原則:

- viewerは麻雀ruleを所有しない
- viewerはPolicy decision logicを所有しない
- viewerはArena evaluation / training protocolを所有しない
- GUI都合の型をengine / Policy contractへ逆流させない
- viewerの停止や失敗がgame execution / Policy decisionへ影響しない設計を目指す
- liveとreplayで共通化可能なpresentation semanticsは再利用する
- early canonical `GameRecord` / `ViewerState` schemaを推測で固定しない

### Human Play boundary

Human Playのphysical ownerは `lisjong-play` です。

```text
lisjong-play
        |
        | SeatObservation
        | ActionDescriptor[]
        | selected ActionDescriptor
        v
lisjong-engine
```

`lisjong-play` はhuman seat assignment、人間向けstate / action表示、input、selection UX、confirmation、CLI / GUI presentation、必要なsession orchestrationを所有します。

game state authority、合法手、reaction priority、精算、終局条件等は `lisjong-engine` が所有します。

AI seatを含む場合はconsumer側が `lisjong` Policyを利用し、`lisjong-engine -> lisjong` の逆依存は作りません。

## Runner responsibilities

「runner」は異なる責務を指し得るため、次を区別します。

```text
game runner
    -> lisjong-engine

integration runner / client
    -> lisjong-arena execution / observation

experiment runner / training harness
    -> lisjong-arena experiment-local research

comparison runner
    -> lisjong-arena evaluation
```

### Game runner

ゲーム状態を所有し、Actionを適用してround / matchを進行します。合法手・終了条件・結果生成はengine側の責務です。

### Integration runner / client

RiichiEnv、RiichiLab、first-party engine等へlisjong Policyを接続します。外部Observation / Action表現とPolicy contractの境界、およびsession lifecycleを扱いますが、麻雀ruleやPolicy判断ロジックは再実装しません。

### Experiment runner / training harness

bounded research questionのためにdata materialization、training、checkpoint selection、diagnostic artifact等を実行します。purpose-specific contractとして設計し、generic ML platformへ自動昇格させません。

### Arena / comparison runner

複数trialのseed、seat、Policy / agent組合せ、evaluation scope、試行数等を計画し、raw resultを収集・集計して比較します。execution / observationを利用できますが、integration layerへcomparison semanticsを逆流させません。

## Issue placement rules

新しい課題のplacementは「どのrepositoryの目的を成立させるために必要な責務か」で判断します。

- 麻雀ゲームを正しく進行するために必要なら `lisjong-engine`
- stableなAI decision semantics、Policy、Policy内部component、production feature / inference / model contractなら `lisjong`
- environment接続、live participation、session lifecycle、retry / reconnect、raw execution observationなら `lisjong-arena` Execution / Observation
- bounded research questionのdataset / training / checkpoint / diagnostic / experiment-local modelなら `lisjong-arena` Experiment-local Research
- Policy / agent matchup、seed、seat rotation、strength / benchmark comparisonなら `lisjong-arena` Evaluation
- Human-facing presentation / input / action-selection UX / Human Play / spectator / replay consumerなら `lisjong-play`
- repository境界、依存方向、ownership、promotion boundary等のproject-wide原則を変更するなら `lisjong-project`

experiment-local researchで新しいfeature / modelを作る場合も、stable semanticsのownerを同時に変更したとはみなしません。

複数repositoryに変更が必要な機能でも、同じ仕様を複数repoへ重複して持たせません。横断契約をどこが所有するかを先に決め、各repoは自分の内部実装だけを持ちます。

execution / research infrastructureがArena外の複数consumerから必要になった場合は、その時点で共通runtime / repositoryへの抽出を再検討します。将来のconsumerを推測してgeneric platformを先行設計しません。

## Source-of-truth boundary

```text
lisjong-project
    ecosystemの構造
    cross-repository principles
    repository ownership / dependency direction
    long-term promotion boundary

各repositoryのarchitecture
    repository内部の構造
    concrete contract / implementation boundary

GitHub Issues / PRs
    現在の仕事
    concrete adoption / protocol decisions
    experiment result / progress
```

この境界を維持し、現在のIssue番号や完了状況、特定OSSのversion、具体的evaluation / training protocol等をproject-wide architectureへ埋め込まないことを基本ルールとします。
