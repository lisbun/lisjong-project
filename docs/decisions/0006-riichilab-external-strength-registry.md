# ADR 0006: RiichiLab external strength registry

## Status

Accepted.

## Context

Arenaのcontrolled / reproducible evaluationと、RiichiLab rankedのlive Ratingは異なる問いに答える。

```text
Arena
  fixed protocol / direct comparator / reproducible evidence

RiichiLab
  live matchmaking / changing population / ecosystem-relative signal
```

RiichiLabはOpenSkill系のRatingを使い、uncertaintyを持つ。新規BotのRatingは初期数十戦で大きく動き、Policy更新後のRatingも一定の対局数を通じて追従する。また長期間対局しない場合はuncertaintyが増えるため、対局数が増えていなくてもdisplay Ratingが動き得る。

Reference:
- https://riichi.dev/docs/rating
- https://riichi.dev/docs/ranked

したがって、RiichiLab RatingをArena evidenceへ統合した単一scoreにはせず、exact Policy revisionに紐づく外部実戦の補助signalとして別管理する。

Issue #59と#60は同じregistry concernを扱っていた。#60をcanonical issueとし、#59にあったinactivity driftとthreshold-semantics versioningも本ADRへ統合する。

## Decision

### One record = one exact deployment

Registryの単位はPolicy表示名ではなく、RiichiLab botへ配置されたexact behavioral deploymentとする。

最低限次をbindする。

```text
policy family
policy identity
repository
source revision
behavior-affecting configuration
checkpoint / weights digest when applicable

RiichiLab bot identity
deployment boundary
rating at deployment when observed
games since deployment
current Rating observation
```

同じPolicy class名でもsource revision、checkpoint、またはbehavior configurationが変われば別deploymentとする。

### Rating is not stored alone

Ratingは必ず少なくとも以下と一緒に解釈する。

```text
exact Policy deployment
+ games since deployment
+ observation time
+ execution-quality evidence
```

Botの通算gamesや通算Ratingを、そのままcurrent Policyのevidenceへ帰属させない。

### Operational maturity states

v1のmaturity classificationはpost-deployment ranked game countだけから決める。

```text
< 50       PROVISIONAL
50-99      PRELIMINARY
100-199    STABLE
>= 200     MATURE
```

これはtrue skillを統計的に既知とする宣言ではない。RiichiLabのlive populationとrating dynamicsに対するproject-level operational thresholdである。

将来thresholdを変更する場合はsemantics revisionを上げる。historical recordのmaturity labelを新thresholdで黙ってretroactiveに書き換えない。

### Canonical 200-game snapshot

各deploymentの最初の200 post-deployment ranked games到達直後のRatingをcanonical snapshotとする。

```text
canonical_200_rating
canonical_200_observed_at
canonical_200_quality
```

200戦後もlive運用してよいが、continuous live Ratingでcanonical snapshotを上書きしない。peak / best observed Ratingをcanonical valueにしない。

結果が期待と違うことやDCが含まれたことを理由に、canonical endpointを201戦目以降へ延長しない。

### Server-counted games and execution quality are separate

RiichiLab側でranked gameとして成立しRating更新対象になった対局は、Policy executionがdegradedしていてもpost-deployment exposureへ含める。

DC、timeout/default action、invalid action、chombo等を理由にgame countから除外して、server-side Rating historyとregistry側countをずらさない。

同時にexecution qualityを別管理する。

```text
clean_games
dc_games
timeout_games
invalid_action_games
chombo_games
defaulted_action_count
```

取得元から厳密に判定できない値は推測せずunknown / unavailableのまま保持する。

Canonical qualityの意味は次の3段階とする。

```text
CLEAN
  no DC / invalid action / chombo / timeout / default action

DEGRADED
  no DC / invalid action / chombo
  but timeout or default action occurred

CONTAMINATED
  DC, invalid action, or chombo occurred
```

必要なquality evidenceが不足している場合、quality classそのものを推測しない。evidence completenessを別fieldで明示する。

### Inactivity-driven Rating movement

RiichiLabでは長期inactivityによりuncertaintyが増え、追加対局なしでもdisplay Ratingが変化し得る。

したがってcurrent/live Ratingを保存する場合は、games_since_deploymentとobservation timeを併記し、可能ならlast ranked game timeも保持する。

```text
games unchanged + display Rating changed
!= Policy regression
```

Historical canonical snapshotは後日のinactivity driftで書き換えない。

### Same-bot reuse and carry-over

RiichiLab bot slotは有限であり、bot identityの再利用を許容する。

新Policyを同一botへdeployした場合:

```text
new deployment
games_since_deployment = 0
```

から開始する。一方server-side Ratingは過去deploymentの状態をcarry overし得るため、fresh independent initializationとは扱わない。

可能なら`rating_at_deployment`を保存し、欠損している場合は再構築・補正しない。

### Candidate scope

全実験PolicyをRiichiLabへ送らない。原則としてlocal / Arena sanityを通過したmeaningful candidateに限定する。

Candidate例:
- current Heuristic Champion
- current Learning Champion
- current Overall candidate / Champion
- meaningful challenger
- materially different research Policy

short-lived ablationやcheap screenでrejectされたcandidateは対象外とする。

### Champion and Arena boundary

RiichiLab RatingだけでChampion promotionを決めない。

```text
RiichiLab Rating
  != Heuristic Champion promotion
  != Learning Champion promotion
  != Overall Champion determination
```

Arena evidenceとRiichiLab evidenceを単一metricへ合成しない。矛盾して見える場合は調査対象とする。

### Historical records are append-oriented

後続revisionで過去deployment recordを上書きしない。canonical 200-game snapshotはimmutable historical evidenceとして保持する。

Current/live fieldsは更新してよいが、どのobservationを置換したかが分かるようobservation timeを保持する。

## Representation

v1はdatabaseやdashboardを導入せず、次の2層だけを持つ。

```text
registry/riichilab-strength.json
  machine-readable source of truth

docs/riichilab-strength-registry.md
  human-readable summary / interpretation
```

Schema自体もregistry file内でversionし、threshold semantics revisionを明示する。

Exact post-deployment game countやrating-at-deploymentを立証できないhistorical dataは、値を推測してcanonical recordへ格上げしない。partial evidenceはpartialのまま表現できる。

## Initial population rule

Historical全Policyのbackfillは行わない。

既に確認できるcurrent meaningful Policyから開始し、必要fieldが不足する場合は:

- known factsだけを記録する;
- unknownはnull / unavailableで保持する;
- maturityを推測しない;
- server通算gamesをpost-deployment gamesへ誤変換しない。

## Repository responsibility

- `lisjong-project`: registry semantics、machine-readable registry、human-readable project summary
- `lisjong-arena`: RiichiLab execution / observation、durable per-game provenance、必要になったbounded acquisition / analysis tooling
- `lisjong`: stable Policy behavior identity / implementation

Generic registry service、database、web dashboard、automatic deploymentは作らない。

## Consequences

- RiichiLabのlive progressをrevision単位で追える。
- Arenaのscientific comparisonとlive ecosystem signalを混同しない。
- bot reuse時のcarry-over、DC、timeout、inactivity driftを隠さない。
- exact provenanceがないhistorical gamesを無理にbackfillしない。
- 200戦時点のcanonical endpointをresult-drivenに動かせない。
- 初期registryはpartialでも正確性を優先し、取得基盤が整った時点でfuture deploymentを完全記録できる。
