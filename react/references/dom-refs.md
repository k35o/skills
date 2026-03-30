# DOM操作とrefパターン

## 基本: DOM要素への参照

```tsx
const inputRef = useRef<HTMLInputElement>(null);
return <input ref={inputRef} />;
// inputRef.currentでDOM要素にアクセス
```

## フォーカス管理

```tsx
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);
  return (
    <>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>
        フォーカス
      </button>
    </>
  );
}
```

## スクロール操作

```tsx
ref.current?.scrollIntoView({
  behavior: 'smooth',
  block: 'nearest',
  inline: 'center'
});
```

## リスト内の複数要素へのref（refコールバック）

ループ内ではuseRefを呼べないため、refコールバックとMapを使う。

```tsx
function List({ items }) {
  const mapRef = useRef<Map<string, HTMLElement>>(new Map());

  return items.map(item => (
    <li
      key={item.id}
      ref={(node) => {
        if (node) {
          mapRef.current.set(item.id, node);
        } else {
          mapRef.current.delete(item.id);
        }
      }}
    >
      {item.text}
    </li>
  ));
}
```

## 子コンポーネントのDOMへのアクセス

### React 19: refをpropとして渡す

```tsx
// 子
function MyInput({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}

// 親
const inputRef = useRef(null);
<MyInput ref={inputRef} />
```

### useImperativeHandle: 公開APIの制限

子コンポーネントのDOMを直接公開せず、必要なメソッドだけを公開する。

```tsx
function MyInput({ ref }) {
  const realRef = useRef<HTMLInputElement>(null);

  useImperativeHandle(ref, () => ({
    focus() {
      realRef.current?.focus();
    },
    // scrollIntoViewやvalueなどは公開しない
  }));

  return <input ref={realRef} />;
}
```

## flushSync: DOM更新の即座反映

state更新後にDOMを参照する必要がある場合、flushSyncで同期的にDOMを更新する。

```tsx
import { flushSync } from 'react-dom';

function handleAdd(text: string) {
  flushSync(() => {
    setItems([...items, { id: nextId++, text }]);
  });
  // この時点でDOMは更新済み
  lastRef.current?.scrollIntoView();
}
```

## refクリーンアップ（React 19）

refコールバックからクリーンアップ関数を返せる。

```tsx
<div ref={(node) => {
  // ノードがDOMに追加されたとき
  const observer = new IntersectionObserver(callback);
  if (node) observer.observe(node);

  return () => {
    // ノードがDOMから削除されたとき
    observer.disconnect();
  };
}} />
```

## DOM操作の安全ルール

**安全な操作（読み取り専用）:**
- フォーカス管理: `focus()`, `blur()`
- スクロール: `scrollIntoView()`, `scrollTo()`
- 測定: `getBoundingClientRect()`, `offsetWidth`

**危険な操作（Reactが管理するDOMの変更）:**
- `removeChild()` — ReactのDOMツリーと不整合になる
- `innerHTML` の直接変更
- `appendChild()` でReact管理ツリーにノードを追加

**例外**: Reactが更新する必要のない部分（常に空の`<div>`の中身など）は自由に変更可能。
