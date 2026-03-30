---
name: commit-creator
description: Conventional Commitsに従ったアトミックなgitコミットを作成する。変更を論理単位に分割し、独立してrevert可能なコミットを生成する。コードの変更をコミットしたいとき、コミットメッセージを作りたいときに使用する。
---

# Commit

独立してrevert可能なアトミックコミットを、Conventional Commits仕様に従って作成する。

## ワークフロー

1. `git status` と `git diff` で変更を確認する
2. `git log --oneline -20` で既存のコミットパターン（構造、スコープ命名、メッセージスタイル、言語）を確認する
3. 各hunkを個別に調べ、独立してrevert可能な単位を特定する
4. 単位ごとに:
   - `git diff <file>` で対象hunkを確認する
   - 必要なhunkだけのパッチを作成する
   - `git apply -v` でパッチを適用する
   - `git add <file>` でステージする
   - コミットメッセージを作成してコミットする
   - `git show HEAD` で確認する

**`git add -p` や `git add --interactive` は使用不可。** Claude Codeは対話的コマンドに対応していない。

## パッチ適用

```bash
# 適用前に必ず検証する
git apply --check patch.patch

# 適用（常に-vをつける）
git apply -v patch.patch

# ホワイトスペース問題がある場合
git apply --whitespace=fix -v patch.patch

# 部分的に失敗する場合
git apply --reject -v patch.patch
```

## コミットメッセージ形式

```
<type>(<scope>): <subject>

<body>
```

**type**: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

**body**: コミット前後の差分がわかるように書く。何が変わったか、なぜ変えたかを後から見返せる形で記述する。72文字で折り返す。

**禁止**: ユーザーの指示をそのままコミットメッセージに転記しない。「〜してください」「〜を修正して」などの依頼文をメッセージにしない。変更の事実と理由を客観的に記述する。

## 原則

- **言語**: `git log` を確認し、直近のコミットが日本語なら日本語、そうでなければ英語で書く
- 迷ったら小さくコミットする（後でsquashできるが、分割は難しい）
- プロジェクトのスコープ命名規則に合わせる
- issue/PR参照を含める
- 各コミットが「これをrevertしたら他の機能が壊れるか？」を満たすこと
