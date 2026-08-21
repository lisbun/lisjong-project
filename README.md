# lisjong-project

Project-wide architecture, repository boundaries, and roadmap for the lisjong ecosystem.

## 目的

`lisjong-project` は、lisjong ecosystem 全体の設計・repository責務・依存方向・長期ロードマップを管理する documentation / project coordination repository です。

実装コードは原則として持ちません。`lisjong`、`lisjong-engine`、`lisjong-arena` のいずれかを親repositoryとして扱うものでもありません。

## 正本の分担

```text
lisjong-project
    プロジェクト横断architecture
    repository責務
    repository間依存方向
    長期ロードマップ
    横断的な設計判断

各実装repositoryのdocs/architecture.md
    そのrepository内部のarchitecture

GitHub Issues / PRs
    現在の作業内容
    acceptance criteria
    進捗・完了状態
```

GitHub上で確認できる現在進捗を本repositoryの文書へ重複して記録しません。

## Repository

| Repository | 主な責務 |
| --- | --- |
| [`lisjong`](https://github.com/lisbun/lisjong) | 麻雀AI decision core。Policy、Policy contract、AI判断ロジック、推論・評価component |
| [`lisjong-engine`](https://github.com/lisbun/lisjong-engine) | 日本式リーチ麻雀のルール、状態遷移、合法手、game / match進行 |
| [`lisjong-arena`](https://github.com/lisbun/lisjong-arena) | lisjongのexternal execution / observationと再現可能なPolicy評価。environment integration、対局記録、matchup、seed、seat rotation、結果収集、metrics |

`lisjong-arena` 内では、environmentへの接続・対局実行・raw observation取得を担う execution / observation と、comparison protocol・metrics・artifactを担う evaluation を別責務として扱います。

詳細な責務境界と依存方向は [Architecture](docs/architecture.md) を参照してください。
長期的な能力ロードマップは [Roadmap](docs/roadmap.md) を参照してください。

## Issue placement

新しい機能や設計課題をどのrepositoryへ置くか迷った場合は、まず [Architecture](docs/architecture.md) の責務境界を基準に判断します。

repository境界そのものを変更する提案や、複数repositoryへまたがる設計判断は `lisjong-project` で扱います。
個別repository内部の実装・設計・進捗は、それぞれのrepositoryで管理します。

## Architecture Decision Records

長期的な影響があり、後から理由を再確認する価値がある横断的な設計判断だけを `docs/decisions/` にADRとして残します。

- [ADR 0001: Repository boundaries](docs/decisions/0001-repository-boundaries.md)
- [ADR 0002: External execution and observation ownership](docs/decisions/0002-external-execution-observation-ownership.md)

## License

このrepositoryの文書は [MIT License](LICENSE) の下で公開します。
