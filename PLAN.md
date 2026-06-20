# PM Tools - 開発計画

## 現状
- Phase 1: Project Management（完了）
  - Schema 設計、API 実装、UI 実装済み
  - Entity: User, Project, Tag
- Phase 2: Tasks Management（進行中）

---

# フェーズ 2: Tasks Management 実装前の設計

## ステップ 0: 設計

### 0.1 要件分析
- Task エンティティの詳細設計（プロジェクト管理ツールとしての Tasks の役割）
- タスクとの関連性（タスクはプロジェクト内の管理単位）
- タスクの属性（タイトル、開始日、終了日、担当者、ステータス等）
- Gantt 表示の要件定義
- リソース管理の要件定義

### 0.2 全体アーキテクチャ設計
- バックエンド層（Spring Boot REST API）
- フロントエンド層（Thymeleaf + jQuery）
- データベース設計（PostgreSQL）
- デプロイ構成（Docker Compose）

### 0.3 エンティティ設計
- Task, TaskAttachment, TaskGroup, ProjectTaskRelationship
- 既存の User, Project, Tag との関連性
- Entity-JPA 設計図

### 0.4 API 設計
- REST API 一覧（Tasks CRUD, Filters, Sorts）
- JSON レスポンス形式
- Swagger/OpenAPI ドキュメント設計

### 0.5 UI デザイン
- Task 一覧表示（テーブル/グリッド）
- Task 詳細画面
- Gantt 表示画面（スケジューリング計算）
- フォーム入力画面

### 0.6 テスト設計
- ユニットテストケース（Entity レベル）
- インテグレーションテストケース（API レベル）
- UI テストケース（主要フロー）
- 境界値・エラーハンドリングケース

---

## ステップ 1: 必要ルールの洗い出しと人間による定義

### 1.1 ABSOLUTE_RULES.md 確認
- rule.md: 変更管理（main→feature, MR 承認）
- rule.md: プログレム単位（50〜100 step）
- rule.md: コードコメント（コードと同量）
- rule.md: 技術スタック（Spring Boot+Thymeleaf+jQuery, PostgreSQL, Docker Compose 以外 NG）
- rule.md: 計画プロセス（設計→定義→テスト製造→実装→README→再計画）
- rule.md: ディレクトリ（docs:MDのみ/子禁止, rules:人間のみ/MDのみ, src/, tools/, README, ABSOLUTE_RULE, PLAN）
- rule.md: 技術要素（thymeleaf は thymeleaf の誤字）
- rule.md: 採用禁止（Vue/React/Angular, Oracle MySQL, Kubernetes/Docker Swarm, 外製パッケージ）

### 1.2 人間によるルール定義
- 倖太郎にルール確認・定義を依頼
- ルールの矛盾・欠落チェック

---

## ステップ 2: テスト製造

### 2.1 テストスケルトン作成
- JUnit テストスケルトン
- Testcontainers PostgreSQL 環境

### 2.2 API テストフック
- REST API テストフック
- JSON レスポンス検証

### 2.3 UI テストスケルトン
- Thymeleaf テスト
- jQuery テスト

### 2.4 エラーハンドリングテスト
- バoundary 値テスト
- インボリッド入力テスト

---

## ステップ 3: 実装

### 3.1 バックエンド実装
- Spring Boot REST API
- Entity-JPA
- Repository パターン

### 3.2 フロントエンド実装
- Thymeleaf テンプレート
- jQuery UI

### 3.3 データベース実装
- PostgreSQL スキーマ
- migration script

### 3.4 Docker Compose 実装
- Dockerfile
- docker-compose.yml
- nginx 設定

---

## ステップ 4: README の作成

### 4.1 機能説明
- 利用方法
- 機能一覧
- インストール方法

### 4.2 技術スタック
- Spring Boot
- Thymeleaf + jQuery
- PostgreSQL
- Docker Compose

### 4.3 ディレクトリ構成
- docs/, rules/, src/, tools/
- 各ディレクトリの役割説明

### 4.4 開発フロー
- feature ブランチ作成
- TDD 方針
- コードコメント要件

---

## ステップ 5: 再計画

### 5.1 完了チェック
- 全てのステップ完了確認
- テスト結果確認

### 5.2 次フェーズ計画
- Phase 3: テスト実行
- Phase 4: 実装完了
- 次回タスクの定義

---

**目標:** 設計フェーズを完了し、実装フェーズへ移行する
**期限:** 2026-06-27
**責任者:** PM Tools Team
