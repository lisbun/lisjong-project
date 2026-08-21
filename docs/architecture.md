# Architecture

## 目的

本書は lisjong ecosystem 全体のrepository責務、repository間の依存方向、横断的な責務境界を定める正本です。

個別repository内部のmodule構成、公開API、実装詳細はそれぞれの `docs/architecture.md` を正本とします。
現在の作業内容・進捗・完了状態はGitHub Issues / PRsを正本とし、本書には重複して記録しません。

## Repository responsibilities

### `lisjong`

`lisjong` は、観測可能な麻雀状態からActionを選ぶAI decision coreを担当します。

主な責務:

- Policy / AI戦略
- `DecisionContext` / Policy contract
- `InternalAction` 等のAI-side contract
- 実行時に利用するPolicy / AI configurationの選択
- Policy返却値の合法性・semantic identity validation
- 向聴数、受け入れ枚数、牌効率、lookahead
- remaining tile information、HandBelief等のhidden-state inference
- 将来的な立直判断、打点・offensive value、鳴き、守備・defensive risk、押し引き、value / utility-aware decision、学習Policy
- Policy内部componentのcorrectness / calibration validation
- AI feature / training example / learned estimatorの意味契約

lisjongの中心的な境界は概念上次とします。

```text
DecisionContext
      |
      v
    Policy
      |
      v
InternalAction
```

RiichiEnv / RiichiLab等の外部environment固有型、WebSocket、credential、matchmaking、retry / reconnect、continuous participation、game record persistence等のexecution concernをPolicy contractへ持ち込みません。

`lisjong` は麻雀ルールそのものを進行するgame engineでも、外部環境へのstandalone runner / clientの長期的な所有者でも、複数trialの評価計画・集計を行うrepositoryでもありません。

個々の既存Adapter / runner / trace contractをどこへ置くかは、consumerとdependencyを確認してconcrete Issueで決定します。project-wide architectureだけを理由に同じsemanticsを複数repositoryへ重複実装しません。

### `lisjong-engine`

`lisjong-engine` は、与えられたActionに従って日本式リーチ麻雀を正しく進行し、対局結果を生成するgame engineを担当します。

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
- engine固有component / rule correctnessのvalidation

`lisjong-engine` はPolicy、AI戦略、学習、外部AI hosting、Mortal / MJAI等のexternal-agent integration、RiichiEnv / RiichiLab固有integration、evaluation orchestrationを持ちません。

### `lisjong-arena`

`lisjong-arena` は、lisjongをconcrete environmentで実行・観測し、そのexecution dataを再現可能なPolicy / game evaluationへ接続するarena基盤を担当します。

同一repository内でも、次の2責務を分離します。

```text
execution / observation
        |
        v
 raw execution data
        |
        v
    evaluation
```

#### Execution / observation

主な責務:

- environment-specific integration
- RiichiLab client / Adapter等のexternal participation能力
- local / external execution runner
- matchmaking participation
- repeated / continuous participation
- retry / backoff等のexecution resilience
- execution profile / credential source resolution
- protocol trace
- raw game record / objective execution eventの取得
- environmentへ実際に送信・適用したActionの記録
- Policy contractとexternal Observation / legal Actionを接続するenvironment-facing conversion

このlayerはPolicy performance metric、comparison protocol、Arena固有seed / seat rotation semanticsへ依存しません。またAI判断ロジックや麻雀ruleを再実装しません。

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
- 将来的な統計的比較・external benchmark / report
- evaluation目的のexternal competitor integration / orchestration

Evaluationはexecution / observation能力をconsumerとして利用できますが、その逆方向の依存は避けます。

`lisjong-arena` はAI判断ロジック、Policy内部componentのcorrectness / calibration test、麻雀ルール、単一gameの状態遷移を再実装しません。

execution infrastructureがArena外の複数concrete consumerから必要になった場合、または24/7 production hosting等の独立したoperational responsibilityへ成長した場合にのみ、共通runtime / repositoryへの抽出を再検討します。

## External execution paths

external executionはArena側のexecution / observation responsibilityとして発展させます。ただし、既存integrationを一括移動することや、project-wideなgeneric backendを先行設計することは要求しません。

### Policy-vs-Policy development evaluation

Policy同士のdevelopment evaluationでは、Arenaが利用可能なconcrete execution pathを選択します。

```text
lisjong-arena
      |
      | execution / observation
      v
external game environment
      |
      | DecisionContext / InternalAction
      v
   lisjong Policy
```

既存のRiichiEnv Adapter / runner等をどのrepositoryが最終所有するかは、concrete migration Issueでconsumerとdependencyを確認して決定します。

### Live / standalone participation

RiichiLab等へlisjong自身を参加させるfirst-party entry pointは、target responsibilityとしてArena側のexecution / observation layerへ寄せます。

接続・session lifecycle・matchmaking・retry / reconnect・continuous participation・protocol trace等はAI decision coreとは分離します。

### Mixed-agent external benchmark

Mortal等のexternal competitorを含むbenchmarkでは、Arenaが選択したOSS execution environment等を直接orchestrateしてよいものとします。

```text
                  lisjong-arena
                       |
              execution environment
                   /         \
                  v           v
          lisjong seat   external competitor
               |               |
               v               v
            lisjong      external agent
```

この場合でも、ArenaがlisjongのPolicy判断ロジックや麻雀ルールを複製してよいことを意味しません。具体的なreuse API、external environment、competitor、protocolはconsumer repository / Issue側で決定します。

## Dependency direction

今回のrepository boundaryで少なくとも維持するfirst-party dependency ruleは次です。

```text
lisjong-arena -> lisjong
lisjong-engine -X-> lisjong
lisjong -X-> lisjong-arena
```

`lisjong -> lisjong-engine` の必要性そのものは、本architecture変更では再設計しません。first-party engineがconcrete execution pathとして利用される時点で、実際のAPIとconsumerを確認し、必要であれば別のproject-wide decisionで再評価します。

これとは別に、consumer repositoryが用途に応じて外部OSSへ依存することを許容します。

```text
first-party dependency
    lisjong-arena -> lisjong

example execution dependency
    lisjong-arena -> selected external game environment
```

external dependencyを許容することと、ArenaがAI判断ロジックや麻雀ルールを重複実装してよいことは別です。

`lisjong-engine` は `lisjong` のPolicyやAI実装を知らずに成立する必要があります。`lisjong` もArenaのexecution lifecycleやcomparison protocolを知らずにAI decision coreとして成立する必要があります。

`lisjong-project` はdocumentation / coordination repositoryであり、このruntime dependency graphには含めません。

## External ecosystem boundary

成熟したOSSや外部実装は、reference、backend、benchmark、toolingとして積極的に評価・利用します。

external benchmark、simulation、game execution、protocol interoperability等に必要な能力を成熟したOSSが既に提供している場合は、それを優先的に評価・利用し、同等機能をlisjong ecosystem内で無目的に重複実装しません。

ただし、project-wideなstable public contract、Policy semantics、repository responsibility、project固有artifact contract、cross-repository dependency directionはlisjong ecosystem側で所有します。

外部OSS固有の型・API・内部設計を上位contractへ直接漏らしません。具体的なOSS名、version、adapter、dependency採否は、それを利用するrepository / Issue側で決定します。

correctness validationで複数実装を比較する場合は、実装系譜・algorithm・backend等が十分に独立していることを確認します。同一backendを薄くwrapした複数実装を独立referenceとして数えません。

複数の独立実装のagreementはcorrectnessの強い証拠になり得ますが、proof of correctnessとは扱いません。差異が発生した場合も多数決をoracleとせず、semantic difference、rule configuration、bug、unsupported case等を調査します。

性能最適化はcorrectness、独立validation、regression protectionの後に行います。Python実装であることだけを理由にnative backendへ移行せず、実測されたbottleneckとsemantic compatibilityを確認してから最適化・backend置換を判断します。

## Evaluation ownership

component quality、Policy decision quality、game performanceは異なる評価対象として扱います。

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

例えばHandBelief accuracyの向上は `lisjong` 側componentのquality claimであり、そのHandBeliefを利用するPolicyが対局上強くなったかはArena側のdecision / game performance claimです。この2つを同一の評価として扱いません。

Arenaへcomponent-specific correctness / calibration testを無理に集約しません。また、deterministic reproducibilityとstatistical strength claimを分離し、同じseed / protocolを再実行できることだけをPolicy strengthの統計的証明とはみなしません。

評価scopeは評価対象に対して最小十分なものを選びます。局内decision qualityを主対象とする段階ではround-level evaluationを高速feedback loopとして利用でき、点棒状況・順位条件・オーラス等のgame-level objectiveが重要になった段階では半荘等のgame-level validationへ拡張します。

AABB / ABBB、具体的なseed / seat rotation contract、sample size、variance、confidence interval、paired comparison等はproject-wide architectureでは固定せず、`lisjong-arena` の正本へ委ねます。

## Execution data, AI improvement, and Visualization / Analysis boundary

external environmentから得るraw execution dataと、それをAI改善・評価・可視化へ利用する意味付けを分離します。

```text
first-party engine / external environment / live integration
                       |
                       v
          lisjong-arena execution / observation
                       |
               raw execution data
                 /          \
                v            v
       arena evaluation    AI improvement input
                              |
                              v
                           lisjong

Policy decision / analysis data --------+
Arena result / provenance / artifact ---+--> analysis consumer
```

Arena側のraw execution dataには、例えば次を含められます。

- raw game record
- protocol trace
- objective execution event
- seat-visible observation record
- environmentへ実際に送信・適用したAction
- game result

一方、shanten / ukeire値、HandBelief、候補評価、選択理由等のAI内部analysisをexecution recordへ暗黙に混在させません。必要な場合はDecisionTrace / AnalysisTrace等の別channelとして設計します。

training example semantics、learned estimatorのinput / target、feature semantics、component calibration等は `lisjong` 側のAI responsibilityです。Arenaが牌譜を取得することと、その牌譜をどのようなtraining exampleへ変換するかは別責務です。

### Visibility / secret boundary

次の情報境界を維持します。

```text
runtime credential / Authorization information
        -X-> trace / game record / evaluation artifact

privileged offline / ground-truth data
        -X-> online Policy input

privileged execution observation
        -X-> Policy decision path
```

- token、Authorization header等のsecretをtrace / game record / artifactへ保存しない
- offline component evaluationで利用可能なhidden ground truthをonline Policy inputへ逆流させない
- privileged observer informationをPolicy-visible stateへ追加しない
- execution / observationの追加がPolicy選択へ干渉しない境界を維持する

Visualization / Analysisは、対局状況・牌譜・Policy意思決定過程・evaluation結果を観察、再生、分析するread-orientedなconsumer能力として扱います。

project-wide canonical event schemaの新設を先行要件としません。共通 `GameEvent` 等を先に発明するのではなく、RiichiLab / RiichiEnv / `lisjong-engine` / future consumers等のconcrete requirementsを実際に扱った上で、必要なadapter / normalization boundaryを抽出します。

Visualization / Analysisの責務原則は次の通りです。

- viewerは麻雀ruleを所有しない
- viewerはPolicy decision logicを所有しない
- viewerはArena evaluation protocolを所有しない
- GUI都合の型をengine / Policy contractへ逆流させない
- viewerの停止や失敗がgame execution / Policy decisionへ影響しない設計を目指す
- liveとreplayで共通化可能なsemanticsは再利用するが、早期のcanonical schema固定を避ける
- Arena artifactをviewer唯一の入力経路としない
- adapter / normalization contractは具体的consumer requirementsから抽出する

Human PlayはVisualization / Analysisとは別の将来能力です。具体的なviewer repository、GUI framework、protocol、OSS採用は現時点では固定しません。

## Runner responsibilities

「runner」は異なる責務を指し得るため、ecosystem全体では次の3種類を区別します。

```text
麻雀そのもののgame runner
    -> lisjong-engine

lisjongをゲーム環境へ接続するintegration runner / client
    -> lisjong-arena execution / observation

複数試行を計画・集計するarena / comparison runner
    -> lisjong-arena evaluation
```

### Game runner

ゲーム状態を所有し、Actionを適用してround / matchを進行します。合法手・終了条件・結果生成はengine側の責務です。

### Integration runner / client

RiichiEnv、RiichiLab、将来利用するfirst-party engine等の実行環境へlisjong Policyを接続します。外部Observation / Action表現とPolicy contractの境界、および必要なstandalone session lifecycleを扱いますが、麻雀ルールそのものやPolicy判断ロジックは再実装しません。

### Arena / comparison runner

複数trialのseed、seat、Policy / agent組合せ、evaluation scope、試行数等を計画し、raw resultを収集・集計して比較します。execution / observation layerを利用できますが、integration layerへcomparison semanticsを逆流させません。

## Issue placement rules

新しい課題のplacementは「どのrepositoryの目的を成立させるために必要な責務か」で判断します。

- 麻雀ゲームを正しく進行するために必要なら `lisjong-engine`
- 観測からActionを選ぶAI、Policy内部component、Policy / AI configuration、AI feature / training semanticsなら `lisjong`
- environment接続、RiichiLab等への参加、session lifecycle、retry / reconnect、raw execution observation、またはPolicy / game evaluationなら `lisjong-arena`
- repository境界、依存方向、evaluation ownership、observable boundary等のproject-wide原則を変更するなら `lisjong-project`

具体的な既存Adapter / runner / trace contractのmigration先は、project-wide ruleだけから機械的に決めず、consumerとdependencyを確認してconcrete Issueで決定します。

Visualization / Analysisの具体的な実装repositoryは現時点で固定しません。consumer requirementsが具体化した時点で、既存repositoryの責務を侵食しないplacementを決定します。

複数repositoryに変更が必要な機能でも、同じ仕様を複数repoへ重複して持たせません。横断契約をどこが所有するかを先に決め、各repoは自分の内部実装だけを持ちます。

execution infrastructureがArena外の複数concrete consumerから必要になった場合は、その時点で共通runtime / repositoryへの抽出を再検討します。将来のconsumerを推測してgeneric runtimeを先行設計しません。

## Source-of-truth boundary

```text
lisjong-project
    ecosystemの構造
    cross-repository principles

各repositoryのarchitecture
    repository内部の構造
    concrete contract / implementation boundary

GitHub Issues / PRs
    現在の仕事
    concrete adoption / protocol decisions
```

この境界を維持し、現在のIssue番号や完了状況、特定OSSのversion、具体的evaluation protocol等をproject-wide architectureへ埋め込まないことを基本ルールとします。
