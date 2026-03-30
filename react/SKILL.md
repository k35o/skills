---
name: react
description: Reactの正しい書き方を徹底するスキル。useRef/useStateの使い分け、useEffectの適切な使用と代替API、React 19の最新機能、Concurrent Modeに沿った記述を指導する。Reactコンポーネントの作成・修正・レビュー時に使用する。古いReactパターンを避け、現代のAsync Reactに沿ったコードを書く。
---

# React

正しいReactを書くためのスキル。パフォーマンス最適化ではなく、Reactのメンタルモデルに沿った正しいコードを書くことに重点を置く。

## 基本原則

### useStateとuseRefの使い分け

| 基準 | useState | useRef |
|------|----------|--------|
| レンダーに影響する値 | ✅ | ❌ |
| 変更時に再レンダーが必要 | ✅ | ❌ |
| タイムアウト/インターバルID | ❌ | ✅ |
| DOM要素への参照 | ❌ | ✅ |
| レンダーに不要な最新値の保持 | ❌ | ✅ |

**判断基準**: その値が画面に表示されるか？ → useState。表示されないが保持が必要か？ → useRef。

**refの鉄則**: レンダー中にref.currentを読み書きしない（一度きりの初期化パターンは例外）。

refでのDOM操作（フォーカス、スクロール、リスト内の複数ref、useImperativeHandle、flushSyncなど）の詳細は [references/dom-refs.md](references/dom-refs.md) を参照。

### useEffectの正しい使い方

useEffectは**外部システムとの同期**のためだけに使う。それ以外の用途は大抵間違い。

**正しい用途**:
- ブラウザAPI（IntersectionObserver、ResizeObserverなど）との同期
- サードパーティライブラリとの連携
- WebSocket/EventSourceへのサブスクリプション
- ネットワークリクエスト（データフェッチング）

**useEffectが不要な11のパターン**: [references/no-effect-patterns.md](references/no-effect-patterns.md) を参照。

### 依存配列のルール

- 依存配列は「選ぶ」ものではなく、コードから導出されるもの
- エフェクト内で使うリアクティブな値は全て含める
- リンタを無視しない。リンタが警告したらコードを修正する
- オブジェクトや関数を依存配列に入れない → エフェクト内で生成するか、プリミティブ値を抽出する

依存値の除去パターンの詳細は [references/effect-dependencies.md](references/effect-dependencies.md) を参照。

## React 19の機能を使う

古いパターンを避け、React 19の機能を積極的に使う。

### Actions

非同期処理の状態管理にはActionsを使う:

```tsx
const [isPending, startTransition] = useTransition();

function handleSubmit() {
  startTransition(async () => {
    await saveData();
  });
}
```

### useActionState

フォームのアクション管理に使う。pending状態、エラー、結果を自動管理する。

### useOptimistic

楽観的UIの実装に使う。リクエスト完了前にUIを即座に更新し、失敗時に自動で戻す。

### use

Promiseやコンテキストをレンダー中に読む。条件分岐内でも使える（useContextやuseStateと違い、条件内で呼べる）。

### ref as prop

`forwardRef`は不要。refは通常のpropとして受け取る:

```tsx
// ❌ 古い
const Input = forwardRef((props, ref) => <input ref={ref} {...props} />);

// ✅ React 19
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}
```

### Context

`<Context.Provider>`ではなく`<Context>`を直接使う:

```tsx
// ❌ 古い
<ThemeContext.Provider value={theme}>

// ✅ React 19
<ThemeContext value={theme}>
```

### refクリーンアップ

refコールバックからクリーンアップ関数を返せる:

```tsx
<div ref={(node) => {
  // セットアップ
  return () => {
    // クリーンアップ
  };
}} />
```

## Concurrent Mode

### useTransition

UIをブロックせずにstate更新する。重い再レンダーをバックグラウンドで実行:

```tsx
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setHeavyState(newValue); // UIをブロックしない
});
```

### useDeferredValue

高頻度で変わる値の更新を遅延させ、入力のレスポンシブさを保つ。

### Suspense

非同期データの読み込みを宣言的に扱う。ローディング状態の管理にuseState + useEffectを使わない。

## React Server Components

Server Componentsがデフォルト。`"use client"`をつけたものだけがClient Component。

**核心ルール:**
- Server ComponentsではuseState/useEffect/イベントハンドラは使えない
- `"use client"`の境界はできるだけ葉に近い位置に置く。ページ全体をClient Componentにしない
- Server Actionsの引数は信頼できない入力として扱い、認証・認可を必ず検証する
- サーバー↔クライアント間で渡すデータはシリアライズ可能でなければならない

詳細（"use client"/"use server"のルール、compositionパターン、セキュリティ、シリアライズ可能な型一覧）は [references/rsc.md](references/rsc.md) を参照。

## 避けるべきパターン

- `useMount`、`useEffectOnce`のようなライフサイクルフック
- `useEffect`内でのstate更新による派生値の計算
- インラインでのコンポーネント定義（レンダーごとにstateがリセットされる）
- `forwardRef`（React 19ではrefをpropとして渡す）
- `useEffect`でのイベントハンドラロジック
- `// eslint-disable-next-line react-hooks/exhaustive-deps`

## カスタムフック

- エフェクトを書くときは常にカスタムフックへの抽出を検討する
- カスタムフックは`use`で始め、大文字を続ける
- stateそのものではなく、stateを扱うロジックを共有する
- 1つのuseStateをラップするだけのフックは作らない

## リファレンス

- [useEffectが不要な11のパターン](references/no-effect-patterns.md)
- [エフェクト依存値の除去パターン](references/effect-dependencies.md)
- [DOM操作とrefパターン](references/dom-refs.md)
- [React Server Components / Server Actions](references/rsc.md)
