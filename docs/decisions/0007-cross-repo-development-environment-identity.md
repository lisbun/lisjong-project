# ADR 0007: Cross-repository development environment identity

- Status: Accepted
- Date: 2026-09-19
- Parent issue: [lisjong-project #64](https://github.com/lisbun/lisjong-project/issues/64)

## Context

lisjong ecosystem の internal Python packages は現在すべて `0.1.0` を維持し、
consumer は exact Git commit direct URL を dependency identity として使っている。

2026-09-19、`lisjong-play #41` の RiichiLab live-viewer smoke 前に、current
consumer checkout が新しい Arena / lisjong / engine commitをpinしている一方、
既存 virtualenv に同一 version の古い VCS distribution が残る事象を実測した。

`pip check` は stale Arena が要求する `riichienv==0.4.8` と installed
`riichienv 0.4.10` のversion mismatchを検出したが、same-version VCS package
自体の source revision identity は package versionだけでは証明できなかった。

危険なのは ImportError だけではない。古い package が import可能な場合、別revisionの
runtime semanticsでevaluation / smoke / artifact generationを行える。

## Decision

### 1. Package identity

internal VCS dependency の current development identity は

```text
distribution name
+ repository URL
+ exact full Git commit
+ package version
```

で判定する。

package version単独を source revision identity として扱わない。

### 2. Expected identity authority

consumer repository の `pyproject.toml [project].dependencies` にある exact VCS
pinを direct dependency のauthorityとする。commit constantsを別ファイルへ複製しない。

installed internal distribution の `Requires-Dist` から relevant transitive exact
VCS pinsも辿り、root direct pinとのrevision disagreementをfail closedで拒否する。

### 3. Installed identity authority

installed VCS sourceは PEP 610 `direct_url.json` の repository URL /
`vcs_info.commit_id` で検証する。

`pip check` も実行するが、same-version VCS source revision一致の十分条件とは扱わない。

### 4. Implementation home

reusable verifier の current implementation home は `lisjong-arena` とする。

理由:

- Arena は既に artifact / formal evaluation で PEP 610 revision provenance を扱う
- result-producing execution / evaluation の preflight と責務が近い
- `lisjong-play` は既に Arena の concrete consumerであり、checkerを重複実装せずreuseできる
- `lisjong-project` はdocumentation / coordination repositoryでありruntime codeを持たない

これは generic package-manager framework をArenaへ所有させる判断ではない。将来、
Arenaへ依存しない複数のconcrete consumerが同じtoolを必要とする等の実需要が出た場合に
placement extractionを再評価する。

### 5. Verification path

current supported command:

```text
python -m lisjong_arena.environment_verify --project pyproject.toml
```

mismatch は non-zero。verification自体はnetworkを使わない。

test / important smoke、RiichiLab network run、result-producing evaluation、formal /
locked evaluation preflight、provenance-sensitive artifact generationの前に利用する。

### 6. Repair / bootstrap

canonical repair は clean virtualenv bootstrap とし、最後に verifier PASS を要求する。

package-by-package blind uninstall / reinstallを標準repairにしない。formal evaluation
中のsilent auto-upgradeも行わない。

### 7. Versioning

current policy は **`0.1.0` + exact VCS commit identity + verifier** を維持する。

この問題だけを理由に、次は導入しない。

- every-commit semantic version bump
- new lockfile / second dependency authority
- Poetry / uv / Conda migration
- monorepo migration
- universal package-manager abstraction

independent release lifecycleやpublished compatibility contractが必要になった場合は、
semantic version policyを別decisionとして再検討する。

## Consequences

Positive:

- stale same-version internal packageを test / network / evaluation 前に検出できる
- current dependency metadataを唯一のexpected pin authorityとして維持できる
- pip resolverの特定version挙動へcorrectnessを依存させない
- play側へchecker copyを作らずに済む

Costs / limitations:

- clean venv repairはforce-reinstallより時間がかかる
- verifierはlocal checkoutのGit HEAD自体を証明しないため、formal protocol固有の
  checkout lockは引き続き必要
- current internal detectionは `github.com/lisbun/lisjong*` exact VCS dependencyを
  project-specific contractとして扱う
- internal VCS marker dependencyはcurrent workflowではunsupportedとしてfail closedする

## Reproduction record

triggering incident の確認済み条件:

```text
Windows / PowerShell
CPython 3.14.6
pip 26.1.2
internal package version 0.1.0
cache state not captured
operational outcome REPRODUCED
```

cache stateが未記録であるため、cache-specific workaroundはdecisionに含めない。
