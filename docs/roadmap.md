# Roadmap

## 目的

本書は lisjong ecosystem 全体として、どの能力をどのような依存関係で成立させたいかを示す長期ロードマップです。

本ロードマップは、すべての能力を上から順番に完了させる直列Phase一覧ではありません。基礎契約を共有した上で、game engine、Policy、external execution / observation、AI research、Policy comparison、external benchmark、Visualization / Analysisをそれぞれの責務境界で並行して発展させ、必要な地点で統合します。

個別Issueの実装順序、現在の進捗、完了状態は各repositoryのGitHub Issues / PRsを正本とします。本書にはIssue番号や完了checkboxを原則として持ちません。

## Roadmap principles

- 接続可能性と正しさを、AIの強さより先に確立する
- 麻雀ルール、AI判断、external execution / observation、研究用学習・分析、比較評価、可視化を明確な責務境界で分離する
- deterministicな実行条件を早期に確立し、後続の回帰testと比較に利用する
- 外部環境固有の都合をPolicyやengineの内部契約へ漏らさない
- execution / observationとevaluationをArena内でも別責務として扱う
- raw execution data acquisitionとAI feature / training semanticsを分離する
- credentialやprivileged observer dataをPolicy inputへ逆流させない
- 成熟したOSSや強い既存AIはreference / backend / benchmark / toolingとして積極的に評価・利用する
- 成熟したOSSがgame executionやprotocol interoperabilityを既に提供する場合は、同等機能の重複実装を避けて優先利用する
- strong-AI referenceをcanonical designとはみなさず、mechanism-level principleへ分解してlisjong自身のevidenceで検証する
- stable public contract、Policy semantics、repository responsibility、project固有artifact contract、cross-repository dependency directionはlisjong ecosystem側で所有する
- 一度に複数の研究軸を変更せず、原則としてone bounded experiment = one primary research questionを維持する
- cheap metricを最終optimization targetへ昇格させず、reject / triage / prioritizationに利用する
- fixed-protocol hanchan performanceを総合strengthのNorth Starとして維持しつつ、より安価なstate / round-level evidenceで明らかな退化を早期に落とす
- component quality、Policy decision quality、game performanceを別のquality claimとして扱う
- negative / inconclusive / invalid resultも次の研究判断に使えるevidenceとして保持する
- correctnessを確立し、独立validationとregression protectionを行った後に性能を計測し、実測されたbottleneckを最適化する
- 評価対象に対して最小十分なevaluation scopeを選び、必要になった時点でより高コストなscopeへ拡張する
- engine完成をPolicy改善や初期Arena開始の不要な前提にしない
- 比較可能な対象と再現可能なgame実行が揃う前にArena evaluationを過剰構築しない
- 実際の複数execution pathやconcrete consumerが揃う前に、将来backend / runtimeを推測した汎用abstractionを先行設計しない
- Arena外の複数consumerや独立したproduction hosting要件が成立した時点で、共通runtime抽出を再検討する
- Visualization / Analysisのためにproject-wide canonical event schemaを先行設計せず、具体的consumer requirementsから必要なadapter / normalization boundaryを抽出する
- 学習Policyは既存のPolicy contract、execution、evaluation、analysis基盤を可能な限り再利用し、学習専用の別世界を作らない
- 自動化は小さく決定的なloopから始め、actual bottleneckが確認された範囲だけremote execution、candidate search、self-play、LLM-assisted researchへ広げる

## How to read this roadmap

長期的な能力の発展は、概ね次の構造で捉えます。

```text
                    Foundation
                  Policy contract
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
      Engine Track   Policy Track    Arena Track
                                      /        \
                                     v          v
                         Execution / Observation  Evaluation
                                     |          |
                                     +-----+----+
                                           v
                                 game / analysis data
                                           |
                              +------------+------------+
                              |                         |
                              v                         v
                        AI improvement        Visualization / Analysis
                              |
                              v
                       Learning Policy
```

Engine / Policy / Arenaの各trackは互いの完成を不必要に待たず進めます。

`lisjong` はAI decision coreとして発展し、external environmentへの接続・session lifecycle・raw observation acquisitionはArena側のexecution / observation capabilityとして発展させます。Arena evaluationはそのexecution能力をconsumerとして利用できますが、execution layerへcomparison semanticsを逆流させません。

Visualization / Analysisはread-orientedなconsumer能力として、具体的なdata sourceとconsumer requirementが成立した地点から発展させます。

## Durable AI strengthening loop

AI強化は、思いついたalgorithmを順番に追加する作業ではなく、**cheap evidenceから高fidelity evidenceへ進む反復的なresearch loop**として扱います。

概念上の基本loopは次です。

```text
strong-AI reference / literature / lisjong evidence
                    |
                    v
        mechanism-level hypothesis
                    |
                    v
        one-axis bounded candidate
                    |
                    v
          cheap diagnostic gates
                    |
                    v
      progressively higher fidelity
                    |
                    v
       fixed-protocol hanchan evidence
                    |
                    v
      evidence ledger / interpretation
                    |
                    +--------------------> next hypothesis
```

### Reference-guided, not reference-driven

Mortal、Suphx、C3等の強い既存AIは、次の研究仮説を作るための重要なreferenceです。ただし、強いsystemの構成要素は複数要因の組み合わせなので、採用実績だけから単独要素の因果効果を主張しません。

```text
used by strong AI
    = REFERENCE EVIDENCE

works for lisjong
    = requires LISJONG EVIDENCE
```

具体的なlayerやalgorithm名をそのまま追うより、例えば次のようなmechanism-level principleへ正規化します。

- deterministic Mahjong structureをnetworkに再発見させない
- tile structureを保つinductive biasを与える
- long-horizon outcomeをlearning target / auxiliary signalへ伝える
- offline dataset support外のaction value過大評価を抑える
- public historyをsequentialに統合する
- data scale / population diversityを必要性に応じて増やす
- self-play / historical opponentをpolicy improvementへ利用する

その後、lisjongのcurrent failureや既存artifactへ最も直接的で、かつ最安に切り分けられるprincipleから1軸ずつ検証します。

### One-axis bounded experiment

1つのbounded experimentでは、可能な限りprimary changed axisを1つにします。

例えば次を同一experimentで同時に変えません。

```text
representation
objective
reward
dataset population / scale
network scale
auxiliary heads
self-play population
```

negative resultでも原因切り分けに情報が残る設計を優先します。結果を見てseed、threshold、feature bundle、救済runを追加して同一candidateを延命しません。

### Multi-fidelity evaluation ladder

総合strengthのNorth Starは、引き続き**fixed-protocol hanchan performance**です。

ただしcandidate iterationのたびに、いきなり最も高価な半荘評価へ進む必要はありません。概念上は次のような段階を使います。

```text
Gate A — player-safe same-state / hand-progression diagnostics
    shanten / ukeire / tenpai progression / teacher disagreement 等

Gate B — low-complexity partial environment
    candidate vs passive / tsumogiri-like opponents
    tenpai / riichi / win / offensive round result 等

Gate C — interactive single-round evaluation
    stronger fixed opponents
    round score EV / win / deal-in / draw outcome 等

Gate D — hanchan evaluation
    overall development strength / North Star
```

重要な関係は次です。

```text
cheap metric
!= final optimization target

cheap metric
= reject / triage / prioritization signal
```

Gate A/Bを通過したことだけで「強いPolicy」とは主張しません。一方、基本的な手牌進行やpassive opponents相手の攻撃能力すら明確に退化しているcandidateは、高価なGate C/Dへ送る前にrejectできる可能性があります。

各proxy自体もvalidation対象とします。同じcandidateについて可能な範囲でcheap / round / hanchan evidenceを併記し、cheap proxyが実際にhigher-fidelity strengthを予測するか継続的に確認します。予測力が弱いproxyにはpromotion / rejection authorityを与えません。

### Curriculum — basic capability before full complexity

初期Learned Policyでは、最初から麻雀の全能力を同時に獲得できることを前提にしません。

conceptually:

```text
straight hand progression
    shanten / ukeire
        ↓
tenpai
        ↓
riichi / win
        ↓
calling / hand value
        ↓
defense / push-fold
        ↓
opponent belief / hidden-state use
        ↓
placement / long-horizon game utility
```

これは永久に直列で学習するという意味ではありません。current learnerが基本能力を獲得できているかをcheap gateで確認し、複雑なfailureを一度に抱え込まないためのresearch curriculumです。

simple / structural Policyやfinite-horizon search等は、最終teacherや最終Policyでなくても、基礎能力を学習・診断するteacher / comparatorとして利用できます。

### HandBelief as a distinct research axis

HandBeliefはLearned Policy強化の単なる付属featureとして扱わず、lisjong独自の重要な研究軸として並行して発展させます。

```text
Track A — belief quality
    hidden-state prediction / calibration / robustness

Track B — decision value
    beliefを実際のAction選択へ使う価値

Overall Policy strength
    hanchan / game-performance evidence
```

この3つを混同しません。belief estimatorの精度向上だけでPolicy strength向上を主張せず、逆にPolicy側の短期的なnegative resultだけでhidden-state inference研究全体を棄却しません。

将来shared representationやjoint learningを検討する場合も、separate estimator / oracle / classical baseline等でdecision valueを切り分けられる設計を優先します。

### Small automatic loop first

自動化は最も小さく、安価で、再現可能なloopから始めます。

```text
candidate artifact / configuration
        ↓
deterministic cheap gates
        ↓
validated result
        ↓
reject / advance
```

このsmall loopはLLMをhard dependencyにしません。大量反復する部分では、deterministic code、machine-readable artifact、predeclared ruleを優先します。

より大きなloopでは、人間またはLLM / coding agentを、evidence review、next hypothesis selection、bounded experiment design、implementation / review assistance、unexpected failure interpretation等の低頻度・高情報価値の判断へ利用できます。

```text
small loop
    high frequency / deterministic / no LLM required

large loop
    lower frequency / research judgment / LLM-assisted where useful
```

自動Issue生成、automatic merge、fully autonomous researchを初期completion conditionにはしません。

### Scale / remote execution only after measured need

AWS、hosted CI、distributed execution、batch inference、large self-play等はresearch velocityを上げる手段であり、goalではありません。

```text
local bounded loop
    ↓
runtime / storage / manual work measurement
    ↓
actual bottleneck identified
    ↓
smallest justified remote / scale capability
```

always-on infrastructureやgeneric cloud abstractionを先行設計しません。remote computeを導入する場合も、bounded maximum runtime、explicit cleanup、artifact identity、cost visibilityを持たせます。

## Near-term AI improvement capability sequence

AIを継続的に強化するための主要dependencyは、固定Phase番号ではなくpartial orderとして捉えます。

```text
execution sources / objective records / AI analysis
                    |
                    v
             offline analysis
                    |
                    v
        weakness / hypothesis selection
                    |
                    v
        bounded candidate creation
                    |
                    v
          multi-fidelity evaluation
                    |
                    v
          high-fidelity evidence
                    |
                    +--------------------> repeat
```

次のsupporting capabilitiesは、このloopへdata source、observation capability、debugging capabilityを供給します。すべてが完了するまでPolicy改善を止める意味ではありません。

### Resilient live participation

RiichiLab等のlive environmentへ安全に繰り返し参加し、disconnect / transient failureからsafeにre-participateできるexecution capabilityをArena execution / observation側で発展させます。

このcapability自体はPolicy strengthを直接変更せず、real-world opponent distributionからobjective execution dataを取得する入口として扱います。

### Policy decision observability

objective execution observationとPolicy-internal analysisを分離します。

```text
objective execution
    what happened

Policy decision / analysis
    what was selected / computed
```

Policy-owned semanticsをraw execution recordへ暗黙に混在させず、observer追加によってprivileged informationをPolicy decision pathへ逆流させません。

### Consumer-driven replay / analysis boundary

historical replayやdecision inspectionに必要なpersisted data / correlation semanticsは、具体的consumer requirementから抽出します。

project-wide canonical `GameEvent` / `GameRecord` / global decision IDを先行発明せず、live observationとhistorical replayの差分を実consumerから学びます。

### First-party engine execution

`lisjong-engine` をecosystem自身で制御できるdeterministic execution substrateとして利用し、lisjong Policyと接続できるpathを発展させます。

これはRiichiEnv等のexternal backendを置き換える方針ではありません。external backendとfirst-party engineを用途別に併用し、複数pathから共通化の必要性が確認される前にgeneric backend abstractionを作りません。

### Human Play / spectator / replay consumers

Human Play、AI spectator、persisted replay等のhuman-facing consumerは、engine / Policy / durable recordの公開boundaryを利用します。

UI都合でrule、legality、scoring、AI semanticsを再実装せず、live interactionとread-oriented analysisを必要に応じて分離します。

### Multiple execution paths convergence

RiichiEnv、first-party engine、live environment等の複数execution pathが揃った場合、それぞれの実差異を観測してから共通化境界を判断します。

future APIを推測したgeneric `GameBackend` / `EvaluationBackend` 等を先行導入しません。

## Foundation — Stable Policy contract and environment boundary

環境非依存のPolicy contractを基礎として、観測可能な状態だけから合法Actionを決定できる状態を確立します。

この基礎には次を含みます。

- `DecisionContext` を中心とした環境非依存のPolicy境界
- `InternalAction` 等のAI-side Action contract
- deterministicなPolicy実行
- Policy返却値の合法性 / semantic identity validation
- seat-visibleな情報だけをPolicyへ渡す情報境界
- 実行時に利用するPolicy / AI configurationを選択できる能力
- 外部environment固有型をPolicy contractへ漏らさない設計
- environment-facing conversionとAI decision logicを分離できる設計

このFoundationは、engine、Policy改善、Arena execution / observation、Arena evaluationの各trackが共有する契約です。個々の実行環境固有の型やprotocolは共通Policy contractへ持ち込みません。

## Engine Track — First-party game engine

`lisjong-engine` で、日本式リーチ麻雀を指定RuleSetとseedに従って開始から最終結果まで決定的に進行できる状態を目指します。

```text
domain model / deterministic wall
        -> winning / scoring / RuleSet
        -> legal actions / round state transition
        -> round completion / result
        -> settlement / match progression
        -> final score / rank
        -> deterministic full-game driver
```

engine完成はAIの強さを条件としません。合法手生成、状態遷移、和了・点数、round / match進行、最終結果の正しさを優先します。

`lisjong-engine` はPolicyを所有せず、callerが選択した合法Actionを適用してgameを進行する責務に集中します。external-agent integrationやevaluation orchestrationもengineへ持ち込みません。

## Policy Track — AI decision core

`lisjong` では、観測可能な情報だけからActionを選ぶAI decision semanticsを所有します。

長期的には次の複数軸を統合します。

```text
Structural efficiency
  shanten / ukeire / lookahead
             |
             +-------------------+
             v                   v
Hidden-state inference      Value estimation
  remaining tiles            offensive value
  HandBelief                 defensive risk
             |                   |
             +---------+---------+
                       v
              Value / utility-aware
                    decision
```

主な能力:

- 向聴数、受け入れ枚数、lookahead等のstructural efficiency
- remaining tile information、HandBelief等のhidden-state inference
- 打点・offensive score potential
- 守備・安全度・defensive risk
- 立直判断、鳴き判断、押し引き
- offensive / defensive valueを統合したvalue-aware decision
- 将来の順位価値やgame-level objectiveを含み得るutility-aware decision
- stable production Policy / inference semantics

Expected valueは重要な候補ですが、単一の局収支EVをproject-wideな最終目的関数として固定しません。

hand-crafted Policyはlearned Policy導入後も、baseline、teacher、regression reference、component comparatorとして利用できます。

## Arena Execution / Observation Track

`lisjong-arena` では、lisjongをconcrete environmentへ接続し、executionを観測・記録する能力をevaluationから分離したlayerとして発展させます。

```text
external / local environment
          |
          v
execution / observation
          |
          +--> Policy-visible projection -> lisjong Policy
          |
          +--> raw execution data
```

主な能力:

- environment-specific integration
- external / local runner / client
- RiichiLab等へのlive participation
- matchmaking / session lifecycle
- repeated / continuous participation
- retry / backoff等のexecution resilience
- execution profile / credential source resolution
- protocol trace
- raw game record / objective event取得
- environmentへ実際に送信・適用したActionの記録
- Policy contractとexternal Observation / legal Actionのsafe conversion

このtrackはAI判断ロジック、麻雀rule、Policy performance metric、comparison protocolを所有しません。

raw execution dataとAI意味付けを分離します。privileged offline / ground-truth dataをonline Policy inputへ混ぜず、Policy-internal analysisをobjective raw recordへ暗黙に混在させません。

## Arena Research / Evaluation Track

bounded experimentのownerがArenaである場合、experiment-localなdataset construction、training harness、diagnostic analysis、candidate artifact、controlled evaluationを、stable production Policy semanticsと分離して扱えます。

```text
raw / retained evidence
        ↓
experiment-local feature / dataset
        ↓
bounded training / candidate
        ↓
diagnostic / evaluation
        ↓
validated research result
        ↓
justified stable semantics only -> owning repository
```

experiment-local modelやfeatureが存在することだけで、`lisjong`のstable production schemaへ昇格させません。production integrationが必要になった時点でowner repository、dependency、artifact delivery、runtime semanticsを改めてlockします。

Evaluation layerはPolicy / game performanceを再現可能な条件で比較・評価します。multi-fidelity ladderを利用できますが、specific protocol、seed / seat rotation、sample size、metric、confidence interval、artifact schemaは`lisjong-arena`側の正本に委ねます。

### Development evidence

state / decision / round-levelのcheap evidenceは、rapid feedback、regression detection、failure diagnosis、candidate filteringに利用します。

### Hanchan / external evidence

総合的なgame performance、順位状況、親番、連荘、南場、オーラス等を含むstrength claimにはhanchan-level evidenceを利用します。

Mortal等のexternal competitorは重要なreference / benchmarkになり得ますが、external benchmarkをcandidate searchの唯一の評価器にはしません。

## Evaluation quality layers

component correctness、Policy decision quality、game performanceは異なる評価対象として扱います。

```text
Component quality
       -> Decision quality
       -> Game performance
```

例えばHandBelief accuracyの向上と、HandBelief-aware Policyの対局成績向上は別の主張です。

概念的なownershipは次の通りです。

```text
Stable AI component semantics
    -> component owning repository

Experiment-local measurement / training
    -> bounded research owner repository

Policy / game evaluation
    -> lisjong-arena evaluation

Live / standalone participation
    -> lisjong-arena execution / observation
```

## Visualization / Analysis Track

長期的なecosystem能力として、対局状況・牌譜・AI意思決定過程を観察、再生、分析できる状態を目指します。

```text
Game / analysis data
        |
        +-> replay
        +-> live spectator
        +-> Policy analysis overlay
        +-> decision debugging
```

Visualization / Analysisはread-orientedなconsumerとして位置付けます。

共通 `GameEvent` 等のproject-wide canonical event schemaを先に発明せず、first-party engine、external environment、Arena execution / observation、durable records等の具体的sourceとconsumer requirementからadapter / normalization boundaryを抽出します。

## Learning Policy

Learning Policyは、再現可能なPolicy comparisonと安全なPolicy-visible input boundaryが成立した段階から導入・発展させます。

既存のPolicy contract、execution / observation、round / hanchan evaluation、artifact、replay / analysisを可能な限り再利用し、学習Policyだけを特別扱いする別系統の実行基盤を作りません。

初期Learned Policyでは、teacher agreementやtraining lossだけをstrength claimにしません。cheap hand-progression / decision diagnostics、partial environment、round-level evidenceを利用して基本能力を確認し、必要なcandidateだけhanchan evaluationへ進めます。

学習アルゴリズム、model形式、training data、self-play方式、計算基盤をproject-wideに先固定しません。reference-guided principle inventoryとlisjong evidenceから、次に切り分ける価値が高い軸を選択します。

## OSS / external ecosystem and validation strategy

成熟した外部実装は、用途を区別した上で積極的に評価・利用します。

- **Reference**: correctness comparison / design principle extraction
- **Backend**: computation / simulation / evaluation
- **Benchmark**: external agent / environment
- **Tooling**: replay / visualization / development support

external benchmark、simulation、game execution、protocol interoperability等に必要な能力を成熟したOSSが既に提供する場合は優先利用を検討します。同等機能をecosystem内で無目的に再実装しません。

一方、外部OSS固有の型・API・内部設計をproject-wide stable contractへ直接漏らしません。

correctnessとperformanceは概ね次の順で扱います。

```text
Correctness baseline
        -> independent validation
        -> regression protection
        -> performance instrumentation
        -> actual bottleneck identification
        -> algorithmic / duplicate-work optimization
        -> backend optimization or replacement
```

## Runtime / compute extraction trigger

独立runtime、remote executor、AWS、distributed system等は先行作成しません。

次のようなconcrete requirementが成立した場合に再検討します。

- operator-local executionがactual iteration bottleneckになった
- evaluation / generation runtimeがbounded hosted jobへ移す価値を持つ
- candidate artifact delivery / retentionを共通化するconcrete needが生じた
- 24/7 production hostingが必要になった
- Arena外の複数consumerが同じruntimeを必要とする
- self-play throughputがselected research stepのblockerになった

抽出・scaleは実consumer / measured bottleneckを根拠に行います。

## Ordering constraints

本ロードマップ上で重要な順序制約は、直列Phase番号ではなく次の依存関係として扱います。

- Policy contractはPolicy改善と複数環境接続の基礎になる
- correctness baselineを確立し、独立validationとregression protectionを行ってから性能最適化へ進む
- Engine TrackはAIの強さとは独立して正しさを完成させる
- Policy TrackとArena execution / observationはfirst-party engine完成前でも進められる
- Arena Evaluation Trackの初期比較はfirst-party engine完成前でも進められる
- new research candidateは可能な限りone changed axisで設計する
- candidate evaluationは最小十分なcheap evidenceから開始し、必要なcandidateだけhigher fidelityへ進める
- cheap proxyはNorth Starを置き換えず、proxy自体のpredictive valueも継続評価する
- overall strength claimではfixed-protocol hanchan performanceを重視する
- external benchmarkはgame-level strategy完成を必須gateとしない
- Learning Policyは再現可能なPolicy comparisonを再利用し、teacher agreementだけをstrength claimにしない
- HandBelief component qualityとdecision value / game strengthを分離する
- remote execution / AWS / self-play scaleはmeasured bottleneckが出てから具体化する
- generic backend / runtime abstractionは複数の実execution pathやconcrete consumerが揃ってから判断する
- Visualization / Analysisの共通境界は具体的なsource / consumer requirementsから抽出する
- LLMは高頻度small loopのhard dependencyにせず、research judgmentを担うlarger loopで必要に応じて使う

## What does not belong here

次の情報は本書では管理しません。

- 現在作業中のIssue番号
- Issueごとのacceptance criteria
- PRの状態
- 「次は#XX」のような直近作業順序
- release日程の推測
- 未確定な内部APIやpackage構成
- 特定OSSのversionや具体adapter / wrapper仕様
- AABB / ABBB等の具体的なevaluation protocol
- individual experimentのseed / threshold / artifact identity
- specific candidateの現在のclassification
- 未検証のproject-wide canonical event schema
- 既存Adapter / runner / trace contractのmigration進捗

具体的なPolicy、reference system、evaluation gateを例示することはありますが、current implementation statusやexact protocolは各repository / Issueを正本とします。

## Updating this roadmap

本書は、個別Issueが完了するたびには更新しません。

ecosystem全体の到達目標、track間の依存関係、AI strengthening loop、evaluation strategy、research governance、repository責務境界、long-term execution / compute strategyなど、**方向性が変わった場合に更新します**。
