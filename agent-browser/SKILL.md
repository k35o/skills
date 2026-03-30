---
name: agent-browser
description: agent-browser（Rust製ヘッドレスブラウザCLI）を使ったブラウザ自動化。Webページの操作、スクレイピング、フォーム入力、スクリーンショット取得、E2Eテスト的な操作を行うときに使用する。
---

# agent-browser

AIエージェント向けの高速なRust製ヘッドレスブラウザCLI。インストール済み。

## 基本操作

```bash
agent-browser open <url>              # ページを開く
agent-browser snapshot                # アクセシビリティツリーとref IDを取得
agent-browser screenshot [path]       # スクリーンショット
```

## 要素の検索

セマンティックロケータで要素を特定する:

```bash
agent-browser find role button --name "Submit"
agent-browser find text "ログイン"
agent-browser find label "メールアドレス"
agent-browser find placeholder "検索"
agent-browser find testid "submit-btn"
```

## 操作

```bash
agent-browser click <selector>        # クリック
agent-browser fill <selector> <text>  # フォーム入力
agent-browser press Enter             # キー入力
agent-browser scroll --dy 500         # スクロール
```

## データ取得

```bash
agent-browser get url                 # 現在のURL
agent-browser get text <selector>     # テキスト内容
agent-browser get attr <selector> <attr>  # 属性値
```

## 待機

```bash
agent-browser wait <selector>                # 要素の出現待ち
agent-browser wait --text "完了"              # テキスト出現待ち
agent-browser wait --load networkidle        # ページ読み込み完了待ち
```

## ストレージ・ネットワーク

```bash
agent-browser cookies                 # クッキー管理
agent-browser storage local           # localStorage操作
agent-browser network requests        # リクエスト追跡
```

## バッチ実行

複数コマンドをJSON形式で一括実行し、起動オーバーヘッドを削減できる。

## 基本フロー

1. `open` でページを開く
2. `snapshot` でアクセシビリティツリーとref IDを取得
3. `find` や ref IDで要素を特定
4. `click`, `fill` などで操作
5. `wait` で結果を待機
6. 必要に応じて `screenshot` や `get` で結果を確認
