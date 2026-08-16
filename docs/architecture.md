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
- 向聴数、受け入れ枚数、牌効率
- 将来的な立直判断、打点評価、鳴き、守備、押し引き、学習Policy
- RiichiEnv / RiichiLab / 将来の `lisjong-engine` とPolicyを接続するAdapter・integration
- Policy選択結果の環境側合法手への対応付け・再検証

`lisjong` は麻雀ルールそのものを進行するgame engineではありません。

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

`lisjong-engine` はPolicy、AI戦略、学習、RiichiEnv / RiichiLab固有integrationを持ちません。

### `lisjong-arena`

`lisjong-arena` は、複数のlisjong Policyを再現可能な条件で対局させ、Policy全体の強さを比較するarena / comparison基盤を担当します。

主な責務:

- Policy同士のmatchup定義
- evaluation用seed集合
- deterministicなseat rotation
- 複数gameの実行計画
- Policy assignmentの記録
- raw game result収集
- 平均順位・平均得点・順位回数等の基本metrics
- 再現可能なPolicy comparison protocol
- 将来的な統計的比較・benchmark / report

`lisjong-arena` はAI判断ロジック、Action / state evaluation、麻雀ルール、単一gameの状態遷移を所有しません。

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

この構成では、Arenaは複数対局の計画と集計に集中し、Policy実行・Observation / Action変換・単一gameのRiichiEnv orchestrationを `lisjong` へ委譲します。

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

## Runner responsibilities

「runner」は異なる責務を指し得るため、ecosystem全体では次の3種類を区別します。

```text
麻雀そのもののgame runner
    -> lisjong-engine

Policyとゲーム環境を接続するintegration runner
    -> lisjong

複数対局を実行・集計してPolicyを比較するarena / comparison runner
    -> lisjong-arena
```

### Game runner

ゲーム状態を所有し、Actionを適用してround / matchを進行します。合法手・終了条件・結果生成はengine側の責務です。

### Integration runner

RiichiEnv、RiichiLab、`lisjong-engine` 等の実行環境とPolicyを接続します。外部Observation / Action表現とPolicy contractの境界を扱いますが、麻雀ルールそのものは再実装しません。

### Arena / comparison runner

複数gameのseed、seat、Policy組合せ、試行数等を計画し、raw resultを収集・集計してPolicyを比較します。単一gameのルール進行、環境固有Adapter、Policy判断ロジックは所有しません。

## Issue placement rules

新しい課題のplacementは「どのrepositoryの目的を成立させるために必要な責務か」で判断します。

- 麻雀ゲームを正しく進行するために必要なら `lisjong-engine`
- 観測からActionを選ぶAI、またはPolicyと環境の接続なら `lisjong`
- 複数対局を実験条件に従って比較・集計するなら `lisjong-arena`
- repository境界や依存方向そのものを変更するなら `lisjong-project`

複数repositoryに変更が必要な機能でも、同じ仕様を複数repoへ重複して持たせません。横断契約をどこが所有するかを先に決め、各repoは自分の内部実装だけを持ちます。

## Source-of-truth boundary

```text
lisjong-project
    ecosystemの構造

各repositoryのarchitecture
    repository内部の構造

GitHub Issues / PRs
    現在の仕事
```

この境界を維持し、現在のIssue番号や完了状況をproject-wide architectureへ埋め込まないことを基本ルールとします。
