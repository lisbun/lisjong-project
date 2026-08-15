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

### 将来の `lisjong-eval`

`lisjong-eval` は、複数の対局結果を再現可能な実験条件で収集し、Policyの強さを比較する評価基盤を担当します。

想定責務:

- Policy A / Bの比較
- 評価用seed集合
- seat rotation
- 複数半荘の実行計画
- 結果収集
- 平均順位・平均得点・順位率等のmetrics
- 統計的比較
- benchmark / report
- 再現可能なevaluation protocol

AI判断ロジックや麻雀ルールそのものは持たせません。

`lisjong-eval` の具体的なpackage依存方向は、repositoryを実際に分離する段階で必要なAPI境界を確認して決定します。早期に固定しません。

## Dependency direction

現在確定しているruntime dependencyは次の一方向です。

```text
lisjong -> lisjong-engine
lisjong-engine -X-> lisjong
```

`lisjong-engine` は `lisjong` のPolicyやAI実装を知らずに成立する必要があります。

`lisjong-project` はdocumentation / coordination repositoryであり、このruntime dependency graphには含めません。

## Runner responsibilities

「runner」は異なる責務を指し得るため、ecosystem全体では次の3種類を区別します。

```text
麻雀そのもののgame runner
    -> lisjong-engine

Policyとゲーム環境を接続するintegration runner
    -> lisjong

複数対局を実行・集計してPolicyを比較するevaluation runner
    -> lisjong-eval
```

### Game runner

ゲーム状態を所有し、Actionを適用してround / matchを進行します。合法手・終了条件・結果生成はengine側の責務です。

### Integration runner

RiichiEnv、RiichiLab、`lisjong-engine` 等の実行環境とPolicyを接続します。外部Observation / Action表現とPolicy contractの境界を扱いますが、麻雀ルールそのものは再実装しません。

### Evaluation runner

複数gameのseed、seat、Policy組合せ、試行数等を計画し、結果を集計して比較します。単一gameのルール進行やPolicy判断ロジックは所有しません。

## Issue placement rules

新しい課題のplacementは「どのrepositoryの目的を成立させるために必要な責務か」で判断します。

- 麻雀ゲームを正しく進行するために必要なら `lisjong-engine`
- 観測からActionを選ぶAI、またはPolicyと環境の接続なら `lisjong`
- 複数対局を実験条件に従って比較・集計するなら将来の `lisjong-eval`
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
