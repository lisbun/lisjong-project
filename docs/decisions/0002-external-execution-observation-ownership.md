# ADR 0002: External execution and observation ownership

- Status: Accepted
- Date: 2026-08-22
- Supersedes in part: [ADR 0001: Repository boundaries](0001-repository-boundaries.md)

## Context

ADR 0001では、`lisjong` がAI decision logicに加えてRiichiEnv / RiichiLab等へのself-integrationとstandalone executionを所有し、`lisjong-arena` は主にPolicy / game performanceの比較・評価を所有する境界を採用した。

この境界は、同じPolicyを複数environmentで早期に動かすためには有効だった。

その後、external environment integrationが次の方向へ拡大した。

- RiichiLab WebSocket client
- environment-specific Adapter
- bot profile / credential resolution
- protocol trace
- game execution trace / observation
- 長時間運転
- disconnect時のrecovery
- continuous matchmaking / repeated participation
- 対局記録・採譜
- 将来的な分析・学習用data acquisition

これらは「観測可能な状態からどのActionを選ぶか」というAI decision coreとは異なるexecution lifecycle / observability concernである。

また、RiichiLab等で取得した対局dataは、単なるonline participationだけでなく、Policy evaluation、regression analysis、component improvement、将来的なtraining data acquisitionにも接続する。

このため、`lisjong` がAI判断とexternal execution lifecycleの双方を拡大し続けるより、AI decision coreとexecution / observationをrepository boundaryで分離する方が長期的に明確である。

## Decision

### `lisjong` をAI decision coreへ収束させる

`lisjong` は、観測可能な情報からActionを選択するAI decision coreを所有する。

概念的な中心境界は次とする。

```text
DecisionContext
      |
      v
    Policy
      |
      v
InternalAction
```

Policy / AI戦略、AI-side contract、向聴・受け入れ・lookahead、HandBelief、offensive / defensive value、立直・鳴き・押し引き、learned estimator / Policy等は `lisjong` に残す。

外部environment固有のWebSocket、credential、matchmaking、session lifecycle、retry / reconnect、continuous participation、game record persistence等をAI decision coreのtarget responsibilityへ含めない。

### external execution / observationを`lisjong-arena`側へ寄せる

`lisjong-arena` は、従来のevaluationに加え、lisjongをconcrete environmentで実行・観測する能力を持つ方向へ拡張する。

ただし、同一repository内でも次の2責務を分離する。

```text
execution / observation
        |
        v
 raw execution data
        |
        v
    evaluation
```

Execution / observationはenvironment integration、runner / client、live participation、retry、protocol trace、raw game record取得等を扱う。

Evaluationはmatchup、seed、seat rotation、trial plan、metrics、artifact、statistical comparison等を扱う。

Evaluationはexecution / observationをconsumerとして利用できるが、execution layerはArena固有のcomparison semanticsへ依存しない。

### individual implementation ownershipは一律に固定しない

このDecisionはtarget responsibilityを定めるものであり、既存の全Adapter / runner / trace contractを一括で移動することを要求しない。

特に、RiichiLab participation、WebSocket client、retry / reconnect、continuous execution、raw online game record acquisitionはArena移管の本命とする。

一方、RiichiEnv Adapter、LocalGameRunner、GameTrace等の最終ownershipは、既存consumerとdependencyを確認してconcrete migration Issueで決定する。

同じsemanticsをlisjong / Arena双方へ恒久的に重複実装しない。

## Execution data boundary

external environmentから取得したraw execution dataと、AI改善のためのderived semanticsを分離する。

```text
external environment
        |
        v
lisjong-arena execution / observation
        |
  raw execution data
      /        \
     v          v
 evaluation   AI improvement input
                  |
                  v
               lisjong
```

Arena側では、raw game record、protocol trace、objective execution event、seat-visible observation、実際に送信・適用したAction、game result等を扱える。

training example semantics、AI feature semantics、learned estimator input / target、component calibration等は `lisjong` の責務とする。

shanten / ukeire値、HandBelief、候補評価、選択理由等のAI内部analysisをraw execution recordへ暗黙に混在させない。必要な場合はDecisionTrace / AnalysisTrace等の別channelとして設計する。

## Visibility / secret boundary

repository ownershipを変更しても、次のinformation-flow restrictionを維持する。

```text
runtime credential / Authorization information
        -X-> trace / game record / evaluation artifact

privileged offline / ground-truth data
        -X-> online Policy input

privileged execution observation
        -X-> Policy decision path
```

execution / observationの追加はPolicyのAction選択へ干渉してはならない。

## Dependency direction

少なくとも次を維持する。

```text
lisjong-arena -> lisjong
lisjong-engine -X-> lisjong
lisjong -X-> lisjong-arena
```

`lisjong -> lisjong-engine` の必要性そのものは本ADRでは再設計しない。

first-party engineがconcrete execution pathとして利用される時点で、実際のAPIとconsumerを確認し、必要であれば別ADR / Issueで再評価する。

Arenaはexecution / interoperabilityのconsumerとしてRiichiEnv等のexternal environmentへ直接依存してよい。ただし外部型をlisjongのstable Policy contractへ漏らさない。

## Alternatives considered

### A. 現状維持

`lisjong` がAI decision、Adapter、external client、standalone executionを引き続き所有する。

利点:

- migrationが不要
- 現在のexecution pathを維持できる

欠点:

- AI decision coreとruntime operational concernが同一repositoryで拡大し続ける
- RiichiLab長時間運転、retry、採譜等がlisjongへ集中する
- repository責務を簡潔に説明しにくくなる

採用しない。

### B. `lisjong-runtime` を新設する

external execution / observationを独立repositoryへ分離する。

利点:

- runtime責務を最も純粋に分離できる
- 将来production hostingへ発展しやすい

欠点:

- repositoryが増える
- 現時点では主要consumerがArena / AI development workflowへ集中している
- concrete requirementより先にgeneric runtime abstractionを作る可能性がある

現時点では採用しない。

### C. `lisjong-arena` をexecution / observation / evaluation基盤へ拡張する

利点:

- `lisjong-arena -> lisjong` の既存依存方向で成立する
- external execution、data acquisition、evaluationのworkflowを近接配置できる
- repositoryを増やさずに済む
- `lisjong` をAI decision coreへ収束できる

欠点:

- Arenaが肥大化する可能性がある
- executionとevaluationを分離しない場合、責務境界が再び曖昧になる

同一repository内でexecution / observationとevaluationを別layerとして維持することを条件にCを採用する。

## Runtime extraction trigger

現時点では `lisjong-runtime` repositoryを新設しない。

次のようなconcrete requirementが成立した場合に再検討する。

- 24/7 production bot hosting
- evaluationとは独立したdeployment
- generic process / agent hosting
- Arena以外の複数consumerが同じruntimeを必要とする
- execution infrastructureがArena固有評価責務から独立して大きく成長する
- Arena repository内のpackage-level分離では責務境界を維持しにくくなる

将来需要を推測して先行抽象化するのではなく、実consumer / operational requirementを抽出判断の根拠とする。

## Consequences

### Positive

- `lisjong` の責務をAI decision coreとして明確化できる
- WebSocket / retry / credential / trace等のoperational concernをPolicyから切り離せる
- RiichiLab participation、採譜、evaluationを一方向のworkflowとして整理できる
- raw execution dataとAI training semanticsを分離できる
- `lisjong-arena -> lisjong` の既存依存方向を利用できる
- 新repositoryを先行作成せずに済む

### Trade-offs

- Arenaにexecutionとevaluationという2つの大きな責務が入る
- package / module境界を意識して依存方向を維持する必要がある
- 既存integrationの段階migrationが必要になる
- RiichiEnv Adapter / LocalGameRunner / GameTrace等の最終ownershipはconcrete migration時に追加判断が必要になる
- 将来production hosting等へ成長した場合はruntime抽出が必要になる可能性がある

これらは、AI decision coreとexecution lifecycleを分離しつつprematureなrepository追加を避ける利点に比べて許容できると判断する。
