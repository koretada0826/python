# Day 55:Stripe 決済深掘り - SaaS必須

## 💡 これは何?(小学生でもわかる説明)

****月額制でお金を取る仕組み****

### なぜ大事?
SaaS で月10万円の不労所得を作る基盤

### イメージ
自販機の**お金を入れる口と缶を出す機械**を作る

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 +30-100万(SaaS収益化)
## ⏱ 15時間

「**お金を取る仕組み**」がないと SaaS は進まない。

---

## 1. Stripe アカウント(10分)

1. https://stripe.com/jp 登録
2. テストモード で開発
3. ダッシュボード → API keys 取得
4. Webhook シークレット取得

---

## 2. Checkout Session(月額)(30分)

```bash
pip install stripe
```

```python
import stripe
stripe.api_key = os.getenv("STRIPE_SECRET")

# ダッシュボードで「商品」作成 → price_id 取得
PRICE_ID = "price_xxxxx"

@app.post("/create-checkout-session")
def create_checkout(user_id: int):
    session = stripe.checkout.Session.create(
        payment_method_types=["card"],
        line_items=[{"price": PRICE_ID, "quantity": 1}],
        mode="subscription",
        success_url="https://app.com/success?session_id={CHECKOUT_SESSION_ID}",
        cancel_url="https://app.com/cancel",
        client_reference_id=str(user_id),
        metadata={"user_id": user_id},
    )
    return {"url": session.url}
```

フロント:
```tsx
const { url } = await fetch("/create-checkout-session", {method: "POST"}).then(r => r.json());
window.location = url;
```

---

## 3. Webhook(超重要)(40分)

決済完了通知を受けて DB 更新。

```python
@app.post("/webhook")
async def webhook(request: Request):
    payload = await request.body()
    sig = request.headers.get("stripe-signature")
    
    try:
        event = stripe.Webhook.construct_event(
            payload, sig, os.getenv("STRIPE_WEBHOOK_SECRET")
        )
    except stripe.error.SignatureVerificationError:
        raise HTTPException(400)
    
    if event["type"] == "checkout.session.completed":
        session = event["data"]["object"]
        user_id = int(session["metadata"]["user_id"])
        
        # ユーザーをプレミアムに昇格
        db.update_user(user_id, plan="premium",
                      stripe_customer_id=session["customer"],
                      stripe_subscription_id=session["subscription"])
    
    elif event["type"] == "customer.subscription.deleted":
        # 解約処理
        ...
    
    elif event["type"] == "invoice.payment_failed":
        # 支払失敗通知
        ...
    
    return {"ok": True}
```

### ローカル Webhook テスト

```bash
stripe listen --forward-to localhost:8000/webhook
```

---

## 4. 顧客ポータル(20分)

```python
# 解約・カード変更を Stripe 任せ
session = stripe.billing_portal.Session.create(
    customer=user.stripe_customer_id,
    return_url="https://app.com/account",
)
return {"url": session.url}
```

→ **解約UI を自分で作らなくていい**。

---

## 5. 使用量ベース課金(20分)

```python
# API呼び出し回数で課金
stripe.SubscriptionItem.create_usage_record(
    "si_xxxxx",
    quantity=1,
    timestamp=int(time.time()),
    action="increment"
)
```

→ AI API SaaS で必須(従量課金)。

---

## 6. ⭐ 課題

```
1. Streamlit + FastAPI で Stripe月額課金実装
2. 無料/プレミアムで機能制限
3. Webhook で DB自動更新
4. 顧客ポータル統合
5. 7日トライアル設定
```

---

## ✅ チェック
- [ ] Checkout Session 作成
- [ ] Webhook で DB更新
- [ ] 顧客ポータル
- [ ] 試用期間・プロモコード
- [ ] **動く SaaS 課金システム完成**

## 次
`56_メール_通知.md`
