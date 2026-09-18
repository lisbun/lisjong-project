# Mahjong evaluation statistics

## 目的

本書は、lisjong ecosystemで麻雀の実戦牌譜・自己対局・Arena評価を解釈するときの**横断的な統計規律**を定めるガイドです。

目的は、短期のブレ、局内相関、post-hoc slicing、opponent / Policy交絡を無視して、偶然の差をPolicy改善・劣化として読みすぎないことです。

本書は特定Policyの強弱を判定するprotocolではありません。具体的なseed、sample size、primary statistic、interval / test、promotion thresholdは、対象experimentのArena-owned contractで事前に固定します。

```text
point estimate
    + sample size / denominator
    + uncertainty
    + cohort / provenance
    + confirmatory or exploratory status
        |
        v
appropriate interpretation

!= short-window overinterpretation
!= post-hoc slice = proven weakness
!= observational result = causal Policy effect
```

## Scope / non-goals

本書が扱うもの:

- 平均順位、順位率、和了率、放銃率等のsampling variation
- hanchan-level / round-level metricの観測単位
- hanchan clusteringを考慮したuncertainty
- primary / secondary / exploratory analysisの分離
- recent / rolling windowの解釈境界
- observational RiichiLab historyとcontrolled Arena evaluationの分離
- Policy / rules / runtime / opponent population provenance
- reporting checklist

本書が固定しないもの:

- universal minimum hanchan count
- p-valueだけによるautomatic promotion
- every analysisへの単一のbootstrap variant / replicate count
- every exploratory sliceへの一律Bonferroni
- Bayesian modelの導入義務
- causal inference framework
- current Champion
- specific Policyの強弱

## 1. 麻雀の平均順位は大きくぶれる

4人麻雀の順位を 1, 2, 3, 4 としたとき、各順位が25%ずつなら1半荘あたりの順位分散は1.25です。公開されている麻雀統計の解説では、現実的な順位分布でも概ね1.1〜1.45程度の範囲が例示されています。

独立・stationaryなhanchanを仮定し、順位分散を1.25と置いた単純モデルでは、平均順位の95% sampling half-widthは概ね

```text
1.96 * sqrt(1.25 / N)
```

です。

| Hanchan | 95% sampling half-widthの目安 |
| ---: | ---: |
| 100 | 約 ±0.22 |
| 500 | 約 ±0.10 |
| 1,000 | 約 ±0.07 |
| 2,000 | 約 ±0.05 |
| 5,000 | 約 ±0.03 |
| 10,000 | 約 ±0.02 |

これは**hard gateではありません**。次の強い仮定を置いたorder-of-magnitude referenceです。

```text
same underlying population
independent hanchan
stable rules / Policy / opponents
no result-driven filtering
approximately stable rank variance
```

実際の必要sample sizeは少なくとも次に依存します。

```text
metric
expected variance / event rate
detectable effect size
desired uncertainty
comparison design
pairing / blocking
opponent / rule stability
decision cost of false positive / false negative
```

したがって、`500半荘なら十分`、`2,000半荘なら証明済み`のような単一thresholdを正本にしません。

### Precisionからsample sizeを考える

平均順位について独立hanchan・分散 `v` の単純モデルを使うなら、目標half-width `h` に必要なsample sizeのrough planning valueは、

```text
N ~= (1.96^2 * v) / h^2
```

で見積もれます。`v = 1.25`なら、`h = 0.05`で約1,921、`h = 0.03`で約5,336、`h = 0.02`で約12,005 hanchanです。

これもplanning aidであり、paired comparisonでは**差分の分散**、clustered / blocked designでは**design unitの分散**を使う方が適切です。実験結果を見た後にdesired precisionやsample sizeを有利な方向へ変更しません。

## 2. Point estimateだけを報告しない

主要metricは原則として次をセットで扱います。

```text
estimate
sample size
numerator / denominator where applicable
uncertainty interval
cohort definition
```

例えば和了率なら、

```text
win rate = 22.1%
wins = 221
eligible rounds = 1,000
95% interval = ...
```

のように、率だけを切り離して表示しません。

平均順位、1着率、4着率、和了率、放銃率、リーチ率、副露率等をpoint estimate単独でPolicy評価に使いません。

interval methodはmetricとdesignに依存します。独立binary trialの近似を機械的にround-level麻雀metricへ適用せず、次節のcluster structureを確認します。

## 3. Half-gameとroundを同じ独立sampleとして数えない

同一半荘内の局は完全独立ではありません。

点棒、順位状況、親番、供託、本場、同一対戦相手、終盤戦略等を半荘内で共有するため、

```text
2,500 rounds
!= 2,500 independent samples
```

です。

### Hanchan-cluster bootstrap

通常のhanchan cohortからround-level metricのuncertaintyを求める場合、**hanchanをclusterとしてresampleするcluster bootstrapをdefault candidate**とします。

conceptually:

```text
N hanchan
   |
   v
sample N hanchan with replacement
   |
   v
selected hanchan内の全roundを保持
   |
   v
metric recompute
   |
   v
repeat B times
   |
   v
bootstrap distribution / interval
```

clusterを丸ごとresampleすることで、同一hanchan内の依存構造を壊しにくくします。

ただし、cluster bootstrapは魔法の補正ではありません。

- cluster数が少ない場合はbootstrap distribution自体が不安定になり得る
- percentile / basic / BCa / normal interval等で性質が異なる
- replicate count不足はMonte Carlo errorを増やす
- selection / survivorship / opponent confoundingはbootstrapでは解消しない

したがって、exact variant、replicate count、random seed、interval construction、minimum cluster handlingは目的別実装で固定し、synthetic testを持ちます。

### Paired / blocked Arena design

paired seed、seat rotation、AABB/ABBB block等、hanchanより上位に比較designの依存単位がある場合は、その**design unitを壊さないresampling / inference**を優先します。

```text
preserve pairing / blocking
> blindly resample individual rows
```

本書の `hanchan-cluster bootstrap` は、round-level observational / ordinary cohortに対するdefault candidateであり、purpose-specific paired protocolの設計を上書きしません。

## 4. Recent / rolling windowはdiagnosticでありstrength proofではない

```text
latest 20
latest 40
latest 100
rolling average
```

は、次の用途には有用です。

- regression / implementation anomalyの発見
- Policy切替時のbehavior change確認
- DC / timeout等の運用品質監視
- follow-up hypothesis生成

しかし単独で、

```text
recent window worsened
    -> Policy became weaker
```

とは判断しません。

UI / report / Issueでshort-window viewを提示する場合、`descriptive / diagnostic`であることを明示します。window数や境界を結果を見ながら都合よく選んだ場合は、さらにexploratoryとして扱います。

## 5. Primary / secondary / exploratoryを分離する

麻雀牌譜には多数のsliceがあります。

```text
riichi / dama
open / closed
1 / 2 / 3 calls
dealer / non-dealer
early / middle / late
good wait / bad wait
score band
opponent riichi state
```

同じcohortを見た後に多数の比較を行えば、偶然extremeなsliceを見つけやすくなります。

lisjongでは少なくとも次を区別します。

```text
Primary metrics
  pre-specified
  confirmation / promotion判断に利用可能

Secondary diagnostics
  pre-specified
  behavior interpretation

Exploratory slices
  post-hoc allowed
  hypothesis generation only
```

exploratory resultはsame cohort上でprimary evidenceへ昇格させません。重要な仮説が見つかった場合は、独立cohort、fresh seed population、paired Arena evaluation等で事前定義して再検証します。

multiple-comparison adjustmentが必要な場合、比較familyと目的を先に定義します。Holm / Bonferroni / simultaneous interval / FDR等のどれを使うかは目的依存であり、v1では単一方式を強制しません。

重要なのは、

```text
look at many slices
find one extreme result
treat it as if pre-specified
```

をしないことです。

## 6. Observational historyとcontrolled evaluationを分ける

RiichiLab live historyでは、

```text
Policy period
time
self Rating
opponent population
matchmaking
table composition
execution quality
```

が共変動し得ます。

したがってlongitudinal observational historyは、

```text
what happened under this cohort
```

の記述や仮説生成には使えますが、

```text
Policy A caused +X strength
Policy A is proven stronger than B
```

の因果判定には使いません。

Policy比較のconfirmationは、可能な限りArenaのcontrolled / paired evaluationへ残します。

長期sampleであってもobservational dataは自動的にcausal evidenceへ変わりません。大量sampleはsampling errorを小さくできますが、systematic confoundingを消しません。

## 7. Denominatorとsmall-N warningを常に見せる

subgroup metricは率だけ表示しません。最低限、

```text
hanchan count
round count
event count
denominator
interval
```

を併記します。

`N < 30なら禁止`のような一律minimumは設けません。metricやevent rateで必要Nが変わるためです。

代わりに、machine-readable summaryでは少なくとも次を保持できる設計を推奨します。

```text
n_hanchan
n_rounds
numerator
denominator
interval_low
interval_high
interval_width
event_count
warning_flags
```

`wide_interval`、`few_clusters`、`sparse_events`等のwarning thresholdを使う場合、そのthresholdはanalysis protocol側で事前固定します。

## 8. Policy provenance / population stabilityを守る

sampleを増やしても、途中でPolicyや条件が変われば一つのpopulationとして扱えません。

可能な限り次を保持します。

```text
Policy identity
source revision / checkpoint
rules
runtime / profile
execution software revision
time range
opponent-strength context
disconnect / abnormal termination
```

例えば、

```text
10,000 historical games with mixed Policies
!= 10,000 games for the current Policy
```

です。

同様に、ruleset、matchmaking population、execution environment等がmaterialに変わった場合は、cohort boundaryを明示します。異なるpopulationを単にsample sizeを稼ぐためにpoolしません。

## 9. Comparisonを始める前に固定するもの

confirmatory comparisonでは、result exposure前に少なくとも次を固定します。

```text
question / hypothesis
candidate and baseline identity
evaluation population
seed / game population or acquisition window
seat / pairing / blocking design
primary metric
uncertainty / test method
sample size or precision target
decision / classification rule
invalid / partial execution handling
```

結果を見た後のseed追加、window変更、metric差替え、subgroup選択、threshold変更は、同一confirmatory eventの継続として扱いません。必要ならfreshな独立Issue / experimentとして設計します。

## 10. lisjongでの評価signalの分離

lisjong ecosystemでは評価signalを概念上次のように分けます。

```text
1. Controlled Arena evaluation
   -> Policy selection / confirmation

2. Long-horizon RiichiLab longitudinal history
   -> external / ecological performance context

3. Mahjong behavior metrics
   -> diagnosis and mechanism hypotheses

4. Teacher / value-based evaluators
   -> sample-efficient auxiliary signal
```

第4層は実戦成績の代替ではなく補助評価です。

MJ-DLVATは、通常の平均順位より分散の小さいvalue-based estimatorを提案し、報告された実験では推定順位の分散を45.5%削減しています。この種のvariance-reduced evaluatorは将来候補ですが、そのestimator自体のbias / calibration / target populationを別途検証する必要があり、本書では導入を義務化しません。

## 11. Reporting checklist

analysis / evaluation reportでは最低限次を確認します。

- cohort definition
- Policy / revision / checkpoint provenance
- rules / runtime / execution revision where relevant
- time range
- hanchan count
- round count where applicable
- metric numerator / denominator
- point estimate
- uncertainty interval
- uncertainty method / resampling unit
- disconnect / abnormal-game handling
- opponent / population context
- confirmatory / secondary / exploratory classification
- missing-data / coverage

Policy comparisonではさらに、

- comparison design
- pairing / blocking / seat rotation
- pre-specified sample size or precision target
- primary statistic / classification rule
- result exposure後のextension有無

を明記します。

## 12. Arena / RiichiLab implementation guidance

`lisjong-arena`のanalyzer / evaluatorは、purpose-specific contractを優先しつつ、本書の原則を参照します。

特にRiichiLab longitudinal analysisでは、

- point estimate onlyを避ける
- hanchan / round sample sizeとdenominatorを常時表示する
- round-level uncertaintyではhanchan clusteringを無視しない
- recent / rolling windowはdiagnostic onlyとする
- exploratory subgroupをconfirmationに使わない
- Policy provenanceとopponent-strength contextを分離する
- observational resultをcausal Policy effectと表現しない

を守ります。

Arenaのcontrolled paired evaluationでは、hanchan単位だけでなくseed / rotation / matchup block等のdesign dependencyを保持し、experiment-specific protocolでuncertaintyを固定します。

## References

- みーにん, 「麻雀における平均順位のぶれ」, 2018-09-27. https://note.com/meaningless777/n/n17c8ad1f6569
- Takuya Ogami, Katsutoshi Amano, Yoshimasa Tsuruoka, “MJ-DLVAT: A Deep Learning Value Assessment Technique for Mahjong,” IEEE Conference on Games 2024. DOI: https://doi.org/10.1109/COG60054.2024.10645601
- University of Tokyo repository copy of MJ-DLVAT: https://repository.dl.itc.u-tokyo.ac.jp/record/2013721/files/48236414.pdf
- Ren et al., “Simple Approaches for Dealing With Correlated Data,” clustered bootstrap discussion. https://pmc.ncbi.nlm.nih.gov/articles/PMC10893847/
- “Nonparametric bootstrap methods for interval estimation ... with correlated diagnostic test data,” cluster bootstrap procedure. https://pmc.ncbi.nlm.nih.gov/articles/PMC10728486/
- NIST/SEMATECH e-Handbook of Statistical Methods, “How can we make multiple comparisons?” https://www.itl.nist.gov/div898/handbook/prc/section4/prc47.htm

## Interpretation summary

```text
large N
    reduces sampling noise
    but does not erase confounding

cluster bootstrap
    respects within-hanchan dependence better
    but does not repair selection bias

exploratory slicing
    generates hypotheses
    but does not become confirmation retroactively

RiichiLab history
    provides ecological context
    but does not replace controlled comparison
```
