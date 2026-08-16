# Roadmap

## 目的

本書は lisjong ecosystem 全体として、どの能力をどのような依存関係で成立させたいかを示す長期ロードマップです。

本ロードマップは、すべての能力を上から順番に完了させる直列Phase一覧ではありません。基礎契約を共有した上で、game engine、Policy、Policy comparisonをそれぞれの責務境界で並行して発展させ、必要な地点で統合します。

個別Issueの実装順序、現在の進捗、完了状態は各repositoryのGitHub Issues / PRsを正本とします。本書にはIssue番号や完了checkboxを原則として持ちません。

## Roadmap principles

- 接続可能性と正しさを、AIの強さより先に確立する
- 麻雀ルール、AI判断、Policy比較を明確な責務境界で分離する
- deterministicな実行条件を早期に確立し、後続の回帰testと比較に利用する
- 外部環境固有の都合をPolicyやengineの内部契約へ漏らさない
- engine完成をPolicy改善や初期Arena開始の不要な前提にしない
- 比較可能な対象と再現可能なgame実行が揃う前にArenaを過剰構築しない
- 実際の複数execution pathが揃う前に、将来backendを推測した汎用abstractionを先行設計しない
- 学習Policyは、手書きPolicyを再現可能に比較できる基盤が整ってから導入する

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
          |              |              |
          +--------------+--------------+
                         |
                    Convergence
          Policy x first-party engine
          Arena x multiple environments
                         |
                         v
                  Learning Policy
```

3つのtrackは完全な独立作業ではありませんが、互いの完成を不必要に待たず進めます。依存が必要になる地点はConvergenceとして明示します。

## Foundation — Stable Policy and integration contract

環境非依存のPolicy contractを基礎として、観測可能な状態だけから合法Actionを決定し、その結果を実行環境側の合法Actionへ安全に対応付けられる状態を確立します。

この基礎には次を含みます。

- `DecisionContext` を中心とした環境非依存のPolicy境界
- deterministicなPolicy実行
- Policy返却値の合法性validation
- 外部環境とPolicyを分離するAdapter / integration境界
- seat-visibleな情報だけをPolicyへ渡す情報境界
- RiichiEnv、RiichiLab、将来の `lisjong-engine` 等へ同じPolicy contractを接続できる設計

このFoundationは、engine、Policy改善、Arenaの各trackが共有する契約です。個々の実行環境固有の型やprotocolは共通Policy contractへ持ち込みません。

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

`lisjong-engine` はPolicyを所有せず、callerが選択した合法Actionを適用してgameを進行する責務に集中します。

## Policy Track — Hand-crafted Policy evolution

`lisjong` では、単純で決定的なPolicyを基準として、観測可能な情報だけからより良いActionを選択する能力を段階的に追加します。

初期の牌効率能力から、必要に応じて次のような判断能力へ発展させます。

- 向聴数
- 受け入れ枚数
- 牌効率
- 打点・offensive value
- 立直判断
- 鳴き判断
- 守備・安全度
- 押し引き

新しい能力を既存Policyへ無条件に上書きするのではなく、比較可能なPolicy世代を残せる設計を優先します。

機能追加そのものではなく、既存Policyに対してどのような改善・退行を生んだかをArenaで評価可能にすることを重視します。

このtrackは `lisjong-engine` の完成を待つ必要はありません。利用可能な実行環境を使ってPolicy改善と検証を継続できます。

## Arena Track — Reproducible Policy comparison

`lisjong-arena` を、複数Policyの強さを再現可能な条件で比較する独立repositoryとして発展させます。

初期Arenaは `lisjong` の既存integrationを利用し、`lisjong-engine` の完成を開始条件にしません。

最初に成立させるcomparison foundationは次です。

- 比較対象となる複数Policy
- fixed seed set
- deterministicなseat rotation
- Policy assignmentの記録
- raw game resultの収集
- average rank / average score / 順位回数等の基本metrics
- 同一comparison protocolの再実行可能性

その後、実際の比較で必要性が確認された段階で、試行数設計、信頼区間、統計的比較、benchmark / report等を追加します。

Elo / rating、database、visualization、distributed execution等は、具体的な利用上の必要性が確認される前に基盤へ取り込みません。

ArenaはPolicy判断ロジック、麻雀ルール、単一gameの状態遷移を再実装しません。

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

環境ごとの差異はAdapter / integration側で吸収し、Policy判断ロジックやengineのルール実装へ外部protocol固有の型や状態を持ち込みません。

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

それ以前に、将来APIを推測した汎用 `GameBackend` / `EvaluationBackend` 等のabstractionは導入しません。

## Learning Policy

手書きPolicyとcomparison protocolが十分に安定し、Policy改善を再現可能に評価できる状態になった後、学習Policyを導入します。

この段階では、既存のPolicy contractとArena comparison protocolを可能な限り再利用し、学習Policyだけを特別扱いする別系統の実行基盤を不用意に作りません。

学習アルゴリズム、model形式、training data、self-play方式、計算基盤は、その時点の要件と実測に基づいて設計し、本ロードマップでは先に固定しません。

## Ordering constraints

本ロードマップ上で重要な順序制約は、直列Phase番号ではなく次の依存関係として扱います。

- Policy contract / integration boundaryはPolicy改善と複数環境接続の基礎になる
- Engine TrackはAIの強さとは独立して正しさを完成させる
- Policy Trackはfirst-party engine完成前でも進められる
- Arena Trackの初期比較はfirst-party engine完成前でも進められる
- 高度な統計機能は最小comparison protocolによる実データを得てから追加する
- first-party engine integrationはengineが実execution pathとして成立してから行う
- generic backend abstractionは複数の実execution pathが揃ってから判断する
- Learning Policyは再現可能なPolicy comparisonが成立してから導入する

## What does not belong here

次の情報は本書では管理しません。

- 現在作業中のIssue番号
- Issueごとのacceptance criteria
- PRの状態
- 「次は#XX」のような直近作業順序
- release日程の推測
- 未確定な内部APIやpackage構成

これらは各repositoryのIssues / PRs / architectureを正本とします。

## Updating this roadmap

本書は、個別Issueが完了するたびには更新しません。

ecosystem全体の到達目標、track間の依存関係、Convergenceの開始条件、repository分離や責務境界など、長期的な方向性が変わった場合に更新します。
