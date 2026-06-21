# Tasks Management Migration Architecture

## 1. 背景と目的

### 1.1 現状の状況
**Phase 1: Project Management** - 完了済み

| Entity | ステータス | 機能 |
|--------|-----------|------|
| User | **未実装** | 登録、ログイン、権限設定 |
| Project | **未実装** | 作成、編集、削除、移動 |
| Tag | **未実装** | 作成、付け替え、削除 |
| Report | **未実装** | データ分析レポート |
| Calendar | **未実装** | イベント管理 |

### 1.2 拡張対象
**Phase 2: Tasks Management** - 設計中

Microsoft Project のようなタスク管理機能を追加・統合する。

---

## 2. 全体アーキテクチャ

```
┌─────────────────────────────────────────┐
│          Frontend (Thymeleaf)           │
│  ┌─────────────────────────────────┐   │
│  │ TaskList.html (一覧/フィルタ/ソート)│   │
│  │ TaskDetail.html (詳細/添付/コメント)│   │
│  │ Gantt.html (依存関係/リソース)      │   │
│  │ TaskForm.html (作成/編集)          │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│      Backend (Spring Boot + JPA)        │
│  ┌─────────────────────────────────┐   │
│  │ Controllers (REST API)          │   │
│  │   - TaskController             │   │
│  │   - GanttController           │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ Services (Business Logic)       │   │
│  │   - TaskService               │   │
│  │   - GanttService             │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ Repositories (Data Access)      │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ Entities (ORM Models)           │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│         Infrastructure (Docker)         │
│  ┌─────────────────────────────────┐   │
│  │ PostgreSQL Container           │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

### 技術スタック

| Layer | Technology | 理由 |
|-------|-----------|------|
| Backend | Spring Boot | 堅牢なエンタープライズフレームワーク |
| ORM | JPA (Hibernate) | 効率的な DB 操作、型安全 |
| Frontend | Thymeleaf | サーサイドレンダリング、テンプレート化 |
| JS Libs | jQuery | 既存ライブラリとの整合性、軽量 |
| Database | PostgreSQL | 堅牢な RDBMS、複合クエリ |
| Orchestration | Docker Compose | 環境統一、容易なデプロイ |

### 既存 Entity の拡張

| Entity | プルイン | 役割 |
|--------|---------|------|
| User | PM Core | タスクの作成者・担当者 |
| Project | PM Core | タスクの所属先 |
| Tag | PM Core | タスクのカテゴリ分類 |
| **Task** | **New** | **タスク本体** |
| TaskAttachment | **New** | ファイル添付管理 |
| TaskGroup | **New** | タスクのグループ化 |
| ProjectTaskRelationship | **New** | 複数プロジェクトへの紐付け |

---

## 3. エンティティ設計

### 3.1 Task (タスク)

```java
@Entity
@Table(name = "task")
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "project_id", nullable = false)
    private Project project;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User assignedTo;

    @Column(name = "title", nullable = false, length = 255)
    private String title;

    @Column(name = "description", columnDefinition = "TEXT")
    private String description;

    @Enumerated(EnumType.STRING)
    private TaskStatus status; // TODO, DOING, DONE, BLOCKED, CANCELLED

    @Column(name = "start_date")
    private LocalDate startDate;

    @Column(name = "due_date")
    private LocalDate dueDate;

    @Column(name = "estimated_hours", precision = 5, scale = 2)
    private BigDecimal estimatedHours;

    @Column(name = "actual_hours", precision = 5, scale = 2)
    private BigDecimal actualHours;

    @OneToMany(mappedBy = "task", cascade = CascadeType.ALL)
    private List<TaskAttachment> attachments;

    @OneToMany(mappedBy = "task", cascade = CascadeType.ALL)
    private List<TaskRelationship> dependencies;

    @CreatedBy
    @CreationDate
    private Instant createdAt;

    @LastModifiedBy
    @LastModifiedDate
    private Instant updatedAt;

    // Javadoc: Validation and Business Logic
    /** Validates task data integrity. */
    public void validate();

    /** Transitions task status with validation. */
    public void transitionStatus(String newStatus);

    /** Calculates task progress percentage. */
    public BigDecimal calculateProgress();
}

// Enum
public enum TaskStatus {
    TODO, DOING, DONE, BLOCKED, CANCELLED
}
```

**設計理由**：
- `projectId` 必須：プロジェクトへの所属は必須
- `assignedTo` 必須：担当者は必須
- `status` 有限値：無効なステータスを防止
- 日付：境界値チェック必要（開始 ≦ 終了）
- `createdAt`/`updatedAt`：監査トレイル

### 3.2 TaskAttachment (ファイル添付)

```java
@Entity
@Table(name = "task_attachment")
public class TaskAttachment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "task_id", nullable = false)
    private Task task;

    @Column(name = "filename", nullable = false, length = 255)
    private String filename;

    @Column(name = "filepath", nullable = false, length = 255)
    private String filepath;

    @Column(name = "filesize", nullable = false)
    private Long filesize;

    @Column(name = "file_type")
    private String fileType;

    @ManyToOne
    @JoinColumn(name = "uploaded_by", nullable = false)
    private User uploadedBy;

    @CreationDate
    private Instant uploadedAt;

    @CreatedBy
    @CreationDate
    private Instant createdAt;

    @LastModifiedBy
    @LastModifiedDate
    private Instant updatedAt;

    // Validation & Business Logic Methods
    public void validateFileSize(Long maxFileSize) throws ConstraintViolationException;
    public boolean isFileTypeAllowed() throws ConstraintViolationException;
    public void uploadFile(Path file) throws IOException;
    public void deleteFile() throws IOException;
}
```

**設計理由**：
- ファイルサイズ制限：100MB（DB ストレージ保護）
- ファイルタイプ制限：白黒リスト方式
- ファイルストレージ：AWS S3 など外部ストレージへの連携想定

### 3.3 TaskGroup (タスクグループ)

```java
@Entity
@Table(name = "task_group")
public class TaskGroup {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "project_id", nullable = false)
    private Project project;

    @Column(name = "group_name", nullable = false, length = 100, unique = true)
    private String groupName;

    @Column(name = "description", columnDefinition = "TEXT")
    private String description;

    @ManyToOne
    @JoinColumn(name = "created_by", nullable = false)
    private User createdBy;

    @CreationDate
    private Instant createdAt;

    @LastModifiedBy
    @LastModifiedDate
    private Instant updatedAt;

    // Validation & Business Logic Methods
    public void rename() throws ConstraintViolationException;
    public void delete() throws IOException;
}
```

**設計理由**：
- `project_id + group_name` 一意：同一プロジェクト内で重複防止
- ファイルストレージ：グループごとのファイル構成

### 3.4 TaskRelationship (タスク間の依存関係)

```java
@Entity
@Table(name = "task_relationship", uniqueConstraints = {
    @UniqueConstraint(columns = "preceding_task_id, succeeding_task_id")
})
public class TaskRelationship {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "preceding_task_id", nullable = false)
    private Task precedingTask; // 先行タスク

    @ManyToOne
    @JoinColumn(name = "succeeding_task_id", nullable = false)
    private Task succeedingTask; // 後続タスク

    @Column(name = "type", nullable = false)
    private RelationshipType type; // FS (Finish-to-Start), FF, etc.

    @CreationDate
    private Instant createdAt;

    @LastModifiedBy
    @LastModifiedDate
    private Instant updatedAt;

    // Validation & Business Logic Methods
    public void validate() throws ConstraintViolationException;
    public void remove() throws ConstraintViolationException;
}

public enum RelationshipType {
    FS, FF, SS, SF
}
```

**設計理由**：
- マイクロソフトプロジェクトの依存関係タイプ対応
- 循環依存防止
- Gantt 表示・計算用

---

## 4. API 設計

### 4.1 Task CRUD エンドポイント

| メソッド | URL | 説明 | 権限 |
|---------|-----|------|------|
| POST | /api/tasks | タスク作成 | 所有者: Project |
| GET | /api/tasks/{id} | タスク詳細取得 | 読込権限 |
| PUT | /api/tasks/{id} | タスク更新 | 所有者 |
| DELETE | /api/tasks/{id} | タスク削除 | 所有者 |

**Response Format**（共通）：
```json
{
    "success": true,
    "data": {
        "id": 1,
        "title": "Example Task",
        "status": "TODO",
        ...
    },
    "errors": []
}
```

### 4.2 Task 一覧・フィルタ API

| メソッド | URL | パラメータ |
|---------|-----|-----------|
| GET | /api/tasks | projectId, status, userId, startDate, dueDate, tags[], keyword |

**Response Format**：
```json
{
    "success": true,
    "data": [...],
    "pagination": {
        "page": 1,
        "size": 20,
        "total": 100,
        "totalPages": 5
    },
    "errors": []
}
```

### 4.3 タスク移動 API

**POST /api/tasks/{id}/move**

**Request Body**：
```json
{
    "projectId": 1,
    "groupId": null,
    "assignedTo": null
}
```

**Response**：
```json
{
    "success": true,
    "message": "Task moved to project 1",
    "affectedDependencies": [...]
}
```

### 4.4 依存関係 API（Gantt 用）

| メソッド | URL | 説明 |
|---------|-----|------|
| GET | /api/tasks/{taskId}/dependencies | 依存関係一覧取得 |
| PUT | /api/tasks/{taskId}/dependencies | 依存関係更新 |

---

## 5. UI デザイン

### 5.1 Task 一覧表示

**Template**: `TaskList.html`

| 機能 | 実装 |
|------|------|
| テーブル表示 | Thymeleaf + jQuery DataTables |
| 並び替え | Status, DueDate, AssignedTo, Title |
| フィルター | Project, Status, Date Range, User, Tags |
| ページネーション | Standard pagination |
| 表示列のカスタマイズ | Yes/No |

### 5.2 Task 詳細画面

**Template**: `TaskDetail.html`

| セクション | 機能 |
|-----------|------|
| Task Info | タイトル、ステータス、日付、担当者 |
| Attachments | ファイル一覧・アップロードボタン |
| Dependencies | Gantt 表示（依存関係） |
| Comments | コメント一覧・投稿フォーム |

### 5.3 Task 作成・編集フォーム

**Template**: `TaskForm.html`

| フィールド | バリデーション |
|-----------|---------------|
| Title (required) | minLength: 1 |
| Description (optional) | maxLength: 1000 |
| Project (select) | Required |
| Status (select) |有限値: TODO/DOING/DONE/...|
| Due Date (date picker) | min: today |
| Estimated Hours (decimal) | min: 0 |
| Tags (multi-select) | Required |

### 5.4 Gantt 表示

**Template**: `Gantt.html`

| 機能 | 実装 |
|------|------|
| 表示粒度 | 日/週/月切り替え |
| 依存関係 | 線引き |
| ドラッグ操作 | タスク移動、依存関係編集 |
| リソース負荷 | 表示/グラフ |

---

## 6. テスト設計

### 6.1 テスト環境

**PostgreSQL Container** (Testcontainers):
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: pmtools
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
    ports:
      - "5432:5432"
```

### 6.2 テストカバージージ目標

| レベル | 対象 | カバレッジ目標 |
|-------|------|---------------|
| Unit | Entity | 80% |
| Integration | API | 90% |
| UI | Thymeleaf/JQuery | 70% |

### 6.3 テストカテゴリ

**Entity レベル (JUnit)**:
- Task: validation, status transition, date calculation
- TaskAttachment: size limit, type whitelist
- TaskGroup: rename constraints

**API レベル (Spring Mock + RestClient)**:
- Task: CRUD, filtering, move
- 権限チェック、エラーレスポンス

**UI レベル (Selenium)**:
- TaskList: 表示、フィルタ、ページネーション
- TaskDetail: モダル、ファイルアップロード
- Gantt: ドラッグ、依存関係

### 6.4 境界値・例外

| テストケース | 期待結果 |
|------------|---------|
| 日付逆転 (開始 > 終了) | Error 400 |
| 空プロジェクトへの割り当て | Error 403 |
| 依存先の完了前終了日設定 | Warning |
| 無効なファイル拡張子 | Error 400 |
| 超過ファイルサイズ | Error 400 |

---

## 7. 開発手順

### ステップ 2: 人間によるルール確認・定義

| チェック | 状態 |
|---------|------|
| ABSOLUTE_RULES.md 確認 | 要実装 |
| rule.md の矛盾チェック | 要実装 |
| エンティティ属性確認 | 倖太郎 承認待ち |
| API 詳細仕様 | 倖太郎 承認待ち |
| UI デザイン意図 | 倖太郎 承認待ち |

### ステップ 3: テスト製造

| チェック | 状態 |
|---------|------|
| Testcontainers 準備 | 要実装 |
| Entity テスト | 要実装 |
| API テスト | 要実装 |
| UI テスト | 要実装 |
| テスト実行スクリプト | 要実装 |

### ステップ 4: 実装

| チェック | 状態 |
|---------|------|
| docs/設計 | 完了 |
| Entity 実装 | 要実装 |
| Repository 実装 | 要実装 |
| Service 実装 | 要実装 |
| Controller 実装 | 要実装 |
| Frontend 実装 | 要実装 |
| tools/ | 要実装 |

### ステップ 5: README

| チェック | 状態 |
|---------|------|
| 機能説明 | 要実装 |
| 技術スタック | 要実装 |
| ディレクトリ構造 | 要実装 |
| 開発フロー | 要実装 |
| 初期環境セットアップ | 要実装 |

---

## 8. リスク管理

| リスク | 影響 | 対策 | 優先度 |
|-------|------|------|-------|
| 開発期間超過 | 延期 | 毎日進捗確認 | 高 |
| データ整合性 | DB 壊壊 | TDD | 高 |
| UI レスポンシブ | 操作性低下 | Material Design | 中 |
| 権限漏れ | セキュリティ | 逐次テスト | 高 |

---

**文書作成日**: 2026-06-21
**担当者**: PM Tools Team
**確認者**: 倖太郎
