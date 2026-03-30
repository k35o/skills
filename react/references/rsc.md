# React Server Components / Server Actions

## Server Components

### できること
- `async/await` でデータを直接取得する（API不要）
- 重いライブラリをサーバーだけで使う（クライアントバンドルに含まれない）
- ビルド時またはリクエスト時に実行される
- Promiseを作ってClient Componentに渡す（`use`で読む）

### できないこと
- `useState`, `useReducer`, `useEffect` などのインタラクティブなフック
- イベントハンドラ（`onClick`, `onChange` など）
- ブラウザAPI（`localStorage`, `window` など）
- Context（読み取りのみ不可。提供も不可）

### 注意
- Server Componentsに`"use server"`ディレクティブは**不要**。`"use server"`はServer Actionsのためのもの
- Server Componentsがデフォルト。`"use client"`をつけたものだけがClient Component

## "use client"

### ルール
- ファイル先頭に記述する（コメントは前に書ける。バッククォートは不可）
- モジュール依存ツリーに基づいてクライアント境界が決まる（レンダーツリーではない）
- `"use client"`をつけたモジュールが依存する全てのモジュールもクライアントで実行される

### つけるべきとき
- `useState`, `useEffect`などのフックを使うとき
- イベントハンドラが必要なとき
- ブラウザAPIにアクセスするとき

### 設計の原則
- `"use client"`の境界はできるだけ葉に近い位置に置く
- レイアウトやページ全体をClient Componentにしない
- インタラクティブな部分だけを切り出してClient Componentにする

```tsx
// ❌ ページ全体をクライアントにしない
'use client';
export default function Page() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <HeavyStaticContent />  {/* これもクライアントバンドルに含まれる */}
      <Counter count={count} setCount={setCount} />
    </div>
  );
}

// ✅ インタラクティブな部分だけを分離
// page.tsx (Server Component)
export default function Page() {
  return (
    <div>
      <HeavyStaticContent />
      <Counter />  {/* これだけClient Component */}
    </div>
  );
}

// counter.tsx
'use client';
export function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

## "use server" / Server Actions

### ルール
- 非同期関数でのみ使用可能
- 引数と返り値はシリアライズ可能でなければならない
- クラス、一般的な関数、イベントオブジェクト、DOM要素は渡せない

### セキュリティ

**Server Actionsの引数は信頼できない入力として扱う。**

```tsx
// ❌ 危険: クライアントからのIDをそのまま信頼
async function deleteNote(noteId: string) {
  'use server';
  await db.notes.delete(noteId);
}

// ✅ 安全: 認証・認可を検証する
async function deleteNote(noteId: string) {
  'use server';
  const user = await getAuthenticatedUser();
  const note = await db.notes.find(noteId);
  if (note.ownerId !== user.id) throw new Error('Unauthorized');
  await db.notes.delete(noteId);
}
```

### フォーム統合

```tsx
// Server Action
async function updateName(formData: FormData) {
  'use server';
  const name = formData.get('name');
  await db.users.update({ name });
}

// フォームから直接呼ぶ
<form action={updateName}>
  <input type="text" name="name" />
  <button type="submit">更新</button>
</form>
```

### useActionStateとの組み合わせ

pending状態、エラー、結果を自動管理する:

```tsx
'use client';
const [state, submitAction, isPending] = useActionState(updateName, { error: null });

<form action={submitAction}>
  <input type="text" name="name" />
  <button disabled={isPending}>
    {isPending ? '送信中...' : '更新'}
  </button>
  {state.error && <p>{state.error}</p>}
</form>
```

## シリアライズ可能な型（サーバー↔クライアント）

**可能:** string, number, bigint, boolean, undefined, null, Date, プレーンオブジェクト, 配列, Map, Set, TypedArray, ArrayBuffer, Promise, JSX要素, Server Actions

**不可:** 一般的な関数, クラス, クラスインスタンス, DOMノード, イベントオブジェクト, 未登録Symbol

## Server ComponentからClient Componentへのデータの渡し方

```tsx
// ✅ シリアライズ可能なpropsを渡す
async function Page() {
  const data = await fetchData();
  return <ClientChart data={data} />;
}

// ✅ Promiseを渡してClient側でuseで読む
async function Page() {
  const dataPromise = fetchData(); // awaitしない
  return <ClientChart dataPromise={dataPromise} />;
}

// client-chart.tsx
'use client';
function ClientChart({ dataPromise }) {
  const data = use(dataPromise); // Suspenseと組み合わせる
  return <Chart data={data} />;
}
```

## compositionパターン

Client ComponentにServer Componentをchildrenとして渡せる:

```tsx
// layout.tsx (Server Component)
export default function Layout({ children }) {
  return (
    <ClientSidebar>
      {children}  {/* Server Componentのレンダー結果 */}
    </ClientSidebar>
  );
}

// client-sidebar.tsx
'use client';
export function ClientSidebar({ children }) {
  const [isOpen, setIsOpen] = useState(true);
  return (
    <div>
      <nav>{/* クライアントのインタラクション */}</nav>
      <main>{children}</main>  {/* サーバーで既にレンダー済み */}
    </div>
  );
}
```
