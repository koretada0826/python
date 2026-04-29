# Day 4:pandas - エクセル/CSV処理

## 💰 この章後の月収:**3〜10万円**(初受注圏)
## ⏱ 所要:3時間

ランサーズ案件の**80%はここの応用**。最重要章。

---

## ゴール

- CSV/Excel を読み書き
- フィルタ・集計・並び替え
- 複数ファイル統合
- グラフ描画
- **「複数Excel統合」を成果物として完成**

---

## 1. インストール(5分)

```
pip install pandas openpyxl matplotlib
```

(anaconda 入ってる人は不要)

---

## 2. 基本(20分)

```python
import pandas as pd

# データ作成
df = pd.DataFrame({
    "名前": ["太郎", "花子", "次郎"],
    "年齢": [25, 30, 22],
    "給料": [30, 40, 28]
})
print(df)
```

```
   名前  年齢  給料
0  太郎  25  30
1  花子  30  40
2  次郎  22  28
```

→ これが **DataFrame**(エクセルの表)。

### CSV/Excel 読み書き

```python
# 読み込み
df = pd.read_csv("data.csv", encoding="utf-8")
df = pd.read_excel("data.xlsx")

# 確認
df.head()       # 最初5行
df.shape        # (行, 列)
df.columns      # 列名
df.describe()   # 統計量

# 書き出し
df.to_csv("out.csv", index=False, encoding="utf-8")
df.to_excel("out.xlsx", index=False)
```

---

## 3. フィルタ・集計(30分)

### フィルタ

```python
df[df["年齢"] >= 25]                          # 25歳以上
df[(df["年齢"] >= 25) & (df["給料"] >= 30)]    # 複数条件
df[df["名前"].str.contains("太")]             # 部分一致
df[df["名前"].isin(["太郎", "花子"])]          # リスト含む
```

### 並び替え

```python
df.sort_values("年齢")
df.sort_values("給料", ascending=False)
```

### 集計

```python
df["給料"].sum()       # 合計
df["給料"].mean()      # 平均
df["給料"].max()       # 最大

# グループ別
df.groupby("部署")["給料"].sum()
df.groupby("部署").agg({"給料": ["mean", "sum"]})
```

### ピボット(超強力)

```python
df.pivot_table(
    index="部署",
    columns="性別",
    values="給料",
    aggfunc="mean"
)
```

→ エクセルのピボットそのもの。

---

## 4. 複数ファイル統合(30分)

```python
import pandas as pd
from pathlib import Path

# 同じ列のCSV/Excelをまとめて読む
dfs = []
for f in Path(".").glob("sales_*.csv"):
    df = pd.read_csv(f)
    df["元ファイル"] = f.name
    dfs.append(df)

統合 = pd.concat(dfs, ignore_index=True)
統合.to_excel("merged.xlsx", index=False)
```

→ **これだけでランサーズ案件1件 5,000-15,000円**。

### 横に結合(merge)

```python
社員 = pd.read_csv("社員.csv")  # id, 名前
給料 = pd.read_csv("給料.csv")  # id, 額

合体 = pd.merge(社員, 給料, on="id")
```

---

## 5. 日付操作(20分)

```python
df["日付"] = pd.to_datetime(df["日付"])

df["月"] = df["日付"].dt.to_period("M")  # 2026-01
df["曜日"] = df["日付"].dt.day_name()

# 月別集計
月別 = df.groupby(df["日付"].dt.to_period("M"))["売上"].sum()

# 期間で絞る
df[(df["日付"] >= "2026-01-01") & (df["日付"] <= "2026-12-31")]
```

---

## 6. グラフ(20分)

```python
import matplotlib.pyplot as plt
import matplotlib
matplotlib.rc('font', family='Hiragino Sans')  # Mac日本語

# 月別売上(折れ線)
月別 = df.groupby(df["日付"].dt.to_period("M"))["売上"].sum()
月別.plot(kind="line", title="月別売上")
plt.savefig("chart.png", dpi=150)
plt.show()

# カテゴリ別(棒)
df["カテゴリ"].value_counts().plot(kind="bar")

# 1枚に複数
fig, axes = plt.subplots(1, 2, figsize=(12, 5))
df["売上"].plot(kind="line", ax=axes[0])
df.groupby("カテゴリ")["売上"].sum().plot(kind="bar", ax=axes[1])
plt.tight_layout()
plt.savefig("dashboard.png")
```

---

## 7. ⭐ 売れる成果物 #1:Excel/CSV統合ツール(60分)

### 仕様

- フォルダの sales_*.xlsx を全部読み込み
- 1つの merged.xlsx に統合
- 月別・カテゴリ別のサマリーシート追加

### クロードに頼む

```
以下を実装するPythonコード書いて。

入力: data/ フォルダの sales_*.csv または sales_*.xlsx
共通の列: [日付, 商品, カテゴリ, 売上]

出力: merged.xlsx
- "all" シート: 全データ統合
- "monthly" シート: 月別売上ピボット
- "category" シート: カテゴリ別合計

サンプルデータも自動生成。
matplotlib でグラフ画像 dashboard.png も作成。
```

### 完成イメージ

```
$ python3 merge_sales.py
12ファイル統合中... 1,200行
merged.xlsx 出力完了
- monthly: 12ヶ月分
- category: 5カテゴリ
- グラフ: dashboard.png
```

→ **ランサーズで10,000-30,000円**で売れる。

---

## ✅ チェック

- [ ] CSV/Excel 読み書きできる
- [ ] フィルタ・groupby・pivot_table 使える
- [ ] 日付操作(年/月/曜日抽出)できる
- [ ] 複数ファイル統合できる
- [ ] matplotlib でグラフ描ける
- [ ] **売れる成果物#1(Excel統合)が動いた**

---

## 進捗

```
Day 4: pandas 完了。Excel統合ツール完成。ポートフォリオ1作品目。
```

## 次

`05_スクレイピング.md` へ。**Webからデータ自動取得で月10万円圏**。
