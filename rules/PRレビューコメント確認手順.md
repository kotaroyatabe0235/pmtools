GitHub PRレビューコメント取得手順

背景

OpenClawはGitHub CLI (gh) を利用してPull Requestの情報を取得できるが、環境や認証状態によってはレビューコメントを正しく取得できない場合がある。

その結果、以下のような問題が発生する。

* AIがレビュー指摘を認識できない
* 修正対象が分からないまま実装を進める
* 「対応済み」のつもりでもレビュー内容を反映できていない

症状

AIに以下のような指示を出してもレビューコメントを取得できない。

PRのレビューコメントを確認して対応してください

または

GitHubの指摘内容を確認して修正してください

対処方法

1. GitHub CLIでPRレビューコメントを取得する

PR番号を確認する。

gh pr view <PR番号> —comments

例:

gh pr view 123 —comments

2. レビュー情報をJSONで取得する

gh pr view <PR番号> \
  —json reviews,comments

例:

gh pr view 123 \
  —json reviews,comments

3. レビューコメントをファイルへ保存する

gh pr view <PR番号> \
  —json reviews,comments \
  > pr-review.json

4. AIへ共有する

取得した内容をそのままAIへ渡す。

例:

以下はGitHubレビューコメントです。
内容を確認し、指摘事項を分類してください。
<レビュー内容貼り付け>

または

以下のレビューコメントに対応するための修正案を作成してください。
<レビュー内容貼り付け>

推奨運用

レビュー対応時は以下の流れを推奨する。

1. GitHub上でレビューを受ける
2. gh pr view —comments で内容取得
3. AIへ共有
4. AIに修正案作成を依頼
5. 修正実施
6. 差分レビュー
7. Push

補足

PR本文だけではレビュー指摘は取得できない。

レビューコメントは以下のいずれかに存在する。

* Review Comments
* Review Threads
* General Comments
* Review Summary

AIが指摘を認識していない場合は、必ずGitHub CLIで取得したレビュー内容を明示的に渡すこと。