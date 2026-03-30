---
name: difit
description: difitを使ったローカルコードレビュー。コード変更後にユーザーにレビューを促す、特定のdiffやPRをレビューしてコメント付きで表示する。コード変更のレビュー、diff確認、PR確認のときに使用する。
---

# difit

ローカルのGitコミット差分をGitHub風のWebUIで表示・レビューするCLIツール。

## 基本コマンド

```bash
difit .                          # コミット前の未コミット変更
difit                            # HEADコミット
difit staged                     # ステージング済み変更
difit working                    # ステージされていない変更のみ
difit <commit>                   # 特定コミットのdiff
difit <target> <compare-with>    # 2つのコミット/ブランチの比較
```

## コメント付きレビュー

diffにコメントを付けて起動できる。レビュー所見やコード説明に有用。

```bash
difit <target> \
  --comment '{"type":"thread","filePath":"src/foo.ts","position":{"side":"new","line":42},"body":"ここは修正が必要"}' \
  --comment '{"type":"thread","filePath":"src/bar.ts","position":{"side":"new","line":{"start":10,"end":15}},"body":"この範囲の処理について"}'
```

**コメントのルール:**
- `type` は常に `"thread"`
- `position.side`: 変更後の行は `"new"`、削除された行は `"old"`
- 複数行にまたがる場合は `line: {"start": N, "end": M}` で範囲指定
- ユーザーの使用言語でbodyを書く
- シークレットやクレデンシャルをコメントに含めない

## 未追跡ファイルの表示

```bash
difit . --include-untracked
```

## 起動方法

difitはサーバーを起動してブラウザで表示するツール。バックグラウンドで起動する。

```bash
difit <target> > /tmp/difit-output.txt 2>&1 &
sleep 2
cat /tmp/difit-output.txt  # URLを確認
```

出力からURLを取得し、ブラウザで開く。cmux内であれば `cmux browser open <url>` を使う。

## レビューフロー

1. コード変更後、difitをバックグラウンドで起動しユーザーにレビューを促す
2. ユーザーがレビューコメントを残すと、difitの終了時にstdoutに出力される
3. コメントがあれば、その内容に対応して作業を続ける
4. コメントなしでサーバーが停止した場合は「コメントなし」として扱う

difitのページが正しく起動したかの手動確認は不要。

## PRレビュー

PRをレビューする場合はローカルで差分を確認し、結果をdifitのコメントとして表示する。リモートのGitHubにコメントを投稿しない。
