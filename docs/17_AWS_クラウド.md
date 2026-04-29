# Day 17:AWS・クラウド - 法人本番運用

## 💡 これは何?(小学生でもわかる説明)

****Amazon様の高性能サーバー**を時間借り**

### なぜ大事?
本番運用には必須。法人案件は90%クラウド

### イメージ
自分でサーバー買う(高額)vs **AWS = ホテル代**で借りる

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 この章後の月収:**+20〜50万円**(クラウド案件)
## ⏱ 所要:3-4時間

「**クラウド使えます**」は月100万への必須スキル。
中小企業のSaaS運用は90%がクラウド。

---

## ゴール

- AWS の基本サービスを使える
- EC2 でサーバー立てる
- S3 でファイル保存
- Lambda でサーバーレス関数
- RDS でマネージドDB
- 月10万案件を取れる状態

---

## 1. クラウド全体像(15分)

### 主要3社

| クラウド | シェア | 特徴 |
|---|---:|---|
| **AWS** | 30% | 案件最多、必修 |
| **Google Cloud** | 11% | AI連携強い |
| **Azure** | 24% | 法人(Microsoft連携) |

→ **AWS を最初に学ぶ**のが王道。

### 代替の安いやつ

| サービス | 用途 | 月額 |
|---|---|---:|
| **Render** | 簡単Webアプリ | 無料〜$7 |
| **Fly.io** | Docker全般 | 無料〜 |
| **Vercel** | フロント | 無料 |
| **Railway** | 簡単DB付き | $5〜 |
| **DigitalOcean** | VPS・Kubernetes | $4〜 |
| **Cloudflare Workers** | サーバーレス | 無料 |

→ 個人ならこっちで十分。**AWSは法人案件用**。

---

## 2. AWS 登録(20分)

1. https://aws.amazon.com/jp/
2. 「アカウント作成」
3. クレジットカード登録(従量課金、無料枠あり)
4. ログイン → コンソールへ

### ⚠️ 課金事故防止(必読)

- **Billing Alert**を設定($10超えで通知)
- 使い終わったリソースは**必ず削除**
- 無料枠の範囲を**毎月確認**

---

## 3. EC2 - 仮想サーバー(40分)

「**ネット越しに使えるパソコン**」。

### 立ち上げ手順

1. EC2 コンソール → 「インスタンス起動」
2. AMI(OS): Amazon Linux 2023
3. インスタンスタイプ: `t2.micro`(無料枠)
4. キーペア作成 → `.pem` ダウンロード
5. セキュリティグループ: 22(SSH)+ 80(HTTP)+ 443(HTTPS)を開放
6. 起動

### 接続

```bash
chmod 400 my-key.pem
ssh -i my-key.pem ec2-user@<パブリックIP>
```

### Pythonアプリ動かす

```bash
sudo yum install -y python3 python3-pip git
git clone https://github.com/xxx/myapp
cd myapp
pip3 install -r requirements.txt
nohup python3 app.py &
```

→ **これで世界中から自分のアプリが見れる**。

### コスト

- t2.micro: 月750時間まで無料(1年間)
- 24時間稼働で月**0円**(初年度)、以降月$10程度

---

## 4. S3 - ファイル保管庫(30分)

「**容量無制限のファイル置き場**」。SaaSのファイルアップロードに必須。

### 使い方(Pythonから)

```bash
pip install boto3
```

```python
import boto3

s3 = boto3.client("s3")

# アップロード
s3.upload_file("local.png", "my-bucket", "uploads/local.png")

# ダウンロード
s3.download_file("my-bucket", "uploads/local.png", "downloaded.png")

# 一覧
response = s3.list_objects_v2(Bucket="my-bucket")
for obj in response.get("Contents", []):
    print(obj["Key"])

# 削除
s3.delete_object(Bucket="my-bucket", Key="uploads/local.png")

# 一時公開URL(プレサインドURL、1時間有効)
url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": "my-bucket", "Key": "uploads/local.png"},
    ExpiresIn=3600
)
```

→ Streamlit のファイルアップロード先 → S3 が定番。

### コスト

- 5GB まで無料(1年)
- 以降 $0.023/GB/月(超安い)

---

## 5. Lambda - サーバーレス関数(40分)

「**コードだけアップ → 必要な時だけ動く**」。

### なぜすごい?

- サーバー管理不要
- 月100万リクエストまで**無料**
- 使った分だけ課金

### ユースケース

- 定期実行(毎朝6時にデータ取得)
- Webhook 受け口
- 画像リサイズ
- API のバックエンド

### 例:毎朝Slack通知

```python
# lambda_function.py
import json
import requests
import os

def lambda_handler(event, context):
    weather = requests.get("https://api.weather.com/...").json()
    
    requests.post(
        os.environ["SLACK_WEBHOOK"],
        json={"text": f"今日の天気: {weather['summary']}"}
    )
    return {"statusCode": 200}
```

### デプロイ手順

1. AWS Lambda コンソール → 「関数作成」
2. ランタイム: Python 3.12
3. コード貼り付け
4. 環境変数: `SLACK_WEBHOOK=xxx`
5. EventBridge で「毎朝6時」トリガー設定

→ **これで月10万円のクライアント案件**(中小企業の自動化)。

---

## 6. RDS - マネージドDB(30分)

「**AWSが管理してくれるPostgreSQL/MySQL**」。

### なぜ?

- バックアップ自動
- アップデート自動
- 高可用性

### 使い方

1. RDS コンソール → 「データベース作成」
2. PostgreSQL選択
3. インスタンス: `db.t3.micro`(無料枠)
4. 認証情報設定
5. パブリックアクセス: 開発時はON

### Pythonから接続

```python
from sqlalchemy import create_engine

DB_HOST = "your-db.xxx.rds.amazonaws.com"
DB_USER = "postgres"
DB_PASS = os.getenv("DB_PASS")

engine = create_engine(f"postgresql://{DB_USER}:{DB_PASS}@{DB_HOST}:5432/mydb")
```

### コスト

- db.t3.micro: 750時間/月 無料(1年)
- 以降月 $15-20

---

## 7. CloudFront - CDN(15分)

「**世界中のユーザーに高速配信**」。

S3に画像置く → CloudFront 経由 → 世界中の人が爆速ダウンロード。

```
S3 → CloudFront → ユーザー
        (キャッシュ)
```

→ 静的サイト(GitHub Pages 代替)にも使える。

---

## 8. ECS / Fargate - コンテナ運用(20分)

Docker コンテナを AWS で動かす。

### Fargate(サーバーレスコンテナ、楽)

1. ECS クラスター作成
2. タスク定義(Dockerイメージ指定)
3. サービス起動

→ コンテナがオートスケールで動く。

### ECR(イメージ置き場)

```bash
# AWS CLI で push
aws ecr get-login-password | docker login --username AWS --password-stdin <account>.dkr.ecr.region.amazonaws.com
docker build -t myapp .
docker tag myapp:latest <account>.dkr.ecr.region.amazonaws.com/myapp:latest
docker push <account>.dkr.ecr.region.amazonaws.com/myapp:latest
```

→ Day 16 で作った Docker イメージをそのまま運用。

---

## 9. IAM - 権限管理(20分)

「**誰が何できるか**を細かく決める**」。法人案件でめちゃ大事。

### 原則

- ❌ ルートアカウントで普段作業しない
- ✅ IAM ユーザー作って必要な権限だけ
- ✅ アクセスキーは**.envに入れる、Gitに上げない**

### 作り方

1. IAM コンソール → 「ユーザー作成」
2. 必要なポリシー添付(例: `AmazonS3FullAccess`)
3. アクセスキー発行
4. Pythonから:

```python
import boto3
s3 = boto3.client("s3")  # ~/.aws/credentials から自動読み込み
```

### MFA(2段階認証)を必ず設定

→ アカウント乗っ取り防止。

---

## 10. CloudWatch - 監視・ログ(15分)

```python
import boto3

logs = boto3.client("logs")

# ログ送信
logs.put_log_events(
    logGroupName="/aws/lambda/myfunc",
    logStreamName="2026-04-29",
    logEvents=[{"timestamp": int(time.time()*1000), "message": "処理開始"}]
)
```

### アラート設定

- CPU 80%超 → Slack通知
- エラー発生 → メール通知
- 予算超過 → SMS

→ **本番運用には必須**。

---

## 11. 安いクラウド(個人/小規模)

### Render を本気で使う(60分)

実は AWS じゃなくても **Render で月100万案件**取れる。

```yaml
# render.yaml
services:
  - type: web
    name: my-saas
    env: docker
    plan: starter  # $7/月
    dockerfilePath: ./Dockerfile
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: my-db
          property: connectionString

databases:
  - name: my-db
    plan: starter  # $7/月
```

→ **月$14で本番運用**。クライアントには「クラウドネイティブ構成」と説明できる。

---

## 12. ⭐ 売れる成果物 #9:本番運用SaaS(120分)

クロードに頼む:

```
Day 8 で作った SaaS を AWS で本番運用する設計書を書いて。

構成:
- ECS Fargate(コンテナ運用)
- ALB(ロードバランサ)
- RDS PostgreSQL(本番DB)
- S3(ファイル保存)
- CloudFront(CDN)
- Route 53(独自ドメイン)
- ACM(SSL証明書)
- CloudWatch(監視)
- Secrets Manager(機密情報)

各サービスのTerraform コードも生成して。
月額コスト試算も含める。
```

→ クロードが**Infrastructure as Code**で生成。
→ クライアントに「**月100万円で本番運用構築**」と提案できる。

---

## 13. 認定資格(おまけ)

| 資格 | 難易度 | 効果 |
|---|---|---|
| AWS Certified Cloud Practitioner | 易 | 入門証明 |
| Solutions Architect Associate | 中 | **本気で月100万狙うなら必須** |
| Solutions Architect Professional | 難 | 月150万圏 |

→ 単価2-3割UP、SES案件の単価交渉に使える。

---

## ✅ チェック

- [ ] AWS アカウント作成・MFA設定
- [ ] EC2 インスタンス起動・Pythonアプリ動かした
- [ ] S3 にファイル UP/DOWN できる
- [ ] Lambda で関数作って動かせた
- [ ] RDS PostgreSQL に Python から接続
- [ ] IAM 理解、ルートアカウント使わない
- [ ] Billing Alert 設定済み
- [ ] **本番運用SaaS の設計書ができた**

---

## 進捗

```
Day 17: AWS・クラウド完了。EC2/S3/Lambda/RDS/Fargate触った。
本番運用設計書完成。
```

## 次

`18_テスト_品質.md` へ。**プロが必ず書く品質保証**。
