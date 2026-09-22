# Architecture

## 目的

本書は lisjong ecosystem 全体のrepository責務、repository間の依存方向、横断的な責務境界を定める正本です。

個別repository内部のmodule構成、公開API、実装詳細はそれぞれの `docs/architecture.md` を正本とします。
現在の作業内容・進捗・完了状態はGitHub Issues / PRsを正本とし、本書には重複して記録しません。

historicalなrepository boundary decisionは `docs/decisions/` のADRへ残しますが、現在のtarget architectureは本書を正本とします。

## Repository responsibilities

### `lisjong`

`lisjong` は、観測可能な麻雀状態からActionを選ぶ **AI decision core** と、Heuristic / Learnedを問わない **canonical AI Learning capability** を担当します。

主な責務:

- Policy / AI戦略
- `DecisionContext` / `PolicyInput` / `InternalAction` 等のAI-side contract
- 向聴数、受け入れ枚数、牌効率、HandBelief、value / risk等のstable AI semantics
- model-facing feature representation / action vocabulary
- Learning用 dataset semantics
- teacher / label generation
- model architecture / training objective / trainer
- model artifact / checkpoint identity
- Learned Policy inference / learned estimator
- Learning objective固有のdiagnostics / intrinsic metric definition
- component correctness / physical-validity semantics

中心的な境界は次です。

```text
DecisionContext / PolicyInput
        |
        v
Heuristic / Learned Policy
        |
        v
InternalAction
```

Learningでは、player-safe sourceからの解釈・feature化・labeling・training・artifact・inferenceを同じownerへ置きます。training semanticsとexecutable entry pointは`lisjong`が所有し、外部environment固有のrunnerやformal strength evaluationは所有しません。

`lisjong` core（Policy contract / hand evaluation / belief / action vocabulary等）はML framework非依存を維持します。ML runtimeはoptional dependency + lazy importとし、weights / checkpointはrepository外のoperator-owned artifactとして扱います。model artifactはimmutable snapshotとし、arbitrary factory / callableを復元せず、identity不整合はload時にfail closedします。

RiichiEnv / RiichiLab固有型、WebSocket、credential、matchmaking、retry / reconnect、continuous participation、game record persistence等のexecution concernをPolicy / Learning contractへ持ち込みません。

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

`lisjong-arena` は、AIをconcrete environmentで **実行・観測** し、controlled conditionで **比較・評価** するrepositoryです。

target responsibilityは次の2本柱です。

```text
Execution / Observation
    what happened
        |
        v
player-safe / objective evidence
        |
        +------------------------------+
        |                              |
        v                              v
lisjong Learning                 Arena Evaluation
builds the candidate             measures the candidate
```

#### Execution / observation

主な責務:

- RiichiEnv / RiichiLab / first-party engine integration
- local / external runner、matchmaking、session lifecycle
- retry / reconnect / backoff等のexecution resilience
- protocol trace / durable game record
- environmentへ実際に送信・適用したActionの記録
- external representationからlisjong-owned Policy contractへのprojection
- Arenaが実行するpopulationのseed / seat / allocation provenance
- reusable player-safe source-record schemaとversioned read/write contract

reusable source recordは、PolicyInput相当のplayer-safe observation、legal actions、selected action、source provenanceを保持し、feature tensorやtraining objective固有labelをcanonical sourceにしません。downstreamの`lisjong`がfeature / datasetを再materializeできることを境界条件とします。

#### Evaluation

主な責務:

- Policy / agent matchup
- evaluation用seed集合とdeterministic seat rotation
- raw result収集
- evaluation artifact / provenance
- statistical comparison
- formal holdout / Champion evaluation
- external benchmark
- evaluation固有diagnostics

Evaluationはexecution / observationをconsumerとして利用しますが、評価結果を見てtraining conditionを暗黙に変更しません。

#### Existing Arena Learning implementation

既存のArena-local dataset / trainer / model / checkpoint / diagnostic implementationは、historical experimentまたは既にlockされたin-flight workloadとして残してよいものとします。これは新しいcanonical ownershipを意味しません。

```text
existing Arena experiment
    = historical/reference implementation

new canonical Learning capability
    = lisjong
```

既存Arena implementationからは、invariants、failure modes、provenance requirements、scientific guardrailsを抽出して再利用します。bulk code migrationやArena code deletionをarchitecture変更の完了条件にしません。

現在すでに成立しているArena AWS execution surfaceは、#331 / #332等のconcrete workloadをhostできます。ただし、all training / all rollout hostingをArenaの恒久責務にはしません。

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

### Learning / research execution

新しいcanonical Learning flowは次を基本とします。

```text
Arena execution / observation
        |
        v
versioned player-safe source record
        |
        v
lisjong
    materialization / feature / label
    training / artifact / LearnedPolicy
        |
        v
Arena evaluation
```

training / data-collection rolloutは、concrete consumerに応じて`lisjong` + `lisjong-engine`等で成立し得ます。formal strength-evaluation rolloutは`lisjong-arena`が所有します。

既存Arena operational surfaceで`lisjong`所有のtraining entry pointを実行することは可能ですが、hosting locationからtraining semanticsのownerを推論しません。

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

`lisjong-arena -> lisjong` は、stable Policy / feature / belief / value semanticsをconsumerとして利用する方向です。historicalなArena-local Learning codeが存在しても、`lisjong -> lisjong-arena` の逆依存を作りません。

`lisjong-arena -> lisjong-engine` はfirst-party execution backend利用のために許可します。Arena固有のevaluation semanticsやlisjong-owned Learning semanticsをengineへ持ち込みません。

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

## Learning and evaluation ownership

component correctness、Learning objective固有metric、Policy decision quality、game performanceは別のclaimとして扱います。

```text
Stable AI semantics / canonical Learning capability
    -> lisjong

Learning objective / intrinsic metric definition
    -> lisjong

Arena-executed population / rotation / seed provenance
    -> lisjong-arena

Policy / game performance evaluation
    -> lisjong-arena

External benchmark
    -> lisjong-arena
```

intrinsic metricがinteractive executionを必要とする場合は、definition / execution / artifactを分離します。

```text
metric definition / semantic threshold
    -> lisjong

population / execution / rotation / seed
    -> execution owner
    -> formal Arena evaluationなら lisjong-arena

artifact
    -> producerが書き、schema ownerを明示
```

例えばHandBeliefでは、field / physical semanticsとcanonical learned-estimator Learningは`lisjong`、Arenaが実行するgame-strength comparisonは`lisjong-arena`です。prediction improvement、decision improvement、game-strength improvementを同一claimとして扱いません。

historical Arena experimentのartifact / identity / evidenceは、その時点のprovenanceを維持し、ownership変更を理由に遡及改名・再指定しません。

## Champion governance

Policy強さのcanonical designationは、Policy architectureのfamilyごとに分離して管理します。

```text
Champion families
├─ Heuristic Champion
└─ Learning Champion

Overall Champion
    = DIRECT cross-family evidence または winning-family succession に基づくderived designation
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

family内promotionはそのfamilyの評価loopとして独立に進められます。Overall Championの初回確立、winning familyの変更、current losing-family Championの変更後の再確立には、比較可能なfamily Championが存在し、両familyへ共通して適用可能なformal strength-evaluation protocolによるevidenceが得られていることを要求します。architectureごとに有利・不利の異なるOverall criterionを使いません。

ただし、current Overall-winning familyがvalidなfamily-internal promotionで自身のexact Championをsuccessorへ置き換え、opposing family Championがdirect cross-family basisから不変である場合は、ADR 0005に従ってOverall designationを継承できます。この継承はproject-governance successionであり、family-internal evidenceとhistorical cross-family evidenceから新pairのstatistical superiorityを推移的に証明する主張ではありません。

いずれのdesignationも `not established` を正式な状態として許容します。あるfamilyのChampionが未確立であることだけを理由に、他方のfamily ChampionをOverall Championへ自動昇格させません。

```text
family Champion
        !=
Overall Champion
```

### Overall designation lifecycle

Overall designationはcurrent family Championsとの関係を明示して管理します。

```text
DIRECT
    exact current family-Champion pairをcanonical cross-family formal eventで直接比較

INHERITED
    current Overall-winning familyがvalidにChampionを更新し、
    opposing family Championがdirect basisから不変
```

`INHERITED` はwinning familyの代表者交代をproject governance上継承する仕組みです。losing-family Championが変わった場合、両familyが変わった場合、exact predecessor / successor / opponent identityを確認できない場合、promotionまたはdirect basisがinvalidatedされた場合は継承しません。そのcurrent pairについてOverallを `not established` とし、fresh cross-family eventを要求します。

current Overall recordでは少なくとも、current Champion identity、winning family、`DIRECT` / `INHERITED` mode、direct cross-family basisを保持します。継承時はさらにpredecessor / successor identity、family-promotion evidence、opposing Champion unchangedの確認を保持し、historical direct evidenceを書き換えません。

このlifecycleはArenaの具体的なstatistical protocol、sample size、rotation、artifact schemaを変更しません。具体的なcross-family evidence contractはArenaが所有します。詳細な判断規則は [ADR 0005: Overall Champion determination by AABB half-game](decisions/0005-overall-champion-aabb-half.md) を参照してください。

### Research-track leader

Learning family内では、BC / Offline Q等のresearch trackごとにleaderを追跡できます。research-track leaderは正式なChampion familyではなく、track固有のdevelopment evidenceで選定してかまいません。一方、Learning Champion自体の決定は、track固有metricだけでは行わず、trackをまたいで共通に適用できるstrength evidenceを使用します。新しいlearning paradigmが増えても、自動的に新しいChampion familyを作りません。

具体的なthreshold、sample size / seed、evaluation頻度、evaluation artifact schema、promotion automationはproject-wide architectureで固定せず、Arena側のpurpose-specific contractと該当Issueへ委ねます。Champion designationのcanonical registry / metadata placementは未決定であり、後続Issueで扱います。判断の背景は [ADR 0004: Champion family separation](decisions/0004-champion-family-separation.md) を参照してください。

## RiichiLab external strength registry

RiichiLab ranked Ratingは、Arenaのcontrolled / reproducible strength evidenceとは別の **external live benchmark** として扱います。

```text
Arena
    fixed protocol / direct comparison / reproducible evidence

RiichiLab
    live matchmaking / changing population / ecosystem-relative signal
```

両者を単一scoreへ合成せず、一方の結果で他方を自動上書きしません。

RiichiLab registryのrecord単位はPolicy表示名ではなく、exact behavioral deploymentです。少なくともPolicy identity / source revision / behavior-affecting configurationまたはcheckpoint、RiichiLab bot identity、deployment boundary、post-deployment game count、Rating observation、execution-quality evidenceを一体として扱います。

v1のpost-deployment exposureは運用上のmaturityとして次に分類します。

```text
0-49 games      PROVISIONAL
50-99 games     PRELIMINARY
100-199 games   STABLE
200+ games      MATURE
```

これらはtrue skillの統計的保証ではありません。threshold semanticsを変更する場合はrevisionを明示し、historical recordへ黙ってretroactiveに適用しません。

各deploymentの最初の200 post-deployment ranked games到達直後のRatingをcanonical snapshotとし、後続のcontinuous live Ratingと分離します。peak Ratingやresult-drivenなendpoint延長をcanonical recordに使いません。

RiichiLab側でranked gameとして成立してRatingへ反映された対局は、DC / timeout / default action等があってもexposure countから除外しません。Policy execution qualityはCLEAN / DEGRADED / CONTAMINATED等の別evidenceとして保持し、取得できないquality情報を推測しません。

bot slotの再利用を許容しますが、new deploymentの`games_since_deployment`は0から開始し、server-side Rating carry-overをfresh independent initializationと同一視しません。可能なら`rating_at_deployment`を保存し、欠損時に独自補正や再構築を行いません。

長期inactivity等により追加gameなしでもdisplay Ratingが動き得るため、Rating observationにはgame countとobservation timeを併記します。no-game Rating movementだけをPolicy regressionとは解釈せず、historical canonical snapshotを後日のdisplay変化で上書きしません。

Responsibility boundary:

```text
lisjong-project
    registry semantics / project-level representation

lisjong-arena
    RiichiLab execution / observation
    durable per-game provenance
    bounded acquisition / analysis tooling when justified

lisjong
    stable Policy behavior identity / implementation
```

RiichiLab RatingだけでHeuristic Champion / Learning Champion / Overall Championをpromotionせず、Champion governanceのcontrolled evidenceは既存project / Arena contractを正本とします。

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
       Arena evaluation      reusable source record
                                  |
                                  v
                            lisjong Learning
                                  |
                                  v
                         candidate / artifact
                                  |
                                  `----> Arena evaluation

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

training entry point
    -> lisjong

training execution host
    -> concrete workload owner / existing Arena operational surface when explicitly reused

comparison runner
    -> lisjong-arena evaluation
```

### Game runner

ゲーム状態を所有し、Actionを適用してround / matchを進行します。合法手・終了条件・結果生成はengine側の責務です。

### Integration runner / client

RiichiEnv、RiichiLab、first-party engine等へlisjong Policyを接続します。外部Observation / Action表現とPolicy contractの境界、およびsession lifecycleを扱いますが、麻雀ruleやPolicy判断ロジックは再実装しません。

### Training entry point / execution host

Learning semantics、dataset interpretation、training objective、executable training entry pointは`lisjong`が所有します。

実際のcomputeをどこでhostするかは別のoperational concernです。既存Arena AWS surfaceをconcrete workloadで再利用できますが、これをproject-wideな恒久hosting責務にはしません。future hosting ownerは具体的なconsumer / operational needが生じた時点で決めます。

### Arena / comparison runner

複数trialのseed、seat、Policy / agent組合せ、evaluation scope、試行数等を計画し、raw resultを収集・集計して比較します。execution / observationを利用できますが、integration layerへcomparison semanticsを逆流させません。

## Issue placement rules

新しい機能や設計課題は、主たる意味契約のownerへ置きます。

- 麻雀rules / state transition / scoringなら `lisjong-engine`
- Policy / AI semantics、feature / action vocabulary、dataset semantics、teacher / label、training、model artifact、Learned Policy / learned estimatorなら `lisjong`
- environment接続、live participation、raw observation、Arena-executed population provenanceなら `lisjong-arena` Execution / Observation
- Policy / agent matchup、seed / seat rotation、formal strength comparison、external benchmarkなら `lisjong-arena` Evaluation
- Human-facing presentation / input / Human Play / spectator / replay consumerなら `lisjong-play`
- repository boundary / dependency / ownership ruleの変更なら `lisjong-project`

既存Arena Learning implementationの保守・historical compatibilityはArena Issueで扱えますが、それを理由に新しいcanonical Learning capabilityをArenaへ追加しません。

複数repositoryに変更が必要でも、同じsemantic contractを重複所有しません。cross-repository artifactはowner / version / provenanceを明示し、artifact経由のhidden reverse dependencyを作りません。

## Seed allocation / provenance

scientific / qualification / evaluation / training populationのseed allocationは、ecosystem-wide single registryではなく **owner-scoped authoritative ledger** で管理します。

```text
lisjong-project
    seed allocation / provenance contract
        |
        +--------------------+
        |                    |
        v                    v
lisjong-arena             lisjong
Arena-executed            lisjong-produced
population owner          population owner when needed
```

seed integer単体をglobal identityとせず、generator / environment / producer / derivation semanticsを含むversioned `seed_domain` をcollision scopeとします。同一domain内ではowner ledgerがreuseをfail closedし、cross-owner / cross-domain non-overlapはscientific protocolが要求する場合だけexplicit provenance contractで検証します。

project-wide interoperabilityでは、少なくとも `allocation_identity`、`seed_domain`、authorizing ledger revision、`seed_membership_identity` を照合可能にします。`RESERVED / COMMITTED / RETIRED` のstate semantics、freshness、authority publication、concurrency、historical bootstrapを含む詳細は [Seed allocation / provenance contract](seed-allocation-provenance.md) を正本とします。判断の背景は [ADR 0009: Owner-scoped seed allocation and provenance](decisions/0009-owner-scoped-seed-allocation-provenance.md) を参照してください。

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
