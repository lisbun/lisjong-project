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

## 2026-10-22の補足決定：重複計算backendの段階的廃止

[lisjong #328](https://github.com/lisbun/lisjong/issues/328)に記録した方針により、上記のPython oracleは移行中の比較手段であり、恒久的な正本ではない。重複計算・consumer・fallback・CI・対応wheelを棚卸しし、仕様から決めた期待値、境界条件、性質・不変条件、binding契約へ検証を移す。Rust出力をそのまま期待値の正当化に使わない。

対応環境を整備してRustを必須化し、利用不能時は明示的に失敗する。検証を移した処理からPython計算実装と比較専用CIを削除する。PythonのPolicy・学習・実験制御・bindingを一括廃止する決定ではない。Arena側の評価gateも明示的に整合させ、過去artifactのverifyへの影響を確認する。

固定済み校正のPython/Rust照合は当初計画のまま保持し、以後の評価でPython全対局比較を毎回一律には要求しない。移行期の追加照合は影響範囲と既存証拠で判断する。seed管理・P3・候補固有の未確認事項・配布provenanceの確認は維持する。
