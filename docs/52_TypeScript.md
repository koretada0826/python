# Day 52:TypeScript - Web案件の必須言語

## 💡 これは何?(小学生でもわかる説明)

**JavaScript の進化版**。**「型(かた)」がついた**JavaScript。

### なぜ大事?
- 2026年、Web案件の**60%は TypeScript必須**
- JS のままだと「**型を書ける人**」より単価が**月20〜30万低い**
- 大規模アプリ(SaaS・社内ツール)はほぼTS
- React / Next.js も TS が標準

### イメージ
- **JavaScript** = 何でも入れられる**ダンボール箱**
  - 中身が「みかん」か「数字の3」か**開けるまで分からない**
  - 「みかん + 3」で計算しようとしてバグる
- **TypeScript** = 箱に **「みかん専用」「数字専用」と書いてある**
  - 違うものを入れようとすると**コードを書く時点で警告**
  - バグが**動かす前**に見つかる

> **もうひとつの例え:** 
> JS=自転車(誰でも乗れるけど転びやすい)
> TS=**自転車にヘルメット**つけたやつ。安全で長距離走れる。

### Claude Code に何を頼めばいいか
- 「この JS コードを TypeScript に書き換えて」
- 「この関数の型定義を書いて」
- 「zod で API レスポンスを検証する型を作って」
- 「このエラーメッセージを直して(型エラー)」

---

## 💰 +30〜80万(Web/SaaS案件の月収目安)
## ⏱ 25時間

JavaScript の進化版。**型がある**ので大規模開発に強い。
2026年、Web案件の**60%は TypeScript**。

---

## ゴール

- 基本型・Union・Literal が書ける
- 関数の型・ジェネリクスが分かる
- interface と type の使い分け
- Partial / Pick / Omit などの**便利型**を使える
- async/await + Promise の型が書ける
- zod でランタイム検証ができる

---

## 1. セットアップ(=「TS の準備運動」)(15分)

### 1-1. インストール

```bash
mkdir my-ts && cd my-ts        # フォルダ作って入る
npm init -y                     # package.json を作る
npm install -D typescript ts-node @types/node  # TypeScript 一式インストール
npx tsc --init                  # tsconfig.json を作る
```

### 1-2. tsconfig.json(=「TSの設定書」)

`tsconfig.json` はTSの**ルール表**。とりあえずこれで動く。

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

> **`strict: true` が超重要**。
> これONなら「型を曖昧に書く」とエラーになる=**安全**。
> プロは絶対ON。

### 1-3. 実行コマンド

```bash
npx ts-node script.ts  # TS を直接実行(開発用)
npx tsc                # TS → JS にコンパイル(本番用)
```

> **なぜコンパイル?**
> ブラウザ・Node.js は**JSしか読めない**。
> TS は最終的に JS に変換して動かす。

---

## 2. 基本型(=「箱に種類を書く」)(20分)

### 2-1. プリミティブ型(基本の4つ)

```typescript
// 文字列専用の箱
let name: string = "太郎";

// 数字専用の箱
let age: number = 25;

// YES/NO 専用の箱
let active: boolean = true;

// 文字列の配列(リスト)
let items: string[] = ["a", "b"];

// タプル=種類が決まった2人組
let tuple: [string, number] = ["太郎", 25];
```

> **`: 型名` の形がポイント。**
> `let 変数: 型 = 値` で「**この箱はこの種類しか入らない**」と宣言。

### 2-2. オブジェクトの型(=「設計図」)

```typescript
// User という設計図を作る
type User = {
  id: number;
  name: string;
  email?: string;  // ? は「あってもなくてもOK(省略可)」
};

// 設計図に従ってオブジェクトを作る
const u: User = { id: 1, name: "太郎" };
```

> **イメージ:** 設計図に「id・name は必須、email は任意」と書いてある。
> 違反すると怒られる。

### 2-3. Union(=「複数OK箱」)

```typescript
// string か number どっちでも入る
let value: string | number = "hello";
value = 42;  // OK
// value = true;  // ← エラー(boolean は許可されてない)
```

### 2-4. Literal Type(=「決まった選択肢のみ」)

```typescript
// "admin" "user" "guest" の3択しか入らない
let role: "admin" | "user" | "guest" = "admin";
// role = "owner";  // ← エラー(選択肢にない)
```

> **超便利!**
> プルダウンメニューみたいに「選択肢を決められた箱」が作れる。

### 2-5. any と unknown(=「逃げ道」)

```typescript
// any = 何でも入る(=型チェックOFF=危険)
let escape: any = "hello";
escape = 42;       // OK
escape = { x: 1 }; // OK(でも型の意味なくなる)

// unknown = 何でも入るけど、使う前にチェック必要(=安全)
let x: unknown = fetchData();
if (typeof x === "string") {
  console.log(x.length);  // string と確定したので使える
}
```

> **`any` を使うのは負け**。
> 困ったら `unknown` で型ガード(typeof チェック)を書く。

---

## 3. 関数の型(=「ボタンの形を決める」)(15分)

### 3-1. 普通の関数

```typescript
// 引数2つ(どちらもnumber)、戻り値はnumber
function add(a: number, b: number): number {
  return a + b;
}
```

### 3-2. アロー関数

```typescript
// 短い書き方
const greet = (name: string): string => `Hello ${name}`;
```

### 3-3. オプショナル引数

```typescript
// level は省略OK("info" か "error" のみ)
const log = (msg: string, level?: "info" | "error") => {
  console.log(`[${level ?? "info"}] ${msg}`);
};

log("起動");                // OK
log("失敗", "error");       // OK
```

### 3-4. 関数型(=「関数の設計図」)

```typescript
// イベントを受け取って何も返さない関数の型
type Handler = (event: Event) => void;

// この型に従う関数を作る
const onClick: Handler = (e) => {
  console.log(e.type);
};
```

> **`void` = 戻り値なし**。`return` を使わない関数の戻り値型。

---

## 4. ジェネリクス(=「型を後で決める仕組み」)(20分)

### 4-1. 何が嬉しいの?

> **「同じ処理を、いろんな型で使い回したい」**ときに使う。

```typescript
// T は「あとで決まる型」のフリ仕込み
function identity<T>(arg: T): T {
  return arg;
}

identity<string>("hello");  // T = string
identity<number>(42);       // T = number
```

> **イメージ:** 
> ジェネリクス=**「サイズフリーの服」**。
> Sサイズの人にも、Lサイズの人にも合うように作れる。
> 「Lの服はLにしか合わない」じゃなくて「**着る人に合わせて変わる**」。

### 4-2. 制約(=「Tに条件をつける」)

```typescript
// K は「T の鍵(プロパティ名)」じゃないとダメ、と制約
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const u = { name: "太郎", age: 25 };
getProperty(u, "name");  // string が返る
getProperty(u, "age");   // number が返る
// getProperty(u, "xxx");  // ← エラー(xxx は鍵じゃない)
```

→ **型を再利用できる**。React/関数ライブラリで頻出。

---

## 5. Interface vs Type(=「2つの設計図書き方」)(10分)

### 5-1. interface(継承できる)

```typescript
interface Animal {
  name: string;
}

// Animal を継承してDog を作る
interface Dog extends Animal {
  breed: string;  // 犬種を追加
}

const taro: Dog = { name: "タロウ", breed: "柴犬" };
```

### 5-2. type(union や intersection が得意)

```typescript
// type は「型を別名で呼ぶ」感じ
type ID = string | number;

// 既存の型を合体できる(intersection)
type AdminUser = User & { admin: boolean };
```

### 5-3. 使い分けの目安

| こんな時 | こっち |
|---|---|
| オブジェクトの形を定義する | **interface** |
| Union や合体型を作る | **type** |
| 迷ったら | **type** でOK |

→ チーム規約があれば従う。**個人なら type 一択でも問題なし**。

---

## 6. ユーティリティ型(=「型の便利道具」必修)(20分)

> **TS最強の便利機能**。設計図を**ちょっとだけ変える**ときに使う。

```typescript
type User = { id: number; name: string; email: string };

// すべて Optional(あってもなくてもOK)
type PartialUser = Partial<User>;
// → { id?: number; name?: string; email?: string }

// すべて必須に変える
type RequiredUser = Required<User>;

// 一部のキーだけ取り出す
type UserName = Pick<User, "name" | "email">;
// → { name: string; email: string }

// 一部のキーを除外する
type UserNoId = Omit<User, "id">;
// → { name: string; email: string }

// すべて読み取り専用(変更不可)
type ReadOnlyUser = Readonly<User>;

// 関数の戻り値の型を取り出す
function fetchUser() { return { id: 1, name: "太郎" }; }
type UserResult = ReturnType<typeof fetchUser>;
// → { id: number; name: string }

// Promise の中身だけ取り出す
type Awaited<T> = T extends Promise<infer U> ? U : T;
```

> **使い分け早見表:**
> - 全部省略可にしたい → `Partial`
> - 一部だけ抜き取りたい → `Pick`
> - 一部だけ除きたい → `Omit`
> - 関数の戻り値型をほしい → `ReturnType<typeof 関数名>`

---

## 7. async/await + 型(=「非同期の型」)(10分)

```typescript
// fetchUser は User を返すPromiseを返す
async function fetchUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error("Not found");
  // .json() の戻り値は any なので、as User で型を確定
  return await res.json() as User;
}

// 使う側
const user = await fetchUser(1);  // user は User 型
```

> **`Promise<User>` の意味:**
> 「**いつかは User を返す約束**」=非同期処理の結果型。

---

## 8. zod(=「実行時の型チェッカー」)(20分)

### 8-1. なぜ必要?

> **TSの型チェックは「コンパイル時」だけ**。
> APIから返ってきた**実際のデータ**が型と違うと、実行時にバグる。
> zod は **実行時にデータを検証**してくれる。

```bash
npm install zod
```

### 8-2. 使い方

```typescript
import { z } from "zod";

// スキーマ(=設計図)を作る
const UserSchema = z.object({
  id: z.number(),
  name: z.string().min(1).max(50),    // 1〜50文字
  email: z.string().email(),           // メール形式
  age: z.number().min(0).max(150),     // 0〜150
});

// スキーマから TS の型を自動生成(これ便利!)
type User = z.infer<typeof UserSchema>;

// 検証する(safeParse なら例外を投げない)
const result = UserSchema.safeParse(data);
if (result.success) {
  console.log(result.data);   // 型安全に使える
} else {
  console.error(result.error); // エラー内容を確認
}
```

→ Pydantic の TypeScript 版。**API レスポンス検証**に必須。

> **使うシーン:**
> - 外部APIから来るデータの検証
> - フォーム入力値のバリデーション
> - 環境変数の型チェック
> - DBから取り出したJSONの検証

---

## 9. よくあるエラーと対処

| エラー | 原因 | 直し方 |
|---|---|---|
| `Type 'string' is not assignable to type 'number'` | 型が違う代入 | 型を確認し、`as` か変換関数(`Number()` 等)を使う |
| `Property 'xxx' does not exist on type` | 存在しないプロパティを使った | スペル確認、interface に追加 |
| `Object is possibly 'undefined'` | undefined の可能性あり | `?.` (オプショナルチェイン) や if で守る |
| `Cannot find module 'xxx'` | パッケージが入ってない or 型定義なし | `npm install xxx` + `npm install -D @types/xxx` |
| `Argument of type 'xxx' is not assignable` | 関数の引数の型違い | 関数定義を確認、型を合わせる |
| `Element implicitly has an 'any' type` | `any` 警告 | 明示的に型を書く |
| `Type 'unknown' is not assignable` | unknown を直接使った | 型ガード(`typeof`/`instanceof`)してから使う |

### 困った時のテンプレ

```
このエラー出た:
<エラー文をそのままコピペ>

コード:
<コードをそのままコピペ>

直して。
```

→ **クロードに丸投げでOK**。

---

## 10. ⭐ 課題

```
1. 既存JSスクリプト → TypeScript化
   - .js を .ts にして、型エラーを全部直す
   
2. zod で API レスポンス検証ライブラリ作成
   - GET したJSONを zod で検証してから返す関数

3. TypeScript で REST API クライアントSDK
   - GET/POST/PUT/DELETE のジェネリック関数
   - Authトークン管理(ヘッダーに付与)
   - エラーハンドリング(zod 検証込み)
```

---

## 11. 用語まとめ表

| 用語 | 一言で |
|---|---|
| 型(type) | 箱の中身の種類 |
| `string` / `number` / `boolean` | 文字 / 数字 / YES-NO |
| `string[]` | 文字列の配列(リスト) |
| Tuple | 種類順序が決まった配列 |
| Optional (`?`) | あってもなくてもOK |
| Union (`|`) | 複数の型のどれかOK |
| Literal Type | 決められた選択肢のみ |
| `any` | 何でも入る(=型チェックOFF) |
| `unknown` | 何でも入るが、使う前にチェック必要 |
| `void` | 戻り値なしの関数 |
| ジェネリクス (`<T>`) | あとで決まる型・サイズフリーの型 |
| interface | オブジェクトの設計図(継承可) |
| type | 型に別名つける(union 得意) |
| `Partial<T>` | 全部 optional に変える |
| `Pick<T, K>` | 一部のキーだけ取り出す |
| `Omit<T, K>` | 一部のキーを除外する |
| `Readonly<T>` | 全部読み取り専用 |
| `ReturnType<typeof f>` | 関数の戻り値の型を取り出す |
| `Promise<T>` | 非同期で T を返す約束 |
| zod | 実行時の型検証ライブラリ |
| `z.infer<typeof S>` | zod スキーマから型を自動生成 |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **TypeScript フロント / バック案件**:単価 50〜100万円(プラットフォーム例:ランサーズ/レバテック/Findy Freelance)
- **Node.js / Next.js / Bun ベースのバックエンド開発**:単価 50〜100万円
- **既存 JavaScript コードの TypeScript 化**:単価 30〜80万円

### クロードへの頼み方(営業文を書かせる例)
```
TypeScript 関連案件向けスキルシート文を書いて。
Day 52 までで習得:基本型 / Union / Generics / interface / zod / Promise / Next.js / FastAPI と連携
売り:Python と TypeScript 両方を「型重視」で書ける(=API契約をzod+Pydanticで二重防御)
過去実績:GitHub 上の Next.js + FastAPI フルスタックサンプル
希望:単発 50万〜 / 月額 60〜90万円
ポイント:単なる JS 屋ではなく「型でバグを未然に防ぐ」プロフィール感を出す
500字
```

### この章だけでは足りないもの(次に学ぶべき)
- React 深掘り(Day 53)で更にフロント単価UP
- Tailwind / Shadcn(Day 54)で UI 実装スピードUP
- Stripe(Day 55) / メール(Day 56)で SaaS 完成度UP

---

## ✅ チェック

- [ ] 基本型(string/number/boolean/array)書ける
- [ ] Union (`|`) と Literal Type 書ける
- [ ] Optional (`?`) を使える
- [ ] 関数の引数・戻り値に型がつけられる
- [ ] ジェネリクス `<T>` で型を再利用できる
- [ ] interface と type の使い分けができる
- [ ] `Partial / Pick / Omit / ReturnType` を使える
- [ ] async/await + Promise の型が書ける
- [ ] zod でランタイム検証できる
- [ ] エラーメッセージを見て自分で直せる

---

## 進捗

```
Day 52: TypeScript 完了。
基本型・関数・ジェネリクス・ユーティリティ型・zod まで体得。
これでWeb案件60%が狙える。
```

## 次

`53_React_深掘り.md` へ。**画面を作る部品(コンポーネント)を組み立てる技術**。
