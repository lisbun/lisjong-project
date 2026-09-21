# ADR 0008: Learning and Learned Policy ownership

- Status: Accepted
- Date: 2026-09-22
- Decision issue: https://github.com/lisbun/lisjong-project/issues/75

## Context

lisjong ecosystemでは、初期のLearned Policy研究を素早く検証するため、`lisjong-arena`にboundedなdataset / training / model / checkpoint / diagnosticsを置いてきた。

この配置は個別experimentには有効だったが、Behavior Cloning、offense foundation、defense、calls、value / placement、opponent-state representation、self-play等を継続的に積み上げる段階では、AIを構築する能力が`lisjong`と`lisjong-arena`へ分散する。

特に次の二重化を避ける必要がある。

```text
Arena trains / defines a learned representation
        ↓
promotion
        ↓
lisjong reimplements the serving behavior
```

また、encode済みfeatureだけをsource artifactとして残すと、将来のcanonical feature schemaへ再materializeできず、artifact経由でhistorical Arena ownershipが固定される。

## Decision

`lisjong`を、Heuristic / Learnedを問わず「麻雀をどう打つか」を実装するcanonical AI repositoryとする。

```text
lisjong
    AI semantics
    Heuristic Policy
    feature / action vocabulary
    Learning dataset semantics
    teacher / label
    model / training
    model artifact
    Learned Policy / learned estimator

lisjong-arena
    execution / observation
    reusable player-safe source record
    Arena-executed population provenance
    controlled comparison / evaluation
```

Short form:

```text
lisjong
    = candidateを作る

lisjong-arena
    = candidateを動かして測る
```

### Data boundary

ArenaがLearning consumer向けに保持するreusable source recordは、player-safe observation、legal actions、selected / applied action、source provenanceを中心とする。

feature tensor、model-specific derived value、training objective固有labelをcanonical source representationにしない。下流の`lisjong`がfeature / datasetを再materializeできることを条件とする。

raw/source record schemaはproducerであるArenaがversioned contractとして所有し、`lisjong`は未知versionをfail closedで拒否する。

### Measurement boundary

```text
Learning objective / intrinsic metric definition
    -> lisjong

Arena-executed population / rotation / seed provenance
    -> lisjong-arena

Policy / game-strength evaluation
    -> lisjong-arena
```

intrinsic metricがinteractive executionを必要とする場合も、metric definition、execution population、artifact producerを分離して記録する。

### Runtime / dependency boundary

`lisjong` coreはML framework非依存を維持する。training / learned inferenceはoptional dependency + lazy importとし、ML frameworkなしでもcore importが成立することを守る。

weights / checkpointはrepository外のoperator-owned artifactとし、Gitへcommitしない。model artifactはimmutable snapshotとし、arbitrary code / factory / callableを復元せず、identity不整合をload時にfail closedする。

`lisjong -> lisjong-arena` のruntime reverse dependencyは作らない。

### Operational hosting

training semanticsとexecutable entry pointは`lisjong`が所有する。

既存のArena AWS execution surfaceはconcrete workloadのhostとして再利用できるが、all training / all rollout hostingをArenaの恒久責務にはしない。future hosting ownerは具体的なconsumer / operational needが生じた時点で決める。

### Migration strategy

bulk code migrationは行わない。

```text
concrete use case
    ↓
existing Arena implementation / evidenceをinspect
    ↓
requirements / invariants / failure modesを抽出
    ↓
lisjongに必要なcanonical capabilityだけ実装
```

既存Arena Learning codeはhistorical / reference implementationとして残せる。ownership変更を理由にhistorical artifactのidentityを遡及変更しない。

## In-flight exception / transition

lisbun/lisjong-arena#331 / #332 は既にscientific protocolがlock済みであるため、locked artifactを遡及変更しない。

代わりに lisbun/lisjong-arena#342 で独立したversioned player-safe source-record sidecarを追加し、将来のlisjong-owned Learningが同じ高価なsource decisionsを再materializeできるようにする。

最初のlisjong-native Learning vertical sliceは lisbun/lisjong#184 とする。

## Supersession

本ADRは、次の既存ADRのうち **Learning / Learned Policy / experiment-local ML ownershipに関する記述だけ** をsupersedeする。

- ADR 0001 — Repository boundaries
- ADR 0002 — External execution / observation ownership
- ADR 0003 — External research source use boundary

それらの他のdecision、historical context、external execution / observation、source-use restrictionsは引き続き有効である。

## Consequences

### Positive

- HeuristicとLearned AIが同じcanonical ownerに揃う
- trainingとinferenceのsemantic driftを避けやすい
- Arenaはexecution / evaluationへ集中できる
- player-safe source recordから将来のfeature schemaを再materializeできる
- historical experimentを壊さずincrementalに移行できる

### Costs / tradeoffs

- 移行期間はArenaにhistorical Learning implementationが残る
- cross-repository source-record contractのversion管理が必要になる
- `lisjong`にoptional ML dependency / artifact discipline / ML CIを追加する必要がある

## Non-goals

- `lisjong-ai` repositoryの新設
- Arena ML codeの一括移動・一括削除
- generic ML platformの先行構築
- historical artifactのrename / rewrite
- Arena evaluation protocolの変更
- `lisjong-engine`へのAI / ML責務の導入
