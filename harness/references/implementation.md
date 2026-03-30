# 実装ガイド

Claude Codeでharness パターンを実践する方法。

## subagentによる分離

Claude Codeのsubagent（Agent tool）で3エージェントを分離する。

### Plannerの起動

```
Agent tool:
  prompt: |
    以下のタスクについて詳細な仕様を作成してください。
    タスク: [ユーザーのリクエスト]

    出力:
    1. harness/plan.md に全体仕様を書く
    2. harness/sprints/sprint-N.md に各スプリントの仕様と完了条件を書く

    ルール:
    - スコープは野心的に、実装詳細は避ける
    - 各スプリントの完了条件は具体的かつ検証可能にする
    - スプリントは1つあたり30分〜1時間程度の粒度にする
```

### Generatorの起動

```
Agent tool:
  prompt: |
    harness/sprints/sprint-N.md の仕様に基づいて実装してください。

    ルール:
    - 仕様の完了条件を全て満たすこと
    - 完了したらgit commitする
    - harness/sprints/sprint-N-self-eval.md に自己評価を書く
```

### Evaluatorの起動

```
Agent tool:
  prompt: |
    harness/sprints/sprint-N.md の完了条件に基づいて、
    実装を評価してください。

    ルール:
    - 実際にコードを実行して動作確認する
    - 完了条件を1つずつ検証し、pass/failを記録する
    - 問題があればharness/sprints/sprint-N-eval.md に修正指示を書く
    - Generatorの自己評価と独立して判断すること
```

## 簡易版: 2エージェント

小〜中規模のタスクでは、PlannerとEvaluatorを省略し、以下で十分な場合がある:

1. メインエージェントが計画と実装を行う
2. 別のsubagentが評価だけを行う

**重要なのは「生成と評価の分離」**。最低限これだけ守る。

## worktreeの活用

subagentをworktreeで起動すると、メインブランチを汚さずに実験できる:

```
Agent tool:
  isolation: worktree
  prompt: Sprint 1の実装...
```

## 品質ループの組み込み

各スプリントの流れ:

```
Plan → Generate → Self-Eval → External Eval → Fix → Commit
  ↑                                              |
  └──────────── 修正が必要な場合 ────────────────┘
```

Evaluatorが不合格を出した場合、Generatorに修正指示を戻してループする。最大3回ループしても解決しない場合はユーザーに判断を仰ぐ。

## プロジェクト品質ゲートとの統合

参考スキル（apply-harness-practices）のMVH Checklistの考え方を取り入れる:

### Stage 1（即座に適用）
- CLAUDE.md/AGENTS.mdを簡潔に保つ（50行以下、ポインタ型）
- PostToolUseフックでリンター/フォーマッター自動実行

### Stage 2（短期改善）
- Pre-commitフックでリンター・型チェック
- Stopフックでテスト通過を完了条件化

### Stage 3（中期高度化）
- カスタムリンターにWHY/FIX/EXAMPLEを含むエラーメッセージ
- PreToolUseで機密ファイル編集をブロック

適用はプロジェクトの状況に応じて段階的に行う。全てを一度に入れる必要はない。
