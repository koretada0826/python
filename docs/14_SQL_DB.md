# Day 14:SQL・データベース - 90%の案件で必要

## 💡 これは何?(小学生でもわかる説明)

****大量データを高速に保管・検索する仕組み**(データベース)**

### なぜ大事?
案件の90%は DB 絡む。CSVだけだと10万件で死ぬ

### イメージ
ノートに書く=CSV / **巨大な図書館**=DB(検索が爆速)

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 この章後の月収:**+10〜30万円**(SQL案件追加)
## ⏱ 所要:3-4時間

「**Pythonできます**」だけでは月100万に届かない。**SQLは必須スキル**。

---

## なぜSQL?

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

- SELECT/INSERT/UPDATE/DELETE 書ける
- JOIN/GROUP BY/サブクエリ理解
- Python から DB操作できる
- SQLite/PostgreSQL/MySQL 違いわかる

---

## 1. データベースとは(15分)

「**表(テーブル)が大量に並んでる倉庫**」。エクセルの強化版。

### CSV vs DB

| CSV | DB |
|---|---|
| 単一ファイル | 複数テーブル |
| ロックなし | 同時アクセス可 |
| 10万行で重い | 1億行でも軽い |
| 検索遅い | インデックスで高速 |

→ 規模が大きくなったら DB一択。

---

## 2. SQLite(練習用、最速)(20分)

ファイル1つで完結する DB。Python標準で使える。

### 起動

```python
import sqlite3

conn = sqlite3.connect("test.db")
cur = conn.cursor()

# テーブル作成
cur.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    name TEXT,
    age INTEGER
)
""")

# データ挿入
cur.execute("INSERT INTO users (name, age) VALUES (?, ?)", ("太郎", 25))
conn.commit()

# 検索
cur.execute("SELECT * FROM users")
for row in cur.fetchall():
    print(row)

conn.close()
```

→ **これだけで動く**。試しに走らせる。

---

## 3. SQL基本構文(60分)

### SELECT(検索)

```sql
SELECT * FROM users;                          -- 全部
SELECT name, age FROM users;                   -- 列指定
SELECT * FROM users WHERE age >= 25;           -- 条件
SELECT * FROM users WHERE name LIKE '太%';     -- 部分一致
SELECT * FROM users ORDER BY age DESC LIMIT 5; -- 並び替え+件数
```

### INSERT(挿入)

```sql
INSERT INTO users (name, age) VALUES ('花子', 30);

-- 複数同時
INSERT INTO users (name, age) VALUES
  ('次郎', 22),
  ('三郎', 28);
```

### UPDATE(更新)

```sql
UPDATE users SET age = 26 WHERE name = '太郎';
```

### DELETE(削除)

```sql
DELETE FROM users WHERE age < 20;
```

### CREATE TABLE(テーブル作成)

```sql
CREATE TABLE products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    price INTEGER,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## 4. JOIN(複数テーブル結合)(40分)

実務の核心。**JOIN わかると単価+5万**。

### サンプル

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

### INNER JOIN(両方にある)

```sql
SELECT users.name, orders.product, orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```

結果:
```
太郎 | 本   | 1000
太郎 | ペン  | 200
花子 | 本   | 1500
```

### LEFT JOIN(左の全件 + 右の一致)

```sql
SELECT users.name, orders.product
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

→ 注文なしの user も「product NULL」で出る。

### JOIN の種類

| 種類 | 意味 |
|---|---|
| `INNER JOIN` | 両方にあるものだけ |
| `LEFT JOIN` | 左の全件 |
| `RIGHT JOIN` | 右の全件(SQLiteは不可) |
| `FULL JOIN` | 両方の全件(SQLiteは不可) |
| `CROSS JOIN` | 全組合せ |

---

## 5. GROUP BY(集計)(30分)

```sql
-- ユーザーごとの購入合計
SELECT user_id, SUM(amount), COUNT(*)
FROM orders
GROUP BY user_id;

-- 商品ごとの売上
SELECT product, SUM(amount) as total
FROM orders
GROUP BY product
ORDER BY total DESC;

-- 集計関数
COUNT(*)     -- 個数
SUM(amount)  -- 合計
AVG(amount)  -- 平均
MAX(amount)  -- 最大
MIN(amount)  -- 最小
```

### HAVING(集計後の条件)

```sql
SELECT user_id, SUM(amount) as total
FROM orders
GROUP BY user_id
HAVING total >= 1000;  -- 合計1000以上のユーザーだけ
```

---

## 6. サブクエリ・CTE(20分)

### サブクエリ

```sql
SELECT name FROM users
WHERE id IN (SELECT user_id FROM orders WHERE amount > 1000);
```

### CTE(WITH句、読みやすい)

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

→ **CTEはプロが好むやり方**。

---

## 7. インデックス(10分)

「**目次**」みたいなもの。検索が爆速になる。

```sql
CREATE INDEX idx_user_id ON orders(user_id);
```

→ `WHERE user_id = 1` の検索が100倍速い。

ただし**INSERTは遅くなる**ので、**よく検索する列だけ**に貼る。

---

## 8. Python から DB(40分)

### sqlite3(標準)

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cur = conn.cursor()
    cur.execute("SELECT * FROM users WHERE age > ?", (25,))
    rows = cur.fetchall()
```

### SQLAlchemy(本格、必修)

```bash
pip install sqlalchemy
```

```python
from sqlalchemy import create_engine, text

engine = create_engine("sqlite:///test.db")

with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM users WHERE age > :age"), {"age": 25})
    for row in result:
        print(row)
```

### pandas で DB読み書き

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("sqlite:///test.db")

# DB → DataFrame
df = pd.read_sql("SELECT * FROM users", engine)

# DataFrame → DB
df.to_sql("users_backup", engine, if_exists="replace", index=False)
```

→ **pandas + SQL の組合せ最強**。

### ORM(SQLAlchemy 2.x の現代的書き方)

```python
from sqlalchemy.orm import declarative_base, sessionmaker
from sqlalchemy import Column, Integer, String

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    age = Column(Integer)

# 検索
session = sessionmaker(bind=engine)()
users = session.query(User).filter(User.age >= 25).all()
for u in users:
    print(u.name, u.age)
```

→ **SQLを書かずにPythonでDB操作**できる。

---

## 9. PostgreSQL(本番用)(20分)

法人案件の80%は PostgreSQL。

### Mac で起動

```bash
brew install postgresql
brew services start postgresql
createdb mydb
```

### Python から接続

```bash
pip install psycopg2-binary
```

```python
from sqlalchemy import create_engine
engine = create_engine("postgresql://user:pass@localhost:5432/mydb")
```

→ SQLite と同じ書き方で動く。**SQLAlchemy が抽象化してくれる**。

### PostgreSQL 特有の便利機能

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
