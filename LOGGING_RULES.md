# PM Tools - Logging Rules

作業中は常に状況を逐一ログとして記録します。

## Log Format

```
[YYYY-MM-DD HH:MM GMT+9] <message>
```

## When to Log

- 重大な判断や決定
- エラーや障害
- 完了した作業タスク
- ブランチの作成・切り替え
- 設計の変更
- 技術選定の理由
- ユーザーからの要望やフィードバック

## Where to Store

- 当日のログ: `memory/YYYY-MM-DD.md`
- 蓄積ログ: `PM_LOGS/YYYY-MM-DD.md`
- 設計/ドキュメント: `docs/` 配下のファイル

## Tools

- `memory_get` / `memory_search` for MEMORY.md
- `read` for local files
- `write` for creating/updating files

---

## Additional Global Rules

### 1. Regular Push at Milestones

- **When**: After committing significant milestones (Phase completion, design docs, major features)
- **Action**: Push changes to remote repository immediately
- **Format**: 

```bash
git add .
git commit -m "<milestone summary>"
git push origin main  # or current branch
git push -u origin <branch>
```

- **Frequency**: Every milestone completion (e.g., Phase 1 complete, major feature delivery)

### 2. In-Chat Progress Logging

- **When**: Every significant step of work
- **Format**: Provide brief summary of current state in chat
- **Include**:
  - Current task being worked on
  - Any blocking issues
  - Progress updates
- **Purpose**: Keep user informed without interrupting work flow

### 3. Token Budget Protection

- **Limit**: Maximum 90% token budget usage
- **Trigger**: When current usage exceeds 90% (after compression)
- **Action**: 
  1. Stop current work immediately
  2. Save current state (git commit)
  3. Push changes to repository
  4. Notify user with message

- **Command**:

```bash
git add .
git commit -m "<interrupt note>"
git push origin <branch>
```

- **Notification Message**:

```
⚠️ Token budget exceeded (90% threshold reached)

Current usage: [XX]%

Action taken:
- Work paused
- Changes saved and pushed

Resuming will require your input. Next step: [next action]

Please confirm when ready to continue.
```

### 4. Token Budget Monitoring

- **Check**: Before starting new task (if >70%) and during heavy operations
- **Action**: Use `session_status` to check token usage
- **Response**: If >90%, execute push and notify user

---

## Priority Order

1. Token Budget Protection (highest - system rule)
2. Regular Push at Milestones (scheduled)
3. In-Chat Progress Logging (ongoing)
4. Standard Logging (daily log, decisions, errors)