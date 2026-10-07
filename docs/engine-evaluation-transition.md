# Heuristic正式評価のengine移行方針

方針・残作業の正本: [project #83](https://github.com/lisbun/lisjong-project/issues/83)。本書は移行の境界を定める。Arenaのprotocol承認・実装・校正の完了を意味しない。

## 範囲と切替条件

次のHeuristic family-internal正式半荘評価は、lisjong-engine上の新protocolを目標にする。既存の開発比較は固定済みの条件で継続し、新protocolの完成を待たせない。

切替前にArenaで以下を満たす。満たさない場合は正式評価を開始せず、旧protocolへの無言のfallbackはしない。

1. engine revision・全RuleSet値・game mode・終了条件を明示し、RiichiEnv `4p-red-half`との差分を表にする。既定値への依存だけで一致を主張しない。
2. 新protocol identityを割り当てる。旧 `arena-heuristic-candidate-aabb-half-v1` へのbackend option追加で済ませない。
3. 現Championと候補の鳴き・槓・リーチ・和了・終局をPolicy bridgeで検証し、固定fixtureのPython/Rust判断・精算一致を確認する。未対応や説明不能な差があれば止める。
4. 新protocol上で固定C0との比較基準を作る。実行時間・RSS・費用を候補に対応する小規模校正で測る。開発用の分散推定と正式評価populationは分離する。
5. 狙う効果、独立なblock差の分散、検出力、標本数、主指標、判定法、費用上限を結果を見る前に固定する。

具体的なprotocol ID、RuleSet差分、block/rotation設計、閾値、artifact schemaはArena所有とし、この文書で未検証の値を埋めない。

## Championと過去証拠

現Championの既存昇格根拠は当時のRiichiEnv protocolに紐づけて保持する。移行だけを理由にChampionを取り消したり、engine上で優越確認済みと扱ったりしない。

移行後は現Championをincumbentとして、候補と同じengine protocolで比較する。C0からの累積進捗と現Championに対する昇格判断を区別する。旧protocolと新protocolのD・標本を混ぜない。

Overallの既存根拠・ADR 0005のAABB半荘・対称席配分は変更しない。Overall等の他protocolは今回の一括移行対象にせず、移す時に別のprotocol承認と根拠の整理を行う。

## 局単位記録と統計

engineの `CompletedRound` 精算を正本に、和了・放銃・ツモられ・流局収支・立直/副露・順位分布を最初から記録する。既存arena #432 / #439の重み付けと指標定義を参照し、欠損をゼロとみなさない。

副指標は診断用で、主判定の救済に使わない。局を独立標本として水増しせず、同一seedの席交代を含むblockをprotocolどおり扱う。結果を見ての標本追加、閾値変更、候補変更を行わない。

固定400半荘・旧protocolのSD・所要時間をそのまま移植しない。非劣性を採る場合は許容幅と判定法を事前固定し、単に有意差がないことを非劣性と呼ばない。承認済み月額総予算と当月既支出を確認して計画する。

## ソース確認による差分と検証の入口

仕様固定のownerは[Arena #452](https://github.com/lisbun/lisjong-arena/issues/452)。下表はソース確認であり、対局・fixture実行による適合認証ではない。

確認対象:

- [RiichiEnv v0.4.10 state](https://github.com/smly/RiichiEnv/blob/479c1faeb33d082965eef8198f63261a79c0fce3/riichienv-core/src/state/mod.rs)の `_advance_round` / `_process_end_game`。
- [engine rules](https://github.com/lisbun/lisjong-engine/blob/96b9796c76ef5db8f3968f689a1ca6f3dfc9aa3b/src/lisjong_engine/rules.py)、同revisionの `match_state.py` / `final_score.py`。
- [Arena旧protocol](https://github.com/lisbun/lisjong-arena/blob/db6e38690c0afb5879e666c561718ca0be28cfe9/src/lisjong_arena/heuristic_candidate_aabb/protocol.py)、同revisionの `pyproject.toml` / `lisjong_engine/hanchan.py`。

Arena通常pinはengine `8735e89e1aea000ab59368d0368d476787827741`、既存#385は `96b9796c76ef5db8f3968f689a1ca6f3dfc9aa3b`。この2版でrules/final_scoreは同じだがdriverは異なる。新protocolは採用revisionを明示し、通常pinの暗黙継承や#385の存在だけで対応済みとしない。

| 項目 | 旧RiichiEnv/Arena v1 | engine project-standard-v1 | 移行時の扱い |
| --- | --- | --- | --- |
| 持点・返し・順位点 | 25000開始、30000返し、uma 30/10/-10/-30、oka先頭20 | 同じ設定 | 全RuleSet値を保存し、名前だけで一致としない |
| 主指標 | 素点差/1000 + uma + oka | 最終得点に端数処理と飛び賞罰を含む | 旧式を重ねて適用しない。別endpointとして明記 |
| 端数・単位 | 素点100点刻みを0.1ptとして保持 | 既定は非首位の基礎点を0方向へ整数pt化し残差を首位へ。final_points内部1=0.1pt | 内部整数と表示ptの10倍差を検証。内訳を保存 |
| 飛び | 負点で終了。旧主指標に飛び賞罰なし | 0未満で終了、飛び賞10pt/罰-10ptの設定 | 複数受取人・複数飛びの配分もfixture化 |
| 南4以降の親終了 | 連荘・首位・30000以上で終了。途中流局は除外 | 親和了/聴牌終了有効、首位・30000以上 | 同点首位、途中流局を境界fixtureにする |
| 西4の上限 | 親流れなら終了。親連荘は上記終了条件次第で継続し得る | 西4終了で打ち切り | 明確な差。親連荘/途中流局時も確認 |
| 終了時の残供託 | 首位に加算、同点は若いseat優先 | final_riichi_stick_awardsから最終素点へ加算 | 局精算と最終供託を二重計上しない |
| 同点順位 | 終局時首位選定は若いseat優先 | final_rank_tie=SEAT_ORDER | 旧LocalGameResult全順位の経路もfixtureで確認 |
| 役・槓・複数ロン | 上流GameRuleと生成経路に依存 | 複数ロン、三家和流局、槓ドラ遅延、リーチ後槓は待ち/構成保持などを明示 | 全項目の一致は未確認。GameRuleのPython constructorとdefault_tenhouでも三家和の既定が違うため、実際の生成経路を追う |

### 最低限の検証範囲

1. **bridge**: 現Champion系列を使い、chi/pon/daiminkan/ankan/kakan、赤牌、reach/ron/tsumo/pass、複数応答、リーチ後槓を固定局面で検査する。MinimalPolicyの半荘integrationだけで完了としない。
2. **backend**: 同一PolicyInputでPython/Rustの合法手・選択・必要な補助出力を照合する。同じengineと初期条件の短い試行で局数・精算・終局理由まで確認する。異なるengine間の同seed同牌山は仮定しない。
3. **記録**: 同一実行のCompletedRoundを正本とする。既存hanchan wrapperが返す`CompletedMatch.history`には同一実行の全局の`CompletedRound`が含まれる。局精算・本場/供託・収支はこの履歴を起点とし、既存`focal_outcome_source/engine_source.py`の射影・整合検査の再利用範囲を固定する。履歴だけでは得られない立直/副露等のイベント情報が必要な指標についてのみ、必要なfactとdriver callbackの利用範囲を別に定義する。callback追加を局精算取得の前提にしない。局ID一意性・本場/供託・収支・最終精算の整合と欠損拒否、保存物だけからの再集計を検査する。
4. **protocol**: #385はfocal paired 8半荘/seedの先例であり、AABBのblock設計を承認する根拠ではない。ID、rotation、participant/artifact hash、主指標、CI、採否を#452で固定する。
5. **校正**: 実装確認後、候補に対応する時間/RSS・当月残予算・独立block分散から標本数を決める工程を別に置く。今回のソース確認だけではseed予約・正式実行へ進めない。
