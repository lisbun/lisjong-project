# Roadmap

## 目的

本書は lisjong ecosystem 全体として、どの能力をどのような依存関係で成立させたいかを示す長期ロードマップです。

本ロードマップは、すべての能力を上から順番に完了させる直列Phase一覧ではありません。基礎契約を共有した上で、game engine、Policy、external execution / observation、Policy comparison、external benchmark、Visualization / Analysisをそれぞれの責務境界で並行して発展させ、必要な地点で統合します。

個別Issueの実装順序、現在の進捗、完了状態は各repositoryのGitHub Issues / PRsを正本とします。本書にはIssue番号や完了checkboxを原則として持ちません。

## Roadmap principles

- 接続可能性と正しさを、AIの強さより先に確立する
- 麻雀ルール、AI判断、external execution / observation、比較評価、分析・可視化を明確な責務境界で分離する
- deterministicな実行条件を早期に確立し、後続の回帰testと比較に利用する
- 外部環境固有の都合をPolicyやengineの内部契約へ漏らさない
- execution / observationとevaluationをArena内でも別責務として扱う
- raw execution data acquisitionとAI feature / training semanticsを分離する
- credentialやprivileged observer dataをPolicy inputへ逆流させない
- 成熟したOSSや外部実装はreference / backend / benchmark / toolingとして積極的に評価・利用する
- 成熟したOSSがgame executionやprotocol interoperabilityを既に提供する場合は、同等機能の重複実装を避けて優先利用する
- stable public contract、Policy semantics、repository responsibility、project固有artifact contract、cross-repository dependency directionはlisjong ecosystem側で所有する
- correctnessを確立し、独立validationとregression protectionを行った後に性能を計測し、実測されたbottleneckを最適化する
- 評価対象に対して最小十分なevaluation scopeを選び、必要になった時点でより高コストなscopeへ拡張する
- engine完成をPolicy改善や初期Arena開始の不要な前提にしない
- 比較可能な対象と再現可能なgame実行が揃う前にArena evaluationを過剰構築しない
- 実際の複数execution pathやconcrete consumerが揃う前に、将来backend / runtimeを推測した汎用abstractionを先行設計しない
- Arena外の複数consumerや独立したproduction hosting要件が成立した時点で、共通runtime抽出を再検討する
- Visualization / Analysisのためにproject-wide canonical event schemaを先行設計せず、具体的consumer requirementsから必要なadapter / normalization boundaryを抽出する
- 学習Policyは、手書きPolicyを再現可能に比較できる基盤が整ってから導入し、既存のexecution・評価・分析基盤を可能な限り再利用する

## How to read this roadmap

長期的な能力の発展は、概ね次の構造で捉えます。

```text
                    Foundation
                  Policy contract
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
      Engine Track   Policy Track    Arena Track
                                      /        \
                                     v          v
                         Execution / Observation  Evaluation
                                     |          |
                                     +-----+----+
                                           v
                                 game / analysis data
                                           |
                              +------------+------------+
                              |                         |
                              v                         v
                        AI improvement        Visualization / Analysis
                              |
                              v
                       Learning Policy
```

Engine / Policy / Arenaの各trackは互いの完成を不必要に待たず進めます。

`lisjong` はAI decision coreとして発展し、external environmentへの接続・session lifecycle・raw observation acquisitionはArena側のexecution / observation capabilityとして発展させます。Arena evaluationはそのexecution能力をconsumerとして利用できますが、execution layerへcomparison semanticsを逆流させません。

Visualization / Analysisはread-orientedなconsumer能力として、具体的なdata sourceとconsumer requirementが成立した地点から発展させます。

## Near-term AI improvement capability sequence

ADR 0002後のnear-term developmentは、固定Phase番号や完全な直列gateではなく、**AIを継続的に強化するための主要dependencyを持つpartial order**として捉えます。

```text
Resilient live participation
        |
        +---------------------------> objective live records

Policy decision observability
        |
        v
Consumer-driven replay / analysis boundary
        |
        +---------------------------> objective records + AI analysis

First-party engine execution
        |                           \
        v                            +-> objective execution records
Human vs AI live consumer
        |
        +---------------------------> Human session data

objective records + AI analysis + execution sources
                    |
                    v
             Offline analysis
                    |
                    v
              AI strengthening
                    |
                    v
       reproducible Arena evaluation
                    |
                    v
          external validation
                    |
                    +--------------------> AI strengthening
```

矢印は主要dependency / recommended orderを示します。すべてのcapabilityが完了するまでPolicy改善を止める意味ではありません。Resilient live participationとFirst-party engine executionは互いの完了を待たず進められ、First-party engine executionもReplay / Analysis boundaryの完成をhard dependencyとしません。

### Resilient live participation

RiichiLab等のlive environmentへ、安全に繰り返し参加できるexecution capabilityをArena側のexecution / observationとして発展させます。repeated / continuous participation、disconnectやtransient transport failureからのsafe re-participation、bounded backoff、graceful shutdown等を通じて、real-world opponent distributionから継続的にobjective execution dataを取得できる入口を作ります。

このcapability自体はPolicy strengthを直接変更しません。live execution lifecycleをAI decision coreへ持ち込まず、長時間のdata acquisitionを可能にすることが目的です。

### Policy decision observability

objective execution observationとは分離して、`lisjong` が1 decisionで何を選択し、どのlisjong-owned typed intermediate valueを実際に生成・利用したかをone-way observationできる方向へ発展させます。

```text
GameTrace / objective execution observation
    what happened

Decision / Analysis observation
    what lisjong selected / computed
```

AI-owned analysis semanticsは `lisjong` に残し、Arenaのraw execution recordへ暗黙に混在させません。observerの追加によってPolicy decision pathへprivileged informationが逆流しない境界も維持します。

### Consumer-driven replay / analysis boundary

objective execution recordとAI-owned decision / analysisを最初のconcrete offline consumerで対応付け、historical replayやdecision inspectionに本当に必要なsemanticsを確認します。

project-wide canonical `GameEvent` / `GameRecord` / global decision ID等を先行発明せず、concrete consumer requirementからのみ、

- historical replayに必要なpersisted data
- objective executionとAI analysisのcorrelation requirement
- consumer-side projection / adapter
- live observationとhistorical replayの差分

を導出します。Visualization / Analysisは引き続きread-oriented consumerとして扱います。

### First-party engine execution

`lisjong-engine` のfirst-party execution substrate上で `lisjong` Policyを実際に動作させるconcrete pathを、Arena側のexecution / integration consumerとして成立させます。

```text
                lisjong-arena
                 /        \
                v          v
          lisjong       lisjong-engine
         decision        execution
```

Arena側がengineのplayer-safe observation / public action boundaryとlisjong Policy contractを接続し、ecosystem自身で完全制御できるdeterministic execution、controlled scenario / regression、将来のground-truth validation、Human vs AIの共通AI execution pathへつなげます。

これはRiichiEnv等のexternal backendを置き換える方針ではありません。複数の実execution pathから共通化の必要性が確認される前に、generic `GameBackend` / backend registry等を先行設計しません。

### Human vs AI live consumer

Human Playは単なる将来UIではなく、first-party engine executionを利用するAI evaluation / data acquisition consumerとしても利用できるようにします。最初から豪華なGUIを前提にせず、minimum live consumerを優先します。

Human Play consumerはhuman-facing presentation、human input、action selection UX、human seat assignment、session orchestrationを所有します。game progressionは `lisjong-engine`、AI decision semanticsは `lisjong` が所有し、UI都合をengine / Policy contractへ逆流させません。

旧 `python-study` のCLI / HumanPlayer implementation自体を互換移植することは目的にせず、そこで抽出済みのbehavior / test / edge-case knowledgeをconcrete implementationの入力として利用します。旧runtimeはfuture Human Play consumer成立までreferenceとして保持し、dependency-aware cleanupは別判断とします。Human Play専用repository名やUI frameworkもconcrete implementation開始前には固定しません。

Human vs AIの**対局成立**は主にFirst-party engine executionを前提とします。一方、Human sessionをAI analysis / data acquisitionへ十分に活用する段階では、Policy decision observabilityとconsumer-driven correlation boundaryを再利用します。

### Offline replay / analysis

live Human interactionとは分離したread-oriented consumerとして、対局後にobjective executionとAI decision / analysisを対応付けて振り返れる状態を目指します。入力sourceはHuman sessionに限定せず、RiichiLab、simulation、first-party engine、Arena evaluation等のconcrete sourceを利用できます。

用途には、regression game / round replay、candidate evaluation inspection、HandBeliefやfuture TenpaiBeliefの変化確認、risk / value / push-fold判断のdebugging、Human opponentに対するinference mismatchの発見等を含み得ます。

Human Play consumerを牌譜viewer / AI思考viewerの母体として肥大化させず、live interactionとoffline analysisを別consumer responsibilityとして維持します。

### AI strengthening feedback loop

上記capabilityは、AI改善を開始するための最後のgateではなく、既存のPolicy improvementをより観測可能・再現可能・data-drivenにするfeedback loopへ接続します。

```text
RiichiLab / simulation / Human
            |
            v
          execution
            |
            v
 objective record + AI analysis
            |
            v
       offline analysis
            |
            v
     weakness identification
            |
            v
       lisjong improvement
            |
            v
 reproducible Arena evaluation
            |
            v
       external validation
            |
            +--------------------> repeat
```

強化対象を特定algorithmへ固定しません。hidden-state inference、HandBelief / future TenpaiBelief、offensive value、defensive risk、push / fold、riichi / dama、call decisions、value / utility-aware decision、learned estimator / learned Policy等を、このloopの中で必要に応じて改善します。

component quality、Policy decision quality、game performanceは引き続き別のquality claimとして評価します。

## Foundation — Stable Policy contract and environment boundary

環境非依存のPolicy contractを基礎として、観測可能な状態だけから合法Actionを決定できる状態を確立します。

この基礎には次を含みます。

- `DecisionContext` を中心とした環境非依存のPolicy境界
- `InternalAction` 等のAI-side Action contract
- deterministicなPolicy実行
- Policy返却値の合法性 / semantic identity validation
- seat-visibleな情報だけをPolicyへ渡す情報境界
- 実行時に利用するPolicy / AI configurationを選択できる能力
- 外部environment固有型をPolicy contractへ漏らさない設計
- environment-facing conversionとAI decision logicを分離できる設計

このFoundationは、engine、Policy改善、Arena execution / observation、Arena evaluationの各trackが共有する契約です。個々の実行環境固有の型やprotocolは共通Policy contractへ持ち込みません。

## OSS / external ecosystem and validation strategy

成熟した外部実装は、用途を区別した上で積極的に評価・利用します。

- **Reference**: correctness comparison / differential validation
- **Backend**: computation / simulation / evaluation
- **Benchmark**: external agent / environment
- **Tooling**: replay / visualization / development support

external benchmark、simulation、game execution、protocol interoperability等に必要な能力を成熟したOSSが既に提供している場合は、それを優先的に評価・利用します。同等の麻雀ルール、protocol変換、legal-action mapping、session補助等をecosystem内で無目的に再実装しません。

一方、外部OSS固有の型・API・内部設計は、project-wideなstable contractへ直接漏らしません。具体的なversion、adapter仕様、dependency詳細は、それを所有するrepository / Issueを正本とします。

correctnessとperformanceは次の順序で扱います。

```text
Correctness baseline
        -> independent validation
        -> regression protection
        -> performance instrumentation
        -> actual bottleneck identification
        -> algorithmic / duplicate-work optimization
        -> backend optimization or replacement
```

複数実装をdifferential validationへ利用する場合は、実装系譜・algorithm・backend等が十分に独立していることを確認します。同一backendを薄くwrapした複数実装を独立referenceとして数えません。

また、独立した複数実装が一致することはcorrectnessの強い証拠になり得ますが、agreement自体をproof of correctnessとは扱いません。差異が生じた場合も単純な多数決をoracleとせず、semantic difference、rule configuration、bug、unsupported case等を調査します。

Python実装であることだけを理由にnative化を先行せず、semantic compatibility / correctness / measured performanceを確認してbackend採否を判断します。readableなfirst-party implementation、external reference、高速backendは、必要に応じて併存可能とします。

## Engine Track — First-party game engine

`lisjong-engine` で、日本式リーチ麻雀を指定RuleSetとseedに従って開始から最終結果まで決定的に進行できる状態を目指します。

主な能力は次の順序関係を持ちます。

```text
domain model / deterministic wall
        -> winning / scoring / RuleSet
        -> legal actions / round state transition
        -> round completion / result
        -> settlement / match progression
        -> final score / rank
        -> deterministic full-game driver
```

engine完成はAIの強さを条件としません。合法手生成、状態遷移、和了・点数、round / match進行、最終結果の正しさを優先します。

`lisjong-engine` はPolicyを所有せず、callerが選択した合法Actionを適用してgameを進行する責務に集中します。Mortal / MJAI等のexternal-agent integrationやevaluation orchestrationもengineへ持ち込みません。

## Policy Track — Hand-crafted Policy evolution

`lisjong` では、単純で決定的なPolicyを基準として、観測可能な情報だけからより良いActionを選択する能力を段階的に追加します。

長期的には、単一軸の牌効率改善ではなく、次の複数軸を統合するPolicyへ発展させます。

```text
Structural efficiency
  shanten / ukeire / lookahead
             |
             +-------------------+
             v                   v
Hidden-state inference      Value estimation
  remaining tiles            offensive value
  HandBelief                 defensive risk
             |                   |
             +---------+---------+
                       v
              Value / utility-aware
                    decision
```

主な能力には次を含みます。

- 向聴数、受け入れ枚数、lookahead等のstructural efficiency
- remaining tile information、HandBelief等のhidden-state inference
- 打点・offensive score potential
- 守備・安全度・defensive risk
- 立直判断、鳴き判断、押し引き
- offensive / defensive valueを統合したvalue-aware decision
- 将来の順位価値やgame-level objectiveを含み得るutility-aware decision

Expected valueは中心的な候補ですが、単一の局収支EVをproject-wideな最終目的関数として固定しません。将来の順位条件やgame-level objectiveに応じて、より適切なutilityを扱える余地を残します。

新しい能力を既存Policyへ無条件に上書きするのではなく、比較可能なPolicy世代を残し、実行時に利用するPolicy / AI configurationを選択できる設計を優先します。具体的なclass名、profile形式、algorithm、内部representationはproject-wide roadmapでは固定しません。

機能追加そのものではなく、既存Policyに対してどのような改善・退行を生んだかをArenaで評価可能にすることを重視します。このtrackは `lisjong-engine` の完成を待つ必要はありません。

## Arena Execution / Observation Track

`lisjong-arena` では、lisjongをconcrete environmentへ接続し、executionを観測・記録する能力をevaluationから分離したlayerとして発展させます。

概念的なruntime directionは次です。

```text
external / local environment
          |
          v
execution / observation
          |
          +--> DecisionContext -> lisjong Policy
          |
          +--> raw execution data
```

主な能力:

- environment-specific integration
- external / local runner / client
- RiichiLab等へのlive participation
- matchmaking / session lifecycle
- repeated / continuous participation
- retry / backoff等のexecution resilience
- execution profile / credential source resolution
- protocol trace
- raw game record / objective event取得
- environmentへ実際に送信・適用したActionの記録
- Policy contractとexternal Observation / legal Actionのsafe conversion

このtrackはAI判断ロジック、麻雀rule、Policy performance metric、comparison protocol、Arena固有seed / seat rotation semanticsを所有しません。

### Raw execution data とAI improvement

Arenaが取得するraw execution dataと、lisjongが所有するAI意味付けを分離します。

```text
RiichiEnv / RiichiLab / future environment
                 |
                 v
        execution / observation
                 |
          raw execution data
            /          \
           v            v
    Arena evaluation   lisjong AI improvement
```

Arena側ではraw game record、protocol trace、objective event、seat-visible observation、実際に適用したAction、game result等を扱えます。

そのdataをどのようなAI feature、training example、learned estimator targetとして解釈するかは `lisjong` 側の責務です。

AI内部のshanten / ukeire値、HandBelief、候補評価、選択理由等をraw execution recordへ暗黙に混在させません。必要ならDecisionTrace / AnalysisTrace等の別channelとして設計します。

### Visibility / secret boundary

```text
runtime credential / Authorization information
        -X-> trace / game record / evaluation artifact

privileged offline / ground-truth data
        -X-> online Policy input

privileged execution observation
        -X-> Policy decision path
```

execution / observationの追加がPolicy decisionへ干渉しない境界を維持します。

## Arena Evaluation Track — Reproducible Policy evaluation

`lisjong-arena` のevaluation layerを、Policy / game performanceを再現可能な条件で比較・評価する基盤として発展させます。

Evaluationには目的の異なる少なくとも2つのlaneを持たせます。

```text
Arena evaluation
    |
    +-- Round-level development evaluation
    |       rapid feedback / many samples
    |       self-improvement / regression detection
    |
    +-- Game-level / external benchmark
            overall game performance
            external competitor comparison
```

上位原則は、**評価対象に対して最小十分なevaluation scopeを選ぶこと**です。2つのlaneは代替関係でも厳格な直列gateでもありません。

### Lane 1 — Round-level development evaluation

局内decision qualityを主対象とする強化段階では、局単位の低コスト評価を高速feedback loopの中心に置けます。

概念的には次のような性格を持ちます。

```text
fixed / controlled seeds
        ×
round-level execution
        ×
many trials
```

主な目的:

- 小さなPolicy変更の高速比較
- regression検出
- Policy世代間比較
- game-level strategyが未成熟な段階での自己強化
- failureを局単位で分析・再実行すること

向聴、受け入れ、lookahead、HandBelief、offensive value、defensive risk、鳴き等の局内能力を強化する段階では、このlaneを中心に利用できます。

AABB / ABBB等の具体protocol、seed / seat rotation contract、sample size、metrics、confidence interval等は `lisjong-arena` 側の正本へ委ねます。

### Lane 2 — Hanchan-level external benchmark

成熟した外部AIに対するlisjongの総合的なgame performanceを確認するexternal benchmarkでは、半荘等のgame-level scopeを基本候補とします。

現在の初期external competitor候補はMortalです。Mortalとのbenchmarkでは大量の1局対戦を主要protocolとはせず、概念的には次の経路を優先検討します。

```text
                     lisjong-arena
                          |
                      RiichiEnv
                     /         \
                    v           v
            lisjong seat    Mortal seat
                |               |
                v               v
             lisjong          Mortal

        hanchan-level evaluation
        + controlled seat rotation
```

RiichiEnvが提供するMortal / MJAI interoperability、game execution、legal Action mapping等を利用できる場合は、それを優先します。同等のMJAI event generator / parser / legal-action mapper / game progressionをlisjong ecosystem側で再実装することを前提にしません。

これはRiichiEnvやMortalをproject-wide永久contractへ固定するものではなく、現在の優先execution / benchmark pathです。具体version、wrapper API、process方式、sample size、statistical protocol等は `lisjong-arena` のconcrete Issueで決定します。

external benchmarkでは将来的に、点棒状況、順位条件、親番、連荘、南場、オーラス、トップ取り、ラス回避、game-level utility等を含めた総合的な意思決定を評価できます。

Mortal benchmarkはgame-level strategy完成後にしか実施できないものとはしません。Policy成熟度を測るexternal referenceとして開発途中でも実施でき、game-level strategyの実装進展に伴ってbenchmarkの意味がより豊かになると捉えます。明確なdiagnostic目的があればMortalとのround-level comparisonを行うこと自体も禁止しません。

### Execution と evaluation の使い分け

Policy-vs-Policy development evaluation、RiichiLab live participation、mixed-agent external benchmarkはいずれもArena側のexecution / observation capabilityを利用できる方向へ発展させます。

ただし、execution layerは「何を比較するか」を知りません。

```text
execution / observation
        ^
        |
Arena evaluation
```

Evaluationがexecution capabilityを利用し、execution側へmatchup / seed rotation / metric semanticsを逆流させないことを原則とします。

ArenaはPolicy判断ロジック、麻雀ルール、単一gameの状態遷移を再実装しません。

## Evaluation quality layers

component correctness、Policy decision quality、game performanceは異なる評価対象として扱います。

```text
Component quality
       -> Decision quality
       -> Game performance
```

例えばHandBelief accuracyの向上と、HandBelief-aware Policyの対局成績向上は別の主張です。

概念的なownershipは次の通りです。

```text
Component validation
    -> component owning repository

Policy / game evaluation
    -> lisjong-arena evaluation

External benchmark for evaluation
    -> lisjong-arena evaluation

Live / standalone participation of lisjong itself
    -> lisjong-arena execution / observation
```

component-specific correctness / calibration testを `lisjong-arena` へ無理に集約しません。

## Convergence — Policy x first-party engine

`lisjong-engine` が実際にgameを完走できる実行環境として成立した段階で、既存Policy contractをfirst-party engineへ接続できる状態を目指します。

```text
lisjong-engine
      |
      v
execution / observation boundary
      |
      v
lisjong Policy
```

具体的なAdapter / integration ownershipと `lisjong -> lisjong-engine` dependencyの要否は、その時点の実APIとconsumerを確認して決定します。今回のrepository boundary変更だけを理由に、そのfirst-party dependencyを永久contractとして固定しません。

環境ごとの差異はPolicy判断ロジックやengineのルール実装へ漏らしません。

## Convergence — Arena x multiple game environments

first-party engineが実execution pathとして成立した後は、Arena execution / observationからRiichiEnvに加えて `lisjong-engine` 経路も利用できる状態を検討します。

概念的には次の形です。

```text
                       lisjong-arena
                 execution / observation
                     /             \
                    v               v
               RiichiEnv      lisjong-engine
                    \               /
                     \             /
                       lisjong Policy
```

RiichiEnvと`lisjong-engine`という複数の実execution pathが揃った時点で、それぞれの差異を実測し、共通化すべき境界が本当に存在するかを判断します。

それ以前に、将来APIを推測した汎用 `GameBackend` / `EvaluationBackend` 等のabstractionは導入しません。

## Visualization / Analysis Track

長期的なecosystem能力として、対局状況・牌譜・AI意思決定過程を観察、再生、分析できる状態を目指します。

```text
Game / analysis data
        |
        +-> replay
        +-> live spectator
        +-> Policy analysis overlay
        +-> decision debugging
```

Visualization / Analysisはread-orientedなconsumerとして位置付けます。Human Playは別の将来能力とし、Visualization Trackの必然的な最終段階とは規定しません。

Visualization / Analysisを特定のgame backendやArena artifactへ従属させません。first-party engine、external environment、Arena execution / observation等のconcrete sourceから得られるobjective events / snapshotsや、Policy固有のdecision / analysis data、Arenaのevaluation result / provenance / artifactを、必要に応じてanalysis consumerへ接続できる境界を目指します。

ただし、共通 `GameEvent` 等のproject-wide canonical event schemaを先に発明しません。RiichiLab / RiichiEnv / `lisjong-engine` / future consumers等の具体的requirementsを実際に扱った上で、必要なadapter / normalization boundaryを抽出します。

長期的には、live対局観戦、保存牌譜再生、regression局の再生、candidate / baseline比較、Policy decision analysis、Policy evolutionの追跡等へ発展できる余地を残します。具体的なviewer repository、GUI framework、protocol、OSS採用はこのroadmapでは固定しません。

## Learning Policy

手書きPolicyとcomparison protocolが十分に安定し、Policy改善を再現可能に評価できる状態になった後、学習Policyを導入します。

```text
Strong hand-crafted baseline
          +
Reproducible evaluation
          +
External validation
          +
Replay / analysis
          -> Learning Policy
```

この段階では、既存のPolicy contract、component validation、execution / observation、round / game-level evaluation、external benchmark、replay / analysisを可能な限り再利用し、学習Policyだけを特別扱いする別系統の実行基盤を不用意に作りません。

hand-crafted Policyもbaseline、regression reference、feature / estimator comparison対象、explanation referenceとして継続利用できるようにします。

学習アルゴリズム、model形式、training data、self-play方式、計算基盤は、その時点の要件と実測に基づいて設計し、本ロードマップでは先に固定しません。

## Runtime extraction trigger

現時点では独立した `lisjong-runtime` repositoryを先行作成しません。

次のようなconcrete requirementが成立した場合に再検討します。

- 24/7 production bot hosting
- evaluationとは独立したdeployment
- generic process / agent hosting
- Arena以外の複数consumerが同じruntimeを必要とする
- execution infrastructureがArena固有評価責務から独立して大きく成長する
- Arena repository内のpackage-level分離では責務境界を維持しにくくなる

抽出は実consumer / operational requirementを根拠に行い、将来需要を推測してgeneric runtime abstractionを先行設計しません。

## Ordering constraints

本ロードマップ上で重要な順序制約は、直列Phase番号ではなく次の依存関係として扱います。

- Policy contractはPolicy改善と複数環境接続の基礎になる
- correctness baselineを確立し、独立validationとregression protectionを行ってから性能最適化へ進む
- Engine TrackはAIの強さとは独立して正しさを完成させる
- Policy TrackとArena execution / observationはfirst-party engine完成前でも進められる
- Arena Evaluation Trackの初期比較はfirst-party engine完成前でも進められる
- 現在の局内decision改善では低コストなround-level evaluationを高速iterationの中心にする
- external benchmarkでは評価したいgame-level semanticsに応じて半荘等のscopeを選択する
- Mortal等のexternal benchmarkはgame-level strategy完成を必須gateとしない
- 高度な統計機能は最小comparison protocolによる実データを得てから追加する
- first-party engine integrationはengineが実execution pathとして成立してから具体化する
- generic backend / runtime abstractionは複数の実execution pathやconcrete consumerが揃ってから判断する
- Visualization / Analysisの共通境界は具体的なsource / consumer requirementsから抽出する
- Learning Policyは再現可能なPolicy comparisonが成立してから導入し、既存のexecution・評価・分析基盤を再利用する

## What does not belong here

次の情報は本書では管理しません。

- 現在作業中のIssue番号
- Issueごとのacceptance criteria
- PRの状態
- 「次は#XX」のような直近作業順序
- release日程の推測
- 未確定な内部APIやpackage構成
- 特定OSSのversionや具体adapter / wrapper仕様
- AABB / ABBB等の具体的なevaluation protocol
- Mortal benchmarkの具体的sample size / statistical threshold
- 未検証のproject-wide canonical event schema
- 既存Adapter / runner / trace contractのmigration進捗

現在の長期的なpreferred execution / benchmark pathとしてOSS名を示すことはありますが、そのversionや具体integration contractは各consumer repository / Issueを正本とします。

## Updating this roadmap

本書は、個別Issueが完了するたびには更新しません。

ecosystem全体の到達目標、track間の依存関係、Convergenceの開始条件、repository分離や責務境界、評価戦略、長期的なpreferred execution pathなど、方向性が変わった場合に更新します。
