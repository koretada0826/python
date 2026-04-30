# Day 8:SaaS化 - 自分のWebサービスで月額収入

## 💡 これは何?(小学生でもわかる説明)

****自分の Web サービス**を作って月額で課金**

### なぜ大事?
寝てる間にも収入が入る仕組み。**月10万円の不労所得**を目指せる。
単発案件は「働いた分だけお金」だが、SaaS は「**作った分が積み上がる**」。
これが **月100万を超える唯一の道**。

### イメージ
- **SaaS** = 「**自販機を1台持つ**」(維持費なしで毎月お金が入る)
- **Streamlit** = 「**自販機の見た目(UI)を作る**」
- **認証(ログイン)** = 「**会員カードチェック**」
- **Stripe** = 「**お金を回収する仕組み**」(銀行みたいな役割)
- **デプロイ** = 「**自販機を街角に置く**」(公開する)
- **MRR** = 「**月額売上の積み上げ**」(Monthly Recurring Revenue)
- **Freemium** = 「**お試し無料 + 本気は有料**」(漫画の試し読みと同じ)

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。
**「Streamlit + Stripe + 認証で 〇〇 SaaS 作って」と頼めば、雛形ができる**。

---

## 💰 この章後の月収:**20〜100万円**(MRR が積み上がる)
## ⏱ 所要:3-4時間

**月額制の自分のサービス**を持つと、寝てても収入が入る。
月100万を目指すなら **この階建てが必須**。

---

## SaaS とは(=「月額制の Web サービス」)

**Software as a Service** = 月額制で使える Web サービス。

例:
- Slack: 月額 800円
- Notion: 月額 1,000円
- ChatGPT: 月額 $20

→ あなたも **月額制の小さい SaaS** を持つ。

---

## なぜ強いか(=「単発案件との違い」)

| 単発案件 | SaaS |
|---|---|
| 30,000円 × 月10件 | 月3,000円 × 100ユーザー |
| 毎月新規営業が必要 | **積み上がり**で増えるだけ |
| 自分の時間 = 売上の上限 | 寝てても入る |
| 納品で関係終わり | **継続収益** |

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

- Streamlit で **Web UI** 作る
- ユーザー認証(ログイン)を組み込む
- **Stripe** で月額課金できる
- Streamlit Cloud / Render に**公開**する
- **動く SaaS プロトタイプ完成**

---

## 1. アイデア選定(=「何を売るか決める」)(15分)

### 1-1. 売れる SaaS の条件

1. **特定の業界の困りごとを解決**(=広く浅くより、狭く深く)
2. **月額数千円で買える**(個人 or 中小企業向け)
3. **Python で作れる**(複雑すぎない)
4. **Day 4〜7 で作ったツールの延長線**(=学んだ知識で作れる)

### 1-2. 例(コピペで作っていいレベル)

| アイデア | ターゲット | 月額 |
|---|---|---:|
| 競合価格自動チェック+通知 | EC 事業者 | 3,000円 |
| AI 議事録要約サービス | 中小企業 | 5,000円 |
| 社内 FAQ Bot(SaaS版RAG) | 50人未満の会社 | 10,000円 |
| YouTube チャンネル分析 | YouTuber | 1,500円 |
| メルカリ相場通知 | 転売・主婦 | 1,000円 |
| 求人スクレイピング+AIマッチング | 転職希望者 | 2,000円 |

→ **Day 4-7 で作ったツールを Web 化するだけ**。

---

## 2. Streamlit で Web UI(=「自販機の見た目」)(60分)

### 2-1. インストール

```bash
pip install streamlit
```

### 2-2. 基本

```python
# app.py
import streamlit as st

st.title("AI議事録要約")

# ファイルアップロード欄
uploaded = st.file_uploader("音声ファイルをアップロード", type=["mp3", "wav"])
if uploaded:
    st.audio(uploaded)
    if st.button("要約する"):
        with st.spinner("AI処理中..."):  # くるくるアイコンを出す
            # ここで Whisper API → Claude で要約
            result = "..."
        st.success("完了!")
        st.write(result)
```

起動:
```bash
streamlit run app.py
```

→ **これだけで Web サービス**。

### 2-3. クロードに頼む文

```
Streamlit で AI 議事録要約 SaaS の UI を作って。

機能:
- 音声/動画ファイルアップロード(.mp3, .wav, .mp4, .m4a)
- OpenAI Whisper API で文字起こし
- Claude API で要約・アクションアイテム抽出
- 結果をマークダウン表示・ダウンロード

サイドバーで:
- API 使用量表示
- 設定(要約の詳しさ)

ファイル名は app.py。
```

---

## 3. ユーザー認証(=「会員カードチェック」)(60分)

> 無料ユーザーと有料ユーザーを **分ける**ために必須。
> 「誰がどのプランか」を覚える仕組み。

### 3-1. Streamlit Authenticator

```bash
pip install streamlit-authenticator
```

クロードに頼む文:

```
streamlit-authenticator でログイン機能を Day 8 アプリに追加。

要件:
- ユーザー登録(メール+パスワード)
- ログイン状態保持
- ユーザーごとの利用回数記録(SQLite)
- 無料: 月3回まで、有料: 無制限
- パスワードはハッシュ化して保存
```

### 3-2. より簡単な代替: Supabase

```bash
pip install supabase
```

> **Supabase って何?**
> Google ログイン・メール認証を **すぐに**作れるクラウドサービス。
> Firebase のオープンソース版みたいなもの。**個人開発の認証はこれが楽**。

→ Google ログイン・メール認証が簡単に作れる。

---

## 4. Stripe で月額課金(=「お金を回収する」)(60分)

> 世界標準の決済システム。
> **個人でも数分で導入できる** = 個人開発者の救世主。

### 4-1. 登録(無料)

1. https://stripe.com/jp 登録
2. **テストモード**でまず動かす(本物のお金を使わずテスト)
3. 商品作成(月額3,000円のプランなど)

### 4-2. Python から決済画面を作る

```bash
pip install stripe
```

```python
import stripe
import os

stripe.api_key = os.getenv("STRIPE_SECRET")  # .env から読む

# Checkout セッション作成(月額制)
session = stripe.checkout.Session.create(
    payment_method_types=["card"],
    line_items=[{
        "price": "price_xxxxx",  # Stripe で作ったプラン ID
        "quantity": 1,
    }],
    mode="subscription",                       # 月額制
    success_url="https://yoursaas.com/success",
    cancel_url="https://yoursaas.com/cancel",
)
print(session.url)  # ここに飛ばすと決済画面
```

クロードに「Stripe Checkout を Streamlit に統合する完全なコード書いて」と頼む。

### 4-3. Webhook で決済完了通知を受ける

```python
@app.route("/webhook", methods=["POST"])
def webhook():
    # Stripe からの通知を受けて、ユーザーをプレミアムに昇格
    ...
```

> **Webhook って何?**
> 「**事件が起きたら向こうから連絡が来る仕組み**」。
> 決済が完了した瞬間に Stripe があなたのサーバーに通知してくれる。

→ FastAPI などで実装。

---

## 5. デプロイ(=「自販機を街角に置く」)(40分)

### 5-1. 無料オプション

| サービス | 用途 | 制限 |
|---|---|---|
| **Streamlit Cloud** | Streamlit 専用 | 無料、公開のみ |
| **Render** | Web アプリ全般 | 無料プラン(スリープあり) |
| **Railway** | Web アプリ全般 | $5/月クレジット |
| **Vercel** | フロント専用 | API は別 |

### 5-2. Streamlit Cloud デプロイ手順

1. GitHub にコード push
2. https://share.streamlit.io にアクセス
3. リポジトリを選んで「Deploy」
4. **シークレット(API キー)**を Streamlit 設定で登録
5. **公開 URL** 取得

→ **30分でデプロイ完了**。
**自分のサービスが世界中からアクセスされる**瞬間、エンジニアになった実感が出る。

---

## 6. ⭐ 売れる成果物 #6:小さな SaaS(120分)

### 6-1. 例:メルカリ相場通知サービス

#### 仕様

- ユーザーが「Switch」「PS5」など **商品名登録**
- 毎日朝9時に自動でメルカリ相場をチェック
- メールで通知
- 月額 1,000円

### 6-2. 実装の流れ

1. Day 5 のスクレイピングを Streamlit から呼ぶ
2. ユーザーごとに商品リスト管理(SQLite or Supabase)
3. **APScheduler** で毎日朝9時に実行
4. **SendGrid** でメール送信
5. Stripe で月額課金
6. Streamlit Cloud にデプロイ

→ **Day 4-8 の集大成**。

### 6-3. クロードに頼む文

```
以下の SaaS を作って。

【サービス】
〇〇相場通知

【機能】
- ユーザー登録(メール+パスワード)
- 商品キーワード登録(最大10件、無料は3件)
- 毎日朝9時 JST に Web スクレイピングで価格取得
- 価格変動が大きい時にメール通知
- ダッシュボードで履歴閲覧

【課金】
- 無料: 商品3件まで、通知は週1
- 有料: 商品10件、毎日通知、月額 1,000円

【スタック】
- Streamlit
- SQLite or Supabase
- APScheduler
- SendGrid
- Stripe(月額制)

ファイル構成と requirements.txt も含めて完全なコード書いて。
.env と .gitignore も用意。
```

→ クロードが書いてくれる。**動かしながら直す**。

---

## 7. SaaS の伸ばし方(=「お客を増やす」)(15分)

### 7-1. 集客

- **X / Twitter** で機能紹介(Day 10 で詳細)
- **ProductHunt** 投稿(海外向け)
- **Note / Brain** で活用事例公開
- **Reddit / Hacker News** 投稿(英語)

### 7-2. 価格戦略

- **Freemium**(無料あり、機能制限) ← おすすめ
- **年額プラン**(2ヶ月分割引で囲い込み)
- **法人向けプラン**(月10万円)

### 7-3. 解約防止

- **月次レポート**を定期送信(「今月これだけ使ってます」)
- **「あなたの利用で時間が〇時間削減されました」**と価値を可視化

---

## 8. よくあるエラーと対処

| エラー / 問題 | 原因 | 直し方 |
|---|---|---|
| Streamlit Cloud で `ModuleNotFoundError` | requirements.txt 未設定 | プロジェクトルートに requirements.txt を置く |
| 環境変数(API キー)が読めない | Streamlit Cloud の Secrets 未設定 | アプリ設定 > Secrets に登録 |
| Stripe Webhook が届かない | URL が違う / 署名検証失敗 | Stripe ダッシュボードでログ確認 |
| メール届かない | SendGrid の送信元未認証 | Sender Authentication を済ませる |
| Render が寝る(無料プラン) | アクセス無いとスリープ | UptimeRobot で定期 ping |
| ログイン状態が消える | session_state 揮発性 | DB に保存(SQLite / Supabase) |
| `.env` が GitHub に上がった | `.gitignore` 設定忘れ | **即座にキー無効化**して再発行 |
| 課金後もユーザーが無料のまま | Webhook が動いてない | Stripe ダッシュボードでイベント確認 |

---

## 9. 用語まとめ

| 用語 | 一言で |
|---|---|
| SaaS | 月額制で使う Web サービス |
| MRR | 月額繰り返し収入(Monthly Recurring Revenue) |
| Streamlit | Python で Web 画面が作れるライブラリ |
| デプロイ | サービスを公開すること |
| Streamlit Cloud | Streamlit 専用の無料公開サーバー |
| Render | Web アプリ汎用ホスティング |
| 認証 | 誰がアクセスしてるか確認する仕組み |
| Stripe | 月額課金できる決済プラットフォーム |
| Checkout | Stripe の決済画面 |
| Webhook | 事件が起きたら向こうから通知が来る仕組み |
| Freemium | 無料 + 有料プランの組み合わせ |
| APScheduler | 定時実行ライブラリ(cron みたいなもの) |
| SendGrid | メール送信サービス |
| Supabase | 認証・DB をまとめてくれる BaaS |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **Streamlit 製の業務ダッシュボード / 簡易 SaaS 開発**:単価 10〜50万円(プラットフォーム例:ランサーズ/クラウドワークス/直契約)
- **小規模 SaaS の MVP 開発(認証 + Stripe 決済)**:単価 30〜80万円
- **社内ツールの Web 化(社内 Excel → Streamlit 化)**:単価 10〜30万円(継続保守 月 3〜10万円)
- **自分の SaaS の MRR**:月 10〜100万円(積み上げ次第)

### クロードへの頼み方(営業文を書かせる例)
```
ランサーズの「業務効率化 Web アプリ開発」案件向け提案文を書いて。
Day 8 までで習得したスキル:Streamlit / FastAPI / 認証 / Stripe / Streamlit Cloud / Render デプロイ
売り:依頼者の Excel 業務を 1〜2週間で Web アプリ化、課金つき SaaS も作れる
過去実績:小規模 SaaS を自分で1本デプロイ済み(URL 貼れる)
納期:2〜3週間
予算:20〜40万円
ポイント:納品後の運用(認証・決済・サーバー保守)も月額で受けられる
```

### この章だけでは足りないもの(次に学ぶべき)
- ポートフォリオとして見せ方(Day 9)
- GitHub / SNS でのアピール(Day 10)
- 本番運用(監視 / ログ / スケーリング)
- フロントエンド UI の強化(React / Tailwind)

---

## ✅ チェック

- [ ] Streamlit で UI 動く
- [ ] ユーザー認証実装
- [ ] Stripe 決済画面動く(テストモード)
- [ ] Webhook で決済完了通知を受け取れる
- [ ] Streamlit Cloud or Render に公開
- [ ] `.env` を `.gitignore` に入れた
- [ ] **小さな SaaS 1本がインターネット上で動いてる**

---

## 進捗

```
Day 8: SaaS化完了。〇〇通知サービスを公開。
URL: https://...
最初の有料ユーザー獲得を狙う。
```

## 次

`09_ポートフォリオ.md` へ。**作品を「お願いします」と言わせる形に仕上げる**。
