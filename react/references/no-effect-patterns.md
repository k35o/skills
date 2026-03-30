# useEffectが不要な11のパターン

useEffectを使いたくなったとき、以下のパターンに当てはまらないか確認する。

## 1. props/stateからの派生値

```tsx
// ❌
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// ✅ レンダー中に計算する
const fullName = firstName + ' ' + lastName;
```

## 2. 重い計算のキャッシュ

```tsx
// ❌
const [filteredList, setFilteredList] = useState([]);
useEffect(() => {
  setFilteredList(heavyFilter(items));
}, [items]);

// ✅ useMemoでメモ化する
const filteredList = useMemo(() => heavyFilter(items), [items]);
```

## 3. props変更時のstate全リセット

```tsx
// ❌
useEffect(() => {
  setComment('');
}, [userId]);

// ✅ keyで別コンポーネントにする
<Profile userId={userId} key={userId} />
```

## 4. props変更時の部分的なstate調整

```tsx
// ❌
useEffect(() => {
  setSelection(null);
}, [items]);

// ✅ レンダー中に前回の値と比較して調整する
const [prevItems, setPrevItems] = useState(items);
if (items !== prevItems) {
  setPrevItems(items);
  setSelection(null);
}
```

## 5. イベントハンドラ間のロジック共有

```tsx
// ❌ isInCartの変化で通知を表示
useEffect(() => {
  if (product.isInCart) {
    showNotification('カートに追加しました');
  }
}, [product.isInCart]);

// ✅ 共通関数をイベントハンドラから呼ぶ
function buyProduct() {
  addToCart(product);
  showNotification('カートに追加しました');
}
```

## 6. イベント固有のPOSTリクエスト

```tsx
// ❌
useEffect(() => {
  post('/api/register', { firstName, lastName });
}, [firstName, lastName]);

// ✅ イベントハンドラで送信する
function handleSubmit(e) {
  e.preventDefault();
  post('/api/register', { firstName, lastName });
}
```

## 7. 計算の連鎖（エフェクトチェーン）

```tsx
// ❌ 複数のエフェクトが連鎖的にstateを更新
useEffect(() => { setB(computeB(a)); }, [a]);
useEffect(() => { setC(computeC(b)); }, [b]);

// ✅ イベントハンドラで一度に計算する
function handleChange(a) {
  setA(a);
  setB(computeB(a));
  setC(computeC(computeB(a)));
}
```

## 8. アプリケーション初期化

```tsx
// ❌ エフェクトで初期化（Strict Modeで2回実行される）
useEffect(() => {
  loadAnalytics();
}, []);

// ✅ モジュールスコープで1回だけ実行
let didInit = false;
function App() {
  if (!didInit) {
    didInit = true;
    loadAnalytics();
  }
}
```

## 9. 親への状態変更通知

```tsx
// ❌
useEffect(() => {
  onChange(isOn);
}, [isOn, onChange]);

// ✅ イベントハンドラで両方更新する
function handleToggle(nextIsOn) {
  setIsOn(nextIsOn);
  onChange(nextIsOn);
}
```

## 10. 親へのデータ受け渡し

```tsx
// ❌ 子がエフェクトで親のstateを更新
function Child({ onData }) {
  const data = useSomeAPI();
  useEffect(() => { onData(data); }, [data, onData]);
}

// ✅ 親がデータを取得してpropsで渡す
function Parent() {
  const data = useSomeAPI();
  return <Child data={data} />;
}
```

## 11. 外部ストアへのサブスクライブ

```tsx
// ❌
const [isOnline, setIsOnline] = useState(true);
useEffect(() => {
  const handler = () => setIsOnline(navigator.onLine);
  window.addEventListener('online', handler);
  window.addEventListener('offline', handler);
  return () => {
    window.removeEventListener('online', handler);
    window.removeEventListener('offline', handler);
  };
}, []);

// ✅ useSyncExternalStoreを使う
const isOnline = useSyncExternalStore(
  (callback) => {
    window.addEventListener('online', callback);
    window.addEventListener('offline', callback);
    return () => {
      window.removeEventListener('online', callback);
      window.removeEventListener('offline', callback);
    };
  },
  () => navigator.onLine,
  () => true // SSR用
);
```
