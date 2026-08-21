# ADR 0001: Repository boundaries

- Status: Accepted, partially superseded by [ADR 0002](0002-external-execution-observation-ownership.md)
- Date: 2026-08-16
- Amended: 2026-08-16 — comparison repositoryの正式名称を `lisjong-arena` とし、初期実行経路を確定
- Amended: 2026-08-21 — `lisjong` self-integrationとevaluation-specific external competitor integrationの境界を明確化
- Partially superseded: 2026-08-22 — external execution / observationとstandalone participationのownershipをADR 0002で再配置

## Context

lisjong ecosystemでは、麻雀AI、麻雀ゲームエンジン、Policy比較基盤を別repositoryへ分離して開発する。

責務境界を明示しないまま機能を追加すると、同じ概念を複数repositoryへ重複実装したり、game engineがAI戦略へ依存したり、比較処理がPolicyやルール実装へ混入したりする可能性がある。

また、「runner」という言葉が単一gameの進行、lisjong自身と環境の接続、複数gameの比較実験という異なる責務を指し得るため、横断的な用語整理も必要である。

当初、Policy比較repositoryは仮称 `lisjong-eval` としていた。しかし `eval` は `lisjong` 内部のAction / state evaluationや将来のvalue functionと混同しやすい。複数Policyを対局させて比較する責務を明確にするため、正式名称を `lisjong-arena` とする。

その後、Mortal等のexternal competitorをlisjongのbenchmarkとして利用する将来経路を検討した。`lisjong` を単なるAction selectorへ縮小するのではなく、Policy / AI configurationを選択し、自分自身をRiichiEnv / RiichiLab / 将来のfirst-party engine等へ接続して参加できるself-integration能力を維持する。一方、external competitorをhostingして比較実験を成立させる責務は、evaluation目的である限りArena側に置く方が責務境界に整合する。

この2026-08-21時点のself-integration ownership判断は、その後external execution / observation責務が拡大したため、ADR 0002で再検討した。以下は当時の判断を歴史として保持し、external execution / observationとstandalone participationの現在のownershipについてはADR 0002を正本とする。

成熟したOSSがgame executionやprotocol interoperabilityを提供する場合はそれを活用し、Mortal benchmarkのためだけにgeneric single-match runtimeや重複したprotocol implementationを先行新設しない。

## Decision

repository責務を次のように分離する。

### `lisjong`

観測可能な麻雀状態からActionを選ぶAIを実装し、そのAI自身を各実行環境へ接続して動作させるself-integration能力を担当する。

Policy / AI戦略、Policy contract、Policy / AI configuration selection、向聴数・受け入れ枚数・牌効率等の判断機能、RiichiEnv / RiichiLab / `lisjong-engine` とのAdapter・integration、lisjong自身のstandalone runner / clientを担当する。

self-integrationは概念上、1 seat分の外部環境表現とPolicy contractを接続するenvironment-facing integrationと、lisjong自身を実際の環境へ参加させるstandalone executionを区別する。ただし、この区別は単一の新classやmoduleを要求せず、具体的な内部分割は `lisjong` 側architectureを正本とする。

他AIのhosting、external competitor専用session orchestration、evaluation matchup / seed / seat rotation / multi-trial aggregationは原則として所有しない。

> **Superseded in part by ADR 0002:** external execution / observation、standalone runner / client、live participationのtarget ownershipは `lisjong-arena` 側へ再配置する。`lisjong` のAI decision core、Policy contract、AI-side semanticsは引き続き有効である。

### `lisjong-engine`

与えられたActionに従って日本式リーチ麻雀を正しく進行し、対局結果を生成する。

ドメインモデル、状態遷移、合法手、和了・役・符・点数、round / match進行、RuleSet、deterministicなseed管理を担当する。

PolicyやAI戦略には依存せず、Mortal / MJAI等のexternal-agent integrationやevaluation orchestrationも所有しない。

### `lisjong-arena`

Policy / game performanceを再現可能な実験条件で比較・評価する。

matchup、seed集合、seat rotation、複数対局の実行計画、Policy / agent assignment、raw result収集、基本metrics、comparison protocol、将来的な統計的比較・external benchmark / reportを担当する。

AI判断ロジック、Action / state evaluation、麻雀ルール、単一gameの状態遷移は持たない。

evaluationだけを目的としてMortal等のexternal competitorを導入する場合、そのwrapper / session orchestrationをArena-private implementation detailとして所有してよい。同じintegrationがArena外の複数concrete consumerから必要になった場合にのみ、共通runtime / repositoryへの抽出を再検討する。

> **Extended by ADR 0002:** Arenaはevaluationだけでなく、lisjongのexternal execution / observationを独立layerとして所有する方向へ拡張する。execution / observationとevaluationは同一repository内でも別責務として分離する。

## Arena execution paths

`lisjong-engine` の完成はArena開始の前提にしない。Arenaの実行経路は用途ごとに扱う。

### Policy-vs-Policy development evaluation

既存のPolicy比較では、すでに利用可能な `lisjong` のself-integration / RiichiEnv integrationを利用できる。

```text
lisjong-arena
    |
    | matchup / seeds / seat rotation / aggregation
    v
lisjong
    |
    | existing self-integration
    v
RiichiEnv
```

この既存経路は当時のmigration前提として有効である。target ownershipと将来のexecution pathはADR 0002および現在のArchitectureを正本とする。

### Mixed-agent external benchmark

Mortal等のexternal competitorを含むevaluationでは、Arenaが選択したOSS execution environmentを直接orchestrateしてよい。

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

この経路でもArenaはlisjong用のObservation / legal Action / selected Action mapping semanticsを独自に複製しない。`lisjong` のstandalone runner全体を必ず再利用することは要求しないが、environment-facing self-integration semanticsを可能な限り再利用する。

現在のMortal benchmarkにおけるpreferred OSS execution path等の具体選択はroadmap / consumer Issueを正本とし、本ADRでは特定OSSを永久contractへ固定しない。

RiichiEnvと`lisjong-engine`という複数のfirst-party-facing実経路が揃う前に、将来APIを推測した汎用 `GameBackend` / `EvaluationBackend` 等のabstractionは導入しない。また、mixed-agent benchmarkという単一consumerだけを理由にgeneric external-player runtimeを先行設計しない。

## Dependency direction

first-party repository間で許可する依存方向は次とする。

```text
lisjong-arena -> lisjong
lisjong -> lisjong-engine
lisjong-engine -X-> lisjong
lisjong -X-> lisjong-arena
```

Policy-vs-Policy evaluationでは `lisjong-arena -> lisjong -> external environment` の経路を使用できる。

一方、evaluation-specificなexternal dependencyとして、Arenaが選択したOSS execution environmentへ直接依存することを許容する。

```text
first-party dependency
    lisjong-arena -> lisjong

example evaluation-specific external dependency
    lisjong-arena -> selected OSS execution environment
```

external dependencyを許容することは、Arenaがlisjong用Adapter / conversion semanticsや麻雀ルールを再実装してよいことを意味しない。

`lisjong-project` はdocumentation / coordination repositoryでありruntime dependency graphには含めない。

> **ADR 0002 clarification:** `lisjong-arena -> lisjong`、`lisjong-engine -X-> lisjong`、`lisjong -X-> lisjong-arena` は維持する。`lisjong -> lisjong-engine` の必要性そのものはADR 0002では再設計せず、concrete first-party engine integration時に必要なら別Decisionで再評価する。

## Runner terminology

runner責務は次の3種類として区別する。

```text
game runner
    -> lisjong-engine

lisjong self-integration runner / client
    -> lisjong

arena / comparison runner
    -> lisjong-arena
```

`lisjong` のintegration runner / clientはlisjong自身を環境へ接続する。Arena runnerは複数trialの計画・集計を担当し、mixed-agent external benchmarkではevaluation全体とexternal competitorをorchestrateしてよい。

> **Superseded in part by ADR 0002:** integration runner / clientのtarget ownershipはArena execution / observationへ移し、comparison runnerはArena evaluationとして区別する。

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
- `lisjong` がPolicy libraryだけでなく、RiichiEnv / RiichiLab等へ自分自身として参加できる能力を維持できる
- Policy-vs-Policyの既存評価経路を壊さず、Mortal等とのmixed-agent benchmarkをArena責務として追加できる
- 成熟したOSS interoperabilityを利用し、MJAI等の重複実装を避けやすい
- external benchmarkのためだけに新repository / generic runtimeを先行作成せずに済む
- `evaluation` という一般概念とPolicy comparison repositoryの名称を区別できる
- `lisjong-engine` 完成を待たず、既存integrationで再現可能なPolicy比較を継続できる
- 現在進捗をproject-wide文書へ複製しないため、staleな情報を減らせる
- 同じPolicyを複数実行環境へ接続する構成を維持しやすい

### Trade-offs

- repositoryをまたぐ変更では複数repoのIssue / PR coordinationが必要になる
- repository境界を維持するため、小規模な実装でもどこが契約を所有するか判断が必要になる
- ArenaにはPolicy-vs-Policy経路とmixed-agent benchmark経路という複数のexecution形態が生じる
- external competitor integrationがArena外でも必要になった場合は、後から共通runtimeへの抽出判断が必要になる
- 初期Arenaは外部environment経路に依存し、`lisjong-engine`との共通backend境界は後から設計する必要がある

これらは、責務重複や依存逆流、premature abstractionを避け、成熟OSSを活用できる利点に比べて許容できると判断する。

ADR 0002以降のcurrent target architectureについては、[Architecture](../architecture.md)を参照する。
