# Architecture

## 目的

本書は lisjong ecosystem 全体のrepository責務、repository間の依存方向、横断的な責務境界を定める正本です。

個別repository内部のmodule構成、公開API、実装詳細はそれぞれの `docs/architecture.md` を正本とします。
現在の作業内容・進捗・完了状態はGitHub Issues / PRsを正本とし、本書には重複して記録しません。

## Repository responsibilities

### `lisjong`

`lisjong` は、観測可能な麻雀状態からActionを選ぶAIと、そのAI自身を実際の対局環境で利用可能にするself-integration能力を担当します。

主な責務:

- Policy / AI戦略
- `DecisionContext` / Policy contract
- 実行時に利用するPolicy / AI configurationの選択
- 向聴数、受け入れ枚数、牌効率、lookahead
- remaining tile information、HandBelief等のhidden-state inference
- 将来的な立直判断、打点・offensive value、鳴き、守備・defensive risk、押し引き、value / utility-aware decision、学習Policy
- Policy内部componentのcorrectness / calibration validation
- RiichiEnv / RiichiLab / 将来の `lisjong-engine` とPolicyを接続するAdapter・integration
- Policy選択結果の環境側合法手への対応付け・再検証
- Arenaなしでlisjong自身を外部環境へ参加させるstandalone execution

self-integrationは、概念上次の2つを区別します。

```text
environment-facing self-integration
    1 seat分の外部環境表現とPolicy contractを接続する能力

standalone execution
    lisjong自身を実際の環境へ参加させるrunner / client能力
```

これはproject-wideな能力境界であり、単一のclassやmoduleがObservation変換、Policy実行、Action mapping、session lifecycleをすべて所有することを要求しません。具体的なAdapter / Policy execution / runner / clientの分割は `lisjong` 側architectureを正本とします。

`lisjong` は麻雀ルールそのものを進行するgame engineではなく、他AIのhosting、external competitor専用integration、複数trialの評価計画・集計も原則として所有しません。

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

`lisjong-arena` は、lisjongのPolicy / game performanceを再現可能な条件で評価するarena / comparison基盤を担当します。

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
- evaluationだけを目的とするexternal competitor integration / orchestration

`lisjong-arena` はAI判断ロジック、Policy内部componentのcorrectness / calibration test、麻雀ルール、単一gameの状態遷移を再実装しません。

external competitor integrationは、まずそのevaluationを成立させるArena-private implementation detailとして開始してよいものとします。同じintegrationがArena外の複数concrete consumerからも必要になった場合にのみ、共通runtime / repositoryへの抽出を再検討します。

## Arena execution paths

Arenaの実行経路は用途ごとに扱います。既存のPolicy-vs-Policy evaluation経路を否定せず、mixed-agent external benchmarkで必要な追加経路だけを許容します。

### Policy-vs-Policy development evaluation

Policy同士のdevelopment evaluationでは、`lisjong` が提供する既存self-integration / standalone executionを利用できます。

```text
lisjong-arena
      |
      v
   lisjong
      |
      v
external game environment
```

この経路では、Arenaはmatchup、seed、seat rotation、trial、result / metricsに集中し、lisjong自身を環境へ接続する処理は `lisjong` 側の責務を再利用します。

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

この場合でも、Arenaがlisjong用のObservation / legal Action / selected Action mapping semanticsを独自に複製してよいことを意味しません。Arenaは `lisjong` のstandalone runner全体を必ず再利用する必要はありませんが、lisjong自身を1 seatとして実行するためのenvironment-facing self-integration semanticsを可能な限り再利用します。

具体的なreuse API、external environment、competitor、protocolはconsumer repository / Issue側で決定します。

## Dependency direction

first-party repository間で許可する依存方向は次の通りです。

```text
lisjong-arena -> lisjong
lisjong -> lisjong-engine
lisjong-engine -X-> lisjong
lisjong -X-> lisjong-arena
```

これとは別に、consumer repositoryが用途に応じて外部OSSへ依存することを許容します。例えばArenaがevaluation execution / interoperabilityのために外部game environmentへ直接依存しても、first-party依存方向を変更したことにはなりません。

```text
first-party dependency
    lisjong-arena -> lisjong

example evaluation-specific external dependency
    lisjong-arena -> selected OSS execution environment
```

external dependencyを許容することと、Arenaがlisjong用Adapter / conversion semanticsや麻雀ルールを重複実装してよいことは別です。

`lisjong-engine` は `lisjong` のPolicyやAI実装を知らずに成立する必要があります。`lisjong` もArenaの比較protocolを知らずに自分自身のintegrationを提供できる必要があります。

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
    -> lisjong-arena

External benchmark for evaluation
    -> lisjong-arena

Live / standalone participation of lisjong itself
    -> lisjong self-integration
```

例えばHandBelief accuracyの向上は `lisjong` 側componentのquality claimであり、そのHandBeliefを利用するPolicyが対局上強くなったかはArena側のdecision / game performance claimです。この2つを同一の評価として扱いません。

また、lisjong自身がRiichiLab等のlive environmentへ参加する能力はArenaを必要とせず `lisjong` が所有します。一方、外部AIとの対戦をlisjongの強さを測るbenchmarkとして実施する場合は `lisjong-arena` が所有します。

Arenaへcomponent-specific correctness / calibration testを無理に集約しません。また、deterministic reproducibilityとstatistical strength claimを分離し、同じseed / protocolを再実行できることだけをPolicy strengthの統計的証明とはみなしません。

評価scopeは評価対象に対して最小十分なものを選びます。局内decision qualityを主対象とする段階ではround-level evaluationを高速feedback loopとして利用でき、点棒状況・順位条件・オーラス等のgame-level objectiveが重要になった段階では半荘等のgame-level validationへ拡張します。

AABB / ABBB、具体的なseed / seat rotation contract、sample size、variance、confidence interval、paired comparison等はproject-wide architectureでは固定せず、`lisjong-arena` の正本へ委ねます。

## Observable data and Visualization / Analysis boundary

Visualization / Analysisは、対局状況・牌譜・Policy意思決定過程・evaluation結果を観察、再生、分析するread-orientedなconsumer能力として扱います。

概念的には次のsourceをconsumerへ接続できる境界を目指します。

```text
first-party engine / external environment / live integration
                       |
                       v
             objective events / snapshots
                       |
              +--------+--------+
              |                 |
              v                 v
        live consumer      persisted log / replay

Policy decision / analysis data --------+
Arena result / provenance / artifact ---+--> analysis consumer
```

ただし、project-wide canonical event schemaの新設を先行要件としません。共通 `GameEvent` 等を先に発明するのではなく、RiichiLab / RiichiEnv / `lisjong-engine` / future consumers等のconcrete requirementsを実際に扱った上で、必要なadapter / normalization boundaryを抽出します。

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

lisjong自身をゲーム環境へ接続するintegration runner / client
    -> lisjong

複数試行を実行・集計し、必要に応じてexternal competitorをorchestrateするarena / comparison runner
    -> lisjong-arena
```

### Game runner

ゲーム状態を所有し、Actionを適用してround / matchを進行します。合法手・終了条件・結果生成はengine側の責務です。

### Integration runner / client

RiichiEnv、RiichiLab、`lisjong-engine` 等の実行環境へlisjong自身を接続します。外部Observation / Action表現とPolicy contractの境界、および必要なstandalone session lifecycleを扱いますが、麻雀ルールそのものは再実装しません。

### Arena / comparison runner

複数trialのseed、seat、Policy / agent組合せ、evaluation scope、試行数等を計画し、raw resultを収集・集計して比較します。Policy-vs-Policy evaluationでは `lisjong` のintegrationを利用できます。mixed-agent external benchmarkではArenaが選択したexternal execution environmentを直接orchestrateしてよい一方、lisjong用のenvironment-facing semanticsやPolicy判断ロジック、麻雀ルールは再実装しません。

## Issue placement rules

新しい課題のplacementは「どのrepositoryの目的を成立させるために必要な責務か」で判断します。

- 麻雀ゲームを正しく進行するために必要なら `lisjong-engine`
- 観測からActionを選ぶAI、Policy内部component、Policy / AI configuration selection、またはlisjong自身を環境へ接続・参加させるself-integrationなら `lisjong`
- Policy / game evaluationを複数trialの実験条件に従って比較・集計する場合、またはevaluation-only external competitor integrationなら `lisjong-arena`
- repository境界、依存方向、evaluation ownership、observable boundary等のproject-wide原則を変更するなら `lisjong-project`

Visualization / Analysisの具体的な実装repositoryは現時点で固定しません。consumer requirementsが具体化した時点で、既存repositoryの責務を侵食しないplacementを決定します。

複数repositoryに変更が必要な機能でも、同じ仕様を複数repoへ重複して持たせません。横断契約をどこが所有するかを先に決め、各repoは自分の内部実装だけを持ちます。

external-agent integrationがArena外の複数concrete consumerから必要になった場合は、その時点で共通runtime / repositoryへの抽出を再検討します。将来のconsumerを推測してgeneric external-player hostを先行設計しません。

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
