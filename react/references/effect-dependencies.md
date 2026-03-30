# エフェクト依存値の除去パターン

依存配列を減らしたいときは、リンタを無視するのではなくコードを修正する。

## 1. イベントハンドラへの移動

特定のユーザー操作に応答するコードはエフェクトではなくイベントハンドラに置く。

```tsx
// ❌ エフェクトでフォーム送信
useEffect(() => {
  post('/api/submit', { data });
}, [data]);

// ✅ イベントハンドラで送信
function handleSubmit() {
  post('/api/submit', { data });
}
```

## 2. エフェクトの分割

独立した同期プロセスを1つのエフェクトにまとめない。

```tsx
// ❌ 国が変わっても都市のフェッチが再実行される
useEffect(() => {
  fetchCountry(countryId);
  fetchCities(cityId);
}, [countryId, cityId]);

// ✅ 分割する
useEffect(() => { fetchCountry(countryId); }, [countryId]);
useEffect(() => { fetchCities(cityId); }, [cityId]);
```

## 3. 更新関数の活用

現在のstate値をエフェクト内で読み取る必要をなくす。

```tsx
// ❌ messagesが依存値に含まれる
useEffect(() => {
  connection.on('message', (msg) => {
    setMessages([...messages, msg]);
  });
}, [messages]); // messagesが変わるたびに再接続

// ✅ 更新関数でmessagesを依存値から除去
useEffect(() => {
  connection.on('message', (msg) => {
    setMessages(prev => [...prev, msg]);
  });
}, []); // 再接続不要
```

## 4. useEffectEventの使用

最新の値を参照するが、その変化に「反応」させたくないロジックを分離する。

```tsx
// ❌ themeが変わるたびにチャットが再接続される
useEffect(() => {
  const conn = createConnection(roomId);
  conn.on('connected', () => {
    showNotification('接続しました', theme);
  });
  conn.connect();
  return () => conn.disconnect();
}, [roomId, theme]);

// ✅ useEffectEventで非リアクティブ部分を分離
const onConnected = useEffectEvent(() => {
  showNotification('接続しました', theme);
});

useEffect(() => {
  const conn = createConnection(roomId);
  conn.on('connected', onConnected);
  conn.connect();
  return () => conn.disconnect();
}, [roomId]);
```

**useEffectEventのルール:**
- エフェクト内部からのみ呼び出す
- 他のコンポーネントやフックに渡さない
- 依存配列に含めない

## 5. 静的なオブジェクト/関数の外部化

props/stateに依存しないオブジェクトや関数はコンポーネント外に移動する。

```tsx
// ❌ 毎レンダーで新しいオブジェクトが生成され、エフェクトが再実行
function Chat() {
  const options = { serverUrl: 'https://example.com' };
  useEffect(() => {
    const conn = createConnection(options);
    conn.connect();
    return () => conn.disconnect();
  }, [options]); // 毎回新しい参照
}

// ✅ コンポーネント外に移動
const options = { serverUrl: 'https://example.com' };

function Chat() {
  useEffect(() => {
    const conn = createConnection(options);
    conn.connect();
    return () => conn.disconnect();
  }, []); // optionsは不変
}
```

## 6. 動的なオブジェクトのエフェクト内部化

propsに依存するオブジェクトはエフェクト内で生成する。

```tsx
// ❌ roomIdが変わるたびに新しいオブジェクト参照が生まれる
function Chat({ roomId }) {
  const options = { serverUrl: 'https://example.com', roomId };
  useEffect(() => {
    const conn = createConnection(options);
    conn.connect();
    return () => conn.disconnect();
  }, [options]);
}

// ✅ エフェクト内でオブジェクトを生成し、プリミティブ値を依存に
function Chat({ roomId }) {
  useEffect(() => {
    const options = { serverUrl: 'https://example.com', roomId };
    const conn = createConnection(options);
    conn.connect();
    return () => conn.disconnect();
  }, [roomId]); // プリミティブ値のみ
}
```

## 7. プリミティブ値の抽出

propsから受け取ったオブジェクトの必要な値を分割代入で取り出す。

```tsx
// ❌ optionsオブジェクト全体が依存値
function Chat({ options }) {
  useEffect(() => {
    const conn = createConnection(options);
    conn.connect();
    return () => conn.disconnect();
  }, [options]); // 親が再レンダーするたびに再実行

// ✅ プリミティブ値を抽出
function Chat({ options }) {
  const { serverUrl, roomId } = options;
  useEffect(() => {
    const conn = createConnection({ serverUrl, roomId });
    conn.connect();
    return () => conn.disconnect();
  }, [serverUrl, roomId]); // 値が変わったときだけ
}
```
