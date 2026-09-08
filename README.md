# lisjong-project

Project-wide architecture, repository boundaries, and roadmap for the lisjong ecosystem.

## 目的

`lisjong-project` は、lisjong ecosystem 全体の設計・repository責務・依存方向・長期ロードマップを管理する documentation / project coordination repository です。

実装コードは原則として持ちません。`lisjong`、`lisjong-engine`、`lisjong-arena`、`lisjong-play` のいずれかを親repositoryとして扱うものでもありません。

## 正本の分担

```text
lisjong-project
    project-wide architecture
    repository責務
    repository間依存方向
    long-term roadmap
    cross-repository promotion boundary

各実装repositoryのdocs/architecture.md
    そのrepository内部のarchitecture
    concrete contract / implementation boundary

GitHub Issues / PRs
    現在の作業内容
    acceptance criteria
    experiment / implementation result
    進捗・完了状態
```

GitHub上で確認できる現在進捗を本repositoryの恒久文書へ重複して記録しません。

## Repository

| Repository | 主な責務 |
| --- | --- |
| [`lisjong`](https://github.com/lisbun/lisjong) | 麻雀AI decision core。Policy、stable AI-side contract、牌効率・HandBelief・value / risk等のstable semantics、production / public Learned Policy semantics |
| [`lisjong-engine`](https://github.com/lisbun/lisjong-engine) | 日本式リーチ麻雀のルール、状態遷移、合法手、game / match進行、first-party deterministic execution substrate |
| [`lisjong-arena`](https://github.com/lisbun/lisjong-arena) | external / local execution・observation、bounded experiment-local dataset / training / analysis、再現可能なPolicy / game evaluation |
| [`lisjong-play`](https://github.com/lisbun/lisjong-play) | first-party `lisjong-engine` を利用するHuman Play / presentation consumer。human input、action-selection UX、GUI / CLI、spectator / replay等 |

## Arenaの3責務

`lisjong-arena` 内では、少なくとも次の3責務を分離します。

```text
Execution / Observation
    what happened

Experiment-local Research / ML
    bounded experimentをどうmaterialize / train / diagnoseするか

Evaluation
    candidate / Policyをどう再現可能に比較するか
```

Arenaは、bounded research questionのためのpurpose-specific feature / dataset / trainer / model / checkpoint / diagnostic artifactを所有できます。

ただし、Arenaにresearch implementationが存在することと、stable AI semanticsをArenaが所有することは別です。

```text
experiment-local model
!= production Policy

experiment-local feature schema
!= stable PolicyInput / production feature contract

experiment result
!= stable public API
```

stableなPolicy / HandBelief / value / feature / inference semanticsへ昇格する場合は、owner repositoryを明示的にreviewし、必要なら `lisjong` のstable contractとしてformalizeします。

`lisjong-play` はgame rulesやstate transitionを再実装せず、`lisjong-engine` のplayer-safe public boundaryをconsumerとして利用します。AI seatを含む場合も、必要なPolicy / execution bridgeを既存ownerからreuseし、presentation都合でAI-side semanticsやengine semanticsを複製しません。

## External research source boundary

外部AI、外部model、外部牌譜・log、program output等をresearch sourceとして利用する場合は、技術的に取得・実行できることと、学習利用できることを分離します。

```text
identity / provenance
    ↓
private execution / acquisition basis
    ↓
retention basis
    ↓
ML / distillation use basis
    ↓
redistribution basis
```

これらは独立した判断です。

```text
technically accessible
!= approved training data

program may be executed
!= generated labels may be used for ML

output may be retained
!= generated corpus may be redistributed
```

外部sourceをtraining dataへ昇格する前に、exact source / revision / model or weight provenanceと該当する利用条件を記録し、曖昧な用途は楽観的に `GO` と解釈しません。player-visible serving inputとprivileged / oracle情報の境界も別途維持します。

詳細な責務境界と依存方向は [Architecture](docs/architecture.md) を参照してください。
長期的な能力ロードマップは [Roadmap](docs/roadmap.md) を参照してください。

## Issue placement

新しい機能や設計課題をどのrepositoryへ置くか迷った場合は、まず [Architecture](docs/architecture.md) の責務境界を基準に判断します。

概略:

```text
mahjong rules / progression
    -> lisjong-engine

stable AI semantics / production Policy contract
    -> lisjong

execution / observation
bounded experiment-local ML / analysis
Policy / game evaluation
    -> lisjong-arena

Human Play / presentation / spectator / replay
    -> lisjong-play

repository boundary / dependency / ownership rule
    -> lisjong-project
```

repository境界そのものを変更する提案や、複数repositoryへまたがる設計判断は `lisjong-project` で扱います。
個別repository内部の実装・設計・進捗は、それぞれのrepositoryで管理します。

## Architecture Decision Records

長期的な影響があり、後から理由を再確認する価値がある横断的な設計判断だけを `docs/decisions/` にADRとして残します。

- [ADR 0001: Repository boundaries](docs/decisions/0001-repository-boundaries.md)
- [ADR 0002: External execution and observation ownership](docs/decisions/0002-external-execution-observation-ownership.md)
- [ADR 0003: External research source use boundary](docs/decisions/0003-external-research-source-use-boundary.md)

ADRはhistorical decision recordです。現在のtarget architectureは [Architecture](docs/architecture.md) を正本とします。

## License

このrepositoryの文書は [MIT License](LICENSE) の下で公開します。
