# Day 52:TypeScript - Web案件の必須言語

## 💰 +30-80万(Web/SaaS案件)
## ⏱ 25時間

JavaScript の進化版。**型がある**ので大規模開発に強い。
2026年、Web案件の**60%は TypeScript**。

---

## 小学生説明
JavaScript は**何でも入れられる箱**。間違って数字を文字として扱ってバグる。
TypeScript は**箱に種類を書く**(`数字専用`/`文字専用`)。コンパイル時にバグ検出。

---

## 1. セットアップ(15分)

```bash
mkdir my-ts && cd my-ts
npm init -y
npm install -D typescript ts-node @types/node
npx tsc --init
```

`tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

```bash
npx ts-node script.ts  # 直接実行
npx tsc                 # JSにコンパイル
```

---

## 2. 基本型(20分)

```typescript
// プリミティブ
let name: string = "太郎";
let age: number = 25;
let active: boolean = true;
let items: string[] = ["a", "b"];
let tuple: [string, number] = ["太郎", 25];

// オブジェクト
type User = {
  id: number;
  name: string;
  email?: string;  // optional
};

const u: User = { id: 1, name: "太郎" };

// Union(複数型)
let value: string | number = "hello";
value = 42;  // OK

// Literal Type
let role: "admin" | "user" | "guest" = "admin";

// any(逃げ)・unknown(安全な any)
let x: unknown = fetch_data();
if (typeof x === "string") {
  console.log(x.length);  // 型ガード後に使える
}
```

---

## 3. 関数の型(15分)

```typescript
function add(a: number, b: number): number {
  return a + b;
}

// アロー
const greet = (name: string): string => `Hello ${name}`;

// オプショナル引数
const log = (msg: string, level?: "info" | "error") => { ... };

// 関数型
type Handler = (event: Event) => void;
```

---

## 4. ジェネリクス(20分)

```typescript
// 型を変数にできる
function identity<T>(arg: T): T {
  return arg;
}

identity<string>("hello");
identity<number>(42);

// 制約
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const u = { name: "太郎", age: 25 };
getProperty(u, "name");  // string
getProperty(u, "age");   // number
```

→ **型を再利用**できる。React/関数ライブラリで頻出。

---

## 5. Interface vs Type(10分)

```typescript
// interface(継承可能、宣言マージ可)
interface Animal {
  name: string;
}
interface Dog extends Animal {
  breed: string;
}

// type(union/intersection)
type ID = string | number;
type Mixed = User & { admin: boolean };
```

→ **オブジェクトの形を定義 = interface**、それ以外 = type、が一般的。

---

## 6. ユーティリティ型(必修)(20分)

```typescript
type User = { id: number; name: string; email: string };

// すべて Optional
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string }

// すべて Required
type RequiredUser = Required<User>;

// 一部だけ取り出す
type UserName = Pick<User, "name" | "email">;

// 一部除外
type UserNoId = Omit<User, "id">;

// すべて読み取り専用
type ReadOnlyUser = Readonly<User>;

// 戻り値の型を取り出し
function fetchUser() { return { id: 1 }; }
type UserResult = ReturnType<typeof fetchUser>;

// Promise の中身
type Awaited<T> = T extends Promise<infer U> ? U : T;
```

---

## 7. async/await + 型(10分)

```typescript
async function fetchUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error("Not found");
  return await res.json() as User;
}

const user = await fetchUser(1);
```

---

## 8. zod(ランタイム検証)(20分)

```bash
npm install zod
```

```typescript
import { z } from "zod";

const UserSchema = z.object({
  id: z.number(),
  name: z.string().min(1).max(50),
  email: z.string().email(),
  age: z.number().min(0).max(150),
});

type User = z.infer<typeof UserSchema>;

// 検証
const result = UserSchema.safeParse(data);
if (result.success) {
  console.log(result.data);  // 型安全
} else {
  console.error(result.error);
}
```

→ Pydantic の TypeScript 版。**API レスポンス検証**に必須。

---

## 9. ⭐ 課題

```
1. 既存JS スクリプト → TypeScript 化
2. zod で API レスポンス検証ライブラリ作成
3. TypeScript で REST API クライアントSDK
   - GET/POST/PUT/DELETE のジェネリック関数
   - Auth トークン管理
   - エラーハンドリング
```

---

## ✅ チェック
- [ ] 基本型・Union・Literal
- [ ] 関数型・ジェネリクス
- [ ] interface vs type
- [ ] ユーティリティ型
- [ ] async/await + 型
- [ ] zod でランタイム検証

## 次
`53_React_深掘り.md`
