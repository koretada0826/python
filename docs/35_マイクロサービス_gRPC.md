# Day 35:マイクロサービス・gRPC・Kafka

## 💡 これは何?(小学生でもわかる説明)

****1つの巨大アプリを小さいアプリ群**に分解する**

### なぜ大事?
Netflix/Uber級のシステム。月単価120万円

### イメージ
1人で全料理 → **寿司部・パン部・ラーメン部**で分業

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 +30-100万(エンタープライズ)
## ⏱ 25時間

「**1つのデカいアプリ**」を「**小さいアプリ群**」に分解する技術。Netflix・Uber級のシステム。

---

## 小学生説明

巨大ファミレスのキッチン = 1人で全料理 → 混乱
   →
**寿司部・ラーメン部・パン部** に分業 → 各々の専門家、独立で動く

これがマイクロサービス。

---

## 1. モノリス vs マイクロサービス(15分)

| | モノリス | マイクロサービス |
|---|---|---|
| 開発 | 簡単(1リポジトリ) | 複雑(N個) |
| デプロイ | 全部一緒 | 個別 |
| スケール | 全部一緒 | 個別 |
| 障害 | 全停止 | 1個だけ |

→ **5人未満ならモノリス、50人超でマイクロ**が経験則。

---

## 2. 分割の指針(15分)

### Bounded Context(DDD)

業務の**境界**で分ける:
- ユーザー管理サービス
- 注文管理サービス
- 在庫管理サービス
- 決済サービス
- 通知サービス

→ 各サービスが**独立して進化**できる。

---

## 3. サービス間通信(40分)

### REST(同期)

```python
# サービスA から サービスB を呼ぶ
import httpx
async def get_user(user_id: int):
    async with httpx.AsyncClient() as client:
        r = await client.get(f"http://user-service/users/{user_id}")
        return r.json()
```

### gRPC(高速・型付き)

```bash
pip install grpcio grpcio-tools
```

`user.proto`:
```protobuf
syntax = "proto3";
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
}
message GetUserRequest { int32 id = 1; }
message User { int32 id = 1; string name = 2; }
```

```bash
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. user.proto
```

→ **REST の3-5倍速**、型安全。Go/Java とも繋がる。

### メッセージキュー(非同期)

#### Kafka

```bash
pip install kafka-python
```

```python
from kafka import KafkaProducer, KafkaConsumer
import json

# Publisher
p = KafkaProducer(bootstrap_servers="localhost:9092",
                  value_serializer=lambda v: json.dumps(v).encode())
p.send("orders", {"order_id": 1, "amount": 1000})

# Consumer(別サービス)
c = KafkaConsumer("orders", bootstrap_servers="localhost:9092",
                  value_deserializer=lambda v: json.loads(v.decode()))
for msg in c:
    print(msg.value)
```

→ **イベント駆動**で疎結合化。

#### RabbitMQ も同様。

---

## 4. Service Mesh(Istio)(15分)

「**サービス間通信の共通機能**を一括管理」

- 認証・認可
- ロードバランス
- 監視・トレース
- リトライ・サーキットブレーカー

→ Kubernetes + Istio でマイクロサービス本番運用。

---

## 5. 分散トランザクション(20分)

サービスまたいだ整合性が課題。

### Saga パターン

各サービスがイベントを送り合って**段階的に整合**。

```
1. 注文作成(Order Service) → OrderCreated イベント
2. 在庫減らす(Inventory Service) → 失敗 → CompensateOrder イベント
3. 注文取消(Order Service)
```

### Outbox パターン

DBに**イベントテーブル**を持つ → Kafka に publish。

---

## 6. 観測可能性(Observability)(20分)

### 3つの柱

1. **Logs**(構造化ログ)
2. **Metrics**(数値時系列、Prometheus)
3. **Traces**(リクエストの流れ追跡、Jaeger/OpenTelemetry)

```python
# OpenTelemetry
from opentelemetry import trace
tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("create_order"):
    with tracer.start_as_current_span("validate"):
        ...
    with tracer.start_as_current_span("save_db"):
        ...
```

→ サービスを跨いだ**1リクエストの足跡**が見える。

---

## 7. ⭐ 課題

クロードに頼む:
```
3つのマイクロサービスで「ECサイト」を実装。

サービス:
1. user-service(ユーザー管理、FastAPI + PostgreSQL)
2. order-service(注文、FastAPI + PostgreSQL)
3. notification-service(メール送信、FastAPI + Redis)

通信:
- user→order: gRPC(同期)
- order→notification: Kafka(非同期)

観測:
- OpenTelemetry でトレース
- Prometheus でメトリクス

docker-compose で全部起動。
```

---

## ✅ チェック

- [ ] モノリス vs MS の判断基準
- [ ] gRPC で型付き通信
- [ ] Kafka で非同期メッセージング
- [ ] Saga パターン理解
- [ ] OpenTelemetry でトレース
- [ ] **3サービスのマイクロサービス完成**

## 次
`36_DB内部.md`
