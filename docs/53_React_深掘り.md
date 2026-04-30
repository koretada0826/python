# Day 53:React 深掘り - Hooks・状態管理・SSR

## 💡 これは何?(小学生でもわかる説明)

**画面を作る「部品(コンポーネント)」を組み合わせる仕組み**を、プロレベルに掘り下げる日。

### なぜ大事?
- フロントエンド案件の**ほぼ全部 React**
- React が使えるだけで**月+30〜50万**
- Hooks(フック)が使いこなせると**プロ判定**で単価+30万
- 状態管理(Zustand/TanStack Query)を覚えると**SaaS開発**ができる

### イメージ
- React = **LEGOブロック**
  - ボタン・入力欄・カードリスト... 全部「部品(コンポーネント)」
  - 部品を組み合わせて1つの画面にする
  - 部品を**使い回せる**から開発が早い
- Hooks = **部品に命を吹き込む魔法**
  - `useState` = 部品が**自分の記憶**を持てる魔法
  - `useEffect` = 部品が**画面に出た時/消えた時**にやることを決める魔法
  - `useMemo` = 重い計算を**メモして覚えておく**魔法

### Claude Code に何を頼めばいいか
- 「TODO リストの React コンポーネント書いて(useState使用)」
- 「TanStack Query で API 呼ぶ部分書いて」
- 「react-hook-form + zod でログインフォーム作って」
- 「Zustand でダークモード切替の store 書いて」

---

## 💰 +30〜80万(フロントエンド案件)
## ⏱ 25時間

Day 29 で軽く触れた React を**プロレベル**に。

---

## ゴール

- JSX とコンポーネントの基本を**説明できる**
- Hooks 5種類(useState/useEffect/useMemo/useCallback/useRef)を使える
- Custom Hook を自分で作れる
- Zustand or TanStack Query で状態管理できる
- react-hook-form + zod でフォーム作れる
- Server vs Client Component の違いが分かる
- 仮想スクロール・lazy load で**速い React** が書ける

---

## 1. JSX & コンポーネント(=「LEGOブロックの作り方」)(15分)

### 1-1. JSX とは?

> **HTML を JavaScript の中に書ける文法**。

```tsx
// 関数コンポーネント=部品
function Welcome({ name }: { name: string }) {
  return <h1>Hello, {name}</h1>;
}

// 使う(他の部品から呼び出せる)
<Welcome name="太郎" />
```

> **`{name}` の意味:**
> JS の値を HTML に**埋め込む**書き方。
> Python の f-string `f"{name}"` と似てる。

### 1-2. ルール3つ

| ルール | 例 |
|---|---|
| 部品名は**大文字始まり** | `<Welcome />` ○ / `<welcome />` × |
| `class` は **`className`** | `<div className="..." />` |
| 1つの部品は**1つの要素を返す** | `<>...</>` でラップ可 |

> **なぜ?**
> React は「大文字 = 自作部品」「小文字 = HTMLタグ」と判断するから。

---

## 2. Hooks(=「部品に命を吹き込む魔法」)(40分)

### 2-1. useState(=「部品の記憶」)

```tsx
// count という記憶を作る、初期値0
const [count, setCount] = useState(0);

// クリックで +1
<button onClick={() => setCount(count + 1)}>
  {count}
</button>
```

> **イメージ:** `useState` = **部品専用のメモ帳**。
> メモ帳の内容が変わると、画面も自動で書き換わる。

### 2-2. useEffect(=「部屋の見張り役」)

```tsx
useEffect(() => {
  // この部品が画面に出た時 + 依存値が変わった時に実行
  fetchData();
  
  return () => {
    // この部品が画面から消える時に実行(掃除)
  };
}, [dep]);  // dep が変わった時だけ再実行
```

> **イメージ:** `useEffect` = **部屋の見張り役**。
> - 部屋に入った時(マウント)= 何かする
> - 部屋から出る時(アンマウント)= 後片付け
> - 中の物が変わった時(依存値変化)= もう1回見直す

> **依存配列の意味:**
> - `[]` 空 → **最初の1回だけ**実行
> - `[x]` → x が変わるたび実行
> - 省略 → **毎回**実行(危険、ほぼ使わない)

### 2-3. useMemo / useCallback(=「メモして覚える」)

```tsx
// 重い計算をキャッシュ(data が変わるまで再計算しない)
const result = useMemo(() => heavyCalc(data), [data]);

// 関数をキャッシュ(子コンポーネントの無駄再描画を防ぐ)
const handleClick = useCallback(() => doSomething(), []);
```

> **なぜ必要?**
> React は **状態が変わると毎回部品を再描画**する。
> 重い計算を毎回やるとカクカクになる。
> `useMemo` で **「変わってない時はメモ使って」** と伝える。

### 2-4. useRef(=「DOMへの直通電話」)

```tsx
// HTMLInput への参照を保持する箱
const inputRef = useRef<HTMLInputElement>(null);

// 再描画されても消えない値(レンダー回数カウントなど)
const renderCount = useRef(0);

// JSXで紐付け
<input ref={inputRef} />

// 直接DOM操作
inputRef.current?.focus();
```

> **`useRef` vs `useState` の違い:**
> - `useState` の値は変わると**再描画される**
> - `useRef` の値は変わっても**再描画されない**(=こっそり保存)

### 2-5. Custom Hook(=「自分専用の魔法」)

```tsx
// useから始まる名前で、Hooks を組み合わせて自作Hookを作る
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  return { count, increment, decrement };
}

// 使う側はシンプル
const { count, increment } = useCounter();
```

> **なぜ作る?**
> 同じ Hooks の組み合わせを**何回も書きたくない**から。
> ロジックを再利用できる=コードがDRY(Don't Repeat Yourself)。

---

## 3. 状態管理ライブラリ(=「部品の壁を超える記憶」)(30分)

### 3-1. なぜ必要?

> `useState` は**1つの部品の中だけ**で使える記憶。
> 「ログインユーザー情報」「カート」みたいな**全部品で共有したい記憶**には使えない。

### 3-2. Zustand(おすすめ・シンプル)

```bash
npm install zustand
```

```tsx
import { create } from "zustand";

// 共通の記憶の設計図
type Store = {
  count: number;
  inc: () => void;
};

// 共通記憶を作る
const useStore = create<Store>((set) => ({
  count: 0,
  inc: () => set((s) => ({ count: s.count + 1 })),
}));

// どの部品からでも呼べる
function Counter() {
  const { count, inc } = useStore();
  return <button onClick={inc}>{count}</button>;
}

function Display() {
  const count = useStore(s => s.count);  // 一部だけ取る
  return <p>現在: {count}</p>;
}
```

→ Redux より**100倍シンプル**。

### 3-3. TanStack Query(API状態の自動管理)

```bash
npm install @tanstack/react-query
```

```tsx
import { useQuery, useMutation } from "@tanstack/react-query";

function Users() {
  // /api/users から取得 + キャッシュ + 再取得
  const { data, isLoading, error } = useQuery({
    queryKey: ["users"],
    queryFn: () => fetch("/api/users").then(r => r.json()),
  });
  
  if (isLoading) return <p>読み込み中...</p>;
  if (error) return <p>エラー</p>;
  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

> **何が嬉しい?**
> - **キャッシュ自動**(同じデータを何度も取得しない)
> - **再フェッチ自動**(ウィンドウフォーカス時に最新を取りに行く)
> - **楽観更新**(送信前に画面を先に更新→失敗時にロールバック)
> - useState + useEffect で書くと100行 → useQuery で**5行**

---

## 4. フォーム(react-hook-form + zod)(=「フォームの最強コンビ」)(20分)

```bash
npm install react-hook-form @hookform/resolvers zod
```

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

// バリデーションルール
const Schema = z.object({
  email: z.string().email("メール形式じゃないよ"),
  password: z.string().min(8, "8文字以上"),
});

type FormData = z.infer<typeof Schema>;

function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(Schema),  // zod でバリデーション
  });
  
  return (
    <form onSubmit={handleSubmit(data => login(data))}>
      <input {...register("email")} placeholder="メール" />
      {errors.email && <p>{errors.email.message}</p>}
      
      <input type="password" {...register("password")} placeholder="パスワード" />
      {errors.password && <p>{errors.password.message}</p>}
      
      <button>送信</button>
    </form>
  );
}
```

> **なぜこの組み合わせ?**
> - **react-hook-form** = フォーム入力を**高速・低再描画**で扱える
> - **zod** = バリデーションを**型と一緒に**書ける
> - 2つ合わせて = **型安全 + バリデーション + パフォーマンス**

---

## 5. Server Components(Next.js)(=「サーバー専用の部品」)(20分)

```tsx
// app/users/page.tsx(サーバーで実行される)
async function UsersPage() {
  // ブラウザに送る前にサーバーでDB直接アクセス
  const users = await db.query("SELECT * FROM users");
  return <UserList users={users} />;
}
```

> **何が嬉しい?**
> - **API挟まずDB直接**触れる(API設計いらない)
> - **JSバンドルが軽量化**(クライアントに送らない)
> - **秘密キーが漏れない**(サーバー内で完結)

### 5-1. Server vs Client Component の使い分け

| Server Component(デフォルト) | Client Component(`"use client"`) |
|---|---|
| データ取得・DBアクセス | useState/useEffect |
| 重い計算 | onClick/onChange イベント |
| 秘密キー使用OK | ブラウザAPI(localStorage等) |
| useState 使えない | サーバー処理は不可 |

> **コツ:** デフォルトはServer。**インタラクションが必要な部分だけ Client** にする。

---

## 6. パフォーマンス最適化(=「速い React」)(15分)

### 6-1. React.memo(=「再描画ストッパー」)

```tsx
// props が変わらない限り再描画しない
const Item = React.memo(({ data }: Props) => <div>{data.name}</div>);
```

### 6-2. 仮想スクロール(=「1万件でも軽く」)

```tsx
import { useVirtualizer } from "@tanstack/react-virtual";

// 画面に映ってる10件だけ実際に描画(残りは非表示)
const virtualizer = useVirtualizer({
  count: 10000,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 40,
});
```

> **なぜ?** 1万件全部描画すると重い。**見えてる分だけ**描画=快適。

### 6-3. Code Splitting(=「必要になるまで読み込まない」)

```tsx
const Heavy = React.lazy(() => import("./Heavy"));

<Suspense fallback={<Loading />}>
  <Heavy />
</Suspense>
```

→ 初期ロードが**3〜10倍速くなる**。

---

## 7. よくあるエラーと対処

| エラー | 原因 | 直し方 |
|---|---|---|
| `Cannot read property of undefined` | データ取得前にアクセス | `data?.name` のオプショナルチェイン |
| `Each child should have a unique "key" prop` | リストに key がない | `<li key={item.id}>` を追加 |
| `Too many re-renders` | レンダー中に setState 呼んでる | useEffect の中に移す |
| `Maximum update depth exceeded` | 依存配列が原因の無限ループ | useEffect の依存配列を確認 |
| `Hooks can only be called inside...` | 関数の外/条件分岐で Hook 呼んだ | 関数の最上部で呼ぶ |
| `Text content does not match server-rendered HTML` | SSR とCSR で内容が違う | useEffect の中で更新 |
| `'use client' directive` | Server Component で useState 使った | ファイル先頭に `"use client"` 追加 |
| `Hydration failed` | サーバー描画とクライアント描画が違う | クライアント限定処理を useEffect で |

### 困った時のテンプレ

```
このエラー出た:
<エラー文をそのままコピペ>

コード:
<該当箇所>

直して。
```

---

## 8. ⭐ 課題

```
1. TanStack Query で「ユーザーCRUD UI」(無限スクロール対応)
   - GET一覧、POST作成、PUT更新、DELETE削除
   - useInfiniteQuery で無限スクロール

2. react-hook-form + zod で複雑フォーム
   - バリデーション5種類以上
   - 配列フィールド(useFieldArray)

3. Zustand でグローバル状態管理(ダークモード切替)
   - localStorage 永続化込み

4. Next.js Server Component で DB直接取得
   - PostgreSQL から SELECT して表示
```

---

## 9. 用語まとめ表

| 用語 | 一言で |
|---|---|
| JSX | HTML を JS の中に書ける文法 |
| コンポーネント | 画面の部品(LEGOブロック) |
| props | 親から子に渡すデータ |
| Hook | 部品に命を吹き込む魔法 |
| useState | 部品の記憶(変わると再描画) |
| useEffect | 部屋の見張り役(マウント/アンマウント) |
| useMemo | 重い計算をメモ |
| useCallback | 関数をメモ |
| useRef | DOMへの直通電話・こっそり保存 |
| Custom Hook | 自作のHook(use〜で始まる) |
| Zustand | シンプルな共通状態管理 |
| TanStack Query | API取得のキャッシュ管理 |
| react-hook-form | 高速フォーム |
| zodResolver | zodをフォームバリデーションに繋ぐ |
| Server Component | サーバーで実行(DB直接OK) |
| Client Component | ブラウザで実行(useState OK) |
| `"use client"` | この部品はClient Componentと宣言 |
| React.memo | 再描画ストッパー |
| Suspense | lazy load の待機表示 |
| 仮想スクロール | 見えてる分だけ描画 |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **React 高単価案件 / Next.js 専任ポジション**:月単価 60〜120万円(プラットフォーム例:レバテック/Findy Freelance/直契約)
- **既存 React アプリのパフォーマンス改善**:単価 30〜100万円(memo / Suspense / 仮想スクロール)
- **複雑な状態管理を含む SaaS フロント開発**:単価 50〜150万円(Zustand / TanStack Query)

### クロードへの頼み方(営業文を書かせる例)
```
React 専任ポジション応募用、スキルシート + 提案文を書いて。
Day 53 までで習得:Hooks 全種 / Custom Hook / Zustand / TanStack Query / react-hook-form + zod / Server vs Client
売り:Pythonバックエンドも書けるフロントエンジニア(=フルスタック寄り)
過去実績:GitHub に Next.js + FastAPI のサンプル
希望:月単価 80〜100万円 / リモート週5
ポイント:Server Component と Client Component を適材適所で使い分けられる旨を強調
600字
```

### この章だけでは足りないもの(次に学ぶべき)
- Tailwind / Shadcn(Day 54)で UI 実装スピードUP
- Stripe(Day 55) / メール(Day 56)で SaaS の仕上げ
- 法人化(Day 57)で大手案件参入

---

## ✅ チェック

- [ ] JSX とコンポーネント書ける
- [ ] Hooks(useState/useEffect/useMemo/useCallback/useRef)使える
- [ ] Custom Hook 作れる
- [ ] Zustand or TanStack Query 使える
- [ ] react-hook-form + zod でフォーム作れる
- [ ] Server vs Client Component の違いが分かる
- [ ] React.memo / lazy で最適化できる
- [ ] よくあるエラー(key/Hook ルール/Hydration)を直せる

---

## 進捗

```
Day 53: React 深掘り完了。
Hooks フル活用 + 状態管理 + フォーム + SSR まで体得。
フロントエンド単価+30万のスキル獲得。
```

## 次

`54_Tailwind_Shadcn.md` へ。**綺麗なUIをCSS知識ゼロで作る技術**。
