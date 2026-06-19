# PM Tools - Phase 2: Task Management Specifications

## 概要

Phase 1: プロジェクト管理の完了後、Phase 2 で**タスク管理機能**を実装します。

---

## 機能要件

### 基本機能（必須）

- [ ] タスク CRUD（作成・編集・削除・一覧取得）
- [ ] 親子タスク関係（階層管理）
- [ ] 完了ステータス管理（完了率/進捗率）
- [ ] タスク詳細表示
- [ ] タスクフィルタリング（検索・ステータス・優先度）

### 詳細機能（希望）

- [ ] ドラッグ＆ドロップ移動
- [ ] カスタムフィールド
- [ ] リソース担当者割り当て
- [ ] E2E テスト

---

## データベーススキーマ

### tasks テーブル

```prisma
model Task {
  id                String    @id @default(uuid())
  projectId         String    @map(project_id)
  name              String    @db.VarChar(255)  // 必須
  description       String?   @db.Text          // オプション
  parentId          String?   @map(parent_id)   // NULL=ルート、子タスクの ID
  startDate         DateTime  @db.Date          // 必須
  endDate           DateTime? @map(end_date)    // 計算または手動
  duration          Int       @map(duration)    // 期間（日数）
  progressRate      Decimal   @map(progress_rate) @default(0) @db.Decimal(5,2) // 進捗率
  completionRate    Decimal   @map(completion_rate) @default(0) @db.Decimal(5,2) // 完了率
  actualStartDate   DateTime? @map(actual_start_date)
  actualEndDate     DateTime? @map(actual_end_date)
  actualDuration    Int?      @map(actual_duration)
  startConstraint   String    @map(start_constraint)
  priority          Int       @default(3)       // 1-5
  resourceId        String?   @map(resource_id)
  customFields      Json?     @default("{}")
  position          Int       @default(0)       // 階層内の順序
  
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  
  // 親子関係
  parent            Task?     @relation("TaskParent", fields: [parentId], references: [id])
  
  // 子タスク
  children          Task[]    @relation("TaskChildren")
  
  // プロジェクトとの関係
  project           Project   @relation("ProjectTasks", fields: [projectId], references: [id])
  
  // タグとの関係（多対多）
  tags              Tag[]     @relation("TaskTags")
}
```

### tasks_tag 関連テーブル

```prisma
model TaskTag {
  id            Int    @id @default(autoincrement())
  taskId        String
  tagId         String
  
  createdAt     DateTime @default(now())
  
  taskId        String  @map("task_id") @unique
}
```

### custom_fields テーブル（拡張用）

```prisma
model CustomField {
  id      Int       @id @default(autoincrement())
  name    String    @unique
  code    String    @unique
  type    String    // 'text' | 'number' | 'select' | 'checkbox' | 'date'
  required Boolean @default(false)
  options  String?[]?  // 選択肢（select の時）
  
  tasks    Task[]?
  
  // 拡張用テーブル（多対多）
  taskFields CustomFieldValue[]
}
```

### custom_field_value テーブル（拡張用）

```prisma
model CustomFieldValue {
  id        Int   @id @default(autoincrement())
  taskId    String
  fieldId   Int   @map("field_id")
  value     Json? @default("{}") // 値データ
  
  fieldId   Int   @map("field_id") @relation("CustomField") @relation("TaskFieldValue")
  task      Task  @relation("TaskFields", fields: [taskId], references: [id])
}
```

---

## API エンドポイント

### 基本エンドポイント

```
GET    /api/v1/tasks              - タスク一覧取得
GET    /api/v1/tasks/:id         - タスク詳細取得
POST   /api/v1/tasks             - タスク作成
PUT    /api/v1/tasks/:id         - タスク編集
DELETE /api/v1/tasks/:id         - タスク削除
```

### 親子関係

```
GET    /api/v1/tasks/:id/children   - 子タスク取得
POST   /api/v1/tasks/:id/children   - 子タスク作成
PATCH  /api/v1/tasks/:id/parent     - 親タスク変更
```

### 完了管理

```
POST   /api/v1/tasks/:id/complete   - タスク完了設定
PATCH  /api/v1/tasks/:id/progress   - 完了率/進捗率更新
```

### フィルタリング

```
GET    /api/v1/tasks?projectId=xxx&status=all&search=xxx
```

---

## フロントエンド構成

### コンポーネント

```
src/components/Task/
├── TaskList.tsx          - タスク一覧コンポーネント
├── TaskItem.tsx         - 個別タスク表示
├── TaskForm.tsx         - 作成/編集フォーム
└── TaskProgressBar.tsx  - 完了率/進捗率バー
```

### ページ

```
src/pages/
├── TasksPage.tsx         - タスク一覧画面
└── TaskDetailPage.tsx    - タスク詳細画面
```

### Redux State

```typescript
{
  tasks: {
    tasks: Task[],         // 全タスク
    tasksById: { ... },    // ID マッピング
    activeTasks: Task[],    // 活動中のタスク
    filters: { ... }       // フィルタ状態
  }
}
```

---

## UI/UX

### タスク一覧画面

- タスク一覧テーブル表示
- 完了率/進捗率表示（パーセント＆バー）
- 優先度バッジ
- 検索・フィルタ機能
- ページネーション

### タスク詳細画面

- 基本情報表示
- 子タスクツリー表示
- タグ表示
- 完了/進捗操作

---

## 優先順位

**1st Priority (必須):**
1. tasks テーブル設計
2. 基本 CRUD API
3. 一覧取得 API
4. 詳細取得 API
5. 完了操作 API
6. タスク一覧コンポーネント
7. タスク詳細コンポーネント

**2nd Priority (重要):**
8. 親子関係実装
9. 完了率/進捗計算
10. フォームバリデーション

**3rd Priority (希望):**
11. ドラッグ＆ドロップ
12. カスタムフィールド
13. E2E テスト

---

## 実装ロードマップ

### Week 1: データベース設計
- Prisma schema 作成
- Migrate 実行
- 初期データ seeded

### Week 2: バックエンド実装
- CRUD API 実装
- 親子関係ロジック
- 完了計算ロジック

### Week 3: フロントエンド実装
- タスク一覧画面
- タスク詳細画面
- 編集フォーム
- API 連携

---

*Phase 2 Specifications*
*Created: 2026-06-19*
