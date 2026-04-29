# Day 53:React 深掘り - Hooks・状態管理・SSR

## 💡 これは何?(小学生でもわかる説明)

****画面を作る部品(コンポーネント)**の組み合わせ術**

### なぜ大事?
フロント案件単価+30万

### イメージ
**LEGOブロック**を組み合わせて画面を作る

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 +30-80万(フロントエンド案件)
## ⏱ 25時間

Day 29 で軽く触れた React を**プロレベル**に。

---

## 小学生説明
**画面を作る部品(コンポーネント)を組み合わせる**仕組み。
ボタン・入力欄・リスト... 全部部品化して**再利用**。

---

## 1. JSX & コンポーネント(15分)

```tsx
// 関数コンポーネント
function Welcome({ name }: { name: string }) {
  return <h1>Hello, {name}</h1>;
}

// 使う
<Welcome name="太郎" />
```

---

## 2. Hooks(必修)(40分)

### useState

```tsx
const [count, setCount] = useState(0);
const [user, setUser] = useState<User | null>(null);
```

### useEffect

```tsx
useEffect(() => {
  // マウント・更新時
  fetchData();
  
  return () => {
    // アンマウント時のクリーンアップ
  };
}, [dep]);  // dep が変わった時だけ実行
```

### useMemo / useCallback(パフォーマンス)

```tsx
// 重い計算をキャッシュ
const result = useMemo(() => heavyCalc(data), [data]);

// 関数をキャッシュ(子コンポーネントの再レンダリング防止)
const handleClick = useCallback(() => doSomething(), []);
```

### useRef

```tsx
const inputRef = useRef<HTMLInputElement>(null);
const renderCount = useRef(0);

// DOM参照
<input ref={inputRef} />
inputRef.current?.focus();
```

### Custom Hook

```tsx
// 自作 hook
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = () => setCount(c => c + 1);
  return { count, increment };
}

// 使う
const { count, increment } = useCounter();
```

---

## 3. 状態管理ライブラリ(30分)

### Zustand(おすすめ・シンプル)

```bash
npm install zustand
```

```tsx
import { create } from "zustand";

type Store = {
  count: number;
  inc: () => void;
};

const useStore = create<Store>((set) => ({
  count: 0,
  inc: () => set((s) => ({ count: s.count + 1 })),
}));

// 使う
function Counter() {
  const { count, inc } = useStore();
  return <button onClick={inc}>{count}</button>;
}
```

→ Redux より**100倍シンプル**。

### TanStack Query(API状態管理)

```bash
npm install @tanstack/react-query
```

```tsx
import { useQuery, useMutation } from "@tanstack/react-query";

function Users() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["users"],
    queryFn: () => fetch("/api/users").then(r => r.json()),
  });
  
  if (isLoading) return <p>Loading...</p>;
  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

→ **キャッシュ・再フェッチ・楽観更新**が自動。

---

## 4. フォーム(react-hook-form + zod)(20分)

```bash
npm install react-hook-form @hookform/resolvers zod
```

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const Schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(Schema),
  });
  
  return (
    <form onSubmit={handleSubmit(data => login(data))}>
      <input {...register("email")} />
      {errors.email && <p>{errors.email.message}</p>}
      <input type="password" {...register("password")} />
      <button>送信</button>
    </form>
  );
}
```

---

## 5. Server Components(Next.js)(20分)

```tsx
// app/users/page.tsx(サーバー実行)
async function UsersPage() {
  const users = await db.query("SELECT * FROM users");  // 直接DB!
  return <UserList users={users} />;
}
```

→ **API挟まずDB直接**。バンドル軽量化。

### Client Component との切り分け

| Server | Client |
|---|---|
| データ取得 | useState/useEffect |
| 重い処理 | onClick/イベント |
| DB/秘密キー | ブラウザAPI |

---

## 6. パフォーマンス最適化(15分)

```tsx
// React.memo(再レンダリング防止)
const Item = React.memo(({ data }: Props) => <div>{data.name}</div>);

// 仮想スクロール(1万件)
import { useVirtualizer } from "@tanstack/react-virtual";

// Code Splitting
const Heavy = React.lazy(() => import("./Heavy"));
<Suspense fallback={<Loading />}>
  <Heavy />
</Suspense>
```

---

## 7. ⭐ 課題

```
1. TanStack Query で「ユーザーCRUD UI」(無限スクロール対応)
2. react-hook-form + zod で複雑フォーム
3. Zustand でグローバル状態管理(ダークモード切替)
4. Next.js Server Component で DB直接取得
```

---

## ✅ チェック
- [ ] Hooks(useState/useEffect/useMemo/Ref)使える
- [ ] Custom Hook 作れる
- [ ] Zustand or TanStack Query
- [ ] react-hook-form + zod
- [ ] Server vs Client Component
- [ ] React.memo / lazy

## 次
`54_Tailwind_デザイン.md`
