# Roadmap

## 目的

本書は lisjong ecosystem 全体として、どの能力をどの順序で成立させたいかを示す長期ロードマップです。

個別Issueの実装順序、現在の進捗、完了状態は各repositoryのGitHub Issues / PRsを正本とします。本書にはIssue番号や完了checkboxを原則として持ちません。

## Roadmap principles

- 接続可能性と正しさを、AIの強さより先に確立する
- 麻雀ルール、AI判断、評価を明確な責務境界で分離する
- deterministicな実行条件を早期に確立し、後続の回帰testと評価に利用する
- 外部環境固有の都合をPolicyやengineの内部契約へ漏らさない
- 比較可能な対象と再現可能なgame実行が揃う前にevaluation基盤を過剰構築しない
- 学習Policyは、手書きPolicyを再現可能に比較できる基盤が整ってから導入する

## Capability roadmap

### 1. Policy foundation

環境非依存のPolicy contractを基礎として、観測可能な状態だけから合法Actionを決定できる状態を確立します。

ここには、単純で決定的なPolicyから始め、向聴数、受け入れ枚数、牌効率などの判断能力を段階的に追加する流れを含みます。

### 2. First-party game engine

`lisjong-engine` で、日本式リーチ麻雀を指定RuleSetとseedに従って開始から最終結果まで決定的に進行できる状態を目指します。

engine完成はAIの強さを条件としません。合法手生成、状態遷移、和了・点数、round / match進行、最終結果の正しさを優先します。

### 3. Common Policy across environments

同じPolicy contractを、RiichiEnv、RiichiLab、`lisjong-engine` 等の複数環境へAdapter / integration層を通じて接続できる状態を目指します。

環境ごとの差異はAdapter側で吸収し、Policy判断ロジックやengineのルール実装へ外部protocol固有の型や状態を持ち込みません。

### 4. Reproducible evaluation foundation

複数Policyを比較する価値が生まれ、かつdeterministicな複数対局を再現できる状態になった段階で、`lisjong-eval` の独立を検討します。

分離時には少なくとも次が成立していることを目安とします。

- 比較対象となる複数Policyが存在する
- seedを指定してgame結果を再現できる
- seat差を制御できる
- game結果を機械的に収集できる
- 単発の勝敗ではなく複数試行で比較する必要がある

### 5. Statistical Policy comparison

評価基盤では、固定されたevaluation protocolに従い、seed集合、seat rotation、試行数、metricsを記録してPolicy差を比較できる状態を目指します。

平均順位、平均得点、順位率等の基本metricsから開始し、必要性が確認された段階で信頼区間や統計的比較を追加します。

### 6. Advanced hand-crafted Policy

再現可能な比較基盤を利用しながら、立直判断、打点評価、鳴き、守備、安全度、押し引き等をPolicyへ段階的に追加します。

機能追加そのものではなく、既存Policyに対してどのような改善・退行を生んだかを評価可能にすることを重視します。

### 7. Learning Policy

手書きPolicyとevaluation protocolが十分に安定した後、学習Policyを導入します。

学習アルゴリズム、model形式、training data、計算基盤は、その時点の要件に基づいて設計し、本ロードマップでは先に固定しません。

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

ecosystem全体の到達目標、能力間の順序、repository分離の開始条件など、長期的な方向性が変わった場合に更新します。
