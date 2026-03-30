---
name: cmux
description: cmuxターミナル内での操作スキル。分割ペインでの並行作業、ブラウザ自動化、通知の送信、トポロジーナビゲーションを行う。CMUX_*環境変数が存在する場合、サブエージェントの隔離実行、Web自動化、完了通知が必要なときに使用する。
---

# cmux

cmuxはAIコーディングエージェント向けのmacOSネイティブターミナル。`CMUX_SOCKET_PATH`が設定されていればcmux内で実行中。

## 基本

```bash
cmux identify --json    # 現在の位置（workspace, pane, surface）
cmux list-workspaces    # 全workspaceの一覧
```

**階層:** Window → Workspace（サイドバータブ） → Pane（分割） → Surface（ターミナル/ブラウザ）

**Ref形式:** `workspace:2`, `pane:1`, `surface:7`（UUIDより優先して使う）

## 分割ペイン

長時間タスクやサブエージェントの隔離実行に使う。数秒で終わるコマンドには不要。

```bash
cmux new-split right                    # 垂直分割
cmux new-split down                     # 水平分割
cmux focus-pane --pane pane:2           # ペイン間移動
cmux send --surface surface:5 "command\n"  # 特定surfaceにコマンド送信
```

## ブラウザ自動化

**基本フロー:** Open → Snapshot（ref取得） → refで操作 → 変更を待機

```bash
# 開く
cmux browser open https://example.com --json

# snapshotでelement refを取得（常に--interactiveをつける）
cmux browser surface:7 snapshot --interactive

# refで操作
cmux browser surface:7 click e2
cmux browser surface:7 fill e3 "text"
cmux browser surface:7 select e7 "value"

# 待機
cmux browser surface:7 wait --selector "#loaded" --timeout-ms 10000
cmux browser surface:7 wait --text "Success"

# データ取得
cmux browser surface:7 get url
cmux browser surface:7 get text e3
cmux browser surface:7 eval 'document.title'
```

**注意:** ナビゲーション後はrefが無効になるので再snapshot。CSSセレクタではなくelement refを使う。

## 通知

| 用途 | コマンド |
|------|---------|
| cmux内の通知（ペインリング） | `cmux notify --title "T" --body "B"` |
| システム通知（他アプリにいるとき） | `osascript -e 'display notification "B" with title "T" sound name "Submarine"'` |

## 環境変数

| 変数 | 用途 |
|------|------|
| `CMUX_SOCKET_PATH` | 制御ソケット |
| `CMUX_WORKSPACE_ID` | ワークスペースUUID |
| `CMUX_SURFACE_ID` | サーフェスUUID |

フラグ省略時は現在のworkspace/surfaceがデフォルト。
