# Day 43:PyTorch・Transformer・LoRA・Fine-tuning

## 💡 これは何?(小学生でもわかる説明)

**自分専用LLM**を作る(Fine-tuning)

### なぜ大事?
2026年最先端。月単価150-300万円。

ChatGPT や Claude は「**全員共通の優等生**」。
でも会社が欲しいのは「**うちの社内ルール・専門用語を覚えた専属社員**」。
だから自社用に追加学習させる仕事は **超高単価**。

「弊社のFAQに特化したBot作って」 → 月150-300万円
「医療カルテ専門のLLM作って」 → 月200-500万円
「自社マニュアルで微調整したアシスタント」 → 月100万円〜

### イメージ
- **Fine-tuning** = 「ChatGPTに**社内マニュアルを丸暗記させる**」作業
- **LoRA** = 「**全部教え直すんじゃなくて、必要な部分だけ追加メモ**を貼る」省エネ学習
- **Quantization** = 「**圧縮**して、安いPCでも動かせるようにする」

### 比喩でひとこと
新人を雇って「**社内研修を3ヶ月**」やる感覚。
入社時は何でも聞ける優等生(=ベースモデル)、
研修後は**自社専用の知識を持ったプロ**(=Fine-tuned モデル)。
LoRA は「研修を**短く・安く・効率よく**やる方法」。

### Claude Code に何を頼めばいいか
ここから先は**かなり技術的**。クロードに頼むときは:
- 「Hugging Face で日本語LLMをロードして質問応答するコード」
- 「LoRA で FAQ データを Fine-tuning するスクリプト」
- 「学習中の loss を tensorboard で監視」
みたいに**目的を伝える**。コード自体は全部クロード任せでOK。

---

## 💰 +50-200万(LLMエンジニア)
## ⏱ 30時間

「**自分専用LLM**を作る」最先端スキル。
2026年時点で**できる人がまだ少ない**ので、希少性が極めて高い。

### なぜ単価が桁違い?

| 仕事 | 単価感 |
|---|---|
| LLM の API を叩くだけ | 月10-30万 |
| RAG(社内文書検索)を作る | 月30-80万 |
| **Fine-tuning できる** | **月100-300万** |
| Fine-tuning + 評価まで | 月200-500万 |

「**API叩く側**」と「**モデルを作る側**」では値段が**5-10倍**違う。

---

## ゴール

- PyTorchの**テンソル・モデル・学習ループ**が分かる
- Transformer の中身(Attention)を**ざっくり**説明できる
- Hugging Face で**オープンソースLLM**を動かす
- **LoRA** で軽量Fine-tuning ができる
- Quantization でメモリを節約できる
- DPO の存在を知ってる

---

## 1. PyTorch 基礎(40分)

### 1-1. PyTorch ってなに?

> **イメージ:** Googleの TensorFlow と並ぶ、Meta製の深層学習ライブラリ。
> 今は研究・商用ともに **PyTorch が主流**。Hugging Face も中身は PyTorch。

### 1-2. テンソル(=多次元配列)

```python
import torch

# ベクトル(1次元)
x = torch.tensor([1.0, 2.0, 3.0])

# GPU へ送る(ある場合)
x = x.cuda()           # NVIDIA
# x = x.to("mps")      # Mac Apple Silicon
# x = x.to("cpu")      # CPU

# 形を確認
x.shape  # torch.Size([3])
```

> **比喩:** テンソル = 「**数字の入れ物**」。
> 1次元=ベクトル、2次元=行列、3次元以上=テンソル。

### 1-3. ニューラルネットワーク(モデル定義)

```python
import torch.nn as nn

class Net(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(10, 64)   # 10入力 → 64ニューロン
        self.fc2 = nn.Linear(64, 1)    # 64 → 1出力
    
    def forward(self, x):              # 順伝播の計算
        x = torch.relu(self.fc1(x))    # 活性化関数 ReLU
        return self.fc2(x)
```

### 1-4. 学習ループ

```python
model = Net()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)  # 学習率
loss_fn = nn.MSELoss()                                      # 二乗誤差

for epoch in range(100):           # 100回学習
    pred = model(X)                # 予測
    loss = loss_fn(pred, y)        # 正解とのズレ
    
    optimizer.zero_grad()          # 前回の勾配をリセット
    loss.backward()                # 微分(勾配)を計算
    optimizer.step()               # パラメータ更新
```

> **学習の本質:** 予測する → 答え合わせ → ズレ分パラメータをちょっと動かす。
> これを何万回も繰り返す。

---

## 2. Transformer の中身(30分)

### 2-1. 構造

```
入力テキスト 
  → Tokenize(単語IDに変換)
  → Embedding(意味のベクトル化)
  → [Self-Attention + FFN] × N層 (= Transformerブロック)
  → 出力(次の単語の確率)
```

### 2-2. Self-Attention(超簡略)

```python
def attention(Q, K, V):
    # Q=Query(質問), K=Key(キー), V=Value(値)
    scores = Q @ K.T / sqrt(d_k)   # 質問とキーの相性を計算
    weights = softmax(scores)       # 確率に正規化
    return weights @ V              # 値を重み付き合計
```

> **比喩:** Self-Attention = **「他のトークンとの関連度を学ぶ」**仕組み。
> 「銀行に**お金**を預ける」と「**お金**持ち」は同じ"お金"でも文脈が違う。
> Self-Attention は**周りの単語を見て意味を決める**。

### 2-3. なぜ Transformer は強い?

| 特徴 | 効果 |
|---|---|
| 並列計算できる | GPU で爆速学習 |
| 長距離の文脈を捉える | 文章全体を理解できる |
| スケールが効く | 大きくするほど賢くなる(GPT-4) |

---

## 3. Hugging Face Transformers(40分)

### 3-1. Hugging Face ってなに?

> **イメージ:** **AIモデルのGitHub**。
> 世界中のオープンソースLLMが共有されている。
> 1行で誰でも最新モデルが使える。

### 3-2. インストール

```bash
pip install transformers datasets accelerate
```

### 3-3. 日本語LLMを動かす

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

# モデル名(Hugging Face Hub にある)
model_name = "rinna/japanese-gpt-neox-3.6b"

# トークナイザ(文字 → ID 変換)
tokenizer = AutoTokenizer.from_pretrained(model_name)

# モデル(GPUに自動配置、float16で軽量化)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.float16,
    device_map="auto"
)

# 推論
inputs = tokenizer("こんにちは、", return_tensors="pt").to("cuda")
output = model.generate(**inputs, max_length=100)
print(tokenizer.decode(output[0]))
```

→ **オープンソースLLM** をローカルで動かせる(API代ゼロ)。

---

## 4. LoRA - 軽量Fine-tuning(60分)

### 4-1. なぜ LoRA が必要?

| 方法 | GPU メモリ | 時間 | コスト |
|---|---|---|---|
| **フルFine-tuning** | 数百GB | 数日〜週 | 数百万円 |
| **LoRA** | 24GB(1枚) | 4〜12時間 | $20〜100 |

> **比喩:** 全部教え直す = 「教科書全部書き直し」(膨大)
> LoRA = 「**マーカーで重要箇所に追記**」(変更箇所だけ)

### 4-2. インストール

```bash
pip install peft trl
```

### 4-3. LoRA設定

```python
from peft import LoraConfig, get_peft_model

lora_config = LoraConfig(
    r=8,                                  # ランク(小さいほど軽量)
    lora_alpha=16,                        # スケーリング係数
    target_modules=["q_proj", "v_proj"],  # どの層を学習対象に
    lora_dropout=0.05,                    # 過学習防止
    bias="none",
    task_type="CAUSAL_LM"                 # 文章生成タスク
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# → "全体の0.1%だけ学習" のような表示
```

### 4-4. 学習(TRL の SFTTrainer)

```python
from trl import SFTTrainer

trainer = SFTTrainer(
    model=model,
    train_dataset=dataset,    # 学習データ(JSON 形式が多い)
    tokenizer=tokenizer,
    max_seq_length=512,
)
trainer.train()
trainer.save_model("./my-model")
```

→ **Google Colab Pro($10/月)で4-12時間**で完了。

### 4-5. データ形式の例

```json
[
  {"prompt": "弊社の有給休暇は何日?", "response": "20日です。"},
  {"prompt": "経費精算の締め日は?", "response": "毎月25日です。"}
]
```

→ こういうFAQを **数百〜数千件** 用意するのが学習のキモ。

---

## 5. RLHF / DPO(20分)

### 5-1. なぜ RLHF が必要?

普通のFine-tuningは「**正解データを暗記**」。
でも実際は「Aの方がBより人間が好む」みたいな**好みデータ**で学ばせたい時もある。
それをやるのが **RLHF / DPO**。

| 手法 | 特徴 |
|---|---|
| **RLHF** | 報酬モデル + PPO(複雑、ChatGPT が使った) |
| **DPO** | 直接最適化(シンプル、最近の主流) |

### 5-2. DPO のコード

```python
from trl import DPOTrainer

# preference dataset: prompt, chosen(良い応答), rejected(悪い応答)
trainer = DPOTrainer(
    model,
    ref_model,                  # 元のモデル(基準として使う)
    args=training_args,
    train_dataset=dataset,
    tokenizer=tokenizer,
)
trainer.train()
```

→ **キャラ性のあるBot**(自社カスタマーサポート風)を作れる。

---

## 6. Quantization - メモリ削減(15分)

### 6-1. なぜ量子化が必要?

70Bモデル = **140GB のメモリ必要**(普通のPCには載らない)
4bit量子化すると **40GB弱** に圧縮 → **RTX 4090 (24GB) でも動く**。

> **比喩:** 高画質写真 → JPEG圧縮。
> 多少画質落ちるが、**ファイルサイズが1/4**。

### 6-2. コード

```python
from transformers import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,                       # 4bitに圧縮
    bnb_4bit_quant_type="nf4",               # NormalFloat4(高精度)
    bnb_4bit_compute_dtype=torch.float16,    # 計算は16bitで
)

model = AutoModelForCausalLM.from_pretrained(
    model_name, quantization_config=bnb_config
)
```

→ **70Bモデルを1枚のRTX 4090 で動かせる**。

---

## 7. よくあるエラー・落とし穴

| 症状 | 原因 | 直し方 |
|---|---|---|
| `CUDA out of memory` | GPUメモリ不足 | バッチサイズを下げる/4bit量子化 |
| 学習が進まない(loss平ら) | 学習率が大きすぎ | `lr=1e-4` などに下げる |
| 出力が壊れる(意味不明) | 過学習 | データ増やす/エポック減らす |
| トークナイザが動かない | モデルと種類が違う | `from_pretrained` を同じモデル名で |
| 学習が遅い | CPUで動いてる | `.to("cuda")` 確認 |

---

## 8. ⭐ 課題(順番にやる)

```
1. Hugging Face で日本語LLM(rinna 3.6B等)をロード→質問応答
2. 自分のFAQ(50件以上)を JSON で用意
3. LoRA で Fine-tuning(Google Colab Pro でOK)
4. 学習前後の比較(社内用語の理解度UP)
5. 完成モデルを Hugging Face Hub に push
6. FastAPI でAPI提供 → curl で叩けるようにする
```

→ ここまでできれば**「Fine-tuningできるエンジニア」**として通用する。

---

## 9. 用語まとめ表

| 用語 | 一言で |
|---|---|
| PyTorch | Meta製の深層学習ライブラリ(主流) |
| テンソル | 多次元配列(数字の入れ物) |
| Transformer | 現代LLMの基本構造 |
| Self-Attention | 単語間の関連度を学ぶ仕組み |
| Hugging Face | オープンソースAIのGitHub |
| Tokenizer | 文字をID列に変換 |
| Fine-tuning | 既存モデルに追加学習 |
| LoRA | 軽量Fine-tuning(必要部分だけ追記) |
| Quantization | モデルを圧縮してメモリ削減 |
| RLHF | 人間の好みで微調整 |
| DPO | RLHFのシンプル版(主流化中) |
| Epoch | データを1周する単位 |
| Loss | 予測と正解のズレ(これを最小化する) |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **LLM Fine-tuning 案件(LoRA / DPO)**:単価 50〜300万円(プラットフォーム例:Findy Freelance/直契約/SES)
- **業界特化AIモデル開発(法務 / 医療 / 不動産)**:単価 100〜300万円
- **Hugging Face 公開モデルのカスタム化**:単価 30〜150万円

### クロードへの頼み方(営業文を書かせる例)
```
法人向け、業界特化LLM Fine-tuning コンサルの提案文を書いて。
Day 43 までで習得:PyTorch / Transformers / LoRA / Quantization / DPO
売り:「APIで OpenAI / Claude を叩く」より一段深い "自社専用モデル" を実現
価格:PoC 100万〜 / 本番モデル構築 300万〜
ターゲット:既に LLM API を使っているが、業界用語に弱くて困っている企業
過去実績:小規模データセットで LoRA 実験した GitHub サンプル
注意:学習用GPUの確保(クラウド or 持ち込み)についても触れる
800字
```

### この章だけでは足りないもの(次に学ぶべき)
- 統計・因果推論(Day 44)で評価指標を厳密に
- ベクトルDB(Day 45) / RAG高度化
- マルチモーダル(Day 46) / Browser Agent(Day 47)

---

## ✅ チェック
- [ ] PyTorch のテンソル・モデル・学習ループが書ける
- [ ] Transformer の概要(Attention)が説明できる
- [ ] Transformers ライブラリでLLMを動かせる
- [ ] LoRA で Fine-tuning した
- [ ] Quantization でメモリ削減を試した
- [ ] DPO の概念を説明できる
- [ ] 自分のデータで微調整したモデルを動かせた

## 次
`44_統計_因果推論.md` で**まぐれを見抜く数学**。
