---
name: apm-maintainer
description: Use when migrating, adding, updating, or reviewing local custom skills in this APM management repository. Applies when the user wants to bring an existing SKILL.md into .apm/skills, create a small reusable skill for personal agent distribution, or check that APM-managed skills follow repository policy.
---

# APM Maintainer

この skill は、このリポジトリで自作 skill を APM 管理下へ移植・追加・更新するときに使う。

## 基本方針

- 回答と作業説明は日本語で行う。
- この skill は原則として APM 管理リポジトリのルートで実行する。
- 別リポジトリで起動した場合は、移植元の調査だけを行い、書き込み先として APM 管理リポジトリを明示確認してから作業する。
- 自作 skill の配布元は `.apm/skills/<skill-name>/SKILL.md` に置く。
- `.agents/skills/` は APM が展開する利用先として扱い、直接編集しない。
- skill 名とディレクトリ名は lowercase kebab-case にする。
- 外部 repo からの取り込みは、ユーザーが明示したものだけ扱う。
- MCP server / plugins は追加しない。hooks はユーザーの明示依頼がある場合だけ検討する。
- 秘密情報、個人 PC 固有の絶対パス、token、credential を skill に含めない。

## 移植ワークフロー

1. 移植元を確認する。
   - ユーザーが指定したパス、URL、既存 skill 名を確認する。
   - ローカルに存在する場合は `SKILL.md` と必要な bundled resources だけ読む。
   - 外部 URL の場合は source repo、ref または commit、対象パスを確認し、未承認の依存として扱わない。

2. skill の責務を絞る。
   - 何をしたいときに発火する skill かを一文で定義する。
   - 既存 skill と責務が重なる場合は、統合・改名・採用見送りのどれが妥当か説明する。
   - 汎用 skill と project-local skill を混ぜない。

3. frontmatter を整える。
   - `name` はディレクトリ名と一致させる。
   - `description` は「いつ使うか」が分かる trigger 条件を含める。
   - client 固有の metadata は、必要になるまで追加しない。

4. 内容を移植する。
   - `.apm/skills/<skill-name>/SKILL.md` を作成または更新する。
   - 手順は具体的に書き、長い背景説明や README 的な説明を入れない。
   - 必要な `scripts/`、`references/`、`assets/` だけを同じ skill ディレクトリ配下に置く。
   - bundled resources を含める場合は、`SKILL.md` からいつ読む・使うかを明記する。

5. APM 管理との整合性を確認する。
   - `apm.yml` がある場合は、`.apm/` 配下の local content が配布対象として妥当か確認する。
   - APM が配置した `.agents/`、`.github/`、`.claude/` などのファイルは直接編集しない。
   - `apm.lock.yaml` は手編集しない。

6. 検証する。
   - YAML frontmatter が構文として壊れていないか確認する。
   - `git diff` で秘密情報、不要な絶対パス、キャッシュ、生成物が混ざっていないか確認する。
   - `apm.yml` と `apm.lock.yaml` が存在する場合は、可能な範囲で `apm install --frozen` と `apm audit --ci` を実行する。
   - 初回 lockfile 作成や依存更新が必要な場合は、通常 install / update を実行する前にユーザーへ説明する。

## 外部 skill を参考にする場合

- 外部 skill の導入は allowlist 方針に従う。
- `mattpocock/skills` は allowlist 済み候補だが、採用対象は AGENTS.md に列挙された skill に限る。
- 外部 skill をそのまま使う場合は APM dependency として扱い、原則 vendoring しない。
- 外部 skill を改変して自作化する場合は、元 repo、参照 commit、変更理由を作業報告に書く。
