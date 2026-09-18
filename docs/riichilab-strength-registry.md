# RiichiLab external strength registry

This page is the human-readable view of the machine-readable registry at [`registry/riichilab-strength.json`](../registry/riichilab-strength.json).

The governing semantics are [ADR 0006](decisions/0006-riichilab-external-strength-registry.md).

## Purpose

RiichiLab Rating is tracked as an **external live benchmark**, alongside but separate from Arena's controlled / reproducible strength evidence.

```text
Arena
  direct controlled comparison

RiichiLab
  live ecosystem-relative observation
```

Neither replaces the other, and the two are not combined into one score.

## Maturity semantics

v1 uses post-deployment ranked games:

| Games since deployment | Status |
| ---: | --- |
| 0-49 | `PROVISIONAL` |
| 50-99 | `PRELIMINARY` |
| 100-199 | `STABLE` |
| 200+ | `MATURE` |

These are operational labels, not guarantees that true skill is known.

The canonical archived observation is the Rating immediately after the first 200 post-deployment ranked games. Later live Rating does not overwrite that snapshot, and the endpoint is not extended because a result is favorable, unfavorable, or contaminated.

## Execution quality

Server-counted ranked games remain part of the exposure count even if execution was degraded, because removing them would make the registry diverge from the Rating history.

Execution quality is recorded separately:

- `CLEAN`: no observed DC / invalid action / chombo / timeout / default action;
- `DEGRADED`: timeout/default action occurred, but no observed DC / invalid action / chombo;
- `CONTAMINATED`: DC, invalid action, or chombo occurred.

Unknown fields remain unknown. Missing evidence is never converted into a clean classification.

## Initial record

### `lisjong-dev` / bot 313 — MechanismRiichiDefenseYakuhaiCallPolicy

Current registry state: **PARTIAL_EVIDENCE**.

Known exact-provenance evidence:

```text
2026-09-12 retained ranked record
profile        lisjong-dev
Policy         MechanismRiichiDefenseYakuhaiCallPolicy
lisjong        d1d3c14e3e1948cff871273b7b8b57a08c655db9
Arena          f0259f082abc45ed31c317044d50dfcc663e70c0
record         97f20dadcbdccedd6699a38947f6d30b3b479fbaf5c3d5e9dd2b07f56c39470c
requests       132
responses      132
end_game       yes
```

Additional retained evidence:

```text
2026-09-14 same-Policy cohort
games          23
disconnected   0
Rating delta   +134

2026-09-14 later bot snapshot
server total games   49
display Rating       1567
```

The 23-game cohort was operator-confirmed to use the same Policy, but exact source revision is not proven for every game. Combined with the exact 2026-09-12 record, this gives a lower bound of 24 games known to use the same Policy identity, but only one game is currently bound to the exact retained source revision.

The following values are therefore intentionally **not inferred**:

```text
rating_at_deployment     unknown
games_since_deployment   unknown
maturity_status          unclassified
canonical_200_rating     not reached / not established
```

The bot's server total of 49 games includes earlier Policy generations, so treating 49 as post-deployment exposure would be incorrect.

## Carry-over and inactivity

Reusing a RiichiLab bot carries its server-side Rating history into the next Policy deployment. A new deployment therefore starts its own `games_since_deployment` counter at zero but is not treated as a fresh independent rating initialization.

RiichiLab may also change display Rating after prolonged inactivity because uncertainty increases. Historical canonical snapshots are not rewritten when that happens, and a no-game Rating decline is not interpreted as Policy regression.

## Update discipline

For future deployments, record the deployment boundary **before** ranked exposure whenever possible:

1. exact Policy identity / source revision / checkpoint;
2. bot identity;
3. server total games and Rating at deployment;
4. subsequent ranked game count and live Rating observations;
5. DC / timeout / invalid-action quality evidence;
6. canonical snapshot immediately after game 200.

Historical gaps are not repaired by guessing.

## Current tooling assessment

No new generic registry service is justified for v1.

Existing Arena responsibilities already provide the building blocks:

- #168 / #232: durable first-party ranked records and bounded acquisition;
- #253: Policy-provenance-aware RiichiLab longitudinal analysis.

If repeated manual registry updates become an actual bottleneck, the smallest follow-up should export a validated registry-update candidate from those existing sources rather than creating a new database or dashboard.
