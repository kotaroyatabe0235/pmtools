# PM Tools - Phase 2 Design: Task Management

ブラウザで動作する Microsoft Project 的なプロジェクト管理ツールの**Phase 2: タスク管理**の設計書。

---

## 期間

- **開始**: 2026-07-03
- **完了目標**: 2026-07-26
- **想定作業日数**: 約 3 週

---

## 全体概要

Phase 1 でプロジェクト管理機能（プロジェクト CRUD、ユーザー管理、タグ）を完了。
Phase 2 で**タスク管理機能**を実装。

### アーキテクチャ

```
Browser (React Frontend)
    │
    └─→ API Server (Express + Node.js)
            │
            └─→ PostgreSQL (Database)
```

**Phase 1 との相違点**:
- 同じアーキテクチャを踏襲
- tasks テーブルの追加
- 親/子関係の階層管理
- 完了率/進捗率のカラム追加

---

## データベース設計

### tasks テーブル

| カラム | タイプ | 必須 | 説明 |
|--------|--------|------|------|
| id | UUID | ○ | 一意の ID |
| name | VARCHAR(255) | ○ | タスク名（必須） |
| description | TEXT | ☓ | 記述（オプション） |
| parent_id | UUID/INT | ☓ | 親タスク ID（NULL=ルート） |
| start_date | DATE | ○ | 開始日（必須） |
| end_date | DATE/NULL | ☓ | 終了日（計算または手動） |
| duration | INT | ☓ | 期間（日数） |
| progress_rate | DECIMAL(5,2) | ☓ | 進捗率 0-100% |
| completion_rate | DECIMAL(5,2) | ☓ | 完了率 0-100% |
| actual_start_date | DATE/NULL | ☓ | 実際の開始日 |
| actual_end_date | DATE/NULL | ☓ | 実際の終了日 |
| actual_duration | INT/NULL | ☓ | 実際の期間 |
| start_constraint | ENUM | ☓ | 開始制約（ASAP/AS_LATE_AS_POSSIBLE） |
| priority | INT(1-5) | ☓ | 優先度 |
| resource_id | UUID/INT | ☓ | 担当者/リソース ID |
| custom_fields | JSONB | ☓ | カスタムフィールド |
| position | INT | ○ | 階層内の順序（親子関係） |

**階層構造**:
- `parent_id` で親子関係管理
- `position` で兄弟タスクの順序管理
- リkurs して子タスク取得可

**完了計算**:
- `completion_rate`: 完了率（0-100%）
- `progress_rate`: 進捗率（0-100%, 可変）
- `actual_*` 日付系列は、完了時の記録用

### tasks_tag 関連テーブル

| カラム | タイプ | 必須 | 説明 |
|--------|--------|------|------|
| id | UUID | ○ | 一意の ID |
| task_id | UUID | ○ | 所属タスク ID |
| tag_id | UUID | ○ | タグ ID |
| created_at | TIMESTAMP | ○ | 作成日時 |
| updated_at | TIMESTAMP | ○ | 更新日時 |

**Note**: tasks_tag でタスクとタグの多対多関係管理

---

## API 設計

### 基本エンドポイント

#### タスク一覧取得

```
GET /api/v1/projects/:projectId/tasks
GET /api/v1/tasks

Queries:
- page, per_page（ページネーション）
- search（文字列検索）
- status（完了/未完了）
- parent_id（親タスク限定）

Response Example:
{
  "data": [
    {
      "id": "1",
      "name": "タスク名",
      "start_date": "2026-07-01",
      "duration": 5,
      "priority": 3,
      "status": "active",
      "parent_id": null,
      "position": 1,
      "children_count": 2
    }
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "per_page": 20
  }
}
```

#### タスク詳細取得

```
GET /api/v1/tasks/:taskId
GET /api/v1/projects/:projectId/tasks/:taskId

Response Example:
{
  "data": {
    "id": "1",
    "name": "タスク名",
    "start_date": "2026-07-01",
    "end_date": "2026-07-06",
    "duration": 5,
    "progress_rate": 50.00,
    "priority": 3,
    "parent_id": null,
    "position": 1
  },
  "children": [],
  "resources": []
}
```

#### タスク作成

```
POST /api/v1/projects/:projectId/tasks
POST /api/v1/tasks

Request Body:
{
  "name": "タスク名",
  "start_date": "2026-07-01",
  "duration": 5,
  "priority": 3,
  "parent_task_id": 1,
  "resource_id": 1,
  "custom_fields": {
    "field1": "value1"
  }
}

Response Example:
{
  "data": {
    "id": "1",
    "name": "タスク名",
    "start_date": "2026-07-01",
    ...
  }
}
```

#### タスク編集

```
PUT /api/v1/tasks/:taskId
PUT /api/v1/projects/:projectId/tasks/:taskId

Request Body:
{
  "name": "新しい名前",
  "start_date": "2026-07-02",
  "priority": 4,
  ...
}

Response Example:
{
  "data": { ... }
}
```

#### タスク削除

```
DELETE /api/v1/tasks/:taskId
DELETE /api/v1/projects/:projectId/tasks/:taskId

Notes:
- 子タスクがある場合はエラー、または CASCADE で削除
```

#### タスクの親子関係変更

```
PATCH /api/v1/tasks/:taskId/parent

Request Body:
{
  "parent_id": 99,
  "position": 5
}
```

### 完了操作

#### タスク完了設定

```
POST /api/v1/tasks/:taskId/complete

Request Body:
{
  "completion_date": "2026-07-06",
  "progress": 100
}

Response:
{
  "data": {
    "completion_rate": 100.00,
    "progress_rate": 100.00,
    "status": "completed",
    "completion_date": "2026-07-06"
  }
}
```

#### 完了率/進捗率更新

```
PATCH /api/v1/tasks/:taskId/progress

Request Body:
{
  "completion_rate": 75.00,
  "progress_rate": 50.00,
  "completion_date": "2026-07-03"
}
```

### 階層操作

#### 子タスク取得

```
GET /api/v1/tasks/:taskId/children

Response:
{
  "data": [
    {
      "id": "2",
      "name": "子タスク 1",
      "parent_id": 1,
      "position": 1
    }
  ]
}
```

#### 子タスク作成

```
POST /api/v1/tasks/:taskId/children

Request Body:
{
  "name": "子タスク 1",
  "start_date": "2026-07-02",
  "duration": 3
}
```

### カスタムフィールド

#### カスタムフィールド取得

```
GET /api/v1/custom-fields

Response:
{
  "data": [
    {
      "id": 1,
      "name": "重要度",
      "type": "select",
      "options": ["低", "中", "高"]
    }
  ]
}
```

#### カスタムフィールド値設定

```
PATCH /api/v1/tasks/:taskId/fields

Request Body:
{
  "field_1": {
    "value": "高",
    "operator": "=",
    "expected": "high"
  }
}
```

---

## Frontend アーキテクチャ

### コンポーネント構成

```
src/
├── components/
│   ├── Task/
│   │   ├── TaskList.tsx           # タスク一覧コンポーネント
│   │   ├── TaskItem.tsx           # 個別タスク表示
│   │   ├── TaskForm.tsx           # タスク作成/編集フォーム
│   │   ├── TaskProgressBar.tsx   # 完了率/進捗率バー
│   │   └── TaskTree.tsx           # 階層ツリー表示
│   ├── TaskMobile/
│   │   └── TaskMobileView.tsx     # スマホ用タスク表示
│   └── Shared/
│       └── PriorityBadge.tsx      # 優先度バッジ
│
└── pages/
    ├── TasksPage.tsx              # タスク一覧ページ
    ├── TaskDetailPage.tsx         # タスク詳細ページ
    └── TaskEditPage.tsx           # タスク編集ページ
```

### Redux State

```typescript
{
  tasks: {
    entities: Task[]          // 全タスク
    byId: Record<string, Task> // ID マッピング
    currentProjectTasks: Task[] // 現在プロジェクトのタスク
    filters: {                // フィルタ状態
      projectId: string
      status: 'all' | 'active' | 'completed'
      search: string
      priority: number[]
    }
  }
}
```

### Redux Actions

```typescript
// Task CRUD
TASK_REQUEST_ADD: [AddTaskPayload]
TASK_REQUEST_UPDATE: [UpdateTaskPayload]
TASK_REQUEST_DELETE: [taskId]

// Task Status
TASK_COMPLETE: [taskId]
TASK_SET_PROGRESS: [taskId, rate]

// Task Hierarchy
TASK_SET_PARENT: [taskId, parentId]
TASK_CREATE_CHILD: [parentId, tasks]

// Filtering
TASK_FILTER_UPDATE: [FilterState]
```

### API Integration

```typescript
// TaskService
class TaskService {
  async fetchTasks(projectId?: string): Promise<Task[]>
  async fetchTask(id: string): Promise<Task>
  async createTask(payload: AddTaskPayload): Promise<Task>
  async updateTask(id: string, payload: UpdateTaskPayload): Promise<Task>
  async deleteTask(id: string): Promise<void>
  async completeTask(id: string, date?: string): Promise<void>
  async updateProgress(id: string, rate: number): Promise<void>
  async setParent(id: string, parentId: string): Promise<void>
}
```

---

## UI/UX 設計

### タスク一覧画面

```
┌────────────────────────────────────────────┐
│ [← Back to Projects]                       │
├────────────────────────────────────────────┤
│                                          │
│ Project: "サンプルプロジェクト"            │
│ Progress: ███████░░ 70%                   │
│                                          │
├────────────────────────────────────────────┤
│ [New Task] [Filter] [Sort] [View Options]  │
├────────────────────────────────────────────┤
│ ┌───┬───────┬────────┬─────┬───────┐     │
│ │ID  │ Name │ Due    │ Est │ Status│     │
│ └───┴───────┴────────┴─────┴───────┘     │
│ ┌───┬───────┬────────┴─────┴───────┐     │
│ │1   │ タスク A│2026/07/01 │ 5d│🟡 70%│     │
│ │2   │ タスク B│2026/07/03 │ 3d│🔴  0%│     │
│ └───┴───────┴──────────────────────┘     │
└────────────────────────────────────────────┘
```

### タスク詳細画面

```
┌────────────────────────────────────────────┐
│ [← Back to Tasks]                          │
├────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────────────────┐│
│ │  タスク名   │ │ 📅 開始日: 2026/07/01   ││
│ │            │ │ 📆 終了日: 2026/07/06   ││
│ │            │ │ ⏱️ 期間: 5 日            ││
│ │            │ │ 📊 完了率: ███████░░ 70%││
│ │            │ │ 📈 進捗率: █████░░░░ 50% ││
│ │            │ │ ⭐ 優先度: [3]/5        ││
│ │            │ │ 👤 担当者: 山田太郎     ││
│ │            │ │ 🏷️ タグ: [デッドライン] ││
│ │            │ │ 📝 記述:               ││
│ │            │ │   タスクの記述内容        ││
│ └─────────────┴ └─────────────────────────┘│
│                                          │
│ [ 子タスク (2)     ]                      │
│ ┌───────┬────────┬─────┬───────┐         │
│ │ タスク C │2026/07/02│2d│🟢100%│         │
│ │ タスク D │2026/07/04│3d│🟡 50% │         │
│ └───────┴────────┴─────┴───────┘         │
└────────────────────────────────────────────┘
```

### タスク作成/編集フォーム

```
┌────────────────────────────────────────────┐
│ タスク編集                                  │
├────────────────────────────────────────────┤
│                                            │
│ タスク名: [_________] *                     │
│ 記述: [_________]                           │
│                                            │
│ 開始日: [📅 2026/07/01] *                   │
│ 期間: [📏 5] 日                             │
│                                            │
│ 優先度: [🎯 ⭐⭐⭐]                           │
│ 担当者: [👤 山田太郎]                        │
│                                            │
│ タグ: [🏷️]                                  │
│                                            │
│ [取消] [保存]                               │
└────────────────────────────────────────────┘
```

### タスク移動 (ドラッグ＆ドロップ)

```
┌────────────────────────────────────────────┐
│                                            │
│  [📍] タスク A ━━━━━━━━━━                 │
│       │                                     │
│       ▼                                     │
│  [📍] タスク B ━━━━━━                      │
│       │                                     │
│       ▼                                     │
│  [📍] タスク C ━━━━━                       │
│       │                                     │
│  [🖱️] タスク D ━━━━━━━━━━                 │
│       │                                     │
│  [📍] 空白エリア                             │
│                                            │
└────────────────────────────────────────────┘
```

### 完了率/進捗率バー

```
┌────────────────────────────────────────────┐
│ タスク名: 「レポート作成」                  │
│ 完了率: [████████░░] 70%                   │
│ 進捗率: [██████░░░░] 50%                   │
│                                            │
│ 最終期限: 2026/07/10                       │
│ 期限遅延: ⚠️ 3 日超過                       │
└────────────────────────────────────────────┘
```

---

## 階層構造詳細

### ツリー構造

```
root (project)
├── position: 1
│   ├── task_1 (parent)
│   │   ├── position: 1
│   │   │   ├── task_1_1
│   │   │   └── task_1_2
│   │   └── position: 2
│   │       └── task_1_3
│   └── task_2
│       └── position: 1
│           └── task_2_1
```

### 親子関係のルール

- root level: `parent_id = NULL` (プロジェクトルートのタスク)
- child level: `parent_id` = 親タスクの ID
- 子タスクの開始日は、親タスクの開始日以降でなければならない

### 階層操作の制約

| 操作 | 制約 |
|------|------|
| 親変更 | 同一プロジェクト内のみ |
| 子作成 | 親が存在する場合のみ |
| 子移動 | 親の変更は不可（子がある場合） |
| 削除 | 子がある場合は CASCADE またはエラー |

---

## 完了/進捗管理

### 完了率計算

```typescript
completion_rate = (completed_subtasks / total_subtasks) * 100
```

**例**:
- 親タスク：「レポート作成」
- 子タスク 1 個：完了 (100%)
- 子タスク 2 個：完了中 (50%)
- 完了率：1/2 = 50%

### 進捗率管理

```typescript
// マニュアル入力
progress_rate = 0-100%

// 自動計算（オプション）
progress_rate = completion_rate
```

### 完了状態

| 状態 | 完了率 | 進捗率 | 色 |
|------|--------|--------|-----|
| active | 0-99% | 0-99% | 🟡 黄色 |
| completed | 100% | 0-100% | 🟢 緑 |
| overdue | 0-100% | 0-100% | 🔴 赤 (期限超過時) |

---

## カスタムフィールド

### データ構造

```typescript
interface CustomField {
  id: number;
  name: string;           // 表示名
  code: string;           // コード
  type: 'text' | 'number' | 'select' | 'checkbox' | 'date';
  required: boolean;
  options?: string[];     // 選択肢 (select の時)
}

interface CustomFieldValue {
  field_id: number;
  value: string | number | boolean | Date;
  operator?: string;      // =, !=, >, < など
  expected?: string;      // 期待値
}
```

### 使用例

```typescript
// Task schema 内
custom_fields: {
  "field_1": {
    "value": "高",
    "operator": "=",
    "expected": "high"
  }
}

// カスタムフィールドの値は、task.custom_fields 内に保存
```

---

## パフォーマンス考慮

### 階層クエリ最適化

```sql
-- 子タスク取得（再帰クエリ）
SELECT * FROM tasks
WHERE parent_id = 'xxx'
  AND parent_id != 'xxx'

-- 階深取得
WITH RECURSIVE subtree AS (
  SELECT id, parent_id, name, start_date
  FROM tasks
  WHERE id = 'xxx'

  UNION ALL

  SELECT t.id, t.parent_id, t.name, t.start_date
  FROM tasks t
  JOIN subtree s ON t.parent_id = s.id
)
SELECT * FROM subtree
```

### データ量増加対策

- ページネーション（per_page 最小 10, 最大 100）
- インデックス作成（parent_id, start_date, status）
- 全タスク取得ではなく、必要な分のみ取得

---

## テスト戦略

### Unit Test

```typescript
describe('Task', () => {
  describe('calculation', () => {
    it('should calculate end_date from start_date + duration', () => {
      expect(calculateEndDate(startDate, duration)).toBe(expectedDate)
    })
    
    it('should handle parent-child relationship correctly', () => {
      // 親子関係の計算ロジック
    })
  })
})
```

### E2E Test

```typescript
// Playwright
async test('Task CRUD', async ({ page }) => {
  await page.goto('/tasks')
  
  // タスク作成
  await createTask(page, { name: 'テスト', startDate: '2026-07-01' })
  
  // 確認
  const task = await page.getByText('テスト')
  await expect(task).toBeVisible()
  
  // 編集
  await editTask(page, 'テスト', { name: '編集済み' })
  
  // 完了
  await completeTask(page, '編集済み')
  
  // 状態確認
  const status = await page.locator('[data-testid="task-status"]')
  await expect(status).toHaveText('Completed')
})
```

---

## マイグレーション

### スキーマ追加

```bash
npx prisma migrate dev --name phase2_tasks
```

### 変更内容

1. tasks テーブル追加
2. tasks_tag 関連テーブル追加
3. カスタムフィールド対応カラム追加

### データ移行

```bash
# 既存プロジェクトに初期タスク追加
npx ts-node scripts/migrate-data-to-tasks.ts
```

---

## 今後のタスク

- [ ] Phase 3: Gantt 表示設計
- [ ] テストスクリプト作成
- [ ] 開発環境セットアップ
- [ ] ドキュメント更新

---

*2026-06-19 作成*
