# PM Tools - 開発計画

## 現状
- Phase 1: Project Management（完了）
  - Schema 設計、API 実装、UI 実装済み
  - Entity: User, Project, Tag
- Phase 2: Tasks Management（進行中）

## フェーズ 2: Tasks Management 実装前の設計

### ステップ 1: 要件分析と詳細化
- Task エンティティの詳細設計（プロジェクト管理ツールとしての Tasks の役割）
- タスクとの関連性（タスクはプロジェクト内の管理単位）
- タスクの属性（タイトル、開始日、終了日、担当者、ステータス等）
- Gantt 表示の要件定義
- リソース管理の要件定義

### ステップ 2: 全体アーキテクチャ設計
- バックエンド層（Spring Boot REST API）
- フロントエンド層（Thymeleaf + jQuery）
- データベース設計（PostgreSQL）
- デプロイ構成（Docker Compose）

### ステップ 3: エンティティ設計
- Task, TaskAttachment, TaskGroup, ProjectTaskRelationship
- 既存の User, Project, Tag との関連性
- Entity-JPA 設計図

### ステップ 4: API 設計
- REST API 一覧（Tasks CRUD, Filters, Sorts）
- JSON レスポンス形式
- Swagger/OpenAPI ドキュメント設計

### ステップ 5: UI デザイン
- Task 一覧表示（テーブル/グリッド）
- Task 詳細画面
- Gantt 表示画面（スケジューリング計算）
- フォーム入力画面

### ステップ 6: テスト設計
- ユニットテストケース（Entity レベル）
- インテグレーションテストケース（API レベル）
- UI テストケース（主要フロー）
- 境界値・エラーハンドリングケース

### ステップ 7: テスト製造（初期）
- JUnit テストスケルトン作成
- Testcontainers で PostgreSQL 環境のセットアップ
- API テストフック

### ステップ 8: README 更新
- 最新機能の追加
- 開発フローの追加記述

---

**目標:** 設計フェーズを完了し、実装フェーズへ移行する
**期限:** 2026-06-27
**責任者:** PM Tools Team
