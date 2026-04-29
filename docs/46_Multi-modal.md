# Day 46:Multi-modal AI - Vision・Voice・Image生成

## 💡 これは何?(小学生でもわかる説明)

****文字+画像+音声**全部扱える AI**

### なぜ大事?
2026年は Multi-modal が標準。新領域単価300万円

### イメージ
AIに**目・耳・口**を持たせる

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 +30-100万(2026最新)
## ⏱ 15時間

文字だけじゃなく**画像・音声・動画**を扱える AI。

---

## 1. Claude Vision / GPT-4o Vision(20分)

```python
import base64
from anthropic import Anthropic

client = Anthropic()
with open("photo.jpg", "rb") as f:
    img_b64 = base64.b64encode(f.read()).decode()

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64", "media_type": "image/jpeg", "data": img_b64}},
            {"type": "text", "text": "この画像を説明して"}
        ]
    }]
)
```

→ **写真→説明文**、**請求書→構造化データ**、**図表→分析** が一発。

---

## 2. Image生成(DALL-E / Stable Diffusion)(20分)

```python
# OpenAI
from openai import OpenAI
client = OpenAI()
img = client.images.generate(
    model="dall-e-3",
    prompt="A cyberpunk city at sunset, photorealistic",
    size="1024x1024",
)
print(img.data[0].url)
```

### Stable Diffusion(ローカル)

```bash
pip install diffusers
```

```python
from diffusers import StableDiffusionPipeline
import torch

pipe = StableDiffusionPipeline.from_pretrained(
    "runwayml/stable-diffusion-v1-5",
    torch_dtype=torch.float16
).to("cuda")

img = pipe("A cat riding a horse").images[0]
img.save("output.png")
```

→ **YouTubeサムネイル自動生成**(月10-30万円案件)。

---

## 3. Voice AI - リアルタイム会話(30分)

```python
# OpenAI Realtime API or Anthropic Voice
# ユーザーが話す → 文字起こし → LLM応答 → 音声で返答
# (ストリーミング、500ms 以下のレイテンシ)
```

→ **電話自動応答 / 音声秘書 / 通訳** などの新領域。

---

## 4. 動画解析(20分)

```python
# Frame extraction + Vision LLM
import cv2

cap = cv2.VideoCapture("video.mp4")
frames = []
for i in range(0, total_frames, fps):  # 1秒ごと
    cap.set(cv2.CAP_PROP_POS_FRAMES, i)
    ret, frame = cap.read()
    frames.append(frame)

# 各フレームを Claude Vision に
for frame in frames:
    description = describe_with_vision(frame)
```

→ 「**動画の何分に何が起きてるか**」自動タグ付け。

---

## 5. ⭐ 売れる成果物 #19:画像→請求書データ抽出

```
請求書写真をアップ → Claude Vision で構造化JSON
→ Excel/会計ソフトに自動入力

中小企業の経理を自動化、月20-50万円。
```

---

## ✅ チェック
- [ ] Vision API で画像理解
- [ ] DALL-E / SD で画像生成
- [ ] Voice API でリアルタイム会話
- [ ] 動画フレーム解析

## 次
`47_Embedded_Browser_Agent.md`
