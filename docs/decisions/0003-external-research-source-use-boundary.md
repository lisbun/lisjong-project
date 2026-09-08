# ADR 0003: External research source use boundary

## Status

Accepted.

## Context

lisjongのresearch trackでは、外部麻雀AI、外部model / weight、公開牌譜・log、program output等を、reference、benchmark、trajectory source、training label候補として扱う可能性がある。

これらは技術的に取得・実行できる場合でも、source code、model asset、generated output、ML利用、再配布の条件が同一とは限らない。

また、外部teacherをtraining sourceへ使う場合、teacher action distributionやvisited state distributionが既存datasetと変わり得るため、単なるdata quantity変更とも異なる。

したがって、technical accessibilityだけを根拠にtraining corpusへ昇格させると、provenance / complianceとscientific comparisonの両方を曖昧にする。

## Decision

外部research sourceは、少なくとも次の境界を独立して判定する。

```text
identity / provenance
        ↓
private execution / acquisition basis
        ↓
output retention basis
        ↓
ML / distillation use basis
        ↓
redistribution basis
```

次を同一視しない。

```text
technically accessible
!= approved training data

open-source code
!= unrestricted model / weight

program may be executed
!= generated labels may be used for ML

output may be retained
!= generated corpus may be redistributed
```

曖昧な用途は、許可されていると楽観的に推定せず `HOLD` 相当として扱う。

外部teacher / modelを利用するbounded experimentでは、exact upstream repository、revision / release、model / weight identity、provenance、該当terms snapshotを記録する。unofficial fork / weight / mirrorをcanonical artifactへ黙って代用しない。

player-side serving / distillation inputではplayer-visible information boundaryを維持する。offline validationでprivileged truthを使う場合も、ordinary deployable teacher / student inputと区別する。

cross-teacher comparisonでは、既存teacherに対するimitation lossをteacher quality metricへ自動昇格させない。teacherやtrajectory sourceが変わる場合は、same-state relabelingやteacher-neutral downstream evaluation等、changed distributionを考慮したbounded comparisonを別途設計する。

## Repository responsibility

- `lisjong-arena`: external/local execution、bounded feasibility、experiment-local corpus / training / analysis、provenance / use-gate evidenceを扱える。
- `lisjong`: approved research resultがstable production Policy / feature / inference semanticsへ昇格する場合のcanonical AI-side contractを所有する。
- `lisjong-project`: data-source / permission / architectureをまたぐresearch directionとpromotion boundaryを調整する。

外部source固有のlicense interpretationやexact experiment resultを恒久architecture文書へ重複維持せず、該当Issue / upstream terms snapshotを正本とする。

## Consequences

- external teacherのtechnical smoke成功だけではtraining data generationを開始できない。
- local-only executionであることだけではML use permissionの根拠にならない。
- corpusを公開しない計画でも、ML / distillation use basisは別途必要になる。
- exact identity / termsが不明なexternal modelは、別artifactで黙って置換せず、そのcandidateをHOLD / blockedとして扱える。
- data-source axisをarchitecture/objective axisと同じexperimentで無自覚に変更しにくくなる。

このdecisionは広範な法的意見をproject内で作ることを要求しない。研究利用に必要な範囲で、利用条件を保守的かつ再現可能に記録するためのproject governance boundaryである。

## Evidence that motivated the decision

Arena #192のbounded Gate 0では、canonical Akochanがplayer-safe local trajectory / replay sourceとして技術的に成立した一方、program outputをML/distillation labelとして利用する明示的basisを確認できず、technical `GO` と ML-use `HOLD` が分離された。

同じGate 0では、historical Arenaで使われたMortal checkpointがcanonical author-released weightではないことも確認され、implementation identityとmodel/weight provenanceを分離する必要性が具体化した。

Exact candidate identities、terms、measurements、classificationはGitHub Issuesを正本とし、このADRには固定しない。
