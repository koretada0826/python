# Day 46:Multi-modal AI - Vision・Voice・Image生成

## 💡 これは何?(小学生でもわかる説明)

**文字+画像+音声**全部扱える AI

### なぜ大事?
2026年は Multi-modal が標準。新領域単価300万円。

これまでの AI は「**文字専門**」だった。
これからは **目で見て・耳で聞いて・口でしゃべって・手で絵を描く** AI が主流。

「請求書の写真 → AIが構造化JSONに変換」
「動画 → AIが何分に何が起きてるかタグ付け」
「電話 → AIが話して受け答え」

これらが全部、API1本で作れる。
**できる人が少ない**ので、新領域は単価が**桁違い**。

### イメージ
- **Vision** = AIに**目**(写真・動画を理解)
- **Voice** = AIに**耳と口**(リアルタイム会話)
- **Image生成** = AIに**手**(絵を描く)
- **動画解析** = 全部の組み合わせ

### 比喩でひとこと
今までのAIが「**手紙のやり取りだけの新人**」だったのが、
Multi-modal AIは「**目・耳・口を持って一緒に働く同僚**」になる。
**できる仕事の幅が10倍**になる。

### Claude Code に何を頼めばいいか
- 「Claude Vision で画像から請求書データを抽出するコード」
- 「DALL-E 3 で YouTube サムネ自動生成」
- 「動画を1秒ごとにフレーム抜き出して Vision に投げて要約」
- 「Realtime API でリアルタイム電話受付」
こういう粒度で頼める。

---

## 💰 +30-100万(2026最新)
## ⏱ 15時間

文字だけじゃなく**画像・音声・動画**を扱える AI。
新領域なので**競合が少なく単価が高い**。

### 案件例の単価感

| 案件 | 単価 |
|---|---|
| 画像 → JSON抽出ツール | 50-200万 |
| 動画自動字幕システム | 30-100万 |
| YouTubeサムネ自動生成 | 月10-30万 |
| 音声秘書(電話自動応答) | 100-500万 |
| 動画解析(タグ付け) | 50-200万 |

---

## ゴール

- **Vision API**(Claude / GPT-4o)で画像を理解
- **画像生成**(DALL-E / Stable Diffusion)を使える
- **Realtime Voice**(リアルタイム会話)の概念
- **動画フレーム解析**ができる
- **Multi-modal RAG** の構成が分かる

---

## 1. Claude Vision / GPT-4o Vision(20分)

### 1-1. Vision って何?

> **イメージ:** **写真を渡したら、AIが説明してくれる**機能。
> しかも「**請求書の金額だけJSONで返して**」みたいな**構造化指示**もできる。

### 1-2. Claude Vision のコード

```python
import base64
from anthropic import Anthropic

client = Anthropic()

# 画像をbase64に変換(APIに送るため)
with open("photo.jpg", "rb") as f:
    img_b64 = base64.b64encode(f.read()).decode()

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/jpeg",
                    "data": img_b64
                }
            },
            {"type": "text", "text": "この画像を説明して"}
        ]
    }]
)
print(response.content[0].text)
```

### 1-3. 構造化抽出(請求書の例)

```python
prompt = """
この請求書から以下のJSONを返して(マークダウン記号なしで):
{
  "invoice_no": "...",
  "date": "YYYY-MM-DD",
  "vendor": "...",
  "items": [{"name": "...", "qty": ..., "price": ...}],
  "total": ...
}
"""
```

→ **写真→説明文**、**請求書→構造化データ**、**図表→分析** が一発。

### 1-4. 何ができるようになる?

| やりたいこと | Vision で実現 |
|---|---|
| 請求書 → 経理システム | 月20-50万円案件 |
| 名刺 → 顧客DB | 営業ツール |
| 商品写真 → カタログ生成 | EC案件 |
| 図表 → データ分析 | コンサル案件 |
| 古い手書き文書 → デジタル化 | 行政・法務 |

---

## 2. Image生成(DALL-E / Stable Diffusion)(20分)

### 2-1. DALL-E 3(OpenAI)

```python
from openai import OpenAI

client = OpenAI()
img = client.images.generate(
    model="dall-e-3",
    prompt="A cyberpunk city at sunset, photorealistic",
    size="1024x1024",
)
print(img.data[0].url)  # 生成された画像のURL
```

### 2-2. Stable Diffusion(ローカル・無料)

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

### 2-3. DALL-E vs Stable Diffusion

| 観点 | DALL-E 3 | Stable Diffusion |
|---|---|---|
| 料金 | $0.04/枚 | 無料(電気代のみ) |
| 速度 | 速い(数秒) | GPU必須・10-30秒 |
| 品質 | 安定して高い | プロンプト次第 |
| カスタマイズ | 限定 | LoRA・ControlNetで自由 |
| 商用利用 | OK(規約確認) | OK(モデル次第) |

→ **YouTubeサムネイル自動生成**(月10-30万円案件)。

### 2-4. プロンプトのコツ

| 要素 | 例 |
|---|---|
| 主題 | "A cat riding a horse" |
| スタイル | "photorealistic", "anime style" |
| 構図 | "close-up", "wide shot" |
| 光・色 | "golden hour", "neon lights" |
| 解像度 | "8k, ultra detailed" |

---

## 3. Voice AI - リアルタイム会話(30分)

### 3-1. 構成

```
ユーザーが話す
  → 文字起こし(Whisper や Realtime API)
  → LLM応答(Claude / GPT-4)
  → 音声で返答(TTS)
  
全部ストリーミングで500ms以下のレイテンシ
```

### 3-2. 用途

| 案件 | 単価 |
|---|---|
| 電話自動応答 | 100-300万 |
| 音声秘書 | 50-200万 |
| 通訳(日↔英) | 50-150万 |
| 子供向け学習アプリ | 30-100万 |
| 介護施設の見守り | 100-500万 |

### 3-3. 技術選択肢

- **OpenAI Realtime API**(`gpt-4o-realtime`)
- **Anthropic Voice**
- **ElevenLabs**(TTS品質最強)
- **Twilio + LLM**(電話統合)

→ ガチ実装はクロード相談で十分(やる気が出てから着手)。

---

## 4. 動画解析(20分)

### 4-1. 動画はフレームの集まり

```python
import cv2

cap = cv2.VideoCapture("video.mp4")

# 動画情報
fps = cap.get(cv2.CAP_PROP_FPS)              # 1秒あたりのフレーム数
total_frames = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))

# 1秒ごとにフレーム抜き出し
frames = []
for i in range(0, total_frames, int(fps)):
    cap.set(cv2.CAP_PROP_POS_FRAMES, i)
    ret, frame = cap.read()
    if ret:
        frames.append(frame)

# 各フレームを Claude Vision に送る
for i, frame in enumerate(frames):
    description = describe_with_vision(frame)
    print(f"{i}秒: {description}")
```

→ 「**動画の何分に何が起きてるか**」を自動でタグ付け。

### 4-2. ユースケース

- **YouTube動画の自動チャプター生成**
- **監視カメラから異常検知**
- **スポーツ動画の自動ハイライト抽出**
- **教育動画の自動字幕+要約**
- **不適切コンテンツ検出**

---

## 5. ⭐ 売れる成果物 #19:画像→請求書データ抽出

```
構成:
1. ブラウザで請求書写真をアップ
2. Claude Vision で構造化JSON取得
3. Excel/会計ソフト(freee/マネーフォワード)に自動入力
4. 月額課金(Stripe)

技術:
- Streamlit / FastAPI
- Anthropic Vision
- pandas (Excel出力)
- freee / MF API(連携)
```

→ 中小企業の経理を自動化、**月20-50万円**の月額契約。

### 営業文の例

```
御社の経理担当者、毎日請求書何枚処理してますか?

弊社の AI ツールなら:
・写真を撮るだけで自動入力
・freee/マネーフォワードに直接連携
・月額3万円(月100枚まで無制限)

担当者の作業時間が 1日3時間 → 30分 になります。
無料トライアルご希望ならご返信ください。
```

---

## 6. Multi-modal RAG(発展)

### 6-1. 構成

```
文書 + 画像 + 図表 を全部 Embedding 化
  → ベクトルDBに保存
  → 質問が来たら、テキストも画像も検索
  → Vision LLM が画像も参照しながら回答
```

→ 工場のマニュアル、医療カルテ、設計図など**画像が多い領域**で破壊力大。

---

## 7. よくあるエラー・落とし穴

| 症状 | 原因 | 直し方 |
|---|---|---|
| Vision API が拒否される | 不適切画像と判定 | 別画像で試す |
| JSON が壊れる | LLMが余計な説明 | 「JSONのみ返せ」を強調 |
| DALL-E が遅い | 待つしかない | 並列リクエスト(レート上限注意) |
| Stable Diffusion 重い | GPU不足 | Colab Pro / Quantization |
| 動画が重すぎる | フル解像度のまま | 解像度落として処理 |

---

## 8. 用語まとめ表

| 用語 | 一言で |
|---|---|
| Multi-modal | 文字+画像+音声を扱う |
| Vision API | 画像を理解するLLM API |
| 構造化抽出 | 画像/テキストから JSON を取り出す |
| DALL-E | OpenAIの画像生成 |
| Stable Diffusion | オープンソースの画像生成 |
| LoRA(画像) | スタイルを足すための軽量学習 |
| ControlNet | 構図を指定して画像生成 |
| Realtime API | 音声リアルタイム会話API |
| TTS | 文字 → 音声 |
| Frame extraction | 動画 → 画像列 |
| Multi-modal RAG | 画像も検索対象に含めるRAG |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **画像 + 文章 + 音声マルチモーダル AI 開発**:単価 100〜500万円(プラットフォーム例:Findy Freelance/直契約/SES)
- **動画解析 / リアルタイム音声 AI システム**:単価 100〜300万円
- **Vision API を使った業務自動化(請求書 OCR / 商品画像分類)**:単価 50〜150万円

### クロードへの頼み方(営業文を書かせる例)
```
法人向け、マルチモーダルAI システム開発の提案文を書いて。
Day 46 までで習得:Vision API / 画像生成 / Realtime Voice / 動画フレーム解析 / マルチモーダル RAG
売り:画像・音声・動画を統合して扱える(テキスト中心の RAG より一段先)
価格:PoC 100万〜 / 本番 300万〜
ターゲット:現場写真の分析、コールセンター音声の解析、教育・研修動画の分析を求める企業
過去実績:学習用に「請求書画像 → 構造化JSON」「会議動画 → 議事録」のサンプルアプリ
800字
```

### この章だけでは足りないもの(次に学ぶべき)
- Browser Agent(Day 47) ★最先端
- 数学の深掘り(Day 48)で更に高単価ML案件へ
- 海外案件・英語(Day 49)

---

## ✅ チェック
- [ ] Vision API で画像を説明できた
- [ ] Vision で構造化JSON抽出できた
- [ ] DALL-E / Stable Diffusion で画像生成できた
- [ ] Realtime Voice の構成を説明できる
- [ ] 動画フレーム解析を試した
- [ ] Multi-modal RAG の概要が分かる

## 次
`47_Browser_Agent.md` で**自分で動くAI**を作る。
