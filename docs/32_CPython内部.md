# Day 32:CPython 内部・GIL・メモリ管理

## 💡 これは何?(小学生でもわかる説明)

**Python が遅い時、中で何が起きてるかを覗いてみる日**

### なぜ大事?
普通のエンジニアは「動けばOK」で終わる。でも本番運用では、
- 「サーバーが急に重くなる」
- 「メモリが膨らんで落ちる」
- 「CPU使い切ってるのに遅い」

という事態が必ず起きる。**ここの原因を当てられる人は、社内で1人しかいない超人材**になる。

### イメージ
- **CPython** = Pythonの**エンジン本体**(車でいうエンジンルーム)
- **GIL** = 食堂の「**1人ルール**」(同時に1人しか厨房に入れない)
- **メモリ管理** = **倉庫の在庫整理**(使い終わった荷物を捨てる係)
- **バイトコード** = エンジンの中で動く**設計図の中間状態**

> 車の調子が悪いとき、ボンネット開けて中を見れる人と見れない人がいる。
> 後者は「修理屋に丸投げ」しかできないが、前者は**自分で原因を当てて直せる**。
> プログラミングも同じ。**エンジンの中身を知っているだけで月収+20〜50万**。

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何が起きうるか**」「**どのツールで調べるか**」。
具体的な計測コードは Claude Code に書かせる。

---

## 💰 +20-50万(パフォーマンス案件)
## ⏱ 30時間

「**Python が遅い時に何が起きてるか**」を理解する。これが分かるとシニアレベル。

> **なぜシニア扱い?**
> 「動かない」を直せる人は5万人いる。
> でも「動くけど遅い」を直せる人は5000人もいない。
> 希少性=単価。経済の鉄則。

---

## ゴール

- CPython の動作原理(ソースコード→バイトコード→実行)
- GIL(Global Interpreter Lock)の本質と回避策
- メモリ管理(参照カウント+GC)
- バイトコード読解
- 高速化テクニック(Cython, Numba, NumPy)

---

## 1. Python はインタプリタ言語(15分)

### 1-1. Python が動く流れ

```
ソースコード(.py)
   ↓ コンパイル(自動でやってくれる)
バイトコード(.pyc)— 機械語に近い中間形式
   ↓ 仮想マシン(CPython VM)が1命令ずつ解釈
実行(=結果が出る)
```

> **イメージ:**
> 日本語のレシピ(=`.py`)を、英語の手順書(=`.pyc`)に翻訳して、
> ロボット(=CPython VM)が手順書通りに料理を作る感じ。

### 1-2. バイトコードを覗く

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
# 1     RESUME      0
# 2     LOAD_FAST   a       ← aを取り出す
#       LOAD_FAST   b       ← bを取り出す
#       BINARY_OP   +       ← 足し算
#       RETURN_VALUE        ← 結果を返す
```

> **なぜバイトコードを読む?**
> 「`a + b` と `a += b` の速度差」「リスト内包表記とforループの差」など、
> **「なぜこっちが速い?」の根拠**がわかる。

### 1-3. なぜ知ると単価UP?

| シーン | 知らない人 | 知ってる人 |
|---|---|---|
| 遅いコード相談 | 「なんとなく書き換える」 | 「`dis` 見たら関数呼び出しが多い、インライン化しよう」 |
| 同期ズレ | 「謎エラー」と諦める | 「GILが原因、multiprocessingに切り替え」 |
| メモリ膨張 | サーバー増強で対処 | `__slots__` でRAM半減 |

---

## 2. GIL の本質(40分)

### 2-1. GIL = Global Interpreter Lock(全体ロック)

> **「**1プロセス内では同時に1スレッドしかPythonコードを実行できない**」**

これが Python 最大の「クセ」。

### 2-2. なぜ存在?

- 内部の**参照カウント方式 GC**(後述)が**スレッドセーフでない**ため
- 複数スレッドが同時に同じオブジェクトのカウントを更新すると壊れる
- それを防ぐために、**全体に鍵を1個かけた**(=GIL)

> **食堂の例え:**
> 厨房に料理人が複数いるが、**同じレシピ本を同時に読むと混乱**するので、
> 「**今レシピ本を持ってる人だけ料理していい**」というルールにした。
> このレシピ本=GIL。

### 2-3. 影響

```python
# CPU重い処理 → マルチスレッドで速くならない
import threading

def heavy():
    # 1000万回の計算
    sum(i*i for i in range(10**7))

# 4スレッドで実行しても、1スレッド分の時間しか変わらない!
threads = [threading.Thread(target=heavy) for _ in range(4)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

> **これに気づかないと、「マルチスレッドにしたのに速くならない!」と1日溶かす**。

### 2-4. 解決策

| 方法 | 用途 | イメージ |
|---|---|---|
| `multiprocessing` | CPU処理を**別プロセス**で実行(GIL回避) | 厨房をもう1つ作る |
| `asyncio` | I/O待機中は GIL を解放、並行可能 | 料理待ちの間に皿洗い |
| **C拡張**(NumPy等) | C内部では GIL 解放、真の並列 | 料理代行業者を呼ぶ |
| **PyPy** | JIT で高速化 | ロボット料理人 |
| **Cython** | Python → C に変換 | レシピを最適化 |
| **Python 3.13+ free-threaded build** | GIL なしビルド(実験的) | レシピ本を複数冊用意 |

### 2-5. どう使い分ける?

| 仕事の種類 | 最適解 |
|---|---|
| CPU重い計算(画像処理、数値計算) | `multiprocessing` または NumPy |
| ネット通信たくさん(API、DB) | `asyncio` |
| 大規模数値処理 | NumPy + Numba |

---

## 3. 参照カウント GC(20分)

### 3-1. 参照カウントとは

> **そのオブジェクトを参照している変数の数を数える**仕組み。
> カウントが0になった瞬間、メモリから消える。

```python
import sys

a = [1, 2, 3]
print(sys.getrefcount(a))  # 2(変数a + getrefcount内の引数)

b = a  # 同じリストを別の名前でも参照
print(sys.getrefcount(a))  # 3

del b  # 参照を1つ削除
print(sys.getrefcount(a))  # 2
```

> **イメージ:** 図書館の「貸し出し中のしおり」。
> 借りる人がいなくなったら(=カウント0)、本は書庫に戻される(=メモリ解放)。

### 3-2. 循環参照の落とし穴

```python
a = []
b = []
a.append(b)
b.append(a)
# a と b はお互いを指しあってる
# 参照カウントだけでは「カウント1のまま」で解放できない!
```

> **問題:** AがBを指して、BがAを指す。
> 外から見ると「誰も使ってない」のに、お互い指し合ってるからカウントが0にならない。

### 3-3. 世代別 GC(=救助隊)

Python は **世代別 GC** を補助で動かしている。
これが循環参照を見つけて切ってくれる。

```python
import gc
gc.get_count()  # 各世代のオブジェクト数を確認
gc.collect()    # 強制GC実行
```

> **3世代の理由:**
> 「すぐ消えるオブジェクト(若い)」と「長生きするオブジェクト(老人)」を区別。
> 若いやつは頻繁にチェック、老人はたまにチェック → 効率的。

---

## 4. メモリ最適化(30分)

### 4-1. `__slots__` でメモリ削減

```python
# 普通のクラス(メモリ大)
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

# __slots__ 使用(50%削減)
class User:
    __slots__ = ("name", "age")  # 使う属性を固定
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

→ 100万インスタンスで**RAM 200MB → 100MB**。

> **なぜ削減される?**
> 普通のクラスは内部に `__dict__`(可変サイズ辞書)を持つ → 容量大。
> `__slots__` だと固定の小さな配列で済む → 容量小。

### 4-2. ジェネレータ(遅延評価)

```python
# NG: 全部メモリに展開(1000万件の計算結果を全部RAM)
data = [process(x) for x in huge_list]

# OK: 1つずつ処理(その場で計算→消費→次)
data = (process(x) for x in huge_list)  # () で囲む
for d in data:
    ...
```

> **イメージ:**
> 弁当を全部一度に作って積み上げる(=リスト)vs 注文ごとに1個ずつ作る(=ジェネレータ)。
> 在庫スペース(=RAM)が圧倒的に少なくて済む。

### 4-3. memory_profiler で計測

```bash
pip install memory_profiler
```

```python
from memory_profiler import profile

@profile
def my_func():
    huge = [i for i in range(10**6)]

my_func()
# 各行のメモリ使用量が出力される
```

---

## 5. プロファイリング(30分)

### 5-1. cProfile(関数ごとの時間計測)

```python
import cProfile
cProfile.run("my_func()", sort="cumulative")
```

→ どの関数で時間を使ってるかランキング表示。

### 5-2. line_profiler(行ごとの時間計測)

```bash
pip install line_profiler
```

```python
@profile
def slow_func():
    ...

# kernprof -l -v script.py で実行
```

→ **どの行で時間使ってるか**が分かる神ツール。

### 5-3. py-spy(本番でも使える)

```bash
pip install py-spy
py-spy top --pid 12345         # リアルタイム監視(top コマンド風)
py-spy record -o profile.svg --pid 12345  # フレームグラフ生成
```

> **なぜ本番でOK?**
> py-spy はターゲットプロセスを**止めずに**スタックを覗く設計。
> 普通のプロファイラは止めないと計測できないので、本番では使えない。

### 5-4. 使い分け

| ツール | 粒度 | 本番OK? | 用途 |
|---|---|---|---|
| cProfile | 関数 | × | 開発中の概略把握 |
| line_profiler | 行 | × | ボトルネック1行特定 |
| py-spy | 関数 | ○ | 本番障害調査 |
| memory_profiler | 行(メモリ) | × | メモリ膨張原因 |

---

## 6. C拡張・Cython・Numba(40分)

### 6-1. Cython(Python → C 変換)

```bash
pip install cython
```

`fast.pyx`:
```python
def fib(int n):           # 型を指定するのがミソ
    cdef int a = 0, b = 1 # cdef = C 言語の変数として定義
    for _ in range(n):
        a, b = b, a + b
    return a
```

→ **純Pythonの100倍速**。

> **なぜ速い?**
> 普通のPythonは「変数の型は実行時に決まる」→ 毎回型チェック。
> Cythonは「最初から型を指定」→ Cと同じ速さ。

### 6-2. Numba(JIT)— ラクで強力

```bash
pip install numba
```

```python
from numba import jit

@jit(nopython=True)         # これを付けるだけ
def heavy_calc(arr):
    total = 0.0
    for x in arr:
        total += x * x
    return total
```

→ 既存Pythonコードに **`@jit` 付けるだけ** で C 並み速度。

> **JIT (Just-In-Time)とは?**
> 実行する直前に機械語にコンパイルする技術。
> 最初の1回だけ遅いが、2回目以降は爆速。

### 6-3. NumPy ベクトル化

```python
# Python ループ(遅い)
result = [x*2 for x in data]

# NumPy(C実装で爆速)
import numpy as np
result = np.array(data) * 2
```

→ 1000万要素で**100倍速**。

> **なぜ速い?**
> NumPy 内部はCで書かれていて、ループの度にPythonに戻らない。
> しかも GIL を解放するので、本当に並列実行される。

---

## 7. メタクラス・descriptor(参考、20分)

### 7-1. descriptor(`@property` の中身)

```python
class CachedProperty:
    """1回だけ計算して結果を覚えておくプロパティ"""
    def __init__(self, func):
        self.func = func
    def __get__(self, obj, _):
        if obj is None:
            return self
        value = self.func(obj)
        obj.__dict__[self.func.__name__] = value  # キャッシュに保存
        return value

class Foo:
    @CachedProperty
    def expensive(self):
        return heavy_compute()  # 1回だけ実行
```

### 7-2. メタクラス(=クラスを作るクラス)

```python
class Singleton(type):
    """インスタンスを1つしか作らせないメタクラス"""
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Config(metaclass=Singleton):
    pass

c1 = Config()
c2 = Config()
print(c1 is c2)  # True ← 同じインスタンス
```

> **使う場面:**
> ライブラリ作者になる時、ORMを作る時、フレームワークを書く時。
> 普通のアプリ開発では出番が少ないので「読めればOK」レベル。

---

## 8. よくある間違い・エラー表

| 症状 | 原因 | 対処 |
|---|---|---|
| マルチスレッドにしたのに速くならない | GILで1スレッドしか動いてない | CPU処理は `multiprocessing`、I/Oは `asyncio` |
| メモリが時間とともに増える(リーク疑い) | 循環参照、グローバル変数の蓄積 | `gc.collect()`、`tracemalloc` で調査 |
| `__slots__` 使ったら属性追加でエラー | `__slots__` で固定した属性以外は不可 | 必要な属性を全部 `__slots__` に書く |
| Numba で `@jit(nopython=True)` がエラー | Numbaが対応しない関数を使ってる | Pure Pythonの基本構文だけにする |
| Cython 使ったがあまり速くならない | 型指定してない | `cdef int x` のように型を明示 |
| py-spy が動かない | 権限不足 | `sudo` で実行(macOS/Linux) |

---

## 9. 課題

1. **dis** で代表的な処理5つをバイトコード化、`+=` と `+` の違いを観察
2. `memory_profiler` で既存ポートフォリオの**ピークメモリ**測定
3. `cProfile` で 一番遅い関数特定 → Cython/Numba で**10倍速化**
4. **Python 3.13 free-threaded** ビルドで GIL 無効テスト

---

## 10. 今日学んだ用語まとめ

| 用語 | 一言で |
|---|---|
| CPython | C言語で書かれた、標準のPython実装 |
| バイトコード | Pythonが内部で実行する中間命令 |
| GIL | 同時に1スレッドしか動かない全体ロック |
| 参照カウント | オブジェクトを使ってる人数を数える方式 |
| 世代別 GC | 循環参照を切ってくれる救助隊 |
| `__slots__` | クラスのメモリを節約する宣言 |
| ジェネレータ | 1個ずつ計算する遅延評価 |
| プロファイリング | どこで時間/メモリを使ってるか計測 |
| Cython | PythonをCに変換する高速化ツール |
| Numba | デコレータ1つで高速化するJIT |
| ベクトル化 | ループをNumPyで一括処理 |
| descriptor | `__get__`/`__set__` で動くプロパティ |
| メタクラス | クラスを作るクラス |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **パフォーマンスチューニング案件**:単価 100〜300万円(プラットフォーム例:直契約/Findy Freelance/SES)
- **大規模 Python アプリの GIL / メモリ問題解決**:単価 100〜200万円
- **Cython / Numba 高速化コンサル**:時給 10,000〜30,000円

### クロードへの頼み方(営業文を書かせる例)
```
法人向け「Pythonパフォーマンスチューニング・専門家」の営業文を書いて。
Day 32 までで習得:バイトコード / GIL / 参照カウント / GC / __slots__ / cProfile / py-spy / Cython / Numba
売り:数値計算系・データ処理系の Python アプリで「メモリ削減 + 速度UP」を両立
価格:プロファイリング 50万〜 / 高速化実装 100万〜
ターゲット:Python で大規模科学計算・金融・分析を回している企業
600字、定量的な改善事例を含めて
```

### この章だけでは足りないもの(次に学ぶべき)
- OSSコード読解(Day 33)で大規模コードベースの理解
- アーキテクチャ設計(Day 34)で全体最適化へ
- DB内部・OS / Linux(Day 36 / 37)で更に低レイヤー対応

---

## ✅ チェック

- [ ] バイトコード読める
- [ ] GIL の影響を口で説明できる
- [ ] 参照カウント・GC 理解
- [ ] `__slots__`・generator でメモリ削減できる
- [ ] cProfile / py-spy 使える
- [ ] Cython / Numba で高速化できる
- [ ] NumPy ベクトル化できる
- [ ] descriptor / metaclass を読める

---

## 進捗

```
Day 32: CPython内部完了。GIL/GC/プロファイリング/C拡張。
パフォーマンス問題を根本解決できる。
```

## 次

`33_OSSコード読解.md` へ。**他人の偉大なコードを読む力**を身につける。
