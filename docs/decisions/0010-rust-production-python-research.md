# ADR 0010: Rust production core and Python research

- Status: Accepted（project #87、2026-10-03のユーザー承認済み方針を文書化）
- Recorded: 2026-10-06
- Related: [project #87](https://github.com/lisbun/lisjong-project/issues/87)

## Context

向聴・batched discard・R5のnative計算を足場に、本番の対局とPolicy判断をPython runtimeなしで完結させる。学習・統計分析・実験はPythonを継続し、計算の意味を二重に実装しない。

## Decision

| Owner | Rustへ段階移行する対象 |
| --- | --- |
| lisjong | player-safe入力・Action、特徴量、牌効率・探索・belief・risk/value、Policy、本番推論 |
| lisjong-engine | ルール・合法手・状態遷移 |
| lisjong-arena | 環境アダプター・対局接続・本番runner |

言語を理由にrepository ownershipや依存方向を変えない。学習・分析は薄いPython bindingで同じRust semanticsを利用する。Arenaの評価・分析・AWS運用まで一括移植しない。

coreはPython / ML framework非依存とする。学習専用truthとonline入力を分離する。NN推論形式は採用モデルと配布要件が具体化した時に決める。

移植と戦術変更を分け、各段階で基準revision、固定入力、判断一致、時間、メモリ、障害時動作を記録する。Python版を移行中のoracleとして用い、差異を解明してから正式経路を切り替える。同じalgorithmのPython/Rust一致を独立したルール正しさの証明とはしない。

architecture上の移行を進めるために、各段階で大幅な高速化を必須とはしない。高速化・棋力向上の主張には別途実測を必要とする。R1800の比較と並行する場合は共有コードの変更を分離し、固定済み実験のrevisionを差し替えない。

## Completion and consequences

本番Policy・推論・runnerがPythonなしで動作し、Python consumerが同じsemanticsを使い、配布・実行品質が確認された時点で全体移行を完了する。最初のcore分離だけでは完了しない。

旧Python計算経路はconsumer確認後に廃止またはhistorical referenceへ隔離する。恒久的な二重保守を目標としない。稼働pin / wheelの変更、追加AWS予算、Champion昇格はこのADRから自動承認されない。
