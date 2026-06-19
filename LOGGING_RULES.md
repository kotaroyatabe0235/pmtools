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