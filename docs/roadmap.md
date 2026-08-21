# Roadmap

## 目的

本書は lisjong ecosystem 全体として、どの能力をどのような依存関係で成立させたいかを示す長期ロードマップです。

本ロードマップは、すべての能力を上から順番に完了させる直列Phase一覧ではありません。基礎契約を共有した上で、game engine、Policy、self-integration、Policy comparison、external benchmark、Visualization / Analysisをそれぞれの責務境界で並行して発展させ、必要な地点で統合します。

個別Issueの実装順序、現在の進捗、完了状態は各repositoryのGitHub Issues / PRsを正本とします。本書にはIssue番号や完了checkboxを原則として持ちません。

## Roadmap principles

- 接続可能性と正しさを、AIの強さより先に確立する
- 麻雀ルール、AI判断、lisjong自身のself-integration、比較評価、分析・可視化を明確な責務境界で分離する
- deterministicな実行条件を早期に確立し、後続の回帰testと比較に利用する
- 外部環境固有の都合をPolicyやengineの内部契約へ漏らさない
- 成熟したOSSや外部実装はreference / backend / benchmark / toolingとして積極的に評価・利用する
- 成熟したOSSがgame executionやprotocol interoperabilityを既に提供する場合は、同等機能の重複実装を避けて優先利用する
- stable public contract、Policy semantics、repository responsibility、project固有artifact contract、cross-repository dependency directionはlisjong ecosystem側で所有する
- correctnessを確立し、独立validationとregression protectionを行った後に性能を計測し、実測されたbottleneckを最適化する
- 評価対象に対して最小十分なevaluation scopeを選び、必要になった時点でより高コストなscopeへ拡張する
- engine完成をPolicy改善や初期Arena開始の不要な前提にしない
- 比較可能な対象と再現可能なgame実行が揃う前にArenaを過剰構築しない
- 実際の複数execution pathやconcrete consumerが揃う前に、将来backend / runtimeを推測した汎用abstractionを先行設計しない
- external competitor integrationはまず具体的なevaluation consumerの内側で始め、Arena外の複数consumerから必要になった時点で共通runtime抽出を再検討する
- Visualization / Analysisのためにproject-wide canonical event schemaを先行設計せず、具体的consumer requirementsから必要なadapter / normalization boundaryを抽出する
- 学習Policyは、手書きPolicyを再現可能に比較できる基盤が整ってから導入し、既存の評価・分析基盤を可能な限り再利用する

## How to read this roadmap

長期的な能力の発展は、概ね次の構造で捉えます。

```text
                    Foundation
              Policy / integration contract
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
      Engine Track   Policy Track    Arena Track
                         |
                  Self-integration
                         |
          +--------------+--------------+
          |                             |
          v                             v
   standalone / live              evaluation
      participation        development / benchmark

          Engine / Policy / Arena convergence
                         |
                         v
                  Learning Policy

Concrete game / analysis data
          |
          v
Visualization / Analysis Track
  replay / live / analysis / debugging
```

Engine / Policy / Arenaの各trackは互いの完成を不必要に待たず進めます。`lisjong` のself-integrationは、PolicyをRiichiEnv / RiichiLab / 将来のfirst-party engine等へ接続し、Arenaなしでもlisjong自身を実環境で動作させる能力として発展させます。

Arenaはその能力を評価対象として利用できますが、lisjong自身のlive / standalone participationを所有しません。Visualization / Analysisもread-orientedなconsumer能力として、具体的なdata sourceとconsumer requirementが成立した地点から発展させます。

## Foundation — Stable Policy and integration contract

環境非依存のPolicy contractを基礎として、観測可能な状態だけから合法Actionを決定し、その結果を実行環境側の合法Actionへ安全に対応付けられる状態を確立します。

この基礎には次を含みます。

- `DecisionContext` を中心とした環境非依存のPolicy境界
- deterministicなPolicy実行
- Policy返却値の合法性validation
- 外部環境とPolicyを分離するAdapter / integration境界
- seat-visibleな情報だけをPolicyへ渡す情報境界
- 実行時に利用するPolicy / AI configurationを選択できる能力
- 複数環境へ同じPolicy contractを接続できる設計
- environment-facing integrationとstandalone runner / clientを必要に応じて分離できる設計

このFoundationは、engine、Policy改善、self-integration、Arenaの各trackが共有する契約です。個々の実行環境固有の型やprotocolは共通Policy contractへ持ち込みません。

## OSS / external ecosystem and validation strategy

成熟した外部実装は、用途を区別した上で積極的に評価・利用します。

- **Reference**: correctness comparison / differential validation
- **Backend**: computation / simulation / evaluation
- **Benchmark**: external agent / environment
- **Tooling**: replay / visualization / development support

external benchmark、simulation、game execution、protocol interoperability等に必要な能力を成熟したOSSが既に提供している場合は、それを優先的に評価・利用します。同等の麻雀ルール、protocol変換、legal-action mapping、session補助等をecosystem内で無目的に再実装しません。

一方、外部OSS固有の型・API・内部設計は、project-wideなstable contractへ直接漏らしません。具体的なversion、adapter仕様、dependency詳細は、それを所有するrepository / Issueを正本とします。

correctnessとperformanceは次の順序で扱います。

```text
Correctness baseline
        -> independent validation
        -> regression protection
        -> performance instrumentation
        -> actual bottleneck identification
        -> algorithmic / duplicate-work optimization
        -> backend optimization or replacement
```

複数実装をdifferential validationへ利用する場合は、実装系譜・algorithm・backend等が十分に独立していることを確認します。同一backendを薄くwrapした複数実装を独立referenceとして数えません。

また、独立した複数実装が一致することはcorrectnessの強い証拠になり得ますが、agreement自体をproof of correctnessとは扱いません。差異が生じた場合も単純な多数決をoracleとせず、semantic difference、rule configuration、bug、unsupported case等を調査します。

Python実装であることだけを理由にnative化を先行せず、semantic compatibility / correctness / measured performanceを確認してbackend採否を判断します。readableなfirst-party implementation、external reference、高速backendは、必要に応じて併存可能とします。

## Engine Track — First-party game engine

`lisjong-engine` で、日本式リーチ麻雀を指定RuleSetとseedに従って開始から最終結果まで決定的に進行できる状態を目指します。

主な能力は次の順序関係を持ちます。

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

`lisjong-engine` はPolicyを所有せず、callerが選択した合法Actionを適用してgameを進行する責務に集中します。Mortal / MJAI等のexternal-agent integrationやevaluation orchestrationもengineへ持ち込みません。

## Policy Track — Hand-crafted Policy evolution

`lisjong` では、単純で決定的なPolicyを基準として、観測可能な情報だけからより良いActionを選択する能力を段階的に追加します。

長期的には、単一軸の牌効率改善ではなく、次の複数軸を統合するPolicyへ発展させます。

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

主な能力には次を含みます。

- 向聴数、受け入れ枚数、lookahead等のstructural efficiency
- remaining tile information、HandBelief等のhidden-state inference
- 打点・offensive score potential
- 守備・安全度・defensive risk
- 立直判断、鳴き判断、押し引き
- offensive / defensive valueを統合したvalue-aware decision
- 将来の順位価値やgame-level objectiveを含み得るutility-aware decision

Expected valueは中心的な候補ですが、単一の局収支EVをproject-wideな最終目的関数として固定しません。将来の順位条件やgame-level objectiveに応じて、より適切なutilityを扱える余地を残します。

新しい能力を既存Policyへ無条件に上書きするのではなく、比較可能なPolicy世代を残し、実行時に利用するPolicy / AI configurationを選択できる設計を優先します。具体的なclass名、profile形式、algorithm、内部representationはproject-wide roadmapでは固定しません。

機能追加そのものではなく、既存Policyに対してどのような改善・退行を生んだかをArenaで評価可能にすることを重視します。このtrackは `lisjong-engine` の完成を待つ必要はありません。

## Self-integration — lisjong自身を実環境で動かす

`lisjong` はPolicy libraryだけでなく、自分自身を対局環境へ接続して動作させる能力を持ちます。

概念的には次の2層を区別します。

```text
environment-facing self-integration
        ↓
Policy / AI configuration
        ↓
standalone runner / client
```

environment-facing self-integrationは、1 seat分のexternal Observation / legal ActionとlisjongのPolicy contractを安全に接続する能力です。standalone runner / clientは、実際のenvironment / session lifecycleを管理してlisjong自身を参加させます。

現在の具体経路として、RiichiEnvでのローカル実行とRiichiLabへのオンライン参加を維持します。将来 `lisjong-engine` が実execution environmentとして成立した場合も、同じPolicy contractをself-integration経由で利用できる状態を目指します。

Arenaはこのself-integrationを評価対象として利用できますが、RiichiLab等へのlisjong自身のlive / standalone participationを所有しません。

## Arena Track — Reproducible Policy evaluation

`lisjong-arena` を、Policy / game performanceを再現可能な条件で比較・評価する独立repositoryとして発展させます。

Arenaには目的の異なる少なくとも2つのevaluation laneを持たせます。

```text
Arena evaluation
    |
    +-- Round-level development evaluation
    |       rapid feedback / many samples
    |       self-improvement / regression detection
    |
    +-- Game-level / external benchmark
            overall game performance
            external competitor comparison
```

上位原則は、**評価対象に対して最小十分なevaluation scopeを選ぶこと**です。2つのlaneは代替関係でも厳格な直列gateでもありません。

### Lane 1 — Round-level development evaluation

局内decision qualityを主対象とする現在の強化段階では、局単位の低コスト評価を高速feedback loopの中心に置きます。

概念的には次のような性格を持ちます。

```text
fixed / controlled seeds
        ×
round-level execution
        ×
many trials
```

主な目的:

- 小さなPolicy変更の高速比較
- regression検出
- Policy世代間比較
- game-level strategyが未成熟な段階での自己強化
- failureを局単位で分析・再実行すること

向聴、受け入れ、lookahead、HandBelief、offensive value、defensive risk、鳴き等の局内能力を強化する段階では、このlaneを中心に利用します。

AABB / ABBB等の具体protocol、seed / seat rotation contract、sample size、metrics、confidence interval等は `lisjong-arena` 側の正本へ委ねます。

### Lane 2 — Hanchan-level external benchmark

成熟した外部AIに対するlisjongの総合的なgame performanceを確認するexternal benchmarkでは、半荘等のgame-level scopeを基本候補とします。

現在の初期external competitor候補はMortalです。Mortalとのbenchmarkでは大量の1局対戦を主要protocolとはせず、概念的には次の経路を優先検討します。

```text
                     lisjong-arena
                          |
                      RiichiEnv
                     /         \
                    v           v
            lisjong seat    Mortal seat
                |               |
                v               v
             lisjong          Mortal

        hanchan-level evaluation
        + controlled seat rotation
```

RiichiEnvが提供するMortal / MJAI interoperability、game execution、legal Action mapping等を利用できる場合は、それを優先します。同等のMJAI event generator / parser / legal-action mapper / game progressionをlisjong ecosystem側で再実装することを前提にしません。

これはRiichiEnvやMortalをproject-wide永久contractへ固定するものではなく、現在の優先execution / benchmark pathです。具体version、wrapper API、process方式、sample size、statistical protocol等は `lisjong-arena` のconcrete Issueで決定します。

external benchmarkでは将来的に、点棒状況、順位条件、親番、連荘、南場、オーラス、トップ取り、ラス回避、game-level utility等を含めた総合的な意思決定を評価できます。

ただしMortal benchmarkはgame-level strategy完成後にしか実施できないものとはしません。Policy成熟度を測るexternal referenceとして開発途中でも実施でき、game-level strategyの実装進展に伴ってbenchmarkの意味がより豊かになると捉えます。明確なdiagnostic目的があればMortalとのround-level comparisonを行うこと自体も禁止しません。

### Arena execution pathの使い分け

既存のPolicy-vs-Policy development evaluationでは、`lisjong` の既存integration / standalone executionを利用する経路を維持できます。

```text
lisjong-arena
      ↓
   lisjong
      ↓
  RiichiEnv
```

mixed-agent external benchmarkでは、ArenaがRiichiEnv等のOSS execution environmentを直接orchestrateしてよいものとします。ただしlisjong側のObservation / legal Action / selected Action mapping semanticsをArenaへ複製せず、environment-facing self-integrationを可能な限り再利用します。

Mortal等のexternal competitor wrapperは、まずArena-private implementation detailとして開始します。Human Play、Arena外のMortal execution、generic external process hosting等、Arena外の複数concrete consumerが成立した場合にのみ共通runtime / repositoryへの抽出を再検討します。

初期Arenaは引き続き `lisjong-engine` の完成を開始条件にしません。deterministic reproducibilityとstatistical strength claimは別の主張として扱い、再実行可能であることだけを強さの統計的証明とはみなしません。

ArenaはPolicy判断ロジック、麻雀ルール、単一gameの状態遷移を再実装しません。

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
Component validation
    -> component owning repository

Policy / game evaluation
    -> lisjong-arena

External benchmark for evaluation
    -> lisjong-arena

Live / standalone participation of lisjong itself
    -> lisjong self-integration
```

component-specific correctness / calibration testを `lisjong-arena` へ無理に集約しません。

## Convergence — Policy x first-party engine

`lisjong-engine` が実際にgameを完走できる実行環境として成立した段階で、`lisjong` にengine向けAdapter / integrationを追加し、既存Policy contractをfirst-party engineへ接続します。

目標となる構造は次です。

```text
                 lisjong Policy
                       |
          +------------+------------+
          |            |            |
          v            v            v
      RiichiEnv    RiichiLab   lisjong-engine
```

環境ごとの差異はself-integration側で吸収し、Policy判断ロジックやengineのルール実装へ外部protocol固有の型や状態を持ち込みません。

このConvergenceでは、`lisjong -> lisjong-engine` の依存方向を維持し、`lisjong-engine` から `lisjong` へ逆依存させません。

## Convergence — Arena x multiple game environments

first-party engine integrationが成立した後は、Arenaから既存のRiichiEnv経路に加えて `lisjong-engine` 経路も利用できる状態を目指します。

概念的には次の形です。

```text
                       lisjong-arena
                            |
                            v
                          lisjong
                       /            \
                      v              v
                 RiichiEnv     lisjong-engine
```

RiichiEnvと`lisjong-engine`という複数の実execution pathが揃った時点で、それぞれの差異を実測し、共通化すべき境界が本当に存在するかを判断します。

それ以前に、将来APIを推測した汎用 `GameBackend` / `EvaluationBackend` 等のabstractionは導入しません。mixed-agent benchmarkのためにArenaが外部environmentを直接利用できることも、汎用backend abstractionを先行導入する理由にはしません。

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

Visualization / Analysisはread-orientedなconsumerとして位置付けます。Human Playは別の将来能力とし、Visualization Trackの必然的な最終段階とは規定しません。

Visualization / Analysisを特定のgame backendやArenaへ従属させません。first-party engine、external environment、live integration等のconcrete sourceから得られるobjective events / snapshotsや、Policy固有のdecision / analysis data、Arenaのevaluation result / provenance / artifactを、必要に応じてanalysis consumerへ接続できる境界を目指します。

ただし、共通 `GameEvent` 等のproject-wide canonical event schemaを先に発明しません。RiichiLab / RiichiEnv / `lisjong-engine` / future consumers等の具体的requirementsを実際に扱った上で、必要なadapter / normalization boundaryを抽出します。

長期的には、live対局観戦、保存牌譜再生、regression局の再生、candidate / baseline比較、Policy decision analysis、Policy evolutionの追跡等へ発展できる余地を残します。具体的なviewer repository、GUI framework、protocol、OSS採用はこのroadmapでは固定しません。

## Learning Policy

手書きPolicyとcomparison protocolが十分に安定し、Policy改善を再現可能に評価できる状態になった後、学習Policyを導入します。

```text
Strong hand-crafted baseline
          +
Reproducible evaluation
          +
External validation
          +
Replay / analysis
          -> Learning Policy
```

この段階では、既存のPolicy contract、component validation、round / game-level evaluation、external benchmark、replay / analysisを可能な限り再利用し、学習Policyだけを特別扱いする別系統の実行基盤を不用意に作りません。

hand-crafted Policyもbaseline、regression reference、feature / estimator comparison対象、explanation referenceとして継続利用できるようにします。

学習アルゴリズム、model形式、training data、self-play方式、計算基盤は、その時点の要件と実測に基づいて設計し、本ロードマップでは先に固定しません。

## Ordering constraints

本ロードマップ上で重要な順序制約は、直列Phase番号ではなく次の依存関係として扱います。

- Policy contract / integration boundaryはPolicy改善と複数環境接続の基礎になる
- correctness baselineを確立し、独立validationとregression protectionを行ってから性能最適化へ進む
- Engine TrackはAIの強さとは独立して正しさを完成させる
- Policy Trackとself-integrationはfirst-party engine完成前でも進められる
- Arena Trackの初期比較はfirst-party engine完成前でも進められる
- 現在の局内decision改善では低コストなround-level evaluationを高速iterationの中心にする
- external benchmarkでは評価したいgame-level semanticsに応じて半荘等のscopeを選択する
- Mortal等のexternal benchmarkはgame-level strategy完成を必須gateとしない
- 高度な統計機能は最小comparison protocolによる実データを得てから追加する
- first-party engine integrationはengineが実execution pathとして成立してから行う
- generic backend / external-player runtime abstractionは複数の実execution pathやconcrete consumerが揃ってから判断する
- Visualization / Analysisの共通境界は具体的なsource / consumer requirementsから抽出する
- Learning Policyは再現可能なPolicy comparisonが成立してから導入し、既存の評価・分析基盤を再利用する

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
- Mortal benchmarkの具体的sample size / statistical threshold
- 未検証のproject-wide canonical event schema

現在の長期的なpreferred execution / benchmark pathとしてOSS名を示すことはありますが、そのversionや具体integration contractは各consumer repository / Issueを正本とします。

## Updating this roadmap

本書は、個別Issueが完了するたびには更新しません。

ecosystem全体の到達目標、track間の依存関係、Convergenceの開始条件、repository分離や責務境界、評価戦略、長期的なpreferred execution pathなど、方向性が変わった場合に更新します。
