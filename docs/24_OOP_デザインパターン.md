# Day 24:OOP・デザインパターン - 設計力で単価2倍

## 💡 これは何?(小学生でもわかる説明)

**部品を再利用できる書き方(クラス設計)**

### なぜ大事?
コードが「動く」と「**保守できる**」は別物。
動くだけのコードは半年後に**自分でも読めなくなる**。
クラス設計ができる=「**3年後の自分や他人が直せるコード**」が書ける。
ここがジュニア(月60万)とシニア(月150万)を分ける**最重要スキル**。

### イメージ
- **クラス**=「**設計図(ブループリント)**」(同じ家を何軒でも建てられる)
- **インスタンス**=「**設計図から建てた家1軒**」
- **継承**=「**親の設計図を引き継いで、追加で部屋を増やす**」
- **ポリモーフィズム**=「**犬・猫それぞれ違う鳴き声、でも同じ「鳴け」コマンドで動く**」
- **デザインパターン**=「**料理の定番レシピ集**」(先人の知恵)
- **SOLID原則**=「**家を建てる時のお作法5箇条**」(これ守ると崩れにくい家になる)

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 この章後の月収:**+10〜30万円**(設計レビュー突破)
## ⏱ 所要:3時間

「**書ける**」と「**設計できる**」の違いがここ。
ジュニアとシニアを分ける**最重要スキル**。

> **なぜ単価が上がる?**
> 大規模システムは「動くコード」より「**変更しやすいコード**」が価値。
> 設計力 = 数年後の保守費用が下がる = 客が高い金を払う理由。

---

## ゴール

- クラス設計・継承・カプセル化を実用レベルで使える
- SOLID原則を説明できる(他人に)
- 主要GoFパターン(Strategy/Factory/Observer/Singleton)を書ける
- 「**動く**」から「**保守できる**」コードへ進化

---

## 1. クラスの基本(20分)

### 1-1. クラスって何?

> **クラス**=「**設計図**」。
> **インスタンス**=「**設計図から作った実物**」。
> 例:「家の設計図(クラス)」から「太郎さんの家・花子さんの家(インスタンス)」を建てる。

```python
class User:
    """ユーザー(クラス = 設計図)"""

    # クラス変数(全インスタンス共有 = 全ての家で共通の数値)
    total_count = 0

    def __init__(self, name: str, age: int):
        # インスタンス変数(=その家ごとの個別データ)
        self.name = name
        self.age = age
        User.total_count += 1  # 家を建てるたびに+1

    def greet(self) -> str:
        # メソッド(=その家の人がやる動作)
        return f"こんにちは、{self.name}です"

    def __repr__(self) -> str:
        # オブジェクトを文字列にした時の表示(デバッグ用)
        return f"User({self.name!r}, {self.age})"

# 使い方
u1 = User("太郎", 25)   # 1軒目
u2 = User("花子", 30)   # 2軒目
print(u1.greet())       # こんにちは、太郎です
print(User.total_count) # 2
```

### 1-2. self って何?

> **self** = 「**この家自身**」。
> メソッドの第1引数は必ず`self`。Pythonのお約束。

| 用語 | 意味 | たとえ |
|---|---|---|
| `class` | 設計図を定義するキーワード | 設計事務所の図面 |
| `__init__` | コンストラクタ(家を建てる時の初期化) | 引っ越し作業 |
| `self` | このインスタンス自身 | 「うちの家」 |
| インスタンス変数 | 各インスタンス固有のデータ | 各家の住人 |
| クラス変数 | 全インスタンス共通のデータ | 街全体のルール |

---

## 2. 継承・ポリモーフィズム(30分)

### 2-1. 継承(=親のクラスを引き継ぐ)

> **イメージ:** 「動物」という親設計図がある。
> 犬・猫はそれを引き継いで「鳴く」という動作を**自分流に上書き**する。

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    """抽象基底クラス(=必ず子に実装させる雛形)"""
    @abstractmethod
    def cry(self) -> str:
        # ... = 中身は子クラスで実装してね、の意味
        ...

class Dog(Animal):
    def cry(self) -> str:
        return "ワン!"

class Cat(Animal):
    def cry(self) -> str:
        return "ニャー!"

# ポリモーフィズム(=同じインターフェイス、違う実装)
animals: list[Animal] = [Dog(), Cat()]
for a in animals:
    print(a.cry())  # ワン! → ニャー!
```

→ `if isinstance` 連発を避けて、**型で動作を切り替える**のが OOP。

### 2-2. なぜポリモーフィズム?

> **悪い書き方:**
> ```python
> if isinstance(animal, Dog): print("ワン!")
> elif isinstance(animal, Cat): print("ニャー!")
> # 動物が増えるたびにif文を増やす(=保守性低)
> ```
> **良い書き方:**
> ```python
> animal.cry()  # 型を意識せずに動く(=保守性高)
> ```

| 用語 | 意味 |
|---|---|
| 継承 | 親の機能を子が引き継ぐ |
| 抽象クラス(ABC) | 直接インスタンス化できない雛形 |
| `@abstractmethod` | 子クラスで必ず実装する印 |
| ポリモーフィズム | 同じ呼び方で違う動作 |
| オーバーライド | 親メソッドを子で上書き |

---

## 3. dataclass(モダン Python の必修)(20分)

### 3-1. なぜ dataclass?

> **問題:** クラスを書くたびに `__init__`, `__repr__`, `__eq__` を毎回書くのが面倒。
> **解決:** `@dataclass` を付けると**自動生成**してくれる。

```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass  # ← この1行で __init__/__repr__/__eq__ 自動生成
class Order:
    id: int
    user_id: int
    amount: float
    # field(default_factory=...) は「毎回新しいリスト・現在時刻を作る」ためのおまじない
    created_at: datetime = field(default_factory=datetime.now)
    items: list[str] = field(default_factory=list)

    @property  # 関数を変数のように使えるおまじない
    def with_tax(self) -> float:
        return self.amount * 1.1

# 使い方
order = Order(id=1, user_id=100, amount=1000)
print(order.with_tax)  # 1100.0
print(order)  # 自動的に __repr__ が生成された見やすい表示
```

→ `__init__`/`__repr__`/`__eq__` を**自動生成**。**業務コードはこれを使う**。

> **なぜ default_factory?**
> 普通に `items: list = []` と書くと、**全インスタンスで同じリストを共有**してしまう(バグの温床)。
> `field(default_factory=list)` だと**毎回新しいリスト**を作る。

---

## 4. Pydantic(API/設定で必修)(20分)

### 4-1. なぜ Pydantic?

> **問題:** APIで受け取ったデータが「正しいか」検証するのが面倒。
> **解決:** Pydantic を使うと**型と制約を書くだけで自動検証**。

```python
from pydantic import BaseModel, Field, EmailStr, field_validator

class UserCreate(BaseModel):
    # min_length, max_length で文字数制限
    name: str = Field(min_length=1, max_length=50)
    email: EmailStr  # メール形式かチェック
    age: int = Field(ge=0, le=150)  # 0以上150以下
    tags: list[str] = []

    @field_validator("name")
    @classmethod
    def no_special_chars(cls, v: str) -> str:
        # name フィールドへのカスタム検証
        if "@" in v:
            raise ValueError("@ 禁止")
        return v.strip()  # 前後空白を削る

# 入力検証付き(不正値はエラー)
u = UserCreate(name="太郎", email="t@x.z", age=25)
```

→ **API・設定ファイル・データインポート**で必須。

> **dataclass との違い:**
> - dataclass = 内部用(検証なし、軽い)
> - Pydantic = 外部入力用(検証あり、API 用)

---

## 5. SOLID 原則(40分)

> プロのコード設計の**5大原則**。
> 「**設計上手**」はこれを無意識に守ってる。

### 5-1. S - Single Responsibility(単一責任の原則)

「**1クラスは1つの責務**」

> **イメージ:** 1人で「料理」「掃除」「営業」「会計」を全部やるカフェ → 倒れる。
> 役割分担すれば、誰かが休んでも回る。

❌ NG:

```python
class User:
    def save_to_db(self): ...     # DB操作
    def send_email(self): ...     # メール送信
    def render_html(self): ...    # 表示
# → User が大きくなりすぎ。修正のたびに全部影響
```

✅ OK:

```python
class User: pass                  # データだけ
class UserRepository: pass        # DB専門
class EmailSender: pass           # メール専門
class UserRenderer: pass          # 表示専門
# → 修正範囲が局所化
```

### 5-2. O - Open/Closed(開放閉鎖の原則)

「**拡張に開き、修正に閉じる**」

> **イメージ:** USB差込口は「**新しいデバイス追加できる**」けど「**差込口の形は変えない**」。

❌ NG:

```python
def calc_price(item, type):
    # 新しい会員種別が出るたびに if を追加
    if type == "regular": return item.price
    if type == "vip": return item.price * 0.8
    if type == "student": return item.price * 0.9
```

✅ OK:

```python
class PricingStrategy(ABC):
    @abstractmethod
    def calculate(self, item) -> float: ...

class RegularPricing(PricingStrategy):
    def calculate(self, item):
        return item.price

class VipPricing(PricingStrategy):
    def calculate(self, item):
        return item.price * 0.8

# 新規追加 = 新クラス、既存修正なし
```

### 5-3. L - Liskov Substitution(リスコフ置換の原則)

「**親クラスの代わりに子クラスを使えなければならない**」

子は親の契約を**破ってはいけない**。

> **例:** 「鳥は飛ぶ」と決めたなら、ペンギンは「鳥」の子クラスにしない方がいい(飛べないので)。

### 5-4. I - Interface Segregation(インターフェイス分離の原則)

「**使わないメソッドを強制するな**」

巨大インターフェイス → 小さく分割。

> **例:** 「全機能スマホ」ではなく「電話だけ」「メールだけ」など機能別に分ける。

### 5-5. D - Dependency Inversion(依存性逆転の原則)

「**具体ではなく抽象に依存**」

❌ NG:

```python
class OrderService:
    def __init__(self):
        self.db = MySQL()  # 具体クラスに直接依存
```

✅ OK:

```python
class OrderService:
    def __init__(self, db: Database):  # 抽象クラスに依存
        self.db = db
```

→ テストで MockDB に差し替えできる。

> **DI(依存性注入)とは?**
> 必要なものを「外から渡す」設計。
> テスト時は「偽物のDB」を渡せる。本番は「本物のDB」を渡す。

### SOLID まとめ

| 文字 | 名前 | 一言で |
|---|---|---|
| S | 単一責任 | 1クラス1仕事 |
| O | 開放閉鎖 | 拡張OK、既存はいじらない |
| L | リスコフ | 親と置換可能な子だけ作る |
| I | インターフェイス分離 | 不要機能を強制しない |
| D | 依存性逆転 | 抽象に依存 |

---

## 6. デザインパターン主要4選(40分)

### 6-1. Strategy(戦略パターン)

「**振る舞いを差し替え可能にする**」

> **イメージ:** 「決済」という同じ動作を、Stripe・PayPal・銀行振込で**差し替えられる**。

```python
class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float): ...

class StripePayment(PaymentStrategy):
    def pay(self, amount):
        print(f"Stripe: {amount}")

class PayPalPayment(PaymentStrategy):
    def pay(self, amount):
        print(f"PayPal: {amount}")

class Checkout:
    def __init__(self, payment: PaymentStrategy):
        self.payment = payment  # どの決済か外から注入

    def execute(self, amount):
        self.payment.pay(amount)

# 同じ Checkout で違う決済方法
Checkout(StripePayment()).execute(1000)
Checkout(PayPalPayment()).execute(1000)
```

### 6-2. Factory(生成パターン)

「**オブジェクト生成を関数化**」

> **イメージ:** 「ストレージください」と注文すると、設定によって S3・ローカル・GCS が出てくる工場。

```python
def create_storage(type: str) -> Storage:
    if type == "s3": return S3Storage()
    if type == "local": return LocalStorage()
    if type == "gcs": return GCSStorage()
    raise ValueError(f"Unknown: {type}")

storage = create_storage("s3")  # 設定で切り替え可
```

### 6-3. Observer(監視パターン)

「**変化を購読する**」

> **イメージ:** YouTube チャンネルのチャンネル登録。
> 動画投稿(=イベント発生)すると、登録者全員に通知が飛ぶ。

```python
from typing import Callable

class EventBus:
    def __init__(self):
        self._listeners: dict[str, list[Callable]] = {}

    def subscribe(self, event: str, callback: Callable):
        # イベントに対するリスナー登録
        self._listeners.setdefault(event, []).append(callback)

    def emit(self, event: str, data):
        # イベント発生 → 登録者全員に通知
        for cb in self._listeners.get(event, []):
            cb(data)

bus = EventBus()
bus.subscribe("order_created", lambda d: print(f"メール送信: {d}"))
bus.subscribe("order_created", lambda d: print(f"在庫確認: {d}"))
bus.emit("order_created", {"order_id": 1})
# → メールも在庫確認も自動実行
```

### 6-4. Singleton(単一インスタンス)

「**世界に1つだけ**を保証」

> **イメージ:** 国の大統領は1人だけ。何回呼んでも同じ人が来る。

```python
class Config:
    _instance = None

    def __new__(cls):
        # まだ作られてなければ作る、あれば既存を返す
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

c1 = Config()
c2 = Config()
print(c1 is c2)  # True(同じインスタンス)
```

→ DB接続・設定・ロガーで使う。**乱用注意**(=テストしづらくなる)。

---

## 7. その他重要パターン(20分)

### 7-1. Repository

「**データアクセスを抽象化**」

> **イメージ:** 倉庫管理係。中身が SQL でも メモリ でも、外からは「取って」「しまって」だけ言えばいい。

```python
class UserRepository(ABC):
    @abstractmethod
    def get(self, id: int) -> User: ...
    @abstractmethod
    def save(self, user: User): ...

class SQLUserRepository(UserRepository):
    def __init__(self, db):
        self.db = db
    def get(self, id): ...
    def save(self, user): ...

class InMemoryUserRepository(UserRepository):
    def __init__(self):
        self._data = {}
    # テスト用(=DBなしで動く)
```

### 7-2. Decorator

「**関数に機能を追加**」

> **イメージ:** プレゼントのラッピング。中身(関数)はそのまま、見た目に飾りを付ける。

`@property`, `@cache`, `@dataclass` 等で使う。

```python
from functools import wraps
import time

def timing(func):
    """関数の実行時間を測るデコレータ"""
    @wraps(func)  # 元関数の情報を保持
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__}: {time.time()-start:.2f}s")
        return result
    return wrapper

@timing  # ← これで slow_func に時間計測機能が追加される
def slow_func():
    time.sleep(1)
```

### 7-3. Context Manager

「**前処理・後処理を自動で**」

> **イメージ:** 「入る → 出る」のセット動作。`with` 文を抜けたら**必ず後処理**が走る。

```python
class Transaction:
    def __enter__(self):
        # with に入った時の処理
        print("開始")
        return self

    def __exit__(self, *args):
        # with から抜ける時の処理(=必ず実行される)
        print("終了")

with Transaction() as t:
    print("処理中")
# 出力: 開始 → 処理中 → 終了
```

→ DB トランザクション・ファイル開閉で活躍。

> **なぜ便利?**
> エラーが出ても「終了処理」が必ず実行される。
> 例:DB トランザクションで失敗したら自動 rollback。

---

## 8. 課題

### 課題A:Repository + Strategy で注文システム

クロードに頼む:

```
以下を実装するPythonコードを書いて。

設計:
- OrderRepository(ABC) → SQLOrderRepository / InMemoryOrderRepository
- PricingStrategy(ABC) → RegularPricing / VipPricing / SeasonPricing
- OrderService が repository と pricing を受け取る(DI)
- pytest で InMemoryOrderRepository でテスト

SOLID 全部満たす。dataclass 使う。
```

→ **これを書ければ「**設計できる人**」**。

### 課題B:既存コードのリファクタ

Day 4-8 で作ったどれかの作品を、SOLID と Strategy/Factory で書き直す。
**コード量増えるが保守性UP**。

---

## 9. よくある間違い・エラー(10分)

| 状況 | 間違いの原因 | 直し方 |
|---|---|---|
| クラス変数とインスタンス変数を混同 | self の付け忘れ | 個別データは必ず `self.xxx` |
| dataclass のデフォルト値で `[]` 直書き | 全インスタンスで共有される | `field(default_factory=list)` |
| 抽象クラスをインスタンス化 | ABC を理解してない | サブクラスをインスタンス化 |
| 何でも継承で解決 | 継承中毒 | コンポジション(持つ関係)も検討 |
| Singleton 乱用 | グローバル変数化 | 本当に1つでいい時だけ |
| 早すぎる抽象化 | 「将来役立つかも」で抽象化 | YAGNI に従い、必要時に抽象化 |
| Pydantic と dataclass を混ぜる | 区別してない | 外部入力=Pydantic、内部=dataclass |

---

## 10. 用語まとめ

| 用語 | 一言で |
|---|---|
| クラス | 設計図 |
| インスタンス | 設計図から作った実物 |
| `self` | このインスタンス自身 |
| `__init__` | 初期化メソッド |
| 継承 | 親の機能を引き継ぐ |
| ABC(抽象基底クラス) | 直接使えない雛形 |
| `@abstractmethod` | 子で必ず実装する印 |
| ポリモーフィズム | 同じ呼び方で違う動作 |
| dataclass | 自動生成つきクラス |
| Pydantic | 検証つきデータクラス |
| SOLID | 設計5原則 |
| DI(依存性注入) | 必要物を外から渡す |
| Strategy | 振る舞い差し替え |
| Factory | 生成を関数化 |
| Observer | 変化を購読 |
| Singleton | 世界に1つだけ |
| Repository | データアクセス抽象化 |
| Decorator | 関数に機能追加 |
| Context Manager | with 文での前後処理 |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **大規模システム設計・実装**:単価 50〜150万円(プラットフォーム例:ランサーズ「ビジネス」/レバテック/直契約)
- **既存コードのリファクタリング・OOP化**:単価 30〜80万円(レガシーコード改善)
- **設計レビュー・コンサル**:時給 8,000〜20,000円(週数時間契約も可)

### クロードへの頼み方(営業文を書かせる例)
```
法人クライアント向けの「設計改善・リファクタリングコンサル」サービス紹介文を書いて。
Day 24 までで習得:OOP / SOLID / Strategy / Factory / Observer / Repository / Decorator / Context Manager
売り:既存コードの「動くけど読めない」状態を、テスト可能で拡張しやすい設計に変える
価格:設計レビューのみ 20万円〜 / リファクタ実装込み 50万円〜
ターゲット:Python 製の社内システムが古くなってきた中小企業
トーン:技術屋として冷静に。"全部書き直す" ではなく "段階的に改善" を強調
600字
```

### この章だけでは足りないもの(次に学ぶべき)
- 非同期 / 並行処理(Day 25)で大規模システム対応
- データエンジニアリング(Day 26) / ML(Day 27)
- アーキテクチャ設計(Day 34)で更に上のレイヤーへ

---

## ✅ チェック

- [ ] クラス・継承・抽象クラス書ける
- [ ] dataclass / Pydantic 使える
- [ ] SOLID 5原則を説明できる
- [ ] Strategy / Factory / Observer / Singleton パターン書ける
- [ ] Repository パターン理解
- [ ] Decorator / Context Manager 書ける
- [ ] 課題2件動かした

---

## 進捗

```
Day 24: OOP・デザインパターン完了。SOLID/GoF。
コードレビューで設計議論できる状態。
```

## 次

`25_非同期_並行処理.md` へ。**速度を10倍にするスキル**。
