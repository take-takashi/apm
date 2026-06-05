# apm
Microsoft APMのためのリポジトリ

## Codex global AGENTS.md

配布用の Codex 向け instruction は `.apm/instructions/*.instructions.md` で管理する。

現在のグローバル Codex 設定へ反映するには、次を実行する。

```sh
apm run install-codex-global
```

生成結果を事前確認する場合は、scratch directory を使う。

```sh
apm compile --target codex --root /tmp/apm-codex-global-preview --clean --verbose
```
