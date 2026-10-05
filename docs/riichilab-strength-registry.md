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

## Champion deployment reports — 2026-09-30 lineage

2026-10-06の文書更新で、project #85の既存報告から2botを追記した。raw履歴・completionを今回独立に再計算した記録ではない。

| Bot | 報告上の版内対局数 | 200戦目更新後 | 101〜200戦平均 | 対象末尾の更新後 |
| --- | ---: | ---: | ---: | ---: |
| lisjong-dev / 313 | 209 | 1753 | 1745.6 | 1742 |
| lisjong-baseline / 394 | 214 | 1717 | 1735.34 | 1697 |

Policyは `PlacementAwareSpeedCallPolicy`、lisjong `51e832e50a0ee71eac65e4a017f46590e7438c04`、Arena `b8458bdaf1f49dabe493caf1399b260eba55dbb0`、Rust native API 3。版内対象期間はplayed_atのUTC解釈を前提に2026-09-30 15:04:13〜2026-10-01 07:19:36。

根拠:

- [devの境界・連続性・4 run品質](https://github.com/lisbun/lisjong-project/issues/85#issuecomment-5968478626)
- [baselineの履歴・200戦評価](https://github.com/lisbun/lisjong-project/issues/85#issuecomment-5968808250)
- [baselineの先行2 run原本照合](https://github.com/lisbun/lisjong-project/issues/85#issuecomment-5968690816)
- [baselineの残り2 run原本照合](https://github.com/lisbun/lisjong-project/issues/85#issuecomment-5968820108)

各botの4 runについてdefaulted / stale / unanswered / failedは0と報告され、履歴のis_disconnected / is_penalizedも0。再試行・切断・readbackのrun別項目は、引用報告に明記された範囲だけ値を記録し、baselineの先行2 runで数値を転記できない項目はnullとする。arena #434が直接覆うのは `3ac0b7c7` だけで、他runへ品質を外挿しない。高負荷局面の網羅、CPU/RSS余裕、追加bot数の適合はこの記録では保証しない。

**記録の確定度:** timezone-naiveなplayed_atのUTC解釈と版内対象集合はrun窓・件数の整合による。直接game ID照合・API時刻仕様による確定ではない。ADR 0006に従い、JSONのcanonical count / maturity / Rating / qualityはnullのまま、上表は `evidence.reported_snapshot` に条件付きで保持する。run PASSを未知のinvalid action / chombo等の全項目へ自動変換しない。正確な200戦目観測時刻、完全なwheel hash・behavior configuration等も推測しない。

上記の前提でも両botのR1800レート条件は未達。上位botの表示レートは観測日時付きの証拠が未確認であり、数値を補わない。末尾レートは歴史的snapshotで、現在のlive Ratingではない。

## R1800の200戦と品質の記録規則

[project #84](https://github.com/lisbun/lisjong-project/issues/84)の運用条件を適用する。

- デプロイ境界より後にレート更新されたranked対局をbotごとに数える。2botの合算は禁止。
- 各対局の値は更新後の表示レート。200戦目と101〜200戦の平均を記録し、両方1800以上をレート条件とする。ピーク値では代用しない。
- source revision / checkpoint / behavior configuration変更時は別deploymentとして数え直す。旧版と新版の200戦をつながない。
- サーバーで計上された対局は品質問題があっても除外せず、CLEAN / DEGRADED / CONTAMINATEDと必要証拠を別記する。品質の不足はunknownのまま保持する。
- DEGRADEDを達成根拠に使う場合の追加証拠は#84の条件に従う。CONTAMINATEDや証拠不足をCLEANへ読み替えない。
- defaulted / stale / unanswered / disconnect / failedに加え、invalid action / chombo / timeout / readbackの根拠をrunごとに残す。

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
