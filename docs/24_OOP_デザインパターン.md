# Day 24:OOP・デザインパターン - 設計力で単価2倍

## 💡 これは何?(小学生でもわかる説明)

****部品を再利用できる**書き方(クラス設計)**

### なぜ大事?
「動く」と「保守できる」の差。シニア=ここで判断

### イメージ
**LEGOブロック**みたいに部品を組み合わせる設計

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 この章後の月収:**+10〜30万円**(設計レビュー突破)
## ⏱ 所要:3時間

「**書ける**」と「**設計できる**」の違いがここ。
ジュニアとシニアを分ける**最重要スキル**。

---

## ゴール

- クラス設計・継承・カプセル化を実用レベルで
- SOLID原則を理解
- 主要GoFパターン(Strategy/Factory/Observer/Singleton)
- 「**動く**」から「**保守できる**」コードへ

---

## 1. クラスの基本(20分)

```python
class User:
    """ユーザー(クラス = 設計図)"""
    
    # クラス変数(全インスタンス共有)
    total_count = 0
    
    def __init__(self, name: str, age: int):
        # インスタンス変数
        self.name = name
        self.age = age
        User.total_count += 1
    
    def greet(self) -> str:
        return f"こんにちは、{self.name}です"
    
    def __repr__(self) -> str:
        return f"User({self.name!r}, {self.age})"

# 使い方
u1 = User("太郎", 25)
u2 = User("花子", 30)
print(u1.greet())  # こんにちは、太郎です
print(User.total_count)  # 2
```

---

## 2. 継承・ポリモーフィズム(30分)

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    """抽象基底クラス"""
    @abstractmethod
    def cry(self) -> str: ...

class Dog(Animal):
    def cry(self) -> str:
        return "ワン!"

class Cat(Animal):
    def cry(self) -> str:
        return "ニャー!"

# ポリモーフィズム(同じインターフェイス、違う実装)
animals: list[Animal] = [Dog(), Cat()]
for a in animals:
    print(a.cry())
```

→ `if isinstance` 連発を避けて、**型で動作を切り替える**のが OOP。

---

## 3. dataclass(モダン Python の必修)(20分)

```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class Order:
    id: int
    user_id: int
    amount: float
    created_at: datetime = field(default_factory=datetime.now)
    items: list[str] = field(default_factory=list)
    
    @property
    def with_tax(self) -> float:
        return self.amount * 1.1

# 使い方
order = Order(id=1, user_id=100, amount=1000)
print(order.with_tax)  # 1100.0
print(order)  # 自動的に __repr__ が生成
```

→ `__init__`/`__repr__`/`__eq__` を**自動生成**。**業務コードはこれを使う**。

---

## 4. Pydantic(API/設定で必修)(20分)

```python
from pydantic import BaseModel, Field, EmailStr, field_validator

class UserCreate(BaseModel):
    name: str = Field(min_length=1, max_length=50)
    email: EmailStr
    age: int = Field(ge=0, le=150)
    tags: list[str] = []
    
    @field_validator("name")
    @classmethod
    def no_special_chars(cls, v: str) -> str:
        if "@" in v: raise ValueError("@ 禁止")
        return v.strip()

# 入力検証付き
u = UserCreate(name="太郎", email="t@x.z", age=25)
```

→ **API・設定ファイル・データインポート**で必須。

---

## 5. SOLID 原則(40分)

プロのコード設計の**5大原則**。

### S - Single Responsibility(単一責任)

「**1クラスは1つの責務**」

❌ NG:
```python
class User:
    def save_to_db(self): ...     # DB
    def send_email(self): ...     # メール
    def render_html(self): ...    # 表示
```

✅ OK:
```python
class User: pass
class UserRepository: pass
class EmailSender: pass
class UserRenderer: pass
```

### O - Open/Closed(開放閉鎖)

「**拡張に開き、修正に閉じる**」

❌ NG:
```python
def calc_price(item, type):
    if type == "regular": return item.price
    if type == "vip": return item.price * 0.8
    if type == "student": return item.price * 0.9
    # 新規追加で関数を毎回修正
```

✅ OK:
```python
class PricingStrategy(ABC):
    @abstractmethod
    def calculate(self, item) -> float: ...

class RegularPricing(PricingStrategy):
    def calculate(self, item): return item.price

class VipPricing(PricingStrategy):
    def calculate(self, item): return item.price * 0.8

# 新規追加 = 新クラス、既存修正なし
```

### L - Liskov Substitution(リスコフ置換)

「**親クラスの代わりに子クラスを使えなければならない**」

子は親の契約を**破ってはいけない**。

### I - Interface Segregation(インターフェイス分離)

「**使わないメソッドを強制するな**」

巨大インターフェイス → 小さく分割。

### D - Dependency Inversion(依存性逆転)

「**具体ではなく抽象に依存**」

❌ NG:
```python
class OrderService:
    def __init__(self):
        self.db = MySQL()  # 具体に依存
```

✅ OK:
```python
class OrderService:
    def __init__(self, db: Database):  # 抽象に依存
        self.db = db
```

→ テストで MockDB に差し替えできる。

---

## 6. デザインパターン主要4選(40分)

### Strategy(戦略パターン)

「**振る舞いを差し替え可能にする**」

```python
class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float): ...

class StripePayment(PaymentStrategy):
    def pay(self, amount): print(f"Stripe: {amount}")

class PayPalPayment(PaymentStrategy):
    def pay(self, amount): print(f"PayPal: {amount}")

class Checkout:
    def __init__(self, payment: PaymentStrategy):
        self.payment = payment
    
    def execute(self, amount):
        self.payment.pay(amount)

Checkout(StripePayment()).execute(1000)
Checkout(PayPalPayment()).execute(1000)
```

### Factory(生成パターン)

「**オブジェクト生成を関数化**」

```python
def create_storage(type: str) -> Storage:
    if type == "s3": return S3Storage()
    if type == "local": return LocalStorage()
    if type == "gcs": return GCSStorage()
    raise ValueError(f"Unknown: {type}")

storage = create_storage("s3")
```

### Observer(監視パターン)

「**変化を購読する**」

```python
from typing import Callable

class EventBus:
    def __init__(self):
        self._listeners: dict[str, list[Callable]] = {}
    
    def subscribe(self, event: str, callback: Callable):
        self._listeners.setdefault(event, []).append(callback)
    
    def emit(self, event: str, data):
        for cb in self._listeners.get(event, []):
            cb(data)

bus = EventBus()
bus.subscribe("order_created", lambda d: print(f"メール送信: {d}"))
bus.subscribe("order_created", lambda d: print(f"在庫確認: {d}"))
bus.emit("order_created", {"order_id": 1})
```

### Singleton(単一インスタンス)

```python
class Config:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

c1 = Config()
c2 = Config()
print(c1 is c2)  # True
```

→ DB接続・設定・ロガーで使う。**乱用注意**。

---

## 7. その他重要パターン(20分)

### Repository

データアクセスを抽象化。

```python
class UserRepository(ABC):
    @abstractmethod
    def get(self, id: int) -> User: ...
    @abstractmethod
    def save(self, user: User): ...

class SQLUserRepository(UserRepository):
    def __init__(self, db): self.db = db
    def get(self, id): ...
    def save(self, user): ...

class InMemoryUserRepository(UserRepository):
    def __init__(self): self._data = {}
    # テスト用
```

### Decorator

`@property`, `@cache`, `@dataclass` 等で使う。

```python
from functools import wraps
import time

def timing(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__}: {time.time()-start:.2f}s")
        return result
    return wrapper

@timing
def slow_func():
    time.sleep(1)
```

### Context Manager

```python
class Transaction:
    def __enter__(self):
        print("開始")
        return self
    def __exit__(self, *args):
        print("終了")

with Transaction() as t:
    print("処理中")
```

→ DB トランザクション・ファイル開閉で活躍。

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
