# PM Tools - Branch Strategy

プロジェクト管理ツールの開発におけるブランチ運用戦略。

---

## ブランチ構成図

```
main (Production branch)
│
├── develop (Integration branch)
│   │
│   ├── feature/*           (新しい機能のブランチ)
│   ├── bugfix/*           (バグ修正のブランチ)
│   └── refactor/*         (リファクタリングのブランチ)
│
└── hotfix/*               (緊急修正のブランチ - 直接 main から)
```

---

## ブランチの概要

### main ブランチ
- 常にリリース可能な状態
- 本番環境へデプロイするブランチ
- 頻繁にマージされるが、マージ頻度は制限
- CI 構築済み、テスト済みコードのみ

### develop ブランチ
- 本番リリース前の最終統合ブランチ
- feature/ブランチからのマージ先
- リリース候補を準備
- 週 1 回 main にマージ

### feature/* ブランチ
- 新しい機能の実装
- 例：`feature/gantt-basic`、`feature/resource-allocation`
- develop ブランチから作成
- 機能完了時に develop へマージ

### bugfix/* ブランチ
- 既存機能のバグ修正
- 例：`bugfix/task-delete`、`bugfix/date-calc`
- 主ブランチ（通常は develop）から作成
- 修正完了後すぐに主ブランチへマージ

### refactor/* ブランチ
- コードの改修（機能変更なし）
- 例：`refactor/api-structure`、`refactor/gantt-performance`
- 主ブランチから作成
- 主ブランチへマージ

### hotfix/* ブランチ
- main ブランチの緊急修正
- 例：`hotfix/security-patch`、`hotfix/critical-bug`
- main ブランチから作成
- 修正完了後すぐに main へマージ

---

## 命名規則

### ブランチ名

| タイプ | 命名形式 | 例 |
|--------|----------|-----|
| feature | `feature/機能名-詳細` | `feature/gantt-basic` |
| bugfix | `bugfix/バグ概要` | `bugfix/task-delete-error` |
| refactor | `refactor/再構築理由` | `refactor/gantt-performance` |
| hotfix | `hotfix/緊急理由` | `hotfix/security-patch` |
| release | `release/VersionMajorMinorPatch` | `release/0.1.0` |
| chore | `chore/タスク名` | `chore/deps-update` |
| docs | `docs/ドキュメント` | `docs/api-documentation` |
| test | `test/テスト対象` | `test/resource-allocation` |
| ci | `ci/CI 関連` | `ci/build-fix` |

### コミットメッセージ

```
<type>: <subject>

<body>

<footer>
```

#### Type

| タイプ | 説明 | 例 |
|--------|------|-----|
| feat | 新機能 | `feat: Gantt 描画を実装` |
| fix | バグ修正 | `fix: タスク削除時のエラー` |
| refactor | 再構築 | `refactor: API 構造改善` |
| perf | パフォーマンス | `perf: Gantt 描画最適化` |
| docs | ドキュメント | `docs: API ドキュメント追加` |
| style | 書式修正 | `style: コードフォーマット` |
| test | テスト追加 | `test: リソース機能テスト` |
| chore | 設定修正 | `chore: パッケージ更新` |
| revert | リバート | `revert: 前のコミット元へ戻す` |

#### Subject

- 大文字小文字
- 最大 50 文字
- 動詞＋動詞名
- 例：`feat: Gantt 描画を実装`、`fix: 日期計算エラーを修正`

#### Body

- 詳細な説明
- 行区切り
- 80 文字以内
- 例：
```
- Gantt チャートコンポーネント作成
- タスクバー描画ロジック実装
- 日付スケール生成
```

#### Footer

- Issue 参照: `Closes #123`
- コードレビュー: `Reviewed-by: 誰々`

---

## ワークフロー

### 基本フロー

```
1. main から develop にブランチ
2. develop から feature ブランチ作成
3. feature で開発・コミット
4. PR を develop に投稿
5. CI 検証（テスト、lint）
6. CI 合格後、develop へマージ
7. develop から main へマージ（リリース前）
```

### 詳細ステップ

#### 1. 初期設定

```bash
# リポジトリclone
git clone https://github.com/kotaroyatabe0235/pmtools

# 初期コミット
git add .
git commit -m "init: プロジェクト初期化"
git branch -M main

# 公式リポジトリへプッシュ
git push -u origin main
```

#### 2. 最初の機能実装

```bash
# develop ブランチ作成
git checkout -b develop

# feature ブランチ作成
git checkout develop
git pull origin develop
git checkout -b feature/gantt-basic

# 開発・コミット
git add .
git commit -m "feat: Gantt コンポーネント作成"

# 詳細コミット
git commit -m "feat: Gantt チャート基本実装

- タスクバーの描画ロジック
- 日付スケールの生成
- カンバス描画

Closes #1
"
```

#### 3. PR 投稿

```bash
# プッシュ
git push origin feature/gantt-basic

# GitHub で PR 作成
# 「Compare & pull request」をクリック
# Target: develop へ
```

#### 4. 開発中のマージ

```bash
# 機能完了後のマージ
git checkout develop
git pull origin develop
git merge feature/gantt-basic

# feature ブランチ削除
git branch -D feature/gantt-basic

# develop へのプッシュ
git push origin develop

# feature ブランチの削除（リモート）
git push origin --delete feature/gantt-basic
```

#### 5. Release

```bash
# develop から main へマージ
git checkout main
git pull origin main
git merge develop

# タグ付け
git tag -a v0.1.0 -m "Initial Release: Project & Gantt Basics"
git push origin main

# タグ削除
git push origin --delete v0.1.0

# 正式タグプッシュ（GitHub UI で）
```

---

## 推奨されるコマンド

### 基本コマンド

```bash
# リポジトリの初期化
git init

# ブランチ作成・切り替え
git checkout -b <ブランチ名>
git checkout main
git checkout develop

# プル
git pull origin <ブランチ名>

# コマンド
git add .
git add <ファイル>

# コミット
git commit -m "<メッセージ>"
git commit -m "commit1" -m "commit2"
git commit -m "commit3" --amend

# プッシュ
git push origin <ブランチ名>
git push -u origin <ブランチ名>

# 統合
git merge <ブランチ名>
git merge --no-ff <ブランチ名>

# 削除
git branch -D <ブランチ名>
git branch -d <ブランチ名>

# ヒストリ表示
git log --oneline --graph --decorate
```

### 衝突解決

```bash
# 衝突の確認
git status

# 衝突の解決
git add <ファイル>

# 再コミット
git commit -m "conflict resolved: <説明>"

# 衝突の拒否
git checkout --theirs <ファイル>
git checkout --ours <ファイル>
```

---

## CI/CD Pipeline

### Workflow 構成

```yaml
name: PM Tools CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [develop, main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '20.x'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Lint
      run: npm run lint
    
    - name: Type check
      run: npm run type-check
    
    - name: Test
      run: npm run test
      env:
        CI: true
    
    - name: Build
      run: npm run build
    
    - name: E2E Test
      run: npm run test:e2e
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```

### 自動マージ

```yaml
name: Auto Merge

on:
  pull_request_target:
    branches: [develop]
    types: [labeled]

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    
    steps:
    - name: Check status
      run: |
        if [[ "$GITHUB_EVENT_NAME" == "pull_request_target" && \
              "$GITHUB_EVENT_LABEL" == "approved" ]]; then
          gh api /repos/ko.../pmtools/pulls/PR_NUM/merge \
            --method PUT \
            --field merge_method=squash
        fi
```

---

## 管理ツール

### GitHub

- ブランチ管理 UI
- PR 管理
- コミット履歴表示
- CI/CD 確認

### 開発環境

```bash
# 開発環境の準備
npm install

# データベース移行
npx prisma migrate dev

# ホスト起動
npm run dev

# テスト実行
npm run test

# ビルド
npm run build
```

---

## メモ

- **main ブランチ**: 常にクリーンな状態を維持
- **develop ブランチ**: 開発中の統合場所
- **feature ブランチ**: 一人一人が担当する機能
- **PR レビュー**: 必須（最低 1 人の承認）
- **コミット**: 短小な単位で
- **ブランチ**: 機能完了後即削除

---

*2026-06-19 作成*