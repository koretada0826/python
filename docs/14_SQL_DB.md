# Day 14:SQL・データベース - 90%の案件で必要

## 💡 これは何?(小学生でもわかる説明)

**大量のデータを「巨大な図書館」に入れて、爆速で取り出せるようにする日**

### なぜ大事?
案件の90%はDB(データベース)絡み。
**Excelだけだと10万件で動かなくなる**。
SQLが書けるかどうかで、月収30万と月100万の壁が決まる。

### イメージ
- **CSV**=「**自分の机のノート**」(置き場所が1つ、検索は目で探す)
- **DB**=「**国会図書館**」(検索ボタン1秒で本が出てくる)
- **テーブル**=「**本棚1つ**」(同じジャンルの本だけ並ぶ)
- **JOIN**=「**本と作者をリンクさせる仕組み**」(複数本棚をつなげる)
- **インデックス**=「**目次・索引**」(これがあると検索が100倍速い)

### Claude Code に何を頼めばいいか
SQL文・テーブル設計・SQLAlchemy のコードは全部クロード。
あなたは「**どんなデータを持ちたいか**」を伝えるだけ。

---

## 💰 この章後の月収:**+10〜30万円**(SQL案件追加)
## ⏱ 所要:3-4時間

「**Pythonできます**」だけでは月100万に届かない。**SQLは必須スキル**。

---

## なぜSQL?(=必修科目の理由)

- 中規模以上の案件は**ほぼ全部DB絡み**
- 「pandas で CSV」だけだと**月30万止まり**
- SQL書けると**SES案件単価+10-20万**

### よくある案件

| 案件 | 単価 |
|---|---:|
| Excel→DB移行 | 10-30万 |
| データ分析(SQL集計) | 20-50万 |
| BIダッシュボード | 30-100万 |
| Web API のDB設計 | 50-200万 |

---

## ゴール

- SELECT/INSERT/UPDATE/DELETE 書ける(=DBの基本4動作)
- JOIN/GROUP BY/サブクエリ理解
- Python から DB操作できる
- SQLite/PostgreSQL/MySQL 違いわかる

---

## 1. データベースとは(=データの倉庫)(15分)

### 1-1. 何がすごい?

「**表(テーブル)が大量に並んでる倉庫**」。エクセルの強化版。

> **CSVの限界:**
> - 10万行で開くのに30秒
> - 同時編集できない(=チームでつかえない)
> - 検索が線形(上から1行ずつ確認)
>
> **DBなら:**
> - 1億行でも検索1秒
> - 100人同時アクセスOK
> - インデックス使えば**爆速**

### 1-2. CSV vs DB

| 項目 | CSV | DB |
|---|---|---|
| 形式 | 単一ファイル | 複数テーブル |
| 同時アクセス | ロックなし | 同時アクセス可 |
| 容量 | 10万行で重い | 1億行でも軽い |
| 検索速度 | 遅い | インデックスで高速 |
| 用途 | 一時的なデータ | 永続的・本番運用 |

→ 規模が大きくなったら DB一択。

### 1-3. なぜ「テーブル」と呼ぶ?

> **エクセルの「シート」みたいなもの**。
> 行=レコード(1人分のデータ)、列=カラム(項目)。
> 1つのDBの中に、テーブルが何個も入ってる(=シートが何枚もあるエクセル)。

---

## 2. SQLite(練習用、最速)(=お試しDB)(20分)

### 2-1. なぜSQLite?

> **ファイル1つで動くDB**。インストール不要、Python標準ライブラリで使える。
> PostgreSQL/MySQLは本格的だけど、設定が面倒。
> **練習はSQLite、本番はPostgreSQL**が定石。

### 2-2. 起動

```python
import sqlite3

# DBに接続(ファイルが無ければ自動作成)
conn = sqlite3.connect("test.db")
cur = conn.cursor()  # カーソル(=操作する手)を作る

# テーブル作成(IF NOT EXISTS で「無ければ作る」)
cur.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,    -- 主キー(自動連番)
    name TEXT,                  -- 文字列
    age INTEGER                 -- 整数
)
""")

# データ挿入(? はプレースホルダ=後で値を入れる場所)
cur.execute("INSERT INTO users (name, age) VALUES (?, ?)", ("太郎", 25))
conn.commit()  # 保存(commit しないと消える)

# 検索
cur.execute("SELECT * FROM users")
for row in cur.fetchall():
    print(row)  # → (1, '太郎', 25)

conn.close()  # 接続を閉じる
```

→ **これだけで動く**。試しに走らせる。

### よくある間違い

| ミス | エラー | 直し方 |
|---|---|---|
| `commit()` 忘れ | データが消える | INSERT/UPDATE後に必ず `conn.commit()` |
| 接続を閉じない | ファイルロックされる | `with` 文で自動クローズ |
| 文字列を直接埋め込む | SQLインジェクション脆弱性 | プレースホルダ `?` を使う |

---

## 3. SQL基本構文(=DBに話しかける言葉)(60分)

### 3-1. SELECT(検索)= 「取り出して」

```sql
SELECT * FROM users;                          -- 全部
SELECT name, age FROM users;                   -- 列指定
SELECT * FROM users WHERE age >= 25;           -- 条件
SELECT * FROM users WHERE name LIKE '太%';     -- 部分一致(太で始まる)
SELECT * FROM users ORDER BY age DESC LIMIT 5; -- 並び替え+件数制限
```

> **読み方:**
> - `*` = 「全部の列」
> - `WHERE` = 「条件」(if みたいなもの)
> - `LIKE '太%'` = 「太で始まる文字列」(%は0文字以上)
> - `ORDER BY ... DESC` = 「降順で並べる」(ASC = 昇順)
> - `LIMIT 5` = 「上から5件だけ」

### 3-2. INSERT(挿入)= 「追加して」

```sql
INSERT INTO users (name, age) VALUES ('花子', 30);

-- 複数同時(高速)
INSERT INTO users (name, age) VALUES
  ('次郎', 22),
  ('三郎', 28);
```

### 3-3. UPDATE(更新)= 「書き換えて」

```sql
UPDATE users SET age = 26 WHERE name = '太郎';
```

> **⚠️ WHERE 必須!**
> WHERE 忘れると**全データを書き換える**(=大事故)。
> 必ず `WHERE` を先に書いてから `UPDATE` で実行。

### 3-4. DELETE(削除)= 「消して」

```sql
DELETE FROM users WHERE age < 20;
```

> **⚠️ こちらもWHERE必須!**
> WHERE 忘れると**全削除**。
> 本番では `BEGIN TRANSACTION` → `DELETE` → 確認 → `COMMIT`の流れで安全に。

### 3-5. CREATE TABLE(テーブル作成)= 「本棚を作って」

```sql
CREATE TABLE products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,  -- 自動採番される番号
    name TEXT NOT NULL,                     -- 必須(NULLダメ)
    price INTEGER,                          -- 価格
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP  -- 自動で現在時刻
);
```

| 制約 | 意味 |
|---|---|
| `PRIMARY KEY` | 主キー(重複ダメ・必須) |
| `AUTOINCREMENT` | 自動連番 |
| `NOT NULL` | 空っぽダメ |
| `UNIQUE` | 値が重複ダメ |
| `DEFAULT` | 省略時の初期値 |

---

## 4. JOIN(=複数テーブル結合)(40分)

実務の核心。**JOIN わかると単価+5万**。

### 4-1. なぜJOINが必要?

> **DBは「データを分割して持つ」のが基本**。
> 例: ユーザー情報と注文情報は別テーブルに分ける。
> 「太郎の注文を見たい」時は、両方のテーブルを**つなぐ(=JOIN)**必要がある。

### 4-2. サンプル

```sql
-- usersテーブル
id | name
1  | 太郎
2  | 花子

-- ordersテーブル
id | user_id | product | amount
1  | 1       | 本     | 1000
2  | 1       | ペン    | 200
3  | 2       | 本     | 1500
```

### 4-3. INNER JOIN(両方にある)= 「両方マッチしたものだけ」

```sql
SELECT users.name, orders.product, orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id;  -- usersのidとordersのuser_idで紐付け
```

結果:
```
太郎 | 本   | 1000
太郎 | ペン  | 200
花子 | 本   | 1500
```

### 4-4. LEFT JOIN(左の全件 + 右の一致)= 「左は全員残す」

```sql
SELECT users.name, orders.product
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

→ 注文なしの user も「product NULL」で出る。

> **使い分け:**
> - 「注文ある人だけ」 → INNER JOIN
> - 「全員見たい(注文無くても)」 → LEFT JOIN

### 4-5. JOIN の種類

| 種類 | 意味 |
|---|---|
| `INNER JOIN` | 両方にあるものだけ |
| `LEFT JOIN` | 左の全件 |
| `RIGHT JOIN` | 右の全件(SQLiteは不可) |
| `FULL JOIN` | 両方の全件(SQLiteは不可) |
| `CROSS JOIN` | 全組合せ |

---

## 5. GROUP BY(集計)(=グループ分け)(30分)

### 5-1. なぜ集計?

> **「商品ごとの売上合計」「曜日ごとのアクセス数」みたいな質問に答える**。
> Excelの**ピボットテーブル**と同じ機能。

### 5-2. 基本

```sql
-- ユーザーごとの購入合計
SELECT user_id, SUM(amount), COUNT(*)
FROM orders
GROUP BY user_id;

-- 商品ごとの売上(降順)
SELECT product, SUM(amount) as total
FROM orders
GROUP BY product
ORDER BY total DESC;
```

### 5-3. 集計関数

| 関数 | 意味 |
|---|---|
| `COUNT(*)` | 個数 |
| `SUM(amount)` | 合計 |
| `AVG(amount)` | 平均 |
| `MAX(amount)` | 最大 |
| `MIN(amount)` | 最小 |

### 5-4. HAVING(集計後の条件)= 「集計後にフィルタ」

```sql
SELECT user_id, SUM(amount) as total
FROM orders
GROUP BY user_id
HAVING total >= 1000;  -- 合計1000以上のユーザーだけ
```

> **WHEREとHAVINGの違い:**
> - `WHERE` = 集計**前**にフィルタ(=元データから選ぶ)
> - `HAVING` = 集計**後**にフィルタ(=合計1000以上だけ)

---

## 6. サブクエリ・CTE(=入れ子クエリ)(20分)

### 6-1. サブクエリ(=クエリの中にクエリ)

```sql
-- 1000円以上買ったユーザーの名前
SELECT name FROM users
WHERE id IN (SELECT user_id FROM orders WHERE amount > 1000);
```

> **読み方:**
> 1. 内側: 「`amount > 1000` の `user_id` を取る」
> 2. 外側: 「その user_id を持つ users の name を取る」

### 6-2. CTE(WITH句、読みやすい)= 「変数みたいな名前を付ける」

```sql
WITH high_spenders AS (
  SELECT user_id, SUM(amount) as total
  FROM orders
  GROUP BY user_id
  HAVING total >= 1000
)
SELECT u.name, h.total
FROM users u
JOIN high_spenders h ON u.id = h.user_id;
```

→ **CTEはプロが好むやり方**。読みやすい・デバッグしやすい。

> **なぜCTE?**
> サブクエリは入れ子が深くなると読めない。
> CTEは「**この計算結果に名前を付ける**」感覚で、上から順に読める。

---

## 7. インデックス(=本の索引)(10分)

### 7-1. なぜ必要?

「**目次**」みたいなもの。検索が爆速になる。

> **検索の仕組み:**
> インデックスなし → 100万行を1行ずつ確認(=線形検索、遅い)
> インデックスあり → 索引を引く(=対数検索、爆速)

### 7-2. 作り方

```sql
CREATE INDEX idx_user_id ON orders(user_id);
```

→ `WHERE user_id = 1` の検索が100倍速い。

### 7-3. 注意点

| メリット | デメリット |
|---|---|
| SELECT が爆速 | INSERT/UPDATE が遅くなる |
| WHERE/JOIN が高速 | ストレージ容量が増える |

→ **よく検索する列だけ**に貼る。全列に貼ると逆効果。

---

## 8. Python から DB(=2つの世界をつなぐ)(40分)

### 8-1. sqlite3(標準)

```python
import sqlite3

# with文で自動クローズ(プロの書き方)
with sqlite3.connect("test.db") as conn:
    cur = conn.cursor()
    cur.execute("SELECT * FROM users WHERE age > ?", (25,))  # プレースホルダ
    rows = cur.fetchall()
```

### 8-2. SQLAlchemy(本格、必修)

```bash
pip install sqlalchemy
```

```python
from sqlalchemy import create_engine, text

# エンジン作成(=DB接続の管理人)
engine = create_engine("sqlite:///test.db")

with engine.connect() as conn:
    # text() でSQL文を渡す、:age はパラメータ
    result = conn.execute(text("SELECT * FROM users WHERE age > :age"), {"age": 25})
    for row in result:
        print(row)
```

### 8-3. pandas で DB読み書き(=実務最強コンビ)

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("sqlite:///test.db")

# DB → DataFrame
df = pd.read_sql("SELECT * FROM users", engine)

# DataFrame → DB(if_exists で挙動指定)
df.to_sql("users_backup", engine, if_exists="replace", index=False)
# replace = 既存テーブル削除して作り直し
# append  = 既存に追記
# fail    = 既存があればエラー
```

→ **pandas + SQL の組合せ最強**。

> **これができると:**
> - DBから取得 → pandasで集計 → DBに書き戻し
> - 月次レポートを5分で生成
> - **月20万案件**が取れる

### 8-4. ORM(SQLAlchemy 2.x の現代的書き方)

```python
from sqlalchemy.orm import declarative_base, sessionmaker
from sqlalchemy import Column, Integer, String

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    age = Column(Integer)

# 検索(SQLを書かない!)
session = sessionmaker(bind=engine)()
users = session.query(User).filter(User.age >= 25).all()
for u in users:
    print(u.name, u.age)
```

→ **SQLを書かずにPythonでDB操作**できる。

> **ORMのメリット:**
> - SQLインジェクション自動防御
> - DB切替が楽(SQLite → PostgreSQLが1行)
> - Pythonの型ヒントで保護

---

## 9. PostgreSQL(=本番用の本気DB)(20分)

### 9-1. なぜPostgreSQL?

法人案件の80%は PostgreSQL。
- 高機能(JSON型・配列型・全文検索)
- オープンソース・無料
- 大規模で安定

### 9-2. Mac で起動

```bash
brew install postgresql
brew services start postgresql
createdb mydb
```

### 9-3. Python から接続

```bash
pip install psycopg2-binary  # PostgreSQL用ドライバ
```

```python
from sqlalchemy import create_engine
engine = create_engine("postgresql://user:pass@localhost:5432/mydb")
```

→ SQLite と同じ書き方で動く。**SQLAlchemy が抽象化してくれる**。

### 9-4. PostgreSQL 特有の便利機能

```sql
-- JSON型(NoSQLっぽい使い方)
SELECT data->>'name' FROM products WHERE data->>'category' = '本';

-- 配列型
SELECT * FROM users WHERE tags && ARRAY['python', 'ai'];

-- ウィンドウ関数(超強力)
SELECT name, salary,
  RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rank
FROM employees;
```

→ これが書けると「**DBエンジニア**」を名乗れる。

---

## 10. ⭐ 売れる成果物 #7:DB連携Webアプリ(60分)

クロードに頼む:

```
Streamlit + SQLAlchemy + SQLite で
顧客管理アプリを作って。

機能:
- 顧客一覧表示(検索・並び替え可能)
- 顧客の追加・編集・削除
- 月別売上集計(SQL GROUP BY 使用)
- グラフ表示
- CSV/Excel エクスポート

DB設計:
- customers テーブル(id, name, email, created_at)
- orders テーブル(id, customer_id, product, amount, ordered_at)
- INNER JOIN で売上集計

requirements.txt も含めて。
```

→ **これでDBスキル証明できる作品ができる**。

---

## 11. 今日学んだ用語まとめ

| 用語 | 一言で |
|---|---|
| データベース(DB) | 大量データを高速に管理する倉庫 |
| テーブル | DBの中の本棚(=エクセルのシート) |
| カラム / 行 | 列 / レコード(エクセルの列・行) |
| 主キー(PRIMARY KEY) | 各行を識別する重複しない列 |
| SELECT | データを取り出す命令 |
| INSERT | データを追加する命令 |
| UPDATE | データを書き換える命令(WHERE必須) |
| DELETE | データを削除する命令(WHERE必須) |
| WHERE | 条件指定(if みたいなもの) |
| JOIN | 複数テーブルを紐付ける操作 |
| GROUP BY | グループ分けして集計 |
| HAVING | 集計後の条件フィルタ |
| インデックス | 検索を爆速化する索引 |
| サブクエリ | クエリの中のクエリ |
| CTE(WITH句) | 名前付きの中間結果(読みやすい) |
| SQLite | ファイル1個で動くお試しDB |
| PostgreSQL | 本番用の高機能DB |
| SQLAlchemy | PythonからDBを操作する標準ライブラリ |
| ORM | SQLを書かずDBを操作する仕組み |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **DB設計・データ移行案件**:単価 5〜30万円(プラットフォーム例:ランサーズ/クラウドワークス/SES)
- **既存システムのDB抽出 + Excel化レポート**:単価 5〜20万円(SQL + pandas で月次集計化)
- **業務DB(SQLite/PostgreSQL)構築 + CRUD GUI**:単価 10〜30万円(Streamlit + SQLAlchemy)

### クロードへの頼み方(営業文を書かせる例)
```
ランサーズの「業務データベース構築・データ抽出」案件への提案文を書いて。
Day 14 までで習得したスキル:SQL(SELECT/JOIN/GROUP BY/CTE)/ SQLAlchemy / pandas <-> DB
売り:Python から DB 操作 + Excel/CSV 出力まで一気通貫で対応
納期:1〜2週間
予算:10万円前後
ポイント:データ量が多くてもバッチ処理で安定して回す方法を提案できる
```

### この章だけでは足りないもの(次に学ぶべき)
- チーム開発で必須の Git 高度(Day 15)
- Docker(Day 16)で環境差をなくす
- DBの内部最適化・インデックスチューニング(Day 36)

---

## ✅ チェック

- [ ] SQLite で SELECT/INSERT/UPDATE/DELETE 書ける
- [ ] JOIN(INNER/LEFT)わかる
- [ ] GROUP BY と集計関数 使える
- [ ] CTE(WITH句)書ける
- [ ] SQLAlchemy で Python から接続できる
- [ ] pandas <-> DB の往復できる
- [ ] PostgreSQL の存在と使い方を知ってる
- [ ] **DB連携Webアプリが動いた**

---

## 進捗

```
Day 14: SQL・DB完了。SQLite/PostgreSQL/SQLAlchemy/JOIN/CTE理解。
顧客管理アプリ完成。
```

## 次

`15_Git高度.md` へ。**チーム開発のための Git**。
