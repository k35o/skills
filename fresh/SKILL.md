---
name: fresh
description: freshターミナルエディタでファイルを開く操作。ユーザーが「freshで開いて」「このファイルをエディタで見たい」「編集したい」と言ったとき、cmux経由でfreshを別ペインに起動してファイルを開く。
---

# Fresh エディタ連携

freshはターミナルテキストエディタ。Claudeはcmux経由でfreshを別ペインに開き、ユーザーが直接編集できるようにする。

## ファイルを開く

cmuxスキルを併用する。

```bash
# 1. 右ペインを作る
cmux new-split right

# 2. freshでファイルを開く
cmux send --surface surface:N "fresh src/main.rs:42\n"
```

行・列指定が可能:
```bash
fresh src/main.rs              # ファイルを開く
fresh src/main.rs:42           # 42行目を開く
fresh src/main.rs:42:10        # 42行目10列目を開く
fresh Cargo.toml src/lib.rs    # 複数ファイルを開く
```

## セッション

```bash
fresh -a myproject             # 名前付きセッションにアタッチ/作成
fresh --cmd session list       # 既存セッション一覧
fresh --cmd session open-file myproject src/main.rs  # 実行中セッションにファイル追加
```

セッションはターミナル切断後も維持される。

## 使い分け

- Claudeが直接コードを編集する → Read/Edit ツールを使う
- ユーザーが自分で確認・編集したい → freshで開く
