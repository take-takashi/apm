# apm
Microsoft APMのためのリポジトリ

## Codex global AGENTS.md

配布用の Codex 向け instruction は `.apm/instructions/*.instructions.md` で管理する。
Codex の `config.toml` 共通設定は `codex/config.managed.toml` で管理し、`config.toml` の managed block だけを更新する。
既存の `[projects.*]`、`[notice.*]`、`[marketplaces.*]` は Codex が更新する可変領域として残す。

現在のグローバル Codex 設定へ反映するには、次を実行する。

```sh
apm run install-codex-global
```

生成結果を事前確認する場合は、scratch directory を使う。

```sh
apm compile --target codex --root /tmp/apm-codex-global-preview --clean --verbose
```
