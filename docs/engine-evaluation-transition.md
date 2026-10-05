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
