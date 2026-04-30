# Day 55:Stripe 決済深掘り - SaaS必須

## 💡 これは何?(小学生でもわかる説明)

**月額制でお金をもらう仕組み**を Stripe というサービスで作る日。

### なぜ大事?
- SaaS(月額サービス)を作るなら**Stripe ほぼ必須**
- 自前で決済を作ると**法律・セキュリティで死ぬ**
- Stripe を使うと**5分で月額課金がスタート**できる
- 月10万円の不労所得を作る基盤

### イメージ
- Stripe = **お店のレジ + 自販機のシステム**
  - お客様の**カード情報を安全に預かる**
  - 毎月**自動で課金**してくれる
  - 解約・払戻もボタン一つで対応
- Webhook = **「決済完了しました!」の通知書**
  - Stripe がお店(あなたのサーバー)に**「お金入りました」と教えてくれる**
  - その通知を受けて、ユーザーをプレミアム会員にする

> **もう1つの例え:**
> 自前決済=**お店で自分でお金を数える**(間違い・盗難リスク)
> Stripe=**プロのレジ係を雇う**(法律・セキュリティ全部おまかせ)

### Claude Code に何を頼めばいいか
- 「FastAPI で Stripe Checkout を呼ぶ /create-checkout-session 書いて」
- 「Webhook 受信エンドポイントを書いて(署名検証込み)」
- 「ユーザーをプレミアム化する DB 更新コードを書いて」
- 「7日トライアル付きの月額プランを Stripe に作る方法教えて」

---

## 💰 +30〜100万(SaaS収益化)
## ⏱ 15時間

「**お金を取る仕組み**」がないと SaaS は進まない。

---

## ゴール

- Stripe アカウントを作って**テストモードで決済成功**できる
- Checkout Session を作って**支払いページに飛ばせる**
- Webhook を受けて**DB を自動更新**できる
- 顧客ポータル(解約・カード変更)を導入できる
- 使用量ベース課金(従量課金)を実装できる

---

## 1. Stripe アカウント(=「お店の開店準備」)(10分)

### 手順

1. https://stripe.com/jp に登録(メアド+パスワード)
2. **テストモード**で開発(本物のお金は動かない)
3. ダッシュボード → 開発者 → API keys → **Secret key** をコピー
4. 開発者 → Webhook → **シークレット** をコピー

> **本番化のタイミング:**
> 本人確認(本名・口座・身分証)を済ませると本番モードに切替可能。
> **開発中はテストモードで完結**できる。

### テストカード(本番では使えない)

| 番号 | 結果 |
|---|---|
| 4242 4242 4242 4242 | 成功 |
| 4000 0000 0000 0002 | 失敗(カード拒否) |
| 4000 0000 0000 9995 | 失敗(残高不足) |

→ 有効期限・CVC は**何でもOK**(将来の日付・3桁数字)。

---

## 2. Checkout Session(=「支払いページに飛ばす」)(30分)

### 2-1. インストール

```bash
pip install stripe
```

### 2-2. 月額プラン作成(ダッシュボードで先に)

1. ダッシュボード → 商品カタログ
2. 「+商品を追加」 → 名前・月額(例: 980円/月)入力
3. 作成後、**price_xxxxx** という ID をコピー

### 2-3. サーバーコード(FastAPI)

```python
import stripe
import os

# 秘密鍵をセット(.env から読む)
stripe.api_key = os.getenv("STRIPE_SECRET")

# ダッシュボードでコピーしたID
PRICE_ID = "price_xxxxx"

@app.post("/create-checkout-session")
def create_checkout(user_id: int):
    # Stripe に「支払いページ作って」と依頼
    session = stripe.checkout.Session.create(
        payment_method_types=["card"],                    # カード決済のみ
        line_items=[{"price": PRICE_ID, "quantity": 1}],  # この商品を1個
        mode="subscription",                              # 月額課金モード
        success_url="https://app.com/success?session_id={CHECKOUT_SESSION_ID}",
        cancel_url="https://app.com/cancel",
        client_reference_id=str(user_id),                 # 誰の支払いか紐付け
        metadata={"user_id": user_id},                    # 後で使う追加データ
    )
    # フロントに支払いページURLを返す
    return {"url": session.url}
```

### 2-4. フロント側

```tsx
// ボタン押したら支払いページに飛ばす
const { url } = await fetch("/create-checkout-session", { method: "POST" })
  .then(r => r.json());

window.location = url;  // Stripe ホストの支払いページへ移動
```

> **なぜ自前で決済画面作らない?**
> 1. **法律(PCI DSS)** が厳しすぎて自作はほぼ不可能
> 2. Stripe が**毎月セキュリティ更新**してくれる
> 3. 多通貨・3Dセキュア・各種カード対応が**標準で全部入り**

---

## 3. Webhook(=「決済完了の通知書」)(40分)

### 3-1. なぜ必要?

> ユーザーがブラウザで決済しても、**ブラウザが閉じられたら通知届かない**かもしれない。
> Stripe → あなたのサーバーに**直接イベント通知**することで確実にDB更新できる。

### 3-2. コード

```python
@app.post("/webhook")
async def webhook(request: Request):
    payload = await request.body()
    sig = request.headers.get("stripe-signature")
    
    # 署名検証=本物の Stripe からの通知か確認
    try:
        event = stripe.Webhook.construct_event(
            payload, sig, os.getenv("STRIPE_WEBHOOK_SECRET")
        )
    except stripe.error.SignatureVerificationError:
        raise HTTPException(400)  # 偽物の通知=拒否
    
    # 決済完了イベント
    if event["type"] == "checkout.session.completed":
        session = event["data"]["object"]
        user_id = int(session["metadata"]["user_id"])
        
        # ユーザーをプレミアムに昇格
        db.update_user(
            user_id,
            plan="premium",
            stripe_customer_id=session["customer"],
            stripe_subscription_id=session["subscription"],
        )
    
    # 解約イベント
    elif event["type"] == "customer.subscription.deleted":
        # サブスクID から user 探して plan="free" に戻す
        ...
    
    # 支払失敗イベント
    elif event["type"] == "invoice.payment_failed":
        # メール送信などの通知処理
        ...
    
    return {"ok": True}
```

> **`stripe-signature` 検証は超重要。**
> これしないと**偽の通知でユーザーを勝手にプレミアム化**される。

### 3-3. ローカル Webhook テスト

```bash
# Stripe CLI をインストールしてログイン
stripe login

# ローカルサーバーに転送
stripe listen --forward-to localhost:8000/webhook
```

→ コマンド実行後に**シークレット**が表示されるので、`.env` の `STRIPE_WEBHOOK_SECRET` に設定。

### 3-4. 主なイベント早見

| イベント名 | 何が起きた |
|---|---|
| `checkout.session.completed` | 初回決済成功(プレミアム化) |
| `customer.subscription.created` | サブスク作成 |
| `customer.subscription.updated` | プラン変更 |
| `customer.subscription.deleted` | 解約 |
| `invoice.payment_succeeded` | 月次決済成功 |
| `invoice.payment_failed` | 支払失敗 |

---

## 4. 顧客ポータル(=「解約・変更を任せる」)(20分)

### なぜ?

> 「解約画面」「カード変更画面」を**自分で作るのは大変**。
> Stripe が**完璧なUI**を提供してくれる。

```python
@app.post("/customer-portal")
def open_portal(user_id: int):
    user = db.get_user(user_id)
    
    # 解約・カード変更を Stripe ホストの画面で
    session = stripe.billing_portal.Session.create(
        customer=user.stripe_customer_id,
        return_url="https://app.com/account",  # 戻り先
    )
    return {"url": session.url}
```

→ **解約UI を自分で作らなくていい**。法的にも安心。

---

## 5. 使用量ベース課金(=「使った分だけ課金」)(20分)

### なぜ?

- AI API を使うSaaS(問い合わせ回数で課金)
- 月額固定 + 超過分は従量(API呼び出し1回 = 0.1円)

### コード

```python
import time

# API 1回呼ばれるごとに使用量を記録
stripe.SubscriptionItem.create_usage_record(
    "si_xxxxx",                          # サブスクアイテムID
    quantity=1,                          # 増分
    timestamp=int(time.time()),          # 現在時刻
    action="increment",                  # 加算モード
)
```

→ 月末に Stripe が**自動で計算 → 請求**してくれる。

> **ダッシュボードで:**
> 商品作成時に「**従量課金**」タイプを選ぶと使用量ベースになる。

---

## 6. よくあるエラーと対処

| 症状 | 原因 | 直し方 |
|---|---|---|
| `No such price: price_xxx` | テスト/本番のIDを混同 | 環境に合わせたIDを使用 |
| `Invalid API key` | キーが古い/間違い | ダッシュボードで再取得 |
| Webhook が届かない | URL が外部から到達不可 | Stripe CLI か ngrok でトンネル |
| 署名検証失敗 | シークレットの不一致 | listen 表示の `whsec_` を再確認 |
| 決済後にDB未更新 | Webhook ハンドラ未動作 | Stripe ダッシュボードのログ確認 |
| 同一イベントで複数処理 | 冪等性チェックなし | `event["id"]` を保存して重複防止 |
| `customer_id` が None | metadata 紐付けなし | `client_reference_id` を渡す |
| トライアル中に課金 | 設定漏れ | 商品設定で `trial_period_days` 指定 |

---

## 7. ⭐ 課題

```
1. Streamlit + FastAPI で Stripe月額課金実装
   - ログイン → プラン選択 → 決済 → プレミアム化

2. 無料/プレミアムで機能制限
   - 無料は1日10回まで、プレミアム無制限

3. Webhook で DB自動更新
   - 決済成功→プレミアム化、解約→無料化
   - 支払失敗→メール通知

4. 顧客ポータル統合
   - 「サブスク管理」ボタン → portal へ

5. 7日トライアル設定
   - 商品設定 + UI で「7日無料」表示
```

---

## 8. 用語まとめ表

| 用語 | 一言で |
|---|---|
| Stripe | 決済サービスの最大手 |
| テストモード | 本物のお金が動かない開発モード |
| Secret Key | サーバー側の秘密鍵(`sk_...`) |
| Publishable Key | フロント公開OKの鍵(`pk_...`) |
| Price ID | 商品の値段ID(`price_...`) |
| Checkout Session | Stripeホストの支払いページ |
| `mode: "subscription"` | 月額課金モード |
| `client_reference_id` | 誰の決済かの紐付け用ID |
| `metadata` | 後で使う追加データ置き場 |
| Webhook | Stripe→サーバーへのイベント通知 |
| `stripe-signature` | 通知の本物検証用ヘッダ |
| `whsec_...` | Webhook シークレット |
| Customer Portal | Stripeホストの解約・カード変更画面 |
| Subscription Item | 使用量ベース課金の単位 |
| 冪等性(べきとうせい) | 同じイベントを何度処理しても結果が同じ |
| Stripe CLI | ローカル開発用ツール |
| トライアル | 無料試用期間(自動課金開始前) |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **Stripe 決済組み込み案件**:単価 30〜100万円(プラットフォーム例:ランサーズ/直契約/Findy Freelance)
- **既存 SaaS への課金プラン追加**:単価 30〜80万円(月額 / 年額 / 従量課金)
- **個人開発SaaSの収益化代行**:単価 20〜50万円(Stripe + Webhook + DB更新まで)

### クロードへの頼み方(営業文を書かせる例)
```
SaaS 開発者向け、「決済機能だけ短期で組み込みます」プランの宣伝文を書いて。
Day 55 までで習得:Stripe Checkout / Subscription / Webhook 署名検証 / 顧客ポータル / トライアル / プロモコード / 使用量ベース
売り:1〜2週間で「決済できる状態」を完成させる、特急対応
価格:月額プランのみ 30万円 / 年額 + 月額 + トライアル 60万円 / 従量課金 100万円
ターゲット:SaaS のローンチ準備中で時間が無い創業者
500字
```

### この章だけでは足りないもの(次に学ぶべき)
- メール / 通知(Day 56)で決済後の連絡を仕上げる
- 法人化(Day 57)で大手 SaaS 案件参入
- 個人ブランディング(Day 58)で指名案件化

---

## ✅ チェック

- [ ] Stripe アカウント作成 + テストモードで決済成功
- [ ] Checkout Session 作成できる
- [ ] Webhook で署名検証 + DB更新できる
- [ ] 顧客ポータル統合できる
- [ ] 試用期間・プロモコード設定できる
- [ ] 使用量ベース課金が分かる
- [ ] **動く SaaS 課金システム完成**

---

## 進捗

```
Day 55: Stripe 決済深掘り完了。
月額課金 + Webhook + 顧客ポータル + 従量課金 まで体得。
SaaS の収益化基盤が動く。
```

## 次

`56_メール_通知.md` へ。**ユーザーに連絡する仕組み**。
