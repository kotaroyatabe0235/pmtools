# PM Tools - 開発計画

## 現状
- **Phase 1:** Project Management（完了）
  - Schema 設計、API 実装、UI 実装済み
  - Entity: User, Project, Tag

- **Phase 2:** Tasks Management（進行中）

---

# フェーズ 2: Tasks Management 実装前の設計

## ステップ 1: 設計

### 1.1 全体アーキテクチャ概要
- [ ] 既存プロジェクト管理ツールの機能分析
- [ ] Tasks Management の拡張・統合ポイント明確化
- [ ] スケーラビリティ要件の再確認（Task 数の増加想定）
- [ ] セキュリティ要件の洗い出し（認証/認可の粒度）

### 1.2 エンティティ設計詳細
- [ ] Task エンティティ属性定義
  - [ ] タイトル・概要・開始日/終了日
  - [ ] ステータス（todo/doing/done/etc）
  - [ ] 担当者・見積もり時間
  - [ ] 属するプロジェクト
  - [ ] 関連タグ
- [ ] TaskAttachment エンティティ設計
  - [ ] ファイル参照（拡張子、サイズ制限）
  - [ ] 添付ファイルの保存先・形式
- [ ] TaskGroup エンティティ設計
  - [ ] グループ名・グループの階層構造
  - [ ] グループに紐づくタスクの分割粒度
- [ ] ProjectTaskRelationship 設計
  - [ ] 複数プロジェクトへの割り当て可否
  - [ ] タスク移動履歴の保持

### 1.3 API 設計詳細
- [ ] Task CRUD エンドポイント定義
  - [ ] POST /api/tasks
  - [ ] GET /api/tasks/{id}
  - [ ] PUT /api/tasks/{id}
  - [ ] DELETE /api/tasks/{id}
- [ ] Task フィルタリング・ソート API
  - [ ] GET /api/tasks?filters=&sort=
  - [ ] 使用可能なフィルターオプション定義
  - [ ] 使用可能なソートキー定義
- [ ] タスク移動 API
  - [ ] POST /api/tasks/{id}/move
- [ ] 依存関係 API（Gantt 用）
  - [ ] GET /api/tasks/{taskId}/dependencies

### 1.4 UI デザイン詳細
- [ ] Task 一覧表示
  - [ ] テーブル形式のレイアウト（並び替え/フィルター）
  - [ ] 表示列のカスタマイズ可否
  - [ ] 詳細モーダル/ページ遷移
- [ ] Task 詳細画面
  - [ ] タスク情報表示（プロパティテーブル）
  - [ ] 添付ファイル一覧・アップロードボタン
  - [ ] コメント一覧・投稿フォーム
  - [ ] Gantt 表示（依存関係/リソース負荷）
- [ ] タスク作成/編集フォーム
  - [ ] 動的なフィールド追加・削除
  - [ ] タグ選択機能（デフォルト/カスタム）
- [ ] Gantt 表示設計
  - [ ] 時間の表示粒度（日/週/月）
  - [ ] 依存関係の線引き
  - [ ] リソース割り当ての表示・ドラッグ操作

### 1.5 テスト設計詳細
#### エンティティレベル（JUnit/JPA 検証）
- [ ] Task Entity
  - [ ] 新規作成時バリデーション（必須項目チェック）
  - [ ] ステータス遷移ロジックテスト
  - [ ] 日付計算ロジック（開始/終了）
  - [ ] 関連エンティティとの連結テスト
  - [ ] 境界値テスト（日付の開始==終了）
- [ ] TaskAttachment Entity
  - [ ] ファイルサイズ制限テスト
  - [ ] 拡張子制限テスト
  - [ ] DB ストレージ容量チェック

#### API レベル（REST + Testcontainers）
- [ ] Task Create/Update/Delete
  - [ ] 有効なリクエストボディ
  - [ ] 必須項目欠落時エラー
  - [ ] 権限不足時（他者のタスク編集）
  - [ ] 非同期処理（Gantt 更新）
- [ ] Task List/Filter
  - [ ] フィルター条件の不正入力
  - [ ] 大量データ（1000 件以上）の性能
  - [ ] パージネーション動作
- [ ] Task Movement
  - [ ] 移動先プロジェクトの権限チェック
  - [ ] 依存タスクへの影響確認

#### UI レベル（Thymeleaf/JQuery）
- [ ] Task 一覧表示フロー
  - [ ] フロータ機能（debounce 処理）
  - [ ] ページネーション
- [ ] 詳細画面の遷移・表示
- [ ] Gantt 表示のドラッグ操作
  - [ ] タスクのドラッグで移動
  - [ ] 依存関係のドラッグ編集

#### エラーハンドリング
- [ ] バンダー値（日付の逆転）
- [ ] 空プロジェクトへのタスク割り当て
- [ ] 依存先の完了前の終了日設定

---

## ステップ 2: 人間によるルール確認・定義

### 2.1 既存ルール再確認
- [ ] ABSOLUTE_RULES.md の確認
  - [ ] 変更管理のルール（main→feature, MR）
  - [ ] プログレムの粒度（50〜100 ステップ）
  - [ ] テスト製造時の注意点
- [ ] rule.md の矛盾チェック（Step 0.1-1.1）

### 2.2 人間に確認依頼
- [ ] 倖太郎にフェーズ 2 の範囲・目標説明
- [ ] エンティティ属性の追加/削除提案
- [ ] API の詳細仕様（エラーコード/レスポンス形式）
- [ ] UI デザインの意図説明

### 2.3 ルール決定
- [ ] 倖太郎の決裁を待つ
- [ ] 決定後の実装方針確定

---

## ステップ 3: テスト製造詳細

### 3.1 テスト環境準備
- [ ] Testcontainers PostgreSQL 起動
  - [ ] コンテナ作成スクリプト作成
  - [ ] データベース初期化 SQL
  - [ ] 適切な初期データ作成（Seed）

### 3.2 エンティティレベルテスト（JUnit）
- [ ] Task Entity テスト
- [ ] TaskAttachment Entity テスト
- [ ] TaskGroup Entity テスト
- [ ] ProjectTaskRelationship Entity テスト

### 3.3 API レベルテスト（Spring Mock + RestClient）
- [ ] Task CRUD テスト
- [ ] Task Filter/Sort テスト
- [ ] Task Move テスト
- [ ] Gantt API テスト
- [ ] 権限チェックテスト

### 3.4 UI レベルテスト（Jquery + DOM テスト）
- [ ] Task 一覧表示検証
- [ ] 詳細画面検証
- [ ] Gantt 表示・操作検証
- [ ] フォーム入力のバリデーション

### 3.5 境界値・例外テスト
- [ ] 日付の境界値
- [ ] 無効な入力値の処理
- [ ] 権限外アクセスの拒否

### 3.6 テスト実行スクリプト
- [ ] テスト実行コマンド作成
- [ ] レポート生成・ログ出力

---

## ステップ 4: 実装詳細

### 4.1 ドキュメント作成（docs/）
- [ ] デザインドキュメント作成
  - [ ] エンティティ図（实体关系図）
  - [ ] API ドキュメント（Swagger/OpenAPI）
  - [ ] UI 画面設計図（Thymeleaf template）
- [ ] 開発ドキュメント
  - [ ] コードコメントの要件明記
  - [ ] パターン決定（Service/Layer）

### 4.2 エンティティ実装（src/main/java/.../entity）
- [ ] Task.java 実装
  - [ ] JPA Annotations 適用
  - [ ] Constructor/Getters/Setters
  - [ ] バリデーションロジック
  - [ ] Business Logic メソッド
- [ ] TaskAttachment.java
- [ ] TaskGroup.java
- [ ] ProjectTaskRelationship.java
- [ ] 既存エンティティとの FK 設定

### 4.3 Repository 実装（src/main/java/.../repository）
- [ ] TaskRepository.java（JPA / Spring Data）
- [ ] TaskAttachmentRepository.java
- [ ] TaskGroupRepository.java
- [ ] 複合クエリ（Query Method）

### 4.4 Service 実装（src/main/java/.../service）
- [ ] TaskService.java
  - [ ] CRUD メソッド実装
  - [ ] フィルタリングロジック
  - [ ] 日付計算ロジック
  - [ ] 権限チェックロジック
- [ ] GanttService.java（如有）
  - [ ] スケジューリング計算
  - [ ] リソース負荷計算

### 4.5 Controller 実装（src/main/java/.../controller）
- [ ] TaskController.java
  - [ ] REST エンドポイント定義
  - [ ] Request/Response 処理
  - [ ] エラーレスポンス
- [ ] GanttController.java（如有）

### 4.6 Frontend 実装（src/main/resources/templates）
- [ ] TaskList.html
  - [ ] jQuery によるデータ取得
  - [ ] テーブル表示/フィルター/ソート
  - [ ] ページネーション
- [ ] TaskDetail.html
  - [ ] 詳細情報表示
  - [ ] 添付ファイル管理 UI
  - [ ] コメント表示/投稿
- [ ] Gantt.html（如有）
  - [ ] グリッド表示
  - [ ] ドラッグ＆ドロップ機能
  - [ ] リソース割り当て UI
- [ ] テンプレート（Common）
  - [ ] Form Validation
  - [ ] Modal Template

### 4.7 フロントエンドアーティファクト（src/main/resources/static）
- [ ] JavaScript（AJAX フック）
- [ ] CSS（共通スタイル）

### 4.8 実装補助スクリプト（tools/）
- [ ] Dockerfile(s) 作成
- [ ] docker-compose.yml 更新
  - [ ] PostgreSQL のコンテナ定義
  - [ ] Application 起動スクリプト
- [ ] Migration script 作成
- [ ] テスト起動スクリプト

---

## ステップ 5: README の作成・更新

### 5.1 機能説明
- [ ] Task 管理機能の概要
- [ ] Feature 一覧（一覧/詳細/移動/Gantt など）
- [ ] 利用方法（手順書）
- [ ] 権限設計の説明

### 5.2 技術スタック
- [ ] バックエンド: Spring Boot + Thymeleaf + jQuery
- [ ] データベース: PostgreSQL
- [ ] コンテナ: Docker Compose

### 5.3 ディレクトリ構造
- [ ] 全体構造図
- [ ] 各ディレクトリの役割（docs/, rules/, src/, tools/）
- [ ] ファイル配置の明細

### 5.4 開発フロー
- [ ] 変更管理のフロー（main → feature → MR → review）
- [ ] TDD の実施順序
- [ ] コードコメントの要件
- [ ] プログレム単位の分割手順

### 5.5 初期環境セットアップ
- [ ] 環境変数の例
- [ ] Docker Compose 起動手順
- [ ] 初期データ導入手順

---

## ステップ 6: 再計画（完了後のリカプレート）

### 6.1 完了チェック
- [ ] ステップ 1〜5 の完了確認
- [ ] テストカバー率（目標%）確認

### 6.2 次フェーズ計画
- [ ] Phase 3: テスト実行（E2E, Integration）
- [ ] Phase 4: デプロイ準備（Docker 画像/CI）

---

**目標:** Tasks Management の設計を完了し、実装フェーズへ移行する
**期限:** 2026-06-27（以降）
**責任者:** PM Tools Team
**確認者:** 倖太郎
