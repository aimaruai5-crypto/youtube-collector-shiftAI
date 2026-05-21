# YouTube競合動画収集スキル

Use when the user wants to collect new YouTube videos from competitor channels into a spreadsheet, highlight collected videos, or run the weekly workflow.

Triggers include: "動画を集めて", "新着動画を更新して", "ハイライトして", "競合チャンネルの動画", "新着動画シートを更新", "YouTubeワークフローを実行", "全部やって".

---

## 設定ファイルの場所

このスキルはリポジトリ内の `config.txt` からチャンネル固有の設定を読み込みます。

**config.txt の raw URL（このリポジトリ用）:**
```
https://raw.githubusercontent.com/aimaruai5-crypto/youtube-collector-shiftAI/main/config.txt
```

> 別のチャンネルで使う場合: リポジトリをフォークして `config.txt` の値を書き換え、
> 上記 URL をフォーク先の raw URL に変更してください。

---

## 実行手順

### STEP 0: config.txt を読み込む

以下の URL を WebFetch で取得し、`APPS_SCRIPT_URL` と `SPREADSHEET_ID` を取り出す:

```
https://raw.githubusercontent.com/aimaruai5-crypto/youtube-collector-shiftAI/main/config.txt
```

パース例:
- `APPS_SCRIPT_URL=` で始まる行の `=` 以降が Apps Script の URL
- `SPREADSHEET_ID=` で始まる行の `=` 以降がスプレッドシート ID
- `#` で始まる行はコメントなので無視する

---

### STEP 1: ハイライト条件を読み取る

プロジェクトの `conditions_doc_url.txt` に記載されている Google Doc URL を WebFetch で取得して内容を読む。

以下のデフォルト値と比較し、Doc に書かれた条件で上書きする:

| パラメータ | デフォルト | 意味 |
|---|---|---|
| `minViews` | 30000 | 再生数の下限 |
| `minViewMult` | 2.0 | 平均比の下限 |
| `minViralRate` | 20.0 | 拡散率%の下限 |
| `minMatch` | 3 | 4条件中いくつ以上でハイライトするか |

`conditions_doc_url.txt` がない場合はデフォルト値を使用する。

---

### STEP 2: Apps Script を呼び出す（WebFetch）

STEP 0 で取得した `APPS_SCRIPT_URL` と `SPREADSHEET_ID`、STEP 1 の条件値を使って URL を構築し WebFetch で取得する:

```
{APPS_SCRIPT_URL}
  ?action=collect
  &spreadsheetId={SPREADSHEET_ID}
  &minViews={minViews}
  &minViewMult={minViewMult}
  &minViralRate={minViralRate}
  &minMatch={minMatch}
```

→ `REDIRECT DETECTED` が返ってくる（正常）

---

### STEP 3: リダイレクト先を WebFetch で取得

`script.googleusercontent.com/...` の URL を WebFetch で取得する。

---

### STEP 4: 結果を日本語で報告

```
✅ 完了しました！
- チャンネル: X件処理（ショート除外済み）
- 新着動画: Y件追記
- ハイライト: Z件（黄色）
- 競合CH更新: W件（登録者数・平均再生回数）
```

エラー時: `error` フィールドの内容をそのまま伝える。

---

## 別チャンネルへの対応方法

1. このリポジトリを GitHub でフォークする
2. `config.txt` を開き、`APPS_SCRIPT_URL` と `SPREADSHEET_ID` を自分のチャンネル用の値に書き換える
3. `skills.md` の STEP 0 にある raw URL をフォーク先のリポジトリの URL に書き換える
   - 例: `https://raw.githubusercontent.com/your-username/your-repo/main/config.txt`
4. Claude Code でスキルとして登録すれば完成

---

## トラブルシューティング

| 症状 | 対処 |
|---|---|
| リダイレクト先が `accounts.google.com` | Apps Script のデプロイ設定を「全員」に変更して再デプロイ |
| `error: スプレッドシートを開けません` | デプロイアカウントにスプレッドシートの編集権限を付与 |
| タイムアウト | Apps Script の6分制限超過。チャンネル数を確認 |
| config.txt が読み込めない | STEP 0 の raw URL がリポジトリのものと一致しているか確認 |
