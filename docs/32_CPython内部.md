# Day 32:CPython 内部・GIL・メモリ管理

## 💰 +20-50万(パフォーマンス案件)
## ⏱ 30時間

「**Python が遅い時に何が起きてるか**」を理解する。これが分かるとシニアレベル。

---

## ゴール

- CPython の動作原理
- GIL(Global Interpreter Lock)の本質
- メモリ管理(参照カウント+GC)
- バイトコード読解
- 高速化テクニック

---

## 1. Python はインタプリタ言語(15分)

```
ソースコード(.py)
   ↓ コンパイル
バイトコード(.pyc)
   ↓ 仮想マシン(CPython VM)が解釈
実行
```

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
# 1     RESUME      0
# 2     LOAD_FAST   a
#       LOAD_FAST   b
#       BINARY_OP   +
#       RETURN_VALUE
```

→ **バイトコード**を読めると最適化の根拠がわかる。

---

## 2. GIL の本質(40分)

### GIL = Global Interpreter Lock

「**1プロセス内では同時に1スレッドしかPythonコードを実行できない**」。

### なぜ存在?

- 参照カウント方式の GC が**スレッドセーフでないため**

### 影響

```python
# CPU バウンド → マルチスレッドで速くならない
import threading

def heavy():
    sum(i*i for i in range(10**7))

# 4スレッドで実行しても、1スレッド分の時間しか変わらない
threads = [threading.Thread(target=heavy) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
```

### 解決策

| 方法 | 用途 |
|---|---|
| `multiprocessing` | CPU処理を**別プロセス**で実行(GIL回避) |
| `asyncio` | I/O待機中は GIL を解放、並行可能 |
| **C拡張**(NumPy等) | C内部では GIL 解放、真の並列 |
| **PyPy** | JIT で高速化 |
| **Cython** | Python → C に変換 |
| **Python 3.13+ free-threaded build** | GIL なしビルド(実験的) |

---

## 3. 参照カウント GC(20分)

```python
import sys

a = [1, 2, 3]
print(sys.getrefcount(a))  # 2(変数a + getrefcount内)

b = a
print(sys.getrefcount(a))  # 3

del b
print(sys.getrefcount(a))  # 2
```

### 循環参照は?

```python
a = []
b = []
a.append(b)
b.append(a)
# 参照カウントだけでは解放できない
```

→ Python は **世代別 GC** が補助で動作。`gc.collect()` で手動実行可。

```python
import gc
gc.get_count()
gc.collect()
```

---

## 4. メモリ最適化(30分)

### `__slots__` でメモリ削減

```python
# 普通(メモリ多)
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

# slots(50%削減)
class User:
    __slots__ = ("name", "age")
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

→ 100万インスタンスで**RAM 200MB → 100MB**。

### generator(遅延評価)

```python
# NG: 全部メモリに展開
data = [process(x) for x in huge_list]

# OK: 1つずつ処理
data = (process(x) for x in huge_list)
for d in data:
    ...
```

### memory_profiler

```bash
pip install memory_profiler
```

```python
from memory_profiler import profile

@profile
def my_func():
    huge = [i for i in range(10**6)]

my_func()
# 各行のメモリ使用量がわかる
```

---

## 5. プロファイリング(30分)

### cProfile

```python
import cProfile
cProfile.run("my_func()", sort="cumulative")
```

### line_profiler

```bash
pip install line_profiler
```

```python
@profile
def slow_func():
    ...

# kernprof -l -v script.py
```

→ **どの行で時間使ってるか**が分かる。

### py-spy(本番でも使える)

```bash
pip install py-spy
py-spy top --pid 12345         # リアルタイム監視
py-spy record -o profile.svg --pid 12345
```

---

## 6. C拡張・Cython・Numba(40分)

### Cython(Python → C 変換)

```bash
pip install cython
```

`fast.pyx`:
```python
def fib(int n):
    cdef int a = 0, b = 1
    for _ in range(n):
        a, b = b, a + b
    return a
```

→ **純Pythonの100倍速**。

### Numba(JIT)

```bash
pip install numba
```

```python
from numba import jit

@jit(nopython=True)
def heavy_calc(arr):
    total = 0.0
    for x in arr:
        total += x * x
    return total
```

→ 既存Pythonコードに **`@jit` 付けるだけ** で C 並み速度。

### NumPy ベクトル化

```python
# Python ループ(遅い)
result = [x*2 for x in data]

# NumPy(C実装で爆速)
import numpy as np
result = np.array(data) * 2
```

→ 1000万要素で**100倍速**。

---

## 7. メタクラス・descriptor(参考、20分)

### descriptor(`@property`の中身)

```python
class CachedProperty:
    def __init__(self, func):
        self.func = func
    def __get__(self, obj, _):
        if obj is None: return self
        value = self.func(obj)
        obj.__dict__[self.func.__name__] = value
        return value

class Foo:
    @CachedProperty
    def expensive(self):
        return heavy_compute()
```

### メタクラス(クラスのクラス)

```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Config(metaclass=Singleton):
    pass

c1 = Config(); c2 = Config()
print(c1 is c2)  # True
```

→ ライブラリ作者になる時に必要。

---

## 8. 課題

1. **dis** で代表的な処理5つをバイトコード化、`+=` と `+` の違いを観察
2. `memory_profiler` で既存ポートフォリオの**ピークメモリ**測定
3. `cProfile` で 一番遅い関数特定 → Cython/Numba で**10倍速化**
4. **Python 3.13 free-threaded** ビルドで GIL 無効テスト

---

## ✅ チェック

- [ ] バイトコード読める
- [ ] GIL の影響を説明できる
- [ ] 参照カウント・GC 理解
- [ ] `__slots__`・generator でメモリ削減
- [ ] cProfile / py-spy 使える
- [ ] Cython / Numba で高速化
- [ ] NumPy ベクトル化
- [ ] descriptor / metaclass を読める

---

## 進捗

```
Day 32: CPython内部完了。GIL/GC/プロファイリング/C拡張。
パフォーマンス問題を根本解決できる。
```

## 次

`33_OSSコード読解.md` へ。**他人のコードを読む力**。
