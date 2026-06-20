# PM Tools

Microsoft Project 風のブラウザベースプロジェクト管理ツール。

## 技術スタック

- **バックエンド:** Spring Boot + Thymeleaf + jQuery
- **データベース:** PostgreSQL
- **デプロイ:** Docker Compose

## 機能

- プロジェクト管理
- タスク管理
- Gantt 表示
- スケジューリング計算
- リソース管理
- レポート
- エクスポート

## ディレクトリ構成

- `docs/` : 設計ドキュメント（MD 形式のみ）
- `rules/` : 開発ルール（人間のみ編集可能）
- `src/` : ソースコード
- `tools/` : Dockerfile, CI 設定
- `README.md` : プロジェクト説明
- `ABSOLUTE_RULES.md` : 絶対ルール（人間のみ）
- `PLAN.md` : 開発計画

## 開発フロー

1. 設計フェーズ
2. テスト製造
3. 実装フェーズ
4. テスト実行
5. 実装完了

## 技術注意

**採用禁止:**
- Vue.js/React/Angular（thymeleaf 必須）
- Oracle MySQL（PostgreSQL 必須）
- Kubernetes/Docker Swarm（Compose 必須）
- その他外製パッケージ

**コード品質:**
- TDD（テストファースト）
- コードに同量のコメント
- feature ブランチの単位で開発（50〜100 step）

## 利用

1. `docker-compose up` で起動
2. URL にアクセス
3. ログイン後、プロジェクト/タスク操作

## 開発者

倖太郎（Kotarou Yatabe）

---

[ABSOLUTE_RULES.md](ABSOLUTE_RULES.md) を必ず確認し、遵守する。
