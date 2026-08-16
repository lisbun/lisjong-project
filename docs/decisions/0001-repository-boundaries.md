# ADR 0001: Repository boundaries

- Status: Accepted
- Date: 2026-08-16
- Amended: 2026-08-16 — comparison repositoryの正式名称を `lisjong-arena` とし、初期実行経路を確定

## Context

lisjong ecosystemでは、麻雀AI、麻雀ゲームエンジン、Policy比較基盤を別repositoryへ分離して開発する。

責務境界を明示しないまま機能を追加すると、同じ概念を複数repositoryへ重複実装したり、game engineがAI戦略へ依存したり、比較処理がPolicyやルール実装へ混入したりする可能性がある。

また、「runner」という言葉が単一gameの進行、Policyと環境の接続、複数gameの比較実験という異なる責務を指し得るため、横断的な用語整理も必要である。

当初、Policy比較repositoryは仮称 `lisjong-eval` としていた。しかし `eval` は `lisjong` 内部のAction / state evaluationや将来のvalue functionと混同しやすい。複数Policyを対局させて比較する責務を明確にするため、正式名称を `lisjong-arena` とする。

## Decision

repository責務を次のように分離する。

### `lisjong`

観測可能な麻雀状態からActionを選ぶAIを実装し、Policyを各実行環境へ接続する。

Policy / AI戦略、Policy contract、向聴数・受け入れ枚数・牌効率等の判断機能、RiichiEnv / RiichiLab / `lisjong-engine` とのAdapter・integrationを担当する。

### `lisjong-engine`

与えられたActionに従って日本式リーチ麻雀を正しく進行し、対局結果を生成する。

ドメインモデル、状態遷移、合法手、和了・役・符・点数、round / match進行、RuleSet、deterministicなseed管理を担当する。

PolicyやAI戦略には依存しない。

### `lisjong-arena`

複数のlisjong Policyを再現可能な実験条件で対局させ、Policy全体の強さを比較する。

matchup、seed集合、seat rotation、複数対局の実行計画、Policy assignment、raw result収集、基本metrics、comparison protocol、将来的な統計的比較・benchmark / reportを担当する。

AI判断ロジック、Action / state evaluation、麻雀ルール、単一gameの状態遷移は持たない。

## Initial arena execution path

`lisjong-engine` の完成はArena開始の前提にしない。初期Arenaは、すでに利用可能な `lisjong` のRiichiEnv integrationを利用する。

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

将来 `lisjong-engine` が対局可能になった段階では、`lisjong` 側のengine integrationをArenaから利用する。

RiichiEnvと`lisjong-engine`という2つの実経路が揃う前に、将来APIを推測した汎用 `GameBackend` / `EvaluationBackend` 等のabstractionは導入しない。共通化すべき境界は、実際の2経路の差異を確認してから決定する。

## Dependency direction

project-wideに許可する依存方向は次とする。

```text
lisjong-arena -> lisjong
lisjong -> lisjong-engine
lisjong-engine -X-> lisjong
lisjong -X-> lisjong-arena
```

初期Arenaでは `lisjong-arena -> lisjong -> RiichiEnv` を使用する。

`lisjong-project` はdocumentation / coordination repositoryでありruntime dependency graphには含めない。

## Runner terminology

runner責務は次の3種類として区別する。

```text
game runner
    -> lisjong-engine

integration runner
    -> lisjong

arena / comparison runner
    -> lisjong-arena
```

## Source of truth

横断情報の正本を次のように分離する。

```text
lisjong-project
    プロジェクト横断architecture
    repository責務
    repository間依存方向
    長期ロードマップ
    横断的な設計判断

各repositoryのarchitecture
    repository内部のarchitecture

GitHub Issues / PRs
    現在の作業内容・進捗・完了状態
```

## Consequences

### Positive

- 新しいIssueのplacement判断に一貫した基準を持てる
- engineからAIへの依存逆流を防ぎやすい
- AI、ルール、Policy比較を独立して発展させやすい
- `evaluation` という一般概念とPolicy comparison repositoryの名称を区別できる
- `lisjong-engine` 完成を待たず、既存RiichiEnv integrationで再現可能なPolicy比較を開始できる
- 現在進捗をproject-wide文書へ複製しないため、staleな情報を減らせる
- 同じPolicyを複数実行環境へ接続する構成を維持しやすい

### Trade-offs

- repositoryをまたぐ変更では複数repoのIssue / PR coordinationが必要になる
- repository境界を維持するため、小規模な実装でもどこが契約を所有するか判断が必要になる
- 初期ArenaはRiichiEnv経路に依存し、`lisjong-engine`との共通backend境界は後から設計する必要がある

これらは、責務重複や依存逆流、premature abstractionを避ける利点に比べて許容できると判断する。
