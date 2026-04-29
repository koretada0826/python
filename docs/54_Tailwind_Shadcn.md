# Day 54:Tailwind・Shadcn UI - デザインできる技術者

## 💰 +20-50万(フロント単価)
## ⏱ 10時間

「**コード書ける + デザインできる**」=希少価値**3倍**。

---

## 1. Tailwind CSS(30分)

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init
```

`tailwind.config.js`:
```js
module.exports = {
  content: ["./src/**/*.{html,tsx,jsx}"],
  theme: { extend: {} },
};
```

```tsx
<div className="flex items-center gap-4 rounded-lg border bg-white p-6 shadow-md hover:shadow-xl transition">
  <img className="w-16 h-16 rounded-full" src={user.avatar} />
  <div className="flex-1">
    <h3 className="font-bold text-lg">{user.name}</h3>
    <p className="text-gray-600 text-sm">{user.email}</p>
  </div>
  <button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
    フォロー
  </button>
</div>
```

→ クラス書くだけで**プロのUI**。

---

## 2. Shadcn/ui(40分)

「**コピペで使えるUIコンポーネント集**」(現代の最強)

```bash
npx shadcn-ui@latest init
npx shadcn-ui@latest add button card dialog input form table
```

```tsx
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardContent } from "@/components/ui/card";
import { Dialog, DialogTrigger, DialogContent } from "@/components/ui/dialog";

<Card>
  <CardHeader>顧客一覧</CardHeader>
  <CardContent>
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

---

## 3. レスポンシブデザイン(15分)

```tsx
// モバイル基準 → md: タブレット → lg: PC
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  ...
</div>
```

---

## 4. ダークモード(15分)

`tailwind.config.js`:
```js
darkMode: 'class',
```

```tsx
<div className="bg-white dark:bg-slate-900 text-black dark:text-white">
  ...
</div>

// 切替
document.documentElement.classList.toggle("dark");
```

---

## 5. アニメーション(Framer Motion)(15分)

```bash
npm install framer-motion
```

```tsx
import { motion } from "framer-motion";

<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.3 }}
>
  Hello
</motion.div>
```

→ **滑らかなUI**で印象3倍。

---

## 6. ⭐ 課題

```
1. Shadcn で管理画面(顧客CRUD)を1日で作る
2. ダークモード切替実装
3. Framer Motion でカードのフェード
4. レスポンシブ完全対応(スマホ/タブ/PC)
```

---

## ✅ チェック
- [ ] Tailwind 主要クラス
- [ ] Shadcn コンポーネント
- [ ] レスポンシブ
- [ ] ダークモード
- [ ] Framer Motion アニメ

## 次
`55_Stripe_決済.md`
