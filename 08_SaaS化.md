# Day 8:SaaS化 - 自分のWebサービスで月額収入

## 💰 この章後の月収:**20〜100万円**(MRRが積み上がる)
## ⏱ 所要:3-4時間

**月額制の自分のサービス**を持つと、寝てても収入が入る。
月100万を目指すなら**この階建てが必須**。

---

## SaaS とは

**Software as a Service** = 月額制で使えるWebサービス。

例:
- Slack: 月額 800円
- Notion: 月額 1,000円
- ChatGPT: 月額 $20

→ あなたも**月額制の小さいSaaS**を持つ。

---

## なぜ強いか

| 単発案件 | SaaS |
|---|---|
| 30,000円 × 月10件 | 月3,000円 × 100ユーザー |
| 毎月新規営業 | **積み上がり**で増えるだけ |
| 自分の時間 = 上限 | 寝てても入る |

→ **SaaS で月10万円積み上がれば、人生のセーフティネット**。

### 想定設計

| ユーザー数 | 月収 |
|---:|---:|
| 50ユーザー | 月15万 |
| 100 | 月30万 |
| 300 | 月90万 |
| 500 | 月150万 |

---

## ゴール

- Streamlit でWeb UI 作る
- ユーザー認証(ログイン)
- Stripe 月額課金
- Streamlit Cloud / Render に公開
- **動くSaaSプロトタイプ完成**

---

## 1. アイデア選定(15分)

### 売れるSaaSの条件

1. **特定の業界の困りごとを解決**
2. **月額数千円で買える**(個人 or 中小企業向け)
3. **Pythonで作れる**(複雑すぎない)

### 例(コピペで作っていいレベル)

| アイデア | ターゲット | 月額 |
|---|---|---:|
| 競合価格自動チェック+通知 | EC事業者 | 3,000円 |
| AI議事録要約サービス | 中小企業 | 5,000円 |
| 社内FAQ Bot(SaaS版RAG) | 50人未満の会社 | 10,000円 |
| YouTubeチャンネル分析 | YouTuber | 1,500円 |
| メルカリ相場通知 | 転売・主婦 | 1,000円 |
| 求人スクレイピング+AIマッチング | 転職希望者 | 2,000円 |

→ **Day 4-7で作ったツールをWeb化するだけ**。

---

## 2. Streamlit でWeb UI(60分)

```bash
pip install streamlit
```

### 基本

```python
# app.py
import streamlit as st

st.title("AI議事録要約")

uploaded = st.file_uploader("音声ファイルをアップロード", type=["mp3", "wav"])
if uploaded:
    st.audio(uploaded)
    if st.button("要約する"):
        with st.spinner("AI処理中..."):
            # ここで Whisper API → Claude で要約
            result = "..."
        st.success("完了!")
        st.write(result)
```

起動:
```bash
streamlit run app.py
```

→ **これだけでWebサービス**。

### クロードに頼む

```
Streamlit でAI議事録要約 SaaS のUIを作って。

機能:
- 音声/動画ファイルアップロード(.mp3, .wav, .mp4, .m4a)
- OpenAI Whisper API で文字起こし
- Claude API で要約・アクションアイテム抽出
- 結果をマークダウン表示・ダウンロード

サイドバーで:
- API使用量表示
- 設定(要約の詳しさ)
```

---

## 3. ユーザー認証(60分)

無料ユーザーと有料ユーザーを分ける。

### Streamlit Authenticator

```bash
pip install streamlit-authenticator
```

クロードに頼む:

```
streamlit-authenticator でログイン機能を Day 8 アプリに追加。

要件:
- ユーザー登録(メール+パスワード)
- ログイン状態保持
- ユーザーごとの利用回数記録(SQLite)
- 無料: 月3回まで、有料: 無制限
```

### より簡単な代替: Supabase

```bash
pip install supabase
```

→ Googleログイン・メール認証が簡単に作れる。

---

## 4. Stripe で月額課金(60分)

世界標準の決済システム。

### 登録(無料)

1. https://stripe.com/jp 登録
2. テストモードでまず動かす
3. 商品作成(月額3,000円のプランなど)

### Pythonから

```bash
pip install stripe
```

```python
import stripe
stripe.api_key = os.getenv("STRIPE_SECRET")

# Checkout セッション作成(月額制)
session = stripe.checkout.Session.create(
    payment_method_types=["card"],
    line_items=[{
        "price": "price_xxxxx",  # Stripeで作ったプランID
        "quantity": 1,
    }],
    mode="subscription",
    success_url="https://yoursaas.com/success",
    cancel_url="https://yoursaas.com/cancel",
)
print(session.url)  # ここに飛ばすと決済画面
```

クロードに「Stripe Checkout を Streamlit に統合する完全なコード書いて」と頼む。

### Webhook で決済完了通知を受ける

```python
@app.route("/webhook", methods=["POST"])
def webhook():
    # Stripeからの通知を受けて、ユーザーをプレミアムに昇格
    ...
```

→ FastAPI などで実装。

---

## 5. デプロイ(公開する)(40分)

### 無料オプション

| サービス | 用途 | 制限 |
|---|---|---|
| **Streamlit Cloud** | Streamlit専用 | 無料、公開のみ |
| **Render** | Webアプリ全般 | 無料プラン(スリープあり) |
| **Railway** | Webアプリ全般 | $5/月クレジット |
| **Vercel** | フロント専用 | API は別 |

### Streamlit Cloud デプロイ手順

1. GitHub にコードpush
2. https://share.streamlit.io
3. リポジトリを選んで「Deploy」
4. シークレット(API キー)を Streamlit 設定で登録
5. 公開URL取得

→ **30分でデプロイ完了**。

---

## 6. ⭐ 売れる成果物 #6:小さなSaaS(120分)

### 例:メルカリ相場通知サービス

#### 仕様

- ユーザーが「Switch」「PS5」など商品名登録
- 毎日朝9時に自動でメルカリ相場をチェック
- メールで通知
- 月額1,000円

### 実装の流れ

1. Day 5 のスクレイピングを Streamlit から呼ぶ
2. ユーザーごとに商品リスト管理(SQLite or Supabase)
3. APScheduler で毎日朝9時に実行
4. SendGrid でメール送信
5. Stripe で月額課金
6. Streamlit Cloud にデプロイ

→ **Day 4-8 の集大成**。

### クロードに頼む

```
以下の SaaS を作って。

【サービス】
〇〇相場通知

【機能】
- ユーザー登録(メール+パスワード)
- 商品キーワード登録(最大10件、無料は3件)
- 毎日朝9時 JST に Webスクレイピングで価格取得
- 価格変動が大きい時にメール通知
- ダッシュボードで履歴閲覧

【課金】
- 無料: 商品3件まで、通知は週1
- 有料: 商品10件、毎日通知、月額1,000円

【スタック】
- Streamlit
- SQLite or Supabase
- APScheduler
- SendGrid
- Stripe(月額制)

ファイル構成と requirements.txt も含めて完全なコード書いて。
```

→ クロードが書いてくれる。**動かしながら直す**。

---

## 7. SaaS の伸ばし方(15分)

### 集客

- X/Twitter で機能紹介(Day 10で詳細)
- ProductHunt 投稿
- Note・Brain で活用事例公開
- Reddit / Hacker News 投稿(英語)

### 価格戦略

- **Freemium**(無料あり、機能制限) ← おすすめ
- 有料の年額プラン(2ヶ月分割引)
- 法人向けプラン(月10万円)

### 解約防止

- 月次レポートを定期送信
- 「あなたの利用で時間が〇時間削減されました」と価値を可視化

---

## ✅ チェック

- [ ] Streamlit でUI動く
- [ ] ユーザー認証実装
- [ ] Stripe 決済画面動く(テストモード)
- [ ] Streamlit Cloud or Render に公開
- [ ] **小さなSaaS 1本がインターネット上で動いてる**

---

## 進捗

```
Day 8: SaaS化完了。〇〇通知サービスを公開。
URL: https://...
最初の有料ユーザー獲得を狙う。
```

## 次

`09_ポートフォリオ.md` へ。**作品を「お願いします」と言わせる形に仕上げる**。
