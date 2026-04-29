# Day 43:PyTorch・Transformer・LoRA・Fine-tuning

## 💰 +50-200万(LLMエンジニア)
## ⏱ 30時間

「**自分専用LLM**を作る」最先端スキル。

---

## 小学生説明

GPT/Claude を**自分の業務に特化**させる「**追加学習**」。
新人に**社内ルール書を渡して教える**ようなもの。

---

## 1. PyTorch 基礎(40分)

```python
import torch
import torch.nn as nn

# テンソル
x = torch.tensor([1.0, 2.0, 3.0])
x.cuda()  # GPU へ
x.shape

# モデル
class Net(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(10, 64)
        self.fc2 = nn.Linear(64, 1)
    
    def forward(self, x):
        x = torch.relu(self.fc1(x))
        return self.fc2(x)

# 学習
model = Net()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.MSELoss()

for epoch in range(100):
    pred = model(X)
    loss = loss_fn(pred, y)
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

---

## 2. Transformer の中身(30分)

```
入力 → Embedding → [Self-Attention + FFN] × N層 → 出力
```

### Self-Attention(超簡略)

```python
def attention(Q, K, V):
    scores = Q @ K.T / sqrt(d_k)
    weights = softmax(scores)
    return weights @ V
```

→ 「**他のトークンとの関連度を学ぶ**」のが Attention。

---

## 3. Hugging Face Transformers(40分)

```bash
pip install transformers datasets accelerate
```

```python
from transformers import AutoTokenizer, AutoModelForCausalLM

model_name = "rinna/japanese-gpt-neox-3.6b"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype=torch.float16, device_map="auto")

inputs = tokenizer("こんにちは、", return_tensors="pt").to("cuda")
output = model.generate(**inputs, max_length=100)
print(tokenizer.decode(output[0]))
```

→ **オープンソースLLM** をローカルで動かせる。

---

## 4. LoRA - 軽量Fine-tuning(60分)

普通の Fine-tuning は数百GB の GPU 必要 → **LoRA は1枚の GPU で OK**。

```bash
pip install peft trl
```

```python
from peft import LoraConfig, get_peft_model

lora_config = LoraConfig(
    r=8,
    lora_alpha=16,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# Only 0.1% of parameters trained
```

### 学習(TRL の SFTTrainer)

```python
from trl import SFTTrainer
trainer = SFTTrainer(
    model=model,
    train_dataset=dataset,
    tokenizer=tokenizer,
    max_seq_length=512,
)
trainer.train()
trainer.save_model("./my-model")
```

→ **Google Colab Pro($10/月)で4-12時間**で完了。

---

## 5. RLHF / DPO(20分)

「**人間の好みでモデル微調整**」する高度技法。

- **RLHF**: 報酬モデル + PPO(複雑)
- **DPO**: 直接最適化(シンプル、最近の主流)

```python
from trl import DPOTrainer

# preference dataset: prompt, chosen, rejected
trainer = DPOTrainer(
    model,
    ref_model,
    args=training_args,
    train_dataset=dataset,
    tokenizer=tokenizer,
)
trainer.train()
```

→ **キャラ性のあるBot**(自社カスタマーサポート風)を作れる。

---

## 6. Quantization - メモリ削減(15分)

```python
from transformers import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16,
)

model = AutoModelForCausalLM.from_pretrained(
    model_name, quantization_config=bnb_config
)
```

→ **70Bモデルを1枚のRTX 4090 で動かせる**。

---

## 7. ⭐ 課題

```
1. Hugging Face で日本語LLM(rinna 3.6B等)をロード→質問応答
2. LoRA で「自社FAQ」データセット で Fine-tuning
3. 学習前後の比較(社内用語の理解度UP)
4. 完成モデルを Hugging Face Hub に push
5. FastAPI でAPI提供
```

---

## ✅ チェック
- [ ] PyTorch 基本
- [ ] Transformers でLLM動かせる
- [ ] LoRA で Fine-tuning
- [ ] Quantization でメモリ削減
- [ ] DPO の存在を知ってる

## 次
`44_統計_因果推論.md`
