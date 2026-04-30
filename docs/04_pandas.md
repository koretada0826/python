# Day 4:pandas - エクセル/CSV処理

## 💡 これは何?(小学生でもわかる説明)

****Excel を Python で動かす**最強ライブラリ**

### なぜ大事?
ランサーズ案件の **80% がエクセル絡み**。これができれば月10万円見える。
人間が手作業で1日かかる集計を、pandas は **1分で終わらせる**。

### イメージ
- **DataFrame(=データフレーム)** = 「**Excel の表そのもの**」(行と列がある)
- **Series(=シリーズ)** = 「**Excel の1列だけ**」
- **groupby** = 「**部活ごとに集めて点数の合計を出す**」みたいな仕分け
- **merge** = 「**2枚の名簿を ID でくっつける**」
- **pivot_table** = 「**Excel のピボットテーブルそのまま**」
- **matplotlib** = 「**お絵描き道具**」(グラフを描く)

100ファイルを手作業で集計 → pandas で **1分で完了**。
これが「**Python で稼げる**」の正体。

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。
**「データの形」を Claude に伝えれば、加工コードはすぐ出てくる**。

---

## 💰 この章後の月収:**3〜10万円**(初受注圏)
## ⏱ 所要:3時間

ランサーズ案件の **80% はここの応用**。最重要章。

---

## ゴール

- CSV / Excel を読み書きできる
- フィルタ(=条件で絞り込み)・集計・並び替えができる
- 複数ファイルを1つに統合できる
- 日付データを扱える
- グラフを描ける
- **「複数 Excel 統合」を成果物として完成させる**

---

## 1. インストール(=「道具を揃える」)(5分)

### 1-1. 何を入れる?

```
pip install pandas openpyxl matplotlib
```

> **何が入るか:**
> - `pandas` = 表データを扱うメインライブラリ
> - `openpyxl` = Excel ファイル(.xlsx)を読み書きする裏方
> - `matplotlib` = グラフ描画ライブラリ

(Anaconda 入れてる人は不要。最初から入ってる)

### 1-2. pip って何?

> **`pip` = Python 専用のアプリストア**。
> ライブラリ(=他人が作った便利な部品)をネットから自動でインストールしてくれる。

---

## 2. DataFrame の基本(=「Excel の表」)(20分)

### 2-1. DataFrame を作る

```python
import pandas as pd   # pd という愛称で呼ぶ(慣習)

# 辞書から作る(列ごとにデータを並べる)
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
左の `0 1 2` は **インデックス**(行番号)。

> **なぜ `pd` という愛称?**
> 業界全員これ使ってる。Claude が書くコードもこの形。**真似するだけでOK**。

### 2-2. CSV / Excel 読み書き

```python
# 読み込み
df = pd.read_csv("data.csv", encoding="utf-8")
df = pd.read_excel("data.xlsx")

# 中身を確認するメソッド(超頻出)
df.head()       # 最初の5行を見る
df.tail()       # 最後の5行を見る
df.shape        # (行数, 列数) を返す
df.columns      # 列名を返す
df.dtypes       # 各列の型を返す
df.describe()   # 平均・最大・最小などの統計を一発で出す
df.info()       # データ全体の情報

# 書き出し
df.to_csv("out.csv", index=False, encoding="utf-8")
df.to_excel("out.xlsx", index=False)
```

> **`index=False` は何?**
> 書き出すとき左端の行番号を **省く**おまじない。これ無いと余計な列が増える。

### 2-3. 列・行へのアクセス

```python
df["年齢"]              # "年齢" 列を取り出す(Series)
df[["名前", "年齢"]]    # 複数列を取り出す(リストで囲む)
df.iloc[0]              # 0行目を取り出す
df.iloc[0:3]            # 0〜2行目
df.loc[df["年齢"] >= 25] # 条件で行を取り出す
```

---

## 3. フィルタ・集計(=「絞り込み・合計」)(30分)

### 3-1. フィルタ(=「条件で絞る」)

```python
df[df["年齢"] >= 25]                          # 25歳以上
df[(df["年齢"] >= 25) & (df["給料"] >= 30)]    # AND(両方満たす)
df[(df["年齢"] < 25) | (df["給料"] >= 40)]     # OR(どちらかを満たす)
df[df["名前"].str.contains("太")]             # 部分一致(「太」を含む)
df[df["名前"].isin(["太郎", "花子"])]          # リストの中にある
```

> **注意:** Python の `and` `or` ではなく **`&` `|`** を使う。
> 各条件を **`( )` で囲む**のも忘れずに。

### 3-2. 並び替え

```python
df.sort_values("年齢")                       # 年齢の昇順
df.sort_values("給料", ascending=False)      # 給料の降順
df.sort_values(["部署", "給料"])             # 複数列で並び替え
```

### 3-3. 集計(=「数字をまとめる」)

```python
df["給料"].sum()       # 合計
df["給料"].mean()      # 平均
df["給料"].max()       # 最大
df["給料"].min()       # 最小
df["給料"].median()    # 中央値
df["給料"].count()     # データの個数
df["部署"].value_counts()  # カテゴリごとの出現回数
```

### 3-4. groupby(=「グループごとに集計」)

```python
# 部署ごとの給料合計
df.groupby("部署")["給料"].sum()

# 複数の集計を一度に
df.groupby("部署").agg({
    "給料": ["mean", "sum"],
    "年齢": "max"
})
```

> **イメージ:** 学校で「**クラスごとの平均点**」を出すのと同じ。
> Excel で言うと「**SUMIF**」「**AVERAGEIF**」を全部一気にやる感じ。

### 3-5. ピボットテーブル(=「Excel のピボットそのもの」)

```python
df.pivot_table(
    index="部署",       # 行に何を出すか
    columns="性別",     # 列に何を出すか
    values="給料",      # マスに入れる数字
    aggfunc="mean"      # 集計方法(平均)
)
```

→ Excel のピボットを **1行で書ける**。これが pandas の真骨頂。

---

## 4. 複数ファイル統合(=「お金になるやつ」)(30分)

### 4-1. 縦に積む(concat)

```python
import pandas as pd
from pathlib import Path

# 同じ列のCSV/Excelをまとめて読む
dfs = []
for f in Path(".").glob("sales_*.csv"):
    df = pd.read_csv(f)
    df["元ファイル"] = f.name      # どのファイル由来か残す(超重要)
    dfs.append(df)

統合 = pd.concat(dfs, ignore_index=True)  # 縦にくっつける
統合.to_excel("merged.xlsx", index=False)
```

> **これだけでランサーズ案件1件 5,000〜15,000円**。
> やってる人が「**Excel 手作業**」だから。Python なら一瞬。

### 4-2. 横に結合(merge=「VLOOKUP」)

```python
社員 = pd.read_csv("社員.csv")  # 列: id, 名前
給料 = pd.read_csv("給料.csv")  # 列: id, 額

合体 = pd.merge(社員, 給料, on="id")  # id を鍵にして合体
```

> **これは Excel の VLOOKUP 関数と同じ**。
> 2つの表を「共通の鍵(=id など)」で結びつける。

### 4-3. merge の種類

| how="..." | 意味 |
|---|---|
| `"inner"` | 両方に存在するものだけ(デフォルト) |
| `"left"` | 左の表は全部残す(右に無いものは空白) |
| `"right"` | 右の表は全部残す |
| `"outer"` | 両方の全データ(片方に無くても残す) |

---

## 5. 日付操作(=「カレンダー処理」)(20分)

### 5-1. 文字列を日付に変換

```python
# まず日付として認識させる(超重要)
df["日付"] = pd.to_datetime(df["日付"])
```

> **なぜ?**
> CSV の日付は **「文字列」** として読まれる。
> 日付として扱うには **`to_datetime`** で変換が必要。

### 5-2. 年月日を取り出す

```python
df["年"] = df["日付"].dt.year
df["月"] = df["日付"].dt.to_period("M")  # 2026-01 形式
df["曜日"] = df["日付"].dt.day_name()    # Monday など
df["日"] = df["日付"].dt.day
```

### 5-3. 月別集計

```python
月別 = df.groupby(df["日付"].dt.to_period("M"))["売上"].sum()
```

### 5-4. 期間で絞る

```python
df[(df["日付"] >= "2026-01-01") & (df["日付"] <= "2026-12-31")]
```

---

## 6. グラフ(=「お絵描き道具」)(20分)

### 6-1. 日本語対応(Mac)

```python
import matplotlib.pyplot as plt
import matplotlib
matplotlib.rc('font', family='Hiragino Sans')  # Mac日本語フォント
```

> **なぜ必要?**
> matplotlib のデフォルトフォントは日本語非対応。
> これを書かないと **□□□(豆腐)**になる。

### 6-2. 折れ線グラフ

```python
月別 = df.groupby(df["日付"].dt.to_period("M"))["売上"].sum()
月別.plot(kind="line", title="月別売上")
plt.savefig("chart.png", dpi=150)  # 画像として保存
plt.show()                          # 画面に表示
```

### 6-3. グラフの種類

| `kind="..."` | 何の絵? |
|---|---|
| `"line"` | 折れ線(時系列向け) |
| `"bar"` | 棒グラフ(カテゴリ比較) |
| `"barh"` | 横棒 |
| `"pie"` | 円グラフ |
| `"scatter"` | 散布図 |
| `"hist"` | ヒストグラム(分布) |

### 6-4. 1枚に複数(ダッシュボード)

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 5))  # 横に2枚
df["売上"].plot(kind="line", ax=axes[0])
df.groupby("カテゴリ")["売上"].sum().plot(kind="bar", ax=axes[1])
plt.tight_layout()           # 重ならないように整える
plt.savefig("dashboard.png")
```

---

## 7. ⭐ 売れる成果物 #1:Excel / CSV 統合ツール(60分)

### 7-1. 仕様

- フォルダの `sales_*.xlsx` を **全部** 読み込み
- 1つの `merged.xlsx` に統合
- **月別・カテゴリ別のサマリーシート**を追加
- グラフ画像も自動生成

### 7-2. クロードに頼む文

```
以下を実装する Python コードを書いて。

入力: data/ フォルダの sales_*.csv または sales_*.xlsx
共通の列: [日付, 商品, カテゴリ, 売上]

出力: merged.xlsx
- "all" シート: 全データ統合(元ファイル名付き)
- "monthly" シート: 月別売上ピボット
- "category" シート: カテゴリ別合計

サンプルデータも自動生成(3ファイル分)。
matplotlib でグラフ画像 dashboard.png も作成。
日本語対応(Mac の Hiragino Sans)。
ファイル名は merge_sales.py。
```

### 7-3. 完成イメージ

```
$ python3 merge_sales.py
12ファイル統合中... 1,200行
merged.xlsx 出力完了
- all: 1200行
- monthly: 12ヶ月分
- category: 5カテゴリ
グラフ: dashboard.png
```

→ **ランサーズで 10,000〜30,000円** で売れる。
**ポートフォリオ作品 #1** として GitHub にアップする(Day 10 で詳しく)。

---

## 8. よくあるエラーと対処

| エラー | 原因 | 直し方 |
|---|---|---|
| `ModuleNotFoundError: pandas` | pandas 入ってない | `pip install pandas` |
| `FileNotFoundError` | ファイルパスが違う | `Path.exists()` で確認 |
| `UnicodeDecodeError` | 文字コードが違う | `encoding="cp932"` を試す |
| `KeyError: '列名'` | 列名のスペル違い | `df.columns` で確認 |
| `SettingWithCopyWarning` | コピー操作の警告 | `.copy()` を付ける |
| 日本語が豆腐になる(グラフ) | フォント未設定 | `matplotlib.rc('font', family='Hiragino Sans')` |
| 日付計算ができない | 文字列のまま | `pd.to_datetime()` で変換 |
| Excel が壊れる | openpyxl 未インストール | `pip install openpyxl` |

---

## 9. 用語まとめ

| 用語 | 一言で |
|---|---|
| pandas | Excel を Python で動かすライブラリ |
| DataFrame (df) | 表データ(Excel のシート相当) |
| Series | 1列だけのデータ |
| `pd.read_csv` / `read_excel` | ファイル読み込み |
| `to_csv` / `to_excel` | ファイル書き出し |
| `head()` | 最初の数行を見る |
| `describe()` | 統計サマリーを出す |
| フィルタ | 条件で行を絞る |
| `groupby` | グループごとに集計 |
| `pivot_table` | クロス集計表(Excel ピボット) |
| `concat` | 縦にくっつける |
| `merge` | 横にくっつける(VLOOKUP) |
| `to_datetime` | 文字列 → 日付に変換 |
| matplotlib | グラフ描画ライブラリ |
| `plt.savefig` | グラフを画像に保存 |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **売上・KPIデータ集計レポート**:単価 10,000〜50,000円(プラットフォーム例:ランサーズ/クラウドワークス/Coconala)
- **複数Excelファイル統合 + ピボット集計**:単価 10,000〜30,000円(店舗別・支店別データの統合)
- **月次グラフ作成 + PDFレポート出力**:単価 20,000〜50,000円(matplotlib + ReportLab で定型化)

### クロードへの頼み方(営業文を書かせる例)
```
ランサーズの「データ集計・レポート作成」案件向けの提案文を書いて。
Day 4 までで習得した pandas(groupby / pivot_table / merge / 日付処理)+ matplotlib を強みに。
納期:1週間
予算:2〜4万円
ポイント:依頼者の Excel フォーマットに合わせて納品(.xlsx + .png グラフ)できることを強調。
過去実績:なし(その代わり、サンプルデータで事前にデモを作ることをアピール)
```

### この章だけでは足りないもの(次に学ぶべき)
- Webからのデータ自動取得(スクレイピング、Day 5)
- 大量データ処理の高速化(numpy / Polars)
- ダッシュボード化(Day 8 Streamlit)

---

## ✅ チェック

- [ ] CSV / Excel 読み書きできる
- [ ] `head()` `describe()` でデータの中身が見られる
- [ ] フィルタ(`&` `|` `isin` `str.contains`)書ける
- [ ] `groupby` で集計できる
- [ ] `pivot_table` 使える
- [ ] 日付操作(年 / 月 / 曜日抽出)できる
- [ ] 複数ファイル統合できる(`concat`)
- [ ] `merge` で2つの表を結合できる
- [ ] matplotlib でグラフ描ける
- [ ] **売れる成果物 #1(Excel 統合)が動いた**

---

## 進捗

```
Day 4: pandas 完了。Excel統合ツール完成。ポートフォリオ1作品目。
ランサーズに出せるレベル到達。
```

## 次

`05_スクレイピング.md` へ。**Webからデータ自動取得で月10万円圏**。
