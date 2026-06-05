---
name: gh-merge-pr
description: ユーザーが明示的に「gh-merge-pr」「GH Merge PR」「コミット、プッシュ、PR作成、コンフリクトなければマージ」など、この一連のGitHub公開フローをコマンドとして実行するよう依頼したときだけ使う。ローカル変更を確認し、日本語Conventional Commitsでコミットし、pushし、非Draft PRを作成し、コンフリクトがなくマージ可能ならmainへマージする。
---

# GH Merge PR

ローカル変更をレビュー可能な単位でコミットし、リモートへ push し、PR を作成して、コンフリクトがなければ main へマージする。

## 前提

- この skill は明示コマンド用。ユーザーが通常の相談をしているだけなら実行しない。
- 既存のユーザー変更を勝手に戻さない。
- 秘密情報や token の値を出力しない。
- リポジトリ固有の `AGENTS.md` がある場合は必ず従う。
- PR は Draft にしない。ただしリポジトリ指示が明示的に異なる場合はそちらを優先する。
- GitHub 操作は利用可能なら GitHub app/connector を優先し、不足する場合だけ `git` / `gh` を使う。

## 手順

1. 状態確認
   - `git status --short --branch`
   - `git diff --name-only`
   - `git diff`
   - `git remote -v`
   - `git branch --show-current`
   - main 直下での編集禁止など、リポジトリ指示に違反していないか確認する。

2. 検証
   - リポジトリの標準検証を優先する。
   - `mise.toml` があり `verify` が定義されていれば `mise run verify` を実行する。
   - `verify` が重すぎる、未定義、またはユーザーが軽量確認を求めている場合は `mise run check` などリポジトリの標準軽量検証を使う。
   - 実行できなかった検証は、理由をコミット本文と最終報告に残す。

3. コミット
   - 差分を意味のある単位に分ける。1コミットは1つの意図に対応させる。
   - 既存の無関係な変更や用途が異なる変更を混ぜない。
   - コミットメッセージは Conventional Commits 1.0.0 形式にする。
   - 本文は日本語で `変更内容:`、`検証:`、`背景:` ブロックを含める。
   - pre-commit hook は迂回しない。

4. push
   - `git push -u origin <branch>` を実行する。
   - push に失敗した場合は原因を確認し、force push はユーザーの明示許可なしに行わない。

5. PR 作成
   - base は通常 `main`。
   - title はコミット件名または変更全体の Conventional Commit 形式に合わせる。
   - body には `変更内容` と `検証` を簡潔に書く。
   - Draft ではなくレビュー可能な PR として作成する。

6. コンフリクト確認
   - GitHub の PR metadata で `mergeable` を確認する。
   - 初回取得で不明または false でも、判定待ちのことがあるため少し待って再取得する。
   - 必要なら `git fetch origin main <branch>` と `git merge-tree $(git merge-base origin/main HEAD) origin/main HEAD` でローカル確認する。
   - コンフリクト、必須チェック失敗、保護ルール、レビュー必須などがある場合はマージせず、理由を報告する。

7. マージ
   - コンフリクトなしでマージ可能なら PR を main にマージする。
   - merge method はリポジトリ設定や既存運用に合わせる。判断できない場合は通常の merge commit を使う。
   - `expected_head_sha` などが使える場合は指定して、意図しない head 移動を防ぐ。

8. 最終確認
   - PR が `merged: true` になったことを確認する。
   - `git fetch origin main` でローカルの `origin/main` を更新する。
   - 最終報告にはコミット、PR URL、merge commit、検証結果、未処理事項を含める。

## 止める条件

- 作業ツリーに対象外の dirty change がある。
- main に未反映のリモート更新があり、現在のブランチが古い。
- 検証が失敗している。
- PR がコンフリクトしている。
- 必須チェックや保護ルールによりマージできない。
- force push、履歴改変、未マージブランチ削除など破壊的操作が必要。
