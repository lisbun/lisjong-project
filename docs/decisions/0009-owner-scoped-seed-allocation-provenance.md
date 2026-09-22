# ADR 0009: Owner-scoped seed allocation and provenance

- Status: Accepted
- Date: 2026-09-22
- Related: lisbun/lisjong-project#77, lisbun/lisjong-project#75, lisbun/lisjong-arena#346

## Context

scientific / qualification / evaluation populationへfresh seedを割り当てる際、repository code、Issue、retained artifact、JSON / JSONL / gzipをoperatorが横断検索して未使用性を確認する運用が発生していた。

一方、単一のglobal integer registryを導入すると、RNG、environment、producer、seed derivation semanticsが異なるpopulationまで不必要に結合する。

#75で整理したrepository ownershipとも整合させる必要がある。

## Decision

project-wideには **共通contractを定義し、authoritative ledgerはpopulation ownerごとに持つ**。

```text
lisjong-project
    rule / interoperability
        |
        +------------------+
        |                  |
        v                  v
lisjong-arena           lisjong
Arena-owned ledger      lisjong-owned ledger when needed
```

- seed integer単体ではなくversioned `seed_domain` をcollision scopeとする。
- 同一domain内のreuseはowner ledgerでfail closedする。
- `RESERVED / COMMITTED / RETIRED` を共通state semanticsとする。
- `allocation_identity / seed_domain / authorizing ledger revision / seed_membership_identity` をcross-repository provenance boundaryとする。
- cross-owner non-overlapはscientific protocolが要求する場合だけexplicitに検証する。
- ownerはcanonical authority ref / storeを明示し、concurrent mutationとstale baseをfail closedする。
- live allocation stateをscientific code revisionと同一Git revisionへ結合することは要求しない。
- failed / incomplete executionでreservationを自動解放しない。
- 通常allocationでrepository / Issue / artifact全文scanをauthorityとしない。

Normative details are defined in [Seed allocation / provenance contract](../seed-allocation-provenance.md).

## Consequences

### Positive

- fresh allocationのauthorityが機械的に検証可能になる。
- repository ownershipとpopulation provenanceの責務が一致する。
- Arenaをecosystem-wide registryへ肥大化させずにinteroperabilityを得られる。
- exact scientific code revisionとlive reservation stateを独立に管理できる。
- future lisjong-owned training producerは同じcontractを採用できる。

### Cost

- ownerごとにcanonical authorityとmutation protocolが必要になる。
- cross-domain non-overlapを必要とするprotocolは比較semanticsを明示する必要がある。
- historical bootstrap時には一度だけconservative auditが必要になる。

## Rejected alternatives

### One ecosystem-wide integer registry

同じ整数が異なるgenerator semanticsで同じpopulationを意味するとは限らず、不要なglobal couplingを作るため採用しない。

### Continue ad-hoc full-text audit

false positive除外がoperator依存で、concurrencyにも弱く、継続運用できないため採用しない。

### Make Arena the global registry owner

#75のownership boundaryと矛盾し、lisjong-owned training producerまでArenaへ逆依存させるため採用しない。

## Reference implementation

Arena child lisbun/lisjong-arena#346 / PR #347 が最初のowner-scoped implementationであり、`refs/heads/seed-registry` をlive authorityとして使用する。
