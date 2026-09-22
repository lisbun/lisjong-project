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
| [`lisjong`](https://github.com/lisbun/lisjong) | 麻雀AIを作るrepository。Policy / stable AI semantics、feature / action vocabulary、dataset / teacher / training、model artifact、Learned Policy / learned estimator |
| [`lisjong-engine`](https://github.com/lisbun/lisjong-engine) | 日本式リーチ麻雀のルール、状態遷移、合法手、game / match進行、first-party deterministic execution substrate |
| [`lisjong-arena`](https://github.com/lisbun/lisjong-arena) | AIをconcrete environmentで実行・観測し、controlled conditionで比較・評価するrepository。raw/source record、population provenance、formal strength evaluation |
| [`lisjong-play`](https://github.com/lisbun/lisjong-play) | first-party `lisjong-engine` を利用するHuman Play / presentation consumer。human input、action-selection UX、GUI / CLI、spectator / replay等 |

## Learning / Arenaの責務境界

短く言うと次です。

```text
lisjong
    = candidateを作る
      feature / dataset / teacher / training / artifact / inference

lisjong-arena
    = candidateを動かして測る
      execution / observation / population provenance / evaluation
```

Arenaが保持するreusable raw/source recordはplayer-safe observation、legal actions、selected action、provenanceを中心とし、feature tensorやtraining objective固有labelをcanonical sourceにしません。下流の`lisjong`が新しいfeature / dataset representationを再materializeできる境界を優先します。

既存Arena-local Learning codeやartifactはhistorical / reference implementationとして残せます。ownership変更を理由にbulk migrationや遡及的なartifact改名を行いません。

現在のArena AWS execution surfaceはconcrete workloadをhostできますが、training semantics / executable entry pointのcanonical ownerは`lisjong`です。hosting場所とsemantic ownershipを分離します。

`lisjong-play` はgame rulesやstate transitionを再実装せず、`lisjong-engine` のplayer-safe public boundaryをconsumerとして利用します。AI seatを含む場合も、既存ownerのPolicy / execution boundaryをreuseします。

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
RiichiLabのexternal live strength recordは [RiichiLab external strength registry](docs/riichilab-strength-registry.md) を参照してください。
麻雀評価のsample size・uncertainty・cluster bootstrap・探索/検証の横断原則は [Mahjong evaluation statistics](docs/mahjong-evaluation-statistics.md) を参照してください。
seed allocation / freshness / cross-owner provenanceの横断contractは [Seed allocation / provenance contract](docs/seed-allocation-provenance.md) を参照してください。

## Issue placement

新しい機能や設計課題をどのrepositoryへ置くか迷った場合は、まず [Architecture](docs/architecture.md) の責務境界を基準に判断します。

概略:

```text
mahjong rules / progression
    -> lisjong-engine

AI semantics / feature / dataset / teacher / training / Learned Policy
    -> lisjong

execution / observation / source record / population provenance
Policy / game evaluation
    -> lisjong-arena

Human Play / presentation / spectator / replay
    -> lisjong-play

repository boundary / dependency / ownership rule
    -> lisjong-project
```

既存Arena Learning implementationのhistorical maintenanceはArenaで扱えますが、新しいcanonical Learning capabilityは`lisjong`へ置きます。

repository境界そのものを変更する提案や、複数repositoryへまたがる設計判断は `lisjong-project` で扱います。個別repository内部の実装・設計・進捗は、それぞれのrepositoryで管理します。

## Architecture Decision Records

長期的な影響があり、後から理由を再確認する価値がある横断的な設計判断だけを `docs/decisions/` にADRとして残します。

- [ADR 0001: Repository boundaries](docs/decisions/0001-repository-boundaries.md)
- [ADR 0002: External execution and observation ownership](docs/decisions/0002-external-execution-observation-ownership.md)
- [ADR 0003: External research source use boundary](docs/decisions/0003-external-research-source-use-boundary.md)
- [ADR 0004: Champion family separation](docs/decisions/0004-champion-family-separation.md)
- [ADR 0005: Overall Champion determination by AABB half-game](docs/decisions/0005-overall-champion-aabb-half.md)
- [ADR 0006: RiichiLab external strength registry](docs/decisions/0006-riichilab-external-strength-registry.md)
- [ADR 0007: Cross-repository development environment identity](docs/decisions/0007-cross-repo-development-environment-identity.md)
- [ADR 0008: Learning and Learned Policy ownership](docs/decisions/0008-learning-and-learned-policy-ownership.md)
- [ADR 0009: Owner-scoped seed allocation and provenance](docs/decisions/0009-owner-scoped-seed-allocation-provenance.md)

ADRはhistorical decision recordです。現在のtarget architectureは [Architecture](docs/architecture.md) を正本とします。

## License

このrepositoryの文書は [MIT License](LICENSE) の下で公開します。
