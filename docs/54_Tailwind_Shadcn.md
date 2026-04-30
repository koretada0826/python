# Day 54:Tailwind・Shadcn UI - デザインできる技術者

## 💡 これは何?(小学生でもわかる説明)

**綺麗な見た目(UI)を、CSS知識ゼロで作る**スキル。

### なぜ大事?
- 「**コード書ける + デザインできる**」エンジニアは**希少**=単価3倍
- 普通のエンジニアは「動くけど見た目が古い」UI を作る
- 見た目がプロっぽいと**そのまま納品**できる(デザイナー雇う必要なし)
- フロント単価+20〜50万

### イメージ
- **Tailwind** = **服のパーツ屋さん**
  - 「丸い」「青い」「大きい」みたいな**短いキーワード**を組み合わせる
  - 自分で布から縫わずに、パーツを当てるだけで服ができる
- **Shadcn/ui** = **プロが作った服を借りられる店**
  - ボタン・カード・ダイアログ等の**完成パーツ**をコピペで使える
  - 自分のプロジェクトに**コピー(=自分のもの)** にできる

> **もう1つの例え:**
> 普通のCSS=家を建てる時に**木を切り出す**
> Tailwind=**プレカット済の木材**を組む
> Shadcn=**モデルハウスごと**コピーする

### Claude Code に何を頼めばいいか
- 「このコンポーネントを Tailwind で書いて」
- 「Shadcn の Card と Button で顧客一覧UI作って」
- 「ダークモード切替を実装して」
- 「Framer Motion でフェードイン入れて」

---

## 💰 +20〜50万(フロント単価)
## ⏱ 10時間

「**コード書ける + デザインできる**」=希少価値**3倍**。

---

## ゴール

- Tailwind の主要クラスを**コピペ無しで書ける**
- Shadcn のコンポーネントを使って**ダッシュボード**作れる
- レスポンシブ(スマホ/タブ/PC)対応
- ダークモード切替が実装できる
- Framer Motion で**滑らかなアニメ**を入れられる

---

## 1. Tailwind CSS(=「服のパーツを当てる」)(30分)

### 1-1. セットアップ

```bash
npm install -D tailwindcss postcss autoprefixer  # 一式インストール
npx tailwindcss init                              # 設定ファイル作成
```

`tailwind.config.js`:
```js
module.exports = {
  // どのファイルを Tailwind で処理するか
  content: ["./src/**/*.{html,tsx,jsx}"],
  theme: { extend: {} },
};
```

### 1-2. 主要クラス早見

| やりたいこと | クラス |
|---|---|
| 横並び | `flex` |
| 縦並び | `flex flex-col` |
| 中央揃え | `items-center justify-center` |
| 隙間4(=16px) | `gap-4` |
| パディング6 | `p-6` |
| 角丸 | `rounded-lg` |
| 影(普通) | `shadow-md` |
| 影(強) | `shadow-xl` |
| 背景白 | `bg-white` |
| 背景青 | `bg-blue-500` |
| 文字白 | `text-white` |
| 文字大 | `text-lg` |
| 文字太 | `font-bold` |
| ホバー時 | `hover:bg-blue-600` |
| 滑らか変化 | `transition` |

### 1-3. 実例(プロフィールカード)

```tsx
<div className="flex items-center gap-4 rounded-lg border bg-white p-6 shadow-md hover:shadow-xl transition">
  {/* アバター画像(丸く切り抜き) */}
  <img className="w-16 h-16 rounded-full" src={user.avatar} />
  
  {/* 名前と説明(残り全部使う) */}
  <div className="flex-1">
    <h3 className="font-bold text-lg">{user.name}</h3>
    <p className="text-gray-600 text-sm">{user.email}</p>
  </div>
  
  {/* フォローボタン */}
  <button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
    フォロー
  </button>
</div>
```

→ クラス書くだけで**プロのUI**。

> **なぜ class を並べるの?**
> 「`.profile-card { ... }` みたいなCSS書かない」のがTailwind流。
> CSSファイルが膨らまない=**チームで管理しやすい**。

### 1-4. 数字の意味(大事)

```
p-1 = padding 4px
p-2 = padding 8px
p-4 = padding 16px
p-6 = padding 24px
p-8 = padding 32px

w-16 = width 64px
w-32 = width 128px
w-full = width 100%
```

→ **数字 × 4px が基本**(覚えると速い)。

---

## 2. Shadcn/ui(=「完成パーツのコピペ屋」)(40分)

### 2-1. なぜ最強?

- **コピペで使える**完成コンポーネント(ボタン/カード/ダイアログ)
- **自分のプロジェクトにコピー**される(=改造自由・依存しない)
- **アクセシビリティ完璧**(キーボード操作・スクリーンリーダー対応)
- 法人ダッシュボードが**1日でできる**

### 2-2. セットアップ

```bash
npx shadcn-ui@latest init                                    # 初期化
npx shadcn-ui@latest add button card dialog input form table # よく使うパーツ追加
```

→ `src/components/ui/` 配下にコンポーネントがコピーされる。

### 2-3. 使い方

```tsx
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardContent } from "@/components/ui/card";
import { Dialog, DialogTrigger, DialogContent } from "@/components/ui/dialog";

<Card>
  <CardHeader>顧客一覧</CardHeader>
  <CardContent>
    {/* ボタン押すとダイアログ表示 */}
    <Dialog>
      <DialogTrigger asChild>
        <Button>追加</Button>
      </DialogTrigger>
      <DialogContent>新規顧客フォーム</DialogContent>
    </Dialog>
  </CardContent>
</Card>
```

→ **法人向けダッシュボード**が1日でできる。

### 2-4. 主なコンポーネント

| 名前 | 用途 |
|---|---|
| Button | ボタン |
| Card | カード(枠つきセクション) |
| Dialog | モーダル(ポップアップ) |
| Input | 入力欄 |
| Form | フォーム(react-hook-form 連携) |
| Table | 表(ソート・ページング対応) |
| DropdownMenu | プルダウンメニュー |
| Toast | 通知バブル |
| Tabs | タブ切替 |
| Sheet | サイドから出てくるパネル |

> **コツ:** 必要なものだけ `add` する。**未使用パーツが入らない**=軽い。

---

## 3. レスポンシブデザイン(=「スマホもPCも綺麗に」)(15分)

### 3-1. 画面サイズ別の指定

```tsx
// 基本=モバイル / md:=タブレット以上 / lg:=PC以上
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  ...
</div>
```

> **読み方:** 
> スマホは1列、タブレットは2列、PCは3列に並ぶグリッド。

### 3-2. ブレークポイント早見

| プレフィックス | 適用サイズ |
|---|---|
| (なし) | 0px〜(モバイル基準) |
| `sm:` | 640px以上 |
| `md:` | 768px以上(タブレット) |
| `lg:` | 1024px以上(PC) |
| `xl:` | 1280px以上 |
| `2xl:` | 1536px以上 |

> **モバイルファースト**=**小さい画面の指定が基本**。
> 大きい画面用は `md:` `lg:` で**上書き**する。

---

## 4. ダークモード(=「夜目に優しい切替」)(15分)

### 4-1. 設定

`tailwind.config.js`:
```js
module.exports = {
  darkMode: 'class',  // <html class="dark"> でダーク化
  ...
};
```

### 4-2. クラス書き方

```tsx
{/* 普通=白背景・黒文字、ダーク=濃い青背景・白文字 */}
<div className="bg-white dark:bg-slate-900 text-black dark:text-white">
  ...
</div>
```

### 4-3. 切替コード

```tsx
// ボタンクリックで切替
<button onClick={() => document.documentElement.classList.toggle("dark")}>
  ダークモード切替
</button>
```

> **永続化(localStorage):**
> 切替時に `localStorage.setItem("theme", "dark")` で保存。
> 起動時に取り出して反映=**次回も同じ設定**。

---

## 5. アニメーション(Framer Motion)(=「滑らかな動き」)(15分)

```bash
npm install framer-motion
```

```tsx
import { motion } from "framer-motion";

<motion.div
  initial={{ opacity: 0, y: 20 }}    // 開始(透明・20px下)
  animate={{ opacity: 1, y: 0 }}     // 終了(不透明・元位置)
  transition={{ duration: 0.3 }}     // 0.3秒で
>
  Hello
</motion.div>
```

→ **滑らかなUI**で印象3倍。

### 主要パターン

```tsx
// フェードイン
<motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} />

// 下からスライドイン
<motion.div initial={{ y: 50 }} animate={{ y: 0 }} />

// ホバーで拡大
<motion.button whileHover={{ scale: 1.1 }}>クリック</motion.button>

// ドラッグ可能
<motion.div drag />
```

> **やりすぎ注意。**
> 動きは**控えめ**が一番プロっぽい。**0.2〜0.4秒**が目安。

---

## 6. よくあるエラーと対処

| 症状 | 原因 | 直し方 |
|---|---|---|
| Tailwind クラスが効かない | `content` の指定漏れ | `tailwind.config.js` の `content` にファイルパス追加 |
| ダークモード反応しない | `darkMode: 'class'` 抜け | config に追加 |
| `cn is not a function` | utils.ts 未配置 | `npx shadcn-ui@latest init` でセット |
| `@/components/ui/...` 解決失敗 | tsconfig の path 未設定 | `paths` を `tsconfig.json` に追加 |
| Tailwind 設定後も色出ない | npm 再起動忘れ | dev サーバー再起動 |
| Shadcn パーツの見た目が変 | globals.css 未import | `import "./globals.css"` 追加 |
| Framer Motion がチカチカ | レンダー毎に新規生成 | コンポーネントの外で variants 定義 |
| レスポンシブが効かない | プレフィックスの順序 | 「モバイル → md → lg」の順 |

---

## 7. ⭐ 課題

```
1. Shadcn で管理画面(顧客CRUD)を1日で作る
   - サイドバー + テーブル + 編集ダイアログ
   - Toast で完了通知

2. ダークモード切替実装
   - localStorage 永続化込み
   - 切替アニメ(Framer Motion)

3. Framer Motion でカードのフェード
   - リスト各項目を順番にフェードイン

4. レスポンシブ完全対応
   - スマホ:1列、タブ:2列、PC:3列
   - ナビをハンバーガー化
```

---

## 8. 用語まとめ表

| 用語 | 一言で |
|---|---|
| Tailwind CSS | クラスでデザインするCSSフレームワーク |
| `flex` / `grid` | 横並び・グリッド配置 |
| `p-4` / `m-4` | パディング・マージン(× 4px) |
| `rounded-*` | 角丸 |
| `shadow-*` | 影 |
| `hover:*` | マウスを乗せた時 |
| `dark:*` | ダークモード時 |
| `md:` / `lg:` | 中サイズ / 大サイズ画面用 |
| Shadcn/ui | コピペで使える完成コンポーネント集 |
| Card / Dialog / Toast | カード / モーダル / 通知 |
| `npx shadcn-ui add ...` | コンポーネント追加コマンド |
| `darkMode: 'class'` | クラス切替式ダークモード |
| Framer Motion | アニメライブラリ |
| `motion.div` | アニメ可能なdiv |
| `initial` / `animate` | 開始 / 終了の状態 |
| `whileHover` | ホバー中の状態 |
| ブレークポイント | 画面サイズの境界線 |
| モバイルファースト | 小画面基準で書くスタイル |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **UI 実装 / LP制作 / コーポレートサイト構築**:単価 30〜80万円(プラットフォーム例:ランサーズ/直契約/制作会社の下請け)
- **既存サイトのリブランディング(Tailwind + Shadcn)**:単価 30〜80万円
- **SaaS の管理画面UI実装**:単価 30〜100万円(Shadcn ベースで爆速)

### クロードへの頼み方(営業文を書かせる例)
```
ランサーズの「Webサイト・LPデザインから実装」案件向け提案文を書いて。
Day 54 までで習得:Tailwind / Shadcn UI / Framer Motion / レスポンシブ / ダークモード
売り:デザイン雛形がなくても、Shadcnベースで素早くプロっぽいUIを組める
納期:LP 1ページ 1〜2週間 / SaaS管理画面 3〜4週間
予算:LP 30万円〜 / SaaS管理画面 80万円〜
ポイント:バックエンド(FastAPI / Stripe)まで対応可能
500字
```

### この章だけでは足りないもの(次に学ぶべき)
- Stripe(Day 55)で課金組み込み
- メール / 通知(Day 56)で SaaS 完成
- 法人化(Day 57)で法人案件への足場固め

---

## ✅ チェック

- [ ] Tailwind 主要クラス(flex/p/m/rounded/shadow/hover)使える
- [ ] Shadcn コンポーネント(Button/Card/Dialog/Toast)使える
- [ ] レスポンシブ(モバイル/タブ/PC)対応できる
- [ ] ダークモード切替実装できる
- [ ] Framer Motion でフェード/スライド入れられる
- [ ] よくあるエラー(content漏れ/path未設定)を直せる

---

## 進捗

```
Day 54: Tailwind + Shadcn 完了。
プロのUIを1日で作れるスキル獲得。
ダッシュボード/SaaS の見た目で困らない。
```

## 次

`55_Stripe_決済.md` へ。**SaaS で月額課金を取る仕組み**。
