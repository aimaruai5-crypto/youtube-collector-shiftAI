# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリの目的

このリポジトリは **YouTube競合動画収集スキル** を提供する Claude Code スキルリポジトリです。トリガーワードを入力するだけで、Google Apps Script 経由で競合チャンネルの新着動画を収集し、Google スプレッドシートへの記録・ハイライト処理を自動実行します。

## リポジトリ構成と役割

| ファイル | 役割 |
|---|---|
| `skills.md` | Claude Code が読み込むスキル定義。実行ステップ（STEP 0〜4）を記述。 |
| `config.txt` | チャンネル固有の設定値（`APPS_SCRIPT_URL` / `SPREADSHEET_ID`）。スキルが WebFetch で取得する。 |
| `使用ガイド.docx` | セットアップ手順・トラブルシューティングをまとめた利用者向けドキュメント。 |

## スキルの実行フロー

1. **STEP 0**: GitHub raw URL から `config.txt` を WebFetch で取得し、`APPS_SCRIPT_URL` と `SPREADSHEET_ID` をパース
2. **STEP 1**: `conditions_doc_url.txt`（存在すれば）に記載の Google Doc URL からハイライト条件を取得。なければデフォルト値を使用（minViews=30000, minViewMult=2.0, minViralRate=20.0, minMatch=3）
3. **STEP 2**: `APPS_SCRIPT_URL?action=collect&spreadsheetId=...&minViews=...` を WebFetch で呼び出す
4. **STEP 3**: レスポンスのリダイレクト先（`script.googleusercontent.com`）を WebFetch で取得
5. **STEP 4**: 処理件数（新着動画数・ハイライト数など）を日本語で報告

## スキルのトリガーワード

「動画を集めて」「新着動画を更新して」「ハイライトして」「競合チャンネルの動画」「新着動画シートを更新」「YouTubeワークフローを実行」「全部やって」

## config.txt の編集

`config.txt` に設定する値は2つのみ：

```
APPS_SCRIPT_URL=<Google Apps Script のデプロイURL（アクセス:全員）>
SPREADSHEET_ID=<スプレッドシートURLの /d/〜/edit の間の文字列>
```

`#` 始まりの行はコメント。`skills.md` の STEP 0 にある raw URL もフォーク先に合わせて変更が必要。

## 別チャンネルへの展開

1. GitHub でこのリポジトリをフォーク
2. フォーク先の `config.txt` を書き換え
3. `skills.md` STEP 0 内の raw URL をフォーク先の URL に変更

## トラブルシューティング

| 症状 | 対処 |
|---|---|
| リダイレクト先が `accounts.google.com` | Apps Script のデプロイ設定「アクセスできるユーザー」を「全員」に変更して再デプロイ |
| `error: スプレッドシートを開けません` | デプロイアカウントにスプレッドシートの編集者権限を付与 |
| タイムアウト | Apps Script の6分制限超過。登録チャンネル数を確認 |
| `config.txt` が読み込めない | `skills.md` STEP 0 の raw URL が正しいか確認 |
