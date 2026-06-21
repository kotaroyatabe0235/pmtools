# Task API Specification

OpenAPI 3.0 形式でタスク管理 API を定義。

## Base URL

```
/api/v1
```

## 権限レベル

| 権限 | レベル |
|------|--------|
| Owner | プロジェクト作成者、スーパーバイザー |
| Writer | プロジェクトメンバー |
| Reader | 閲覧権限あり |

---

## 4. Task CRUD エンドポイント

### POST /tasks

**説明**: タスクを作成

**権限**: `owner` (所属プロジェクトの作成者)

**Request Body**:

```yaml
components:
  schemas:
    CreateTaskRequest:
      type: object
      required:
        - title
        - projectId
        - assignedTo
        - status
      properties:
        title:
          type: string
          minLength: 1
          maxLength: 255
          description: タスクのタイトル
        projectId:
          type: integer
          minimum: 1
          description: 所属プロジェクト ID
        assignedTo:
          type: integer
          minimum: 1
          description: 担当ユーザー ID
        description:
          type: string
          maxLength: 1000
          nullable: true
        status:
          $ref: '#/components/schemas/TaskStatus'
        startDate:
          type: string
          format: date
          description: 開始日
        dueDate:
          type: string
          format: date
          description: 期限日
          nullable: true
        estimatedHours:
          type: number
          format: double
          minimum: 0
          precision: 2
          description: 見積もり時間（時間）
        tags:
          type: array
          items:
            type: integer
          minItems: 1
          description: タグ ID 一覧
```

**Response 201**:

```yaml
components:
  schemas:
    ApiResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          $ref: '#/components/schemas/Task'
        errors:
          type: array
          items:
            $ref: '#/components/schemas/Error'

    Task:
      type: object
      properties:
        id:
          type: integer
        title:
          type: string
        projectId:
          type: integer
        assignedTo:
          type: integer
        status:
          $ref: '#/components/schemas/TaskStatus'
        startDate:
          type: string
          format: date
        dueDate:
          type: string
          format: date
          nullable: true
        estimatedHours:
          type: number
          format: double
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time
```

**Error Codes**:

| コード | メッセージ | 条件 |
|-------|-----------|------|
| 400 | Invalid request body | 必須項目の欠落、形式エラー |
| 403 | Forbidden | 権限不足 |
| 400 | Invalid date range | 開始日 > 終了日 |
| 400 | Invalid tags | タグの未選択 |

---

### GET /tasks/{id}

**説明**: タスクの詳細を取得

**権限**: `reader` (タスクの所属プロジェクトのメンバー)

**Path Parameters**:

| パラメータ | タイプ | 必須 | 説明 |
|----------|-------|------|------|
| id | integer | true | タスク ID |

**Response 200**:

```yaml
components:
  schemas:
    TaskDetailResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          $ref: '#/components/schemas/Task'
        attachments:
          type: array
          items:
            $ref: '#/components/schemas/TaskAttachment'
        dependencies:
          type: array
          items:
            $ref: '#/components/schemas/TaskRelationship'
        errors:
          type: array
          items:
            $ref: '#/components/schemas/Error'
```

**Error Codes**:

| コード | メッセージ | 条件 |
|-------|-----------|------|
| 404 | Task not found | タスクが存在しない |
| 403 | Forbidden | 権限不足 |

---

### PUT /tasks/{id}

**説明**: タスクを更新

**権限**: `owner` (所属プロジェクトの作成者)

**Path Parameters**:

| パラメータ | タイプ | 必須 | 説明 |
|----------|-------|------|------|
| id | integer | true | タスク ID |

**Request Body**: （CreateTaskRequest と同一）

**Response 200**: （CreateTaskResponse と同一）

**Error Codes**:

| コード | メッセージ | 条件 |
|-------|-----------|------|
| 404 | Task not found | タスクが存在しない |
| 403 | Forbidden | 権限不足 |
| 400 | Invalid status transition | 無効なステータス遷移 |

---

### DELETE /tasks/{id}

**説明**: タスクを削除

**権限**: `owner`

**Path Parameters**:

| パラメータ | タイプ | 必須 | 説明 |
|----------|-------|------|------|
| id | integer | true | タスク ID |

**Response 200**:

```yaml
components:
  schemas:
    DeleteResponse:
      type: object
      properties:
        success:
          type: boolean
        message:
          type: string
        errors:
          type: array
          items:
            $ref: '#/components/schemas/Error'
```

**Error Codes**:

| コード | メッセージ | 条件 |
|-------|-----------|------|
| 404 | Task not found | タスクが存在しない |
| 403 | Forbidden | 権限不足 |
| 400 | Circular dependency | 依存先を削除中 |

---

## 5. Task 一覧・フィルタ API

### GET /tasks

**説明**: タスク一覧を取得（フィルタ・ソート可能）

**権限**: `reader`

**Query Parameters**:

| パラメータ | タイプ | 必須 | 説明 | 値 |
|----------|-------|------|------|-----|
| projectId | integer | false | 所属プロジェクト ID | [1-1000] |
| status | string | false | ステータスフィルタ | TODO,DOING,DONE,BLOCKED,CANCELLED |
| assignedTo | integer | false | 担当者 | [1-100000] |
| startDate | date | false | 開始日フィルタ | YYYY-MM-DD |
| dueDate | date | false | 期限日フィルタ | YYYY-MM-DD |
| tags | integer[] | false | タグフィルタ | [1-1000] |
| keyword | string | false | 検索キーワード | [1-100 char] |
| page | integer | false | ページ番号 | [1-100000] |
| size | integer | false | 表示サイズ | [1-100], デフォルト 20 |
| sort | string | false | ソートキー | id,title,status,dueDate,assignedTo |
| order | string | false | ソート順序 | ASC,DESC |

**Response 200**:

```yaml
components:
  schemas:
    TaskListResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: array
          items:
            $ref: '#/components/schemas/Task'
        pagination:
          $ref: '#/components/schemas/Pagination'
        errors:
          type: array
          items:
            $ref: '#/components/schemas/Error'

    Pagination:
      type: object
      properties:
        page:
          type: integer
        size:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer
```

**Error Codes**:

| コード | メッセージ | 条件 |
|-------|-----------|------|
| 400 | Invalid filter | フィルター形式エラー |
| 400 | Invalid sort key | 無効なソートキー |
| 422 | Invalid pagination | ページ/サイズ制限超過 |

---

## 6. タスク移動 API

### POST /tasks/{id}/move

**説明**: タスクを移動（プロジェクト/グループ/担当者変更）

**権限**: `owner` (移動先プロジェクトの作成者)

**Path Parameters**:

| パラメータ | タイプ | 必須 | 説明 |
|----------|-------|------|------|
| id | integer | true | タスク ID |

**Request Body**:

```yaml
components:
  schemas:
    MoveTaskRequest:
      type: object
      properties:
        projectId:
          type: integer
          nullable: true
          description: 移動先プロジェクト ID
        groupId:
          type: integer
          nullable: true
          description: 移動先グループ ID
        assignedTo:
          type: integer
          nullable: true
          description: 新担当者 ID
          description: groupId が指定された場合無効
```

**Response 200**:

```yaml
components:
  schemas:
    MoveTaskResponse:
      type: object
      properties:
        success:
          type: boolean
        message:
          type: string
        taskId:
          type: integer
        affectedDependencies:
          type: array
          items:
            type: object
            properties:
              relationshipId:
                type: integer
              action:
                type: string
                enum: [delete, update]
        errors:
          type: array
          items:
            $ref: '#/components/schemas/Error'
```

**Error Codes**:

| コード | メッセージ | 条件 |
|-------|-----------|------|
| 404 | Task not found | タスクが存在しない |
| 404 | Target not found | 移動先が存在しない |
| 403 | Forbidden | 権限不足 |
| 400 | Invalid move | 無効な移動操作 |
| 400 | Circular dependency | 依存先が削除中 |

---

## 7. 依存関係 API（Gantt 用）

### GET /tasks/{taskId}/dependencies

**説明**: タスクの依存関係一覧を取得

**権限**: `owner`

**Path Parameters**:

| パラメータ | タイプ | 必須 | 説明 |
|----------|-------|------|------|
| taskId | integer | true | タスク ID |

**Response 200**:

```yaml
components:
  schemas:
    DependencyListResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: array
          items:
            $ref: '#/components/schemas/TaskRelationship'
        errors:
          type: array
          items:
            $ref: '#/components/schemas/Error'
```

### PUT /tasks/{taskId}/dependencies

**説明**: タスクの依存関係を更新

**権限**: `owner`

**Path Parameters**:

| パラメータ | タイプ | 必須 | 説明 |
|----------|-------|------|------|
| taskId | integer | true | タスク ID |

**Request Body**:

```yaml
components:
  schemas:
    UpdateDependencyRequest:
      type: object
      items:
      - type: object
        required:
          - precedingTaskId
          - succeedingTaskId
          - type
        properties:
          precedingTaskId:
            type: integer
          succeedingTaskId:
            type: integer
          type:
            type: string
            enum: [FS, FF, SS, SF]
```

**Response 200**: （DependencyListResponse と同一）

**Error Codes**:

| コード | メッセージ | 条件 |
|-------|-----------|------|
| 404 | Task not found | タスクが存在しない |
| 400 | Circular dependency | 循環依存を検出 |
| 400 | Invalid date conflict | 日付の競合 |
| 403 | Forbidden | 権限不足 |

---

## Shared Components

### Enum: TaskStatus

```yaml
components:
  schemas:
    TaskStatus:
      type: string
      enum: [TODO, DOING, DONE, BLOCKED, CANCELLED]
```

### Schema: Error

```yaml
components:
  schemas:
    Error:
      type: object
      properties:
        code:
          type: string
        message:
          type: string
        field:
          type: string
          nullable: true
```

---

## Security Schemes

```yaml
securitySchemes:
  BearerAuth:
    type: http
    scheme: bearer
    bearerFormat: JWT
```

**Authentication**: Bearer token (JWT) が必須。
**Authorization**: 要求ヘッダー `Authorization: Bearer {token}`

---

**バージョン**: 1.0.0
**日付**: 2026-06-21
**責任者**: PM Tools Team
