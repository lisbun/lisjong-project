# Seed allocation / provenance contract

## Purpose

本書は、lisjong ecosystem における scientific / qualification / evaluation / training population の seed allocation と provenance を相互運用可能にする **project-wide normative contract** です。

このcontractは、ecosystem全体を単一の整数seed namespaceへ結合しません。各populationのproducer / execution ownerが authoritative ledger を所有し、必要な場合だけ cross-owner non-overlap を明示的に検証します。

関連:
- lisbun/lisjong-project#77
- lisbun/lisjong-project#75
- lisbun/lisjong-arena#346

## Ownership model

```text
lisjong-project
    contract / terminology / interoperability
        |
        +---------------------------+
        |                           |
        v                           v
owner repository A              owner repository B
authoritative ledger            authoritative ledger
allocation / freshness          allocation / freshness
```

原則:

- `lisjong-project` は executable ledger を所有しない。
- population を生成・実行する owner repository が、そのpopulationの authoritative allocation record を所有する。
- Arenaが実行・評価するpopulationは `lisjong-arena` が所有する。
- lisjong自身が生成するtraining / self-play populationは、concrete producerが成立した時点から `lisjong` が所有する。
- repository間の非重複は、scientific protocolが要求する場合だけ explicit contract として検証する。
- ownerの違いだけを理由に、同じ整数seedを常に衝突扱いしない。

## Seed domain

### Definition

`seed_domain` は、seed integer がどの stochastic population を識別するかを定めるversioned identityです。

同じ整数seedでも、少なくとも次が異なれば同じpopulationを意味するとは限りません。

- RNG / generator implementation
- environment / ruleset / game mode
- population producer
- seed derivation semantics
- seat / episode expansion semantics

例:

```text
riichienv-4p-red-half-hanchan-v1
lisjong-selfplay-v1
lisjong-training-shuffle-v1
```

### Collision scope

同一 `seed_domain` 内では、authoritative ledger により seed membership の再利用を防止します。

異なるdomain間では、整数の一致だけをcollisionとみなしません。cross-domain non-overlapが科学的に必要な場合、consumer protocolが次を明示しなければなりません。

1. 比較対象となるdomain
2. 何を同一population memberとみなすか
3. membershipを比較するprojection / normalization
4. overlap時のfail-closed条件

暗黙の「ecosystem中の全整数seedは永久に一意」というruleは置きません。

## Allocation record

各owner ledger recordは少なくとも次をlosslessに保持します。

```text
schema_version

allocation_identity
owner_repository
owner_issue_or_protocol

seed_domain
purpose
population_or_split

seed_membership
    explicit seeds
    or lossless range representation
seed_membership_identity

state
    RESERVED
    COMMITTED
    RETIRED

producer_revision
protocol_or_population_identity
allocation_timestamp
provenance_reference
```

必要に応じて次を追加できます。

```text
parent_allocation_identity
cross_population_exclusion_set
external_execution_run_id
artifact_identity
```

### Stable identities

- `allocation_identity` はowner内で一意かつimmutableとする。
- `seed_membership_identity` はmembership内容から決定的に検証可能なidentityとする。
- state transitionやunrelated ledger mutationで、既存allocationのidentity / membershipを黙って変更しない。
- membership変更が必要なら、新しいallocationとして扱う。

## Allocation states

### RESERVED

populationとして正式に予約された状態です。

- result exposure前でも、同一domainの適用対象allocationと再衝突させない。
- protocol lockとmembershipを照合できる。
- ownerが定義した canonical authority ref / store へpublishされた時点でauthorityを持つ。
- scientific code revisionとlive allocation stateを同一Git revisionへ結合する必要はない。

### COMMITTED

execution開始、data generation開始、またはprotocolが定めた exposure boundary を越えた状態です。

- failed / interrupted / incomplete runでも自動解放しない。
- selective rescueやseed replacementを許可する状態ではない。
- `RESERVED -> COMMITTED` はmembershipを変更しない。

### RETIRED

今後の新規利用対象から外したhistorical / legacy allocationです。

`RETIRED` は「再利用可能」を意味しません。既定では **NEVER_REUSE** として扱います。

自動的な `FREE` stateや、失敗を理由にしたrelease transitionは定義しません。

## Freshness semantics

あるcandidate membershipが `fresh` であるためには、allocationをauthoriseする時点で少なくとも次を満たす必要があります。

1. ownerのcanonical authorityを最新状態で読み直している。
2. schema / ledger integrity validationに成功している。
3. applicableな同一domainの `RESERVED` / `COMMITTED` / `RETIRED` allocationとmembershipが衝突しない。
4. protocolがcross-owner / cross-domain exclusionを要求する場合、そのexplicit ruleにも合格する。
5. 検証したauthorizing ledger revisionを記録できる。

historical artifact / Issue / JSON / JSONL / gzipの全文scanは、bootstrapや監査には利用できますが、通常のfresh allocationごとのauthorityにはしません。

## Common operations

各owner implementationは、同等semanticsとして少なくとも次を提供します。

```text
list
show
check
reserve
commit
retire
validate-ledger
```

名前やCLI構文はrepositoryごとに異なっても構いませんが、semanticsは一致させます。

### Required behavior

- `check`: candidate membershipと現行authorityを照合し、collisionをfail closedする。
- `reserve`: current authorityを再読込したうえで、collision-freeな新規allocationをpublishする。
- `commit`: allocation identityとmembershipを維持してstate transitionする。
- `retire`: allocation identityとmembershipを維持してstate transitionする。
- `validate-ledger`: schema、identity、membership表現、重複、state invariantを検証する。

validation failure、stale base、concurrent collision、unknown schemaは成功扱いにしません。

## Authority publication and concurrency

owner repositoryは canonical authority ref / store を明示します。

Git-managed ledgerの場合、そのauthorityはowner repositoryの `main` と同じrefである必要はありません。scientific code revisionをexact pinするconsumerでは、live allocation stateをcode revisionから分離できます。

mutationは次を満たします。

1. mutation直前にcurrent authorityを再読込する。
2. schema / collision / stale-baseを再検証する。
3. compare-and-swap等によりlost updateを防ぐ。
4. concurrent mutationをsilent overwriteせずfail closedする。
5. force overwriteを通常運用にしない。
6. failed executionでreservationを自動解放しない。
7. later unrelated mutationでscientific code revisionを暗黙に変更しない。

## Protocol binding

consumer protocol / lockは必要に応じて次をbindできます。

```text
owner_repository
allocation_identity
seed_domain
authorizing_ledger_revision
seed_membership_identity
```

実行前validationでは、少なくとも次を確認します。

- allocation identityがauthority上で解決できる。
- seed domainがlockと一致する。
- membership identityがlockと一致する。
- stateがprotocol上許可されたactive stateである。
- required cross-owner exclusion evidenceが存在し、対象revisionと一致する。

live ledgerの後続unrelated mutationそのものを、過去のprotocol lockの改変とはみなしません。lockは「どのauthority revisionがそのallocationをauthoriseしたか」を保持します。

## Cross-owner non-overlap

cross-owner non-overlapを要求するprotocolでは、整数listの手作業コピーではなく authoritative allocation identities を比較します。

検証手順:

1. 比較するowner / allocation / domainをprotocolが列挙する。
2. 各ownerのcanonical authorityからallocationを解決する。
3. allocation identity / seed domain / membership identity / authorizing revisionを検証する。
4. 同一domainならlossless membership集合のintersectionを計算する。
5. 異なるdomainなら、protocolが定義したprojection / normalization後にintersectionを計算する。
6. overlapが1件でもあればfail closedする。
7. 比較入力と結果をconsumer provenanceへ記録する。

「異なるownerだから安全」「整数が違うから必ず独立」「domainが違うから自動的に比較不要」といった推論はprotocolの明示ruleなしでは行いません。

## Historical bootstrap

owner implementation導入時には、自身がauthoritative ownerである既知populationをbootstrapします。

情報源の例:

- historical scientific / qualification protocol locks
- repository-declared allocation knowledge
- retained manifests
- durable execution provenance

bootstrapは conservative に行います。不確実な既知populationをfreshとして再利用するより、`RETIRED` / never-reuse側へ倒すことを許容します。

bootstrap完了後の通常allocationでは、operatorによるrepository / Issue / artifact全文検索を要求しません。

## Future owner adoption checklist

新しいproducer repositoryまたは新しいseed domainを追加する場合、実装前に次を定義します。

- authoritative owner repository
- canonical authority ref / store
- seed domain identityとversioning rule
- allocation record schema
- deterministic membership identity
- state transition / exposure boundary
- concurrency / stale-base protection
- bootstrap source
- protocol lock binding fields
- 必要なcross-owner exclusion rule

generic registry platformを先回りで作る必要はありません。concrete producerが必要になった時点で、このcontractに適合する最小owner implementationを追加します。

## Reference implementation

最初のconcrete implementationは `lisjong-arena` の Arena-owned seed registryです。

- implementation issue: lisbun/lisjong-arena#346
- merged implementation: lisbun/lisjong-arena#347
- live authority: `refs/heads/seed-registry`
- scientific code revisionとlive allocation authorityを分離

この実装はproject-wide contractのreferenceであり、Arenaをecosystem-wide registry ownerにするものではありません。

## Non-goals

本contractは次を要求しません。

- ecosystem-wide single physical ledger
- global integer namespace
- 全repositoryへの即時registry実装
- historical artifactsの遡及的なidentity変更
- failed runの自動seed再利用
- repository間の暗黙non-overlap
