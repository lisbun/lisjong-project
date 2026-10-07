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

上記の前提でも両botのR1800レート条件は未達。上位botの表示レートは下の「上位botの表示レート（参考観測）」に観測日時付きで記録した。末尾レートは歴史的snapshotで、現在のlive Ratingではない。

## 上位botの表示レート（参考観測）

JSONの `reference_rating_observations` に保持する。deployment recordではなく比較の参考であり、Arena評価やR1800判定の代わりにしない。

- 観測日時: **2026-10-07T06:05:21.248281Z**（arena #441の `riichilab_coplayer select` 実行時刻。Arena `db6e386`、`selection.json` sha256 `96679302…cad6`）
- 選定規則（[arena #441で事前固定](https://github.com/lisbun/lisjong-arena/issues/441#issuecomment-6018456067)）: rating 1800以上、total_games 1000超、最終対局が観測時刻から30日以内
- rating 1800以上の候補102体のうち38体が該当（除外: 対局数と30日超の両方40、30日超のみ15、対局数のみ9）
- 値はサーバー表示値の転記で、再計算していない。`last_played_at` はAPIのtimezone-naiveな値をそのまま記録した
- 1回の観測snapshotであり、後で値を書き換えない。新しい観測は別entryとして追加する
- 旧固定の参考bot（FuuroMaster、zero-test2）はこの群に含まれない（FuuroMasterは今回の1800以上の候補に含まれず、zero-test2は対局数・最終対局で除外）。Mortal-v4bは含まれる

同卓比較の結果は[arena #441](https://github.com/lisbun/lisjong-arena/issues/441#issuecomment-6034292374)を参照。

| 順位 | Bot / ID | 表示レート | 対局数 | 最終対局（naive） |
| ---: | --- | ---: | ---: | --- |
| 9 | NO.24 / 323 | 1985 | 4418 | 2026-09-25T15:52:53 |
| 14 | Deckard / 386 | 1981 | 1655 | 2026-09-29T01:35:40 |
| 15 | MahjongAI-S70 / 305 | 1965 | 5866 | 2026-09-11T02:12:41 |
| 16 | test / 241 | 1953 | 5994 | 2026-09-13T14:58:55 |
| 17 | LuckyJK / 275 | 1935 | 2457 | 2026-09-19T21:06:25 |
| 19 | doge / 31 | 1927 | 6033 | 2026-10-04T22:36:38 |
| 20 | みーにょ6段 / 203 | 1926 | 24179 | 2026-10-07T06:04:23 |
| 29 | Mortal-v4b / 120 | 1893 | 7104 | 2026-10-07T06:04:23 |
| 31 | Momochan / 292 | 1891 | 5616 | 2026-10-07T06:00:48 |
| 33 | old / 350 | 1889 | 9491 | 2026-10-07T06:03:42 |
| 37 | a / 355 | 1879 | 1260 | 2026-10-03T17:34:28 |
| 38 | Nshinki / 125 | 1877 | 9542 | 2026-09-14T15:16:24 |
| 39 | 一号机 / 398 | 1876 | 2744 | 2026-10-07T06:04:11 |
| 40 | nodoka-latest / 299 | 1873 | 7312 | 2026-10-07T06:03:42 |
| 41 | Mamba Out / 123 | 1872 | 9539 | 2026-09-14T15:16:24 |
| 42 | RIN / 381 | 1872 | 1694 | 2026-09-24T19:50:07 |
| 43 | RLActor-170 / 418 | 1871 | 1194 | 2026-10-07T06:04:23 |
| 45 | cherry / 22 | 1869 | 35792 | 2026-10-06T08:10:00 |
| 50 | baseline_bot1 / 337 | 1863 | 1566 | 2026-09-11T08:27:38 |
| 54 | 花大荔 / 127 | 1855 | 9548 | 2026-09-14T15:15:59 |
| 55 | COEDO緑 / 278 | 1855 | 6397 | 2026-10-07T05:51:01 |
| 58 | Epsilon-Nano / 354 | 1853 | 2638 | 2026-09-20T21:43:38 |
| 59 | 荔猪 / 129 | 1849 | 9600 | 2026-09-14T15:15:19 |
| 67 | RLActor-50 / 411 | 1838 | 1134 | 2026-10-04T14:51:35 |
| 72 | Akasha-0923 / 390 | 1833 | 3368 | 2026-10-07T06:04:11 |
| 73 | Akasha-cand1 / 414 | 1833 | 1622 | 2026-10-07T06:04:11 |
| 74 | mjrl / 320 | 1832 | 5420 | 2026-10-07T06:01:43 |
| 75 | Epsilon-265706 / 357 | 1832 | 2802 | 2026-10-05T12:23:12 |
| 78 | ねじまき鳥-atk / 408 | 1828 | 2288 | 2026-10-07T06:01:44 |
| 79 | Momochan-vvv / 293 | 1823 | 5839 | 2026-10-07T06:03:42 |
| 80 | Kanachan / 269 | 1821 | 7149 | 2026-10-07T06:01:37 |
| 81 | new_1 / 351 | 1821 | 9571 | 2026-10-07T06:01:43 |
| 84 | 花芽荔枝 / 128 | 1817 | 9187 | 2026-09-14T15:15:14 |
| 87 | policy-170k / 388 | 1816 | 2892 | 2026-10-02T00:48:40 |
| 90 | Noob-bot / 322 | 1810 | 3997 | 2026-09-23T06:07:49 |
| 94 | serre / 291 | 1808 | 1386 | 2026-09-22T00:02:48 |
| 98 | PigRaichi / 124 | 1805 | 9583 | 2026-09-14T15:15:19 |
| 100 | 吃貨小北北 / 277 | 1801 | 6485 | 2026-10-07T06:01:16 |

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
