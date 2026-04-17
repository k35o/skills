---
name: codex-reviewer
description: OpenAI Codex CLIを使ってコードや計画に対する外部視点のレビューを取得する。計画の妥当性確認、複雑な問題で行き詰まったとき、アーキテクチャ判断のセカンドオピニオン、難しいデバッグのときに使用する。「Codexに聞いて」「セカンドオピニオンが欲しい」などのリクエストにも対応する。
---

# Codex Reviewer

`codex exec` CLIを使い、外部視点からのレビューを取得する。

## 使い方

### レビュー依頼

Bashツールで `codex exec` を実行する:

```bash
codex exec --full-auto -C <対象ディレクトリ> -o /tmp/codex-review.txt "<レビュー依頼プロンプト>"
```

- `--full-auto`: サンドボックス内で自動実行
- `-C`: 対象プロジェクトのディレクトリ
- `-o`: 結果をファイルに出力（Readツールで読み取る）
- `--ephemeral`: セッションを永続化しない場合に付与

### コードレビュー専用

リポジトリの変更に対するレビューには `codex exec review` を使う:

```bash
# 未コミットの変更をレビュー
codex exec review --full-auto --uncommitted -o /tmp/codex-review.txt

# 特定ブランチとの差分をレビュー
codex exec review --full-auto --base main -o /tmp/codex-review.txt

# 特定コミットをレビュー
codex exec review --full-auto --commit <SHA> -o /tmp/codex-review.txt

# カスタム指示付き
codex exec review --full-auto --uncommitted -o /tmp/codex-review.txt "セキュリティの観点で確認してください"
```

### 結果の取得

`-o` で指定したファイルをReadツールで読み取り、ユーザーに要約して伝える。

## レビュー依頼の書き方

効果的なレビューのために、以下を含める:

```markdown
[依頼内容]

## 背景
[問題や要件、制約条件]

## 現状のアプローチ
[現在の計画や試したこと]

## 懸念点
[認識している問題点や判断に迷っている点]
```

## 使用タイミング

- **計画のレビュー**: Planモードで計画を立てた後、実行前に妥当性を確認する
- **行き詰まり**: 問題解決の糸口が見つからないとき
- **アーキテクチャ判断**: 複数の選択肢で迷っているとき
- **デバッグ困難**: 根本原因の特定が難しいとき
- **コードレビュー**: 変更内容のレビューが必要なとき

## 注意事項

- Codexの提案は参考意見として扱い、最終判断は自分で行う
- 機密性の高いコードやデータは送信しない
