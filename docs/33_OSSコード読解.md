# Day 33:OSSコード読解 - 他人のコードを読む力

## 💰 +20-50万(複雑案件突破)
## ⏱ 30時間

「**コード書ける**」 と 「**コード読める**」は別。**プロは8割の時間を読むこと**に使う。

---

## 小学生でもわかる導入

「料理本を書ける」 と 「他人の料理本を読んで再現できる」 は違うよね。
プログラミングも一緒。**他人のコードを読めるようになると、無限に勉強できる**。

---

## ゴール

- GitHub の有名OSSのコードを読める
- 「**この機能はどこで実装されてる?**」を5分で見つけられる
- バグ修正のPRが出せる
- アーキテクチャを理解できる

---

## 1. なぜOSSコード読み?(15分)

### こんなメリット

1. **無料の最高の教材**(Django/FastAPI/pandas は世界最高峰のコード)
2. **「正しい書き方」が分かる**(個人ブログより信頼度高い)
3. **GitHub に貢献(PR)→ 経歴に書ける**
4. **シニアの思考が学べる**

---

## 2. コード読みの基本フロー(20分)

### Step 1: README + Architecture を読む

「**この OSS は何を解決してるか**」を理解。

### Step 2: 全体構造を把握(`tree` コマンド)

```bash
git clone https://github.com/tiangolo/fastapi
cd fastapi
tree -L 2 -I "__pycache__|*.pyc"
```

→ **どこに何があるか**を地図化。

### Step 3: エントリポイントから読む

```bash
# FastAPI なら
cat fastapi/__init__.py
cat fastapi/applications.py  # FastAPI クラスが住んでる
```

### Step 4: 自分の興味ある機能を**追いかける**

例: `app.get("/")` がどう動くか?
1. `@app.get` の定義を探す → `applications.py`
2. `add_api_route` を呼んでる → routing.py へ
3. `APIRoute` クラスを読む

→ **1行ずつ追いかける**のがコツ。**全部読まなくていい**。

---

## 3. 必須ツール(15分)

### VS Code の機能

- **F12**: 定義に飛ぶ
- **Shift+F12**: どこで使われてるか
- **Cmd+P**: ファイル名検索
- **Cmd+Shift+F**: 全文検索

→ コード読みの**必修ショートカット**。

### grep / ripgrep

```bash
brew install ripgrep
rg "class FastAPI" fastapi/
rg "def get" --type py
```

### Sourcegraph

ブラウザで GitHub のコードを検索/ジャンプできる。

---

## 4. 読むべき OSS リスト(本気の人)

### 初心者向け(コード少なめ・読みやすい)

| OSS | 学べること |
|---|---|
| **requests**(8千行) | HTTP・API設計 |
| **Flask**(2万行) | Webフレームワーク基礎 |
| **click**(1万行) | CLI ツール作り |

### 中級(設計が美しい)

| OSS | 学べること |
|---|---|
| **FastAPI**(7万行) | 型ヒント・依存注入 |
| **pydantic** | バリデーション設計 |
| **httpx** | 非同期HTTP |
| **rich** | ターミナルUI |

### 上級(規模デカい・本物)

| OSS | 学べること |
|---|---|
| **Django**(60万行) | 大規模Web設計 |
| **pandas**(80万行) | 数値計算最適化 |
| **scikit-learn**(50万行) | ML実装 |
| **PyTorch**(200万行+C++) | テンソル計算 |

→ **1つを2週間かけて読む**のが最強の勉強。

---

## 5. 実践:FastAPI を読んでみる(60分)

### 課題:`app.get` の中身を追え

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def root():
    return {"msg": "hello"}
```

→ この`@app.get` はどこで定義されてる?

#### Step1
```bash
git clone https://github.com/tiangolo/fastapi
cd fastapi/fastapi
rg "def get\(" applications.py
```

→ `class FastAPI` の中に `def get` あり。

#### Step2
中身を読む:
```python
def get(self, path, ...):
    return self.router.get(path, ...)
```

→ `self.router` を探す。`__init__` で `APIRouter` が入る。

#### Step3
`routing.py` の `APIRouter.get` を見る → `add_api_route` を呼んでる → `APIRoute` を作って `self.routes` に append。

→ これで「**`@app.get`は最終的に Routeリストへの追加**」と理解できる。

---

## 6. PR の出し方(30分)

### ステップ

1. リポジトリを **Fork**
2. **Issue** で「これ困ってる人いる」を確認、コメントで「やります」と宣言
3. 自分のFork で branch 作成
4. 修正・テスト追加
5. **PR を出す**(Description にIssue番号 `Fixes #42`)
6. レビュー → 修正 → マージ

### 最初の PR は何書く?

**typo修正・docstring追加・テスト追加** が定番。

```python
# Before
def calc(x):
    """calc"""
    return x * 2

# After(PR)
def calc(x: int) -> int:
    """Calculate double of x.
    
    Args:
        x: Input number
    
    Returns:
        Double of x
    """
    return x * 2
```

→ こんな小さな改善でも**プロの経歴に書ける**。

---

## 7. ⭐ 課題

1. **FastAPI** リポジトリを clone、`@app.get` を5分以内で追いかけ完了
2. **requests** ライブラリを読む(`requests.get` がどう動くか)
3. 任意のOSSで**typo / docstring** PR を1件出す
4. リポジトリの**アーキテクチャ図**を Mermaid で書く

---

## ✅ チェック

- [ ] OSS コードの読み方フローを理解
- [ ] VS Code のショートカットを使いこなせる
- [ ] FastAPI のコードを5分でナビゲートできる
- [ ] PR を1件以上出した
- [ ] アーキテクチャを言語化できる

---

## 進捗

```
Day 33: OSS コード読解。FastAPI/requests 読了。PR 1件マージ済み。
複雑案件・既存システム改修案件に対応可能。
```

## 次

`34_アーキテクチャ設計.md` へ。**設計力で月+50万**。
