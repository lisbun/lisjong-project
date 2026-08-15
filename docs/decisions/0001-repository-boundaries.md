# ADR 0001: Repository boundaries

- Status: Accepted
- Date: 2026-08-16

## Context

lisjong ecosystemでは、麻雀AI、麻雀ゲームエンジン、将来の評価基盤を別repositoryへ分離して開発する。

責務境界を明示しないまま機能を追加すると、同じ概念を複数repositoryへ重複実装したり、game engineがAI戦略へ依存したり、評価処理がPolicyやルール実装へ混入したりする可能性がある。

また、「runner」という言葉が単一gameの進行、Policyと環境の接続、複数gameの評価実験という異なる責務を指し得るため、横断的な用語整理も必要である。

## Decision

repository責務を次のように分離する。

### `lisjong`

観測可能な麻雀状態からActionを選ぶAIを実装し、Policyを各実行環境へ接続する。

Policy / AI戦略、Policy contract、向聴数・受け入れ枚数・牌効率等の判断機能、RiichiEnv / RiichiLab / `lisjong-engine` とのAdapter・integrationを担当する。

### `lisjong-engine`

与えられたActionに従って日本式リーチ麻雀を正しく進行し、対局結果を生成する。

ドメインモデル、状態遷移、合法手、和了・役・符・点数、round / match進行、RuleSet、deterministicなseed管理を担当する。

PolicyやAI戦略には依存しない。

### 将来の `lisjong-eval`

複数の対局結果を再現可能な実験条件で収集し、Policyの強さを比較する。

seed集合、seat rotation、複数対局の実行計画、結果収集、metrics、統計的比較、benchmark / report、evaluation protocolを担当する。

AI判断ロジックや麻雀ルール自体は持たない。

具体的なpackage依存方向はrepository分離時まで固定しない。

## Dependency direction

現在確定するruntime dependencyは次とする。

```text
lisjong -> lisjong-engine
lisjong-engine -X-> lisjong
```

`lisjong-project` はdocumentation / coordination repositoryでありruntime dependency graphには含めない。

## Runner terminology

runner責務は次の3種類として区別する。

```text
game runner
    -> lisjong-engine

integration runner
    -> lisjong

evaluation runner
    -> lisjong-eval
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
- AI、ルール、評価を独立して発展させやすい
- 現在進捗をproject-wide文書へ複製しないため、staleな情報を減らせる
- 同じPolicyを複数実行環境へ接続する構成を維持しやすい

### Trade-offs

- repositoryをまたぐ変更では複数repoのIssue / PR coordinationが必要になる
- repository境界を維持するため、小規模な実装でもどこが契約を所有するか判断が必要になる
- `lisjong-eval` の依存構造など、現時点で不要な詳細は後から追加判断する必要がある

これらは、責務重複や依存逆流を避ける利点に比べて許容できると判断する。
