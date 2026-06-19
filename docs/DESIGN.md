# PM Tools - Design Overview

ブラウザで動作する Microsoft Project 的なプロジェクト管理ツール。

---

## 全体アーキテクチャ

```
┌─────────────────────────────────────────┐
│                 Browser (Client)         │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │   Frontend  │  │   Gantt Editor  │   │
│  │   (React)   │  │   (Canvas/SVG)  │   │
│  └─────────────┘  └─────────────────┘   │
│       │                     │           │
│       └───────── API ───────┘           │
└─────────────────────────────────────────┘
│                     │
│      ┌──────────────┴──────────────┐    │
│      │           Backend           │    │
│      │   (Node.js + Express)       │    │
│      │   ┌───────────────┐         │    │
│      │   │   API Server  │         │    │
│      │   │   (REST/GraphQL)│        │    │
│      │   └───────────────┘         │    │
│      │   ┌───────────────┐         │    │
│      │   │   Database    │         │    │
│      │   │   (PostgreSQL)│         │    │
│      │   └───────────────┘         │    │
│      └─────────────────────────────┘    │
└─────────────────────────────────────────┘
```

---

## 主要コンポーネント

### 1. Frontend (React)

**パッケージ**: `react`, `redux`, `antd`, `react-gantt`（または自作）

**主要コンポーネント**:

```
components/
├── App/
│   ├── ProjectList.tsx
│   ├── ProjectDetail.tsx
│   └── TaskEditor.tsx
├── Gantt/
│   ├── GanttChart.tsx          # タスクのタイムライン表示
│   ├── GanttHeader.tsx         # 日付/月のインデックス
│   ├── GanttTaskBar.tsx        # 単一タスクのバー
│   └── GanttToolbar.tsx        # 操作ツール
├── Kanban/
│   ├── KanbanBoard.tsx         # カンバン表示
│   ├── KanbanCard.tsx
│   └── KanbanLane.tsx
├── Calendar/
│   └── CalendarView.tsx        # カレンダー表示
├── Network/
│   └── NetworkDiagram.tsx      # パート関係図
├── Resource/
│   ├── ResourceList.tsx
│   └── ResourceAllocation.tsx
└── Reports/
    └── ReportGenerator.tsx
```

### 2. Backend (Node.js + Express)

**フォルダ構成**:

```
server/
├── src/
│   ├── routes/                 # API エンドポイント
│   │   ├── projects.routes.ts
│   │   ├── tasks.routes.ts
│   │   ├── resources.routes.ts
│   │   └── relationships.routes.ts
│   ├── controllers/
│   │   ├── project.controller.ts
│   │   ├── task.controller.ts
│   │   └── resource.controller.ts
│   ├── models/                 # データベースモデル
│   │   ├── Project.model.ts
│   │   ├── Task.model.ts
│   │   ├── Resource.model.ts
│   │   └── Relationship.model.ts
│   ├── services/
│   │   ├── scheduler.service.ts     # スケジュール計算
│   │   └── gantt.service.ts         # Gantt 描画ロジック
│   └── middleware/
│       ├── auth.middleware.ts
│       └── validation.middleware.ts
├── migrations/                 # DB スキーマ変遷
│   └── 001_initial_schema.sql
└── tests/
    └── api.test.js
```

### 3. データベース (PostgreSQL)

**テーブル**:

```sql
-- プロジェクト
CREATE TABLE projects (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    description TEXT,
    start_date DATE NOT NULL,
    end_date DATE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    owner_id INTEGER,
    CONSTRAINT fk_owner FOREIGN KEY (owner_id) REFERENCES users(id)
);

-- タスク
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    project_id INTEGER NOT NULL,
    parent_id INTEGER,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    start_date DATE NOT NULL,
    end_date DATE,
    duration_days INTEGER NOT NULL,
    percent_complete DECIMAL(5,2),
    progress DECIMAL(5,2),
    actual_start_date DATE,
    actual_end_date DATE,
    actual_duration_days INTEGER,
    constraints VARCHAR(50),
    priority INTEGER,
    assigned_to INTEGER,
    resource_group_id INTEGER,
    custom_fields JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    CONSTRAINT fk_project FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    CONSTRAINT fk_parent FOREIGN KEY (parent_id) REFERENCES tasks(id) ON DELETE CASCADE,
    CONSTRAINT fk_assigned FOREIGN KEY (assigned_to) REFERENCES resources(id) ON DELETE SET NULL
);

-- リソース
CREATE TABLE resources (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    type VARCHAR(20) NOT NULL,  -- 'person', 'team', 'material'
    email VARCHAR(255),
    available FROM NOW(),
    daily_capacity INTEGER DEFAULT 8,  -- 時間数
    rate_per_hour DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 依存関係
CREATE TABLE relationships (
    id SERIAL PRIMARY KEY,
    predecessor_id INTEGER NOT NULL,
    successor_id INTEGER NOT NULL,
    type VARCHAR(10) NOT NULL,  -- FS, SS, FF, SF (Finish-to-Start, etc.)
    lag_days INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    CONSTRAINT fk_pre FOREIGN KEY (predecessor_id) REFERENCES tasks(id),
    CONSTRAINT fk_succ FOREIGN KEY (successor_id) REFERENCES tasks(id)
);
```

### 4. Gantt 描画エンジン

**実装方法**:

- `react-gantt-timeline` などのサードパーティライブラリを使用、または
- Canvas API で自作

**ロジック**:

```javascript
// Gantt チャートの作成
function createGanttChart(projectId, view) {
    const project = await getProject(projectId);
    const tasks = await getTasks(projectId);
    
    const { scale, unit } = view; // day, week, month
    const startDate = getProjectStartDate(project);
    const endDate = calculateEndDate(tasks);
    
    // 日付スケールの作成
    const headerRows = generateDateHeader(startDate, endDate, unit, scale);
    
    // タスクバーの配置
    const taskBars = tasks.map(task => ({
        ...task,
        barStart: task.start_date,
        barEnd: task.end_date,
        barDuration: task.duration_days,
        resourceId: task.assigned_to
    }));
    
    return {
        header: headerRows,
        tasks: taskBars,
        gridSize: calculateGridSize(unit, scale)
    };
}
```

---

## 主要機能

### 基本機能

1. **プロジェクト管理**
   - プロジェクトの作成・編集・削除
   - プロジェクトの詳細表示
   - プロジェクトの複製

2. **タスク管理**
   - タスクの追加・編集・削除
   - 親子関係（サブタスク）
   - タスク名・説明・日付・期間・進捗
   - 優先度・制約（タスク開始方式など）
   - 担当者割り当て
   - カスタムフィールド

3. **Gantt 表示**
   - 日付/週/月表示切り替え
   - スケール調整（1 日/3 日/7 日など）
   - タスクバーのドラッグ操作
   - クリティカルパスのハイライト
   - リソース負荷の可視化

4. **リソース管理**
   - リソースの登録（person, team, material）
   - リソース割り当て
   - リソース負荷表示

5. **依存関係**
   - タスク間依存（FS, SS, FF, SF）
   - ドラッグで依存関係の作成・切断
   - 遅延（lag）の指定

### 詳細機能

1. **スケジュール計算**
   - 始期計算（正解計算）
   - クリティカルパスの自動計算
   - フロート計算
   - 遅延・超過の影響分析

2. **データビュー**
   - データ一覧表示（タスク、リソース、時間）
   - カスタムビュー作成
   - フィルター・ソート

3. **レポート**
   - 基本レポート（タスクリスト、リソース使用率）
   - Gantt レポート
   - 時間軸レポート
   - カスタムレポート

4. **印刷/エクスポート**
   - PDF 出力
   - CSV/Excel エクスポート

---

## テクノロジースタック

### フロントエンド

- **React 18+** - メイン UI フレームワーク
- **Redux Toolkit** - ステート管理
- **TypeScript** - 型安全
- **Tailwind CSS** - スタイリング
- **Ant Design** - コンポーネントライブラリ
- **Framer Motion** - アニメーション
- **Canvas/SVG** - Gantt 図描画

### バックエンド

- **Node.js 20+** - 実行環境
- **Express** - Web フレームワーク
- **PostgreSQL** - データベース
- **Prisma** - ORM
- **pg-javascript** - Node.js 用 PostgreSQL ドライバー
- **ExcelJS** - Excel エクスポート

### インフラ

- **Docker** - 環境構築
- **Redis** - キャッシュ
- **Nginx** - リバースプロキシ

---

## 開発フェーズ

1. **Phase 1: プロジェクト管理**
   - プロジェクト CRUD
   - 基本プロジェクト表示

2. **Phase 2: タスク管理**
   - タスク CRUD
   - タスク一覧表示
   - 親子関係

3. **Phase 3: Gantt 表示**
   - Gantt チャート作成
   - 日付スケール
   - タスクバー表示

4. **Phase 4: スケジュール計算**
   - 正解計算
   - 依存関係
   - クリティカルパス

5. **Phase 5: リソース管理**
   - リソース CRUD
   - 割り当て機能
   - 負荷表示

6. **Phase 6: レポート・エクスポート**
   - レポート機能
   - Excel/PDF エクスポート

---

## 設計のポイント

1. **ブラウザ中心の実装**
   - 計算も表示もブラウザ完結
   - バックエンドはデータ管理、単純な API 提供
   - 大規模計算は Web Worker 使用

2. **パフォーマンス考慮**
   - 大量タスク時の描画最適化
   - Virtual scroll 適用
   - データローダー活用

3. **レスポンシブ対応**
   - PC/スマホ両対応
   - 画面サイズに応じた表示変形

4. **ユーザー体験**
   - ダークモード対応
   - キーボードショートカット
   - ドラッグ＆ドロップ

---

## 参考

- Microsoft Project の機能
- ProjectLibre の実装
- 既存の Gantt ライブラリ

---

*2026-06-19 作成*