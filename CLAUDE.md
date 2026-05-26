# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

このリポジトリは **YouTube競合動画収集スキル** を提供する Claude Code スキルリポジトリです。

| 項目 | 内容 |
|---|---|
| スキル名 | youtube-collector-shiftAI |
| 対象チャンネル | ShiftAI チャンネルグループ（デフォルト） |
| 処理内容 | 競合動画の収集 / スプレッドシート追記 / ハイライト |
| 必要なツール | WebFetch（Claude Code 標準） |
| 外部依存 | Google Apps Script / Google スプレッドシート |

トリガーワードを入力するだけで、Google Apps Script 経由で競合チャンネルの新着動画を収集し、Google スプレッドシートへの記録・ハイライト処理を自動実行します。

---

## リポジトリ構成と役割

| ファイル | 役割 | 編集の要否 |
|---|---|---|
| `skills.md` | Claude Code が読み込むスキル定義。実行ステップ（STEP 0〜4）を記述。 | 他チャンネル対応時のみ |
| `config.txt` | チャンネル固有の設定値（`APPS_SCRIPT_URL` / `SPREADSHEET_ID`）。スキルが WebFetch で取得する。 | 他チャンネル対応時は必須 |
| `使用ガイド.docx` | セットアップ手順・トラブルシューティングをまとめた利用者向けドキュメント。 | 不要 |

---

## スキルの起動方法

このスキルは Claude Code（ブラウザ版）からリポジトリを接続することで利用できます。

| 手順 | 操作 |
|---|---|
| ① ブラウザで Claude Code を開く | https://claude.ai を開き、Claude Code を起動する |
| ② 「リポジトリを追加」ボタンをクリック | チャット入力欄の上に表示されているボタンをクリック |
| ③ リポジトリを接続する | `https://github.com/aimaruai5-crypto/youtube-collector-shiftAI` を入力 |
| ④ トリガーワードを入力する | 「動画を集めて」などのトリガーワードを入力するとスキルが自動実行 |

> リポジトリを一度接続すれば、以降は毎回接続し直す必要はありません。

---

## スキルの実行フロー（STEP 0〜4）

「動画を集めて」などと入力するとスキルが起動し、以下の手順を自動実行します。

| STEP | 処理内容 | 詳細 |
|---|---|---|
| STEP 0 | config.txt の読み込み | GitHub の raw URL から `config.txt` を WebFetch で取得し、`APPS_SCRIPT_URL` と `SPREADSHEET_ID` を取り出す |
| STEP 1 | ハイライト条件の取得 | `conditions_doc_url.txt` に記載の Google Doc URL を取得してハイライト条件を読み込む（ファイルがなければデフォルト値を使用） |
| STEP 2 | Apps Script の呼び出し | STEP 0 の URL と STEP 1 の条件値でリクエストを送信し、動画収集・スプレッドシート書き込みを実行する |
| STEP 3 | リダイレクト先を取得 | Apps Script から返ってくるリダイレクト先（`script.googleusercontent.com`）を WebFetch で取得する |
| STEP 4 | 結果レポート | 処理件数（新着動画数・ハイライト数など）を日本語で報告する |

### STEP 2 のリクエスト URL 構造

```
{APPS_SCRIPT_URL}
  ?action=collect
  &spreadsheetId={SPREADSHEET_ID}
  &minViews={minViews}
  &minViewMult={minViewMult}
  &minViralRate={minViralRate}
  &minMatch={minMatch}
```

STEP 2 で `REDIRECT DETECTED` が返ってくるのは正常動作です。

### STEP 4 の出力形式

```
✅ 完了しました！
- チャンネル: X件処理（ショート除外済み）
- 新着動画: Y件追記
- ハイライト: Z件（黄色）
- 競合CH更新: W件（登録者数・平均再生回数）
```

エラー時は `error` フィールドの内容をそのまま伝える。

---

## ハイライト条件

| パラメータ | デフォルト値 | 意味 |
|---|---|---|
| `minViews` | 30,000 | 再生数の下限 |
| `minViewMult` | 2.0 | チャンネル平均比の下限（平均の2倍以上） |
| `minViralRate` | 20.0% | 拡散率の下限（再生数 / 登録者数） |
| `minMatch` | 3 | 上記4条件のうち何個以上満たした場合にハイライトするか |

条件をカスタマイズしたい場合は `conditions_doc_url.txt` を作成して Google Doc の URL を記載します。

---

## トリガーワード一覧

| トリガーワード | 動作 |
|---|---|
| 動画を集めて | 競合チャンネルの新着動画を収集してスプレッドシートに追記 |
| 新着動画を更新して | 同上 |
| ハイライトして | ハイライト条件に該当する動画を黄色でマーク |
| 競合チャンネルの動画 | 同上 |
| 新着動画シートを更新 | 同上 |
| YouTubeワークフローを実行 | 動画収集からハイライトまで一括実行 |
| 全部やって | 同上 |

---

## config.txt の設定項目

`config.txt` に設定する値は2つのみです。`#` 始まりの行はコメントとして無視されます。

### APPS_SCRIPT_URL

Google Apps Script のデプロイURLです。このスクリプトが競合チャンネルの動画収集・スプレッドシートへの書き込み・ハイライト処理を実行します。

```
APPS_SCRIPT_URL=https://script.google.com/macros/s/(ID)/exec
```

**Apps Script URL の取得方法:**

1. Google Apps Script（script.google.com）でプロジェクトを開く
2. 右上の「デプロイ」→「新しいデプロイ」をクリック
3. 種類「ウェブアプリ」を選択
4. 「アクセスできるユーザー」を **「全員」** に設定
5. 「デプロイ」ボタンを押す
6. 表示された「ウェブアプリ URL」をコピーして `config.txt` に貼り付ける

### SPREADSHEET_ID

動画データを書き込む Google スプレッドシートのIDです。

```
SPREADSHEET_ID=<スプレッドシートURLの /d/〜/edit の間の文字列>
```

例: `https://docs.google.com/spreadsheets/d/【ここがID】/edit`

> Apps Script のデプロイに使用したアカウントに、スプレッドシートの「編集者」権限が必要です。

---

## 別チャンネルへの展開

このリポジトリをフォークして `config.txt` を書き換えることで、他のチャンネルでも同じスキルが使えます。

1. GitHub でこのリポジトリをフォークする
2. フォーク先の `config.txt` を開き、`APPS_SCRIPT_URL` と `SPREADSHEET_ID` を書き換える
3. `skills.md` の STEP 0 にある raw URL をフォーク先の URL に変更する

```
# 変更前
https://raw.githubusercontent.com/aimaruai5-crypto/youtube-collector-shiftAI/main/config.txt

# 変更後
https://raw.githubusercontent.com/[あなたのユーザー名]/[リポジトリ名]/main/config.txt
```

4. Claude Code でスキルとして登録すれば完成

---

## トラブルシューティング

| 症状 | 原因 | 対処法 |
|---|---|---|
| リダイレクト先が `accounts.google.com` | Apps Script のアクセス設定が「自分のみ」 | 「アクセスできるユーザー」を「全員」に変更して再デプロイ |
| `error: スプレッドシートを開けません` | デプロイアカウントに編集権限がない | スプレッドシートの共有設定でデプロイアカウントを「編集者」として追加 |
| タイムアウトが発生する | Apps Script の6分制限を超過 | 登録チャンネル数を確認し、多すぎる場合は分割処理を検討 |
| `config.txt` が読み込めない | STEP 0 の raw URL がリポジトリと一致していない | `skills.md` の STEP 0 の URL がフォーク先の正しい raw URL になっているか確認 |

---

## 関連リンク

- GitHub リポジトリ: https://github.com/aimaruai5-crypto/youtube-collector-shiftAI
- Google Apps Script: https://script.google.com

