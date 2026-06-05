---
name: git-cleanup-main
description: ユーザーが明示的に「git-cleanup-main」「Git Cleanup Main」「クリーンナップしてmainをpull」など、このGitクリーンナップ手順をコマンドとして実行するよう依頼したときだけ使う。マージ済みでクリーンな作業用worktreeとローカルブランチを削除し、リモート追跡ブランチをpruneし、mainブランチをfast-forwardで更新する。
---

# Git Cleanup Main

マージ済みの作業用 worktree とローカルブランチを安全に片付け、main を最新化する。

## 前提

- この skill は明示コマンド用。通常の相談や曖昧な依頼だけでは実行しない。
- dirty な worktree、未マージのブランチ、main 以外へ checkout 不能な状態は削除しない。
- `git reset --hard`、`git checkout -- <file>`、`rm -rf` などの破壊的操作はユーザーの明示許可なしに使わない。
- リポジトリ固有の `AGENTS.md` がある場合は必ず従う。

## 手順

1. 状態確認
   - `git status --short --branch`
   - `git worktree list`
   - `git branch --merged main`
   - `git branch -r`
   - `git fetch --prune origin`

2. main worktree を特定する
   - `git worktree list` で `main` の path を確認する。
   - main worktree が存在しない場合は停止して報告する。
   - 削除対象 worktree の中から自分自身を削除しないよう、以降の削除操作は main worktree から実行する。

3. 削除対象を決める
   - main 以外の worktree を対象候補にする。
   - 候補ごとに `git status --short --branch` を確認し、dirty change があれば削除しない。
   - 対応する branch が `git branch --merged main` に含まれる場合だけ削除対象にする。
   - upstream が `[gone]` のものは stale として扱ってよいが、未マージなら削除しない。

4. worktree とローカルブランチを削除する
   - `git worktree remove <path>` を使う。
   - worktree 削除後、`git branch -d <branch>` を使う。
   - `git branch -D` は使わない。

5. リモート追跡を掃除する
   - `git fetch --prune origin` を実行する。
   - リモートブランチがまだ存在し、対応 PR がマージ済みで、ユーザーがリモート削除も求めている場合だけ `git push origin --delete <branch>` を実行する。
   - `remote ref does not exist` は既に削除済みとして扱う。

6. main を更新する
   - main worktree で `git status --short --branch` を確認し、dirty change があれば pull しない。
   - `git checkout main` が必要な場合は通常 checkout を使う。
   - `git pull --ff-only origin main` を実行する。
   - fast-forward できない場合は停止して報告する。

7. 最終確認
   - `git worktree list`
   - `git branch --list <deleted-branch>`
   - `git branch -r --list origin/<deleted-branch>`
   - `git status --short --branch`
   - 最終報告には削除した worktree / branch、残したもの、main の更新結果を含める。

## 止める条件

- 削除対象 worktree に未保存変更がある。
- branch が main にマージされていない。
- main worktree が見つからない。
- main に dirty change がある。
- `git pull --ff-only` が失敗する。
- リモートブランチ削除や強制削除が必要だが、ユーザーが明示許可していない。
