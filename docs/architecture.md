# Architecture

## 目的

本書は lisjong ecosystem 全体のrepository責務、repository間の依存方向、横断的な責務境界を定める正本です。

個別repository内部のmodule構成、公開API、実装詳細はそれぞれの `docs/architecture.md` を正本とします。
現在の作業内容・進捗・完了状態はGitHub Issues / PRsを正本とし、本書には重複して記録しません。

## Repository responsibilities

### `lisjong`

`lisjong` は、観測可能な麻雀状態からActionを選ぶAIと、そのPolicyを各実行環境へ接続するintegration層を担当します。

主な責務:

- Policy / AI戦略
- `DecisionContext` / Policy contract
- 向聴数、受け入れ枚数、牌効率、lookahead
- remaining tile information、HandBelief等のhidden-state inference
- 将来的な立直判断、打点・offensive value、鳴き、守備・defensive risk、押し引き、value / utility-aware decision、学習Policy
- Policy内部componentのcorrectness / calibration validation
- RiichiEnv / RiichiLab / 将来の `lisjong-engine` とPolicyを接続するAdapter・integration
- Policy選択結果の環境側合法手への対応付け・再検証

`lisjong` は麻雀ルールそのものを進行するgame engineではなく、複数gameを集計するArena evaluation protocolも所有しません。

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

`lisjong-engine` はPolicy、AI戦略、学習、RiichiEnv / RiichiLab固有integrationを持ちません。

### `lisjong-arena`

`lisjong-arena` は、複数のlisjong Policyを再現可能な条件で評価し、Policy decision qualityやgame performanceを比較するarena / comparison基盤を担当します。

主な責務:

- Policy同士のmatchup定義
- evaluation用seed集合
- deterministicなseat rotation
- round / game等のevaluation scopeに応じた実行計画
- Policy assignmentの記録
- raw result収集
- evaluation artifact / provenance
- 評価scopeに応じたmetrics
- 再現可能なPolicy comparison protocol
- 将来的な統計的比較・benchmark / report

`lisjong-arena` はAI判断ロジック、Policy内部componentのcorrectness / calibration test、麻雀ルール、単一gameの状態遷移を所有しません。

## Initial arena execution path

`lisjong-engine` の完成は `lisjong-arena` 開始の前提にしません。初期Arenaは、すでに存在する `lisjong` のRiichiEnv integrationを利用してPolicy比較を成立させます。

```text
lisjong-arena
    |
    | matchup / seeds / seat rotation / aggregation
    v
lisjong
    |
    | existing RiichiEnv integration
    v
RiichiEnv
```

この構成では、Arenaは複数試行の計画と集計に集中し、Policy実行・Observation / Action変換・単一gameのRiichiEnv orchestrationを `lisjong` へ委譲します。

将来 `lisjong-engine` が実際に対局可能になった段階では、`lisjong` 側のengine integrationをArenaから利用できるようにします。RiichiEnvと`lisjong-engine`という2つの実経路が揃う前に、将来APIを推測した汎用 `GameBackend` / `EvaluationBackend` 等のabstractionは導入しません。共通化は実経路の差異を確認してから判断します。

## Dependency direction

project-wideに許可する依存方向は次の通りです。

```text
lisjong-arena -> lisjong
lisjong -> lisjong-engine
lisjong-engine -X-> lisjong
lisjong -X-> lisjong-arena
```

初期Arenaでは `lisjong-arena -> lisjong -> RiichiEnv` の経路を使用します。`lisjong-arena` がRiichiEnv固有のObservation / Action変換や麻雀ルールを再実装しないことを重要な境界とします。

`lisjong-engine` は `lisjong` のPolicyやAI実装を知らずに成立する必要があります。`lisjong` もArenaの比較protocolを知らずに単一game integrationを提供できる必要があります。

`lisjong-project` はdocumentation / coordination repositoryであり、このruntime dependency graphには含めません。

## External ecosystem boundary

成熟したOSSや外部実装は、reference、backend、benchmark、toolingとして利用できます。ただし、project-wideなstable public contract、Policy semantics、repository responsibility、project固有artifact contract、cross-repository dependency directionはlisjong ecosystem側で所有します。

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

External / live validation
    -> relevant integration boundary
```

例えばHandBelief accuracyの向上は `lisjong` 側componentのquality claimであり、そのHandBeliefを利用するPolicyが対局上強くなったかはArena側のdecision / game performance claimです。この2つを同一の評価として扱いません。

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

Policyとゲーム環境を接続するintegration runner
    -> lisjong

複数試行を実行・集計してPolicyを評価するarena / comparison runner
    -> lisjong-arena
```

### Game runner

ゲーム状態を所有し、Actionを適用してround / matchを進行します。合法手・終了条件・結果生成はengine側の責務です。

### Integration runner

RiichiEnv、RiichiLab、`lisjong-engine` 等の実行環境とPolicyを接続します。外部Observation / Action表現とPolicy contractの境界を扱いますが、麻雀ルールそのものは再実装しません。

### Arena / comparison runner

複数trialのseed、seat、Policy組合せ、evaluation scope、試行数等を計画し、raw resultを収集・集計してPolicyを比較します。単一gameのルール進行、環境固有Adapter、Policy判断ロジックは所有しません。

## Issue placement rules

新しい課題のplacementは「どのrepositoryの目的を成立させるために必要な責務か」で判断します。

- 麻雀ゲームを正しく進行するために必要なら `lisjong-engine`
- 観測からActionを選ぶAI、Policy内部component、またはPolicyと環境の接続なら `lisjong`
- Policy / game evaluationを複数trialの実験条件に従って比較・集計するなら `lisjong-arena`
- repository境界、依存方向、evaluation ownership、observable boundary等のproject-wide原則を変更するなら `lisjong-project`

Visualization / Analysisの具体的な実装repositoryは現時点で固定しません。consumer requirementsが具体化した時点で、既存repositoryの責務を侵食しないplacementを決定します。

複数repositoryに変更が必要な機能でも、同じ仕様を複数repoへ重複して持たせません。横断契約をどこが所有するかを先に決め、各repoは自分の内部実装だけを持ちます。

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
