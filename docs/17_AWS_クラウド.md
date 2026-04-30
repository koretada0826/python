# Day 17:AWS・クラウド - 法人本番運用

## 💡 これは何?(小学生でもわかる説明)

**Amazonの世界一強いコンピュータを「ホテル」みたいに時間借りして本番アプリを動かす日**

### なぜ大事?
本番運用には必須。**法人案件は90%クラウド**。
自分のPCでアプリ動かしっぱなし=ダサい・遅い・落ちる。
AWS使えると単価が一気に+20-50万円。

### イメージ
- **AWS**=「**世界最高級のホテル**」(部屋・食堂・倉庫・警備が全部揃ってる)
- **EC2**=「**ホテルの1部屋**」(=自分専用の仮想サーバ)
- **S3**=「**ホテルの倉庫**」(=容量無制限のファイル置き場)
- **Lambda**=「**呼び出しベルで来てくれる仲居さん**」(=必要な時だけ動く関数)
- **RDS**=「**ホテル管理のDB**」(=バックアップも全部やってくれるPostgreSQL)
- **IAM**=「**鍵の発行係**」(=誰がどこに入れるか管理)
- **CloudWatch**=「**監視カメラ**」(=何かあったら通知)

### Claude Code に何を頼めばいいか
EC2 起動・S3 操作・Lambda 関数・Terraform 設計書は全部クロード。
あなたは「**何のサービスを作るか**」を伝えるだけ。

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

## 1. クラウド全体像(=世界のホテル業界マップ)(15分)

### 1-1. なぜクラウド?

> **昔:** 自分でサーバーを買う(数十万円)→ 設置場所・電気・冷却が必要
> **今:** AWS で時間借り(月数千円〜)→ 必要な分だけ使える
>
> 「**自分でホテル建てる vs ホテルに泊まる**」と同じ違い。
> 個人や中小企業は**100%クラウド**で正解。

### 1-2. 主要3社

| クラウド | シェア | 特徴 |
|---|---:|---|
| **AWS** | 30% | 案件最多、必修 |
| **Google Cloud** | 11% | AI連携強い |
| **Azure** | 24% | 法人(Microsoft連携) |

→ **AWS を最初に学ぶ**のが王道。

### 1-3. 代替の安いやつ(=ビジネスホテル)

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

### 2-1. ⚠️ 課金事故防止(=破産しないために必読)

| 防御策 | やり方 |
|---|---|
| **Billing Alert** を設定 | $10超えで通知メール |
| 使い終わったリソースを必ず削除 | EC2 / RDSは止めるだけでなく**Terminate** |
| 無料枠の範囲を毎月確認 | コンソールの「Billing」で残り無料枠表示 |
| MFA(2段階認証)を有効化 | 乗っ取り防止 |

> **怖い話:** 学習者の事例で「EC2 を立てたまま放置 → 1ヶ月で15万円請求」が頻発。
> Billing Alert を**最初の30分以内に必ず設定**。

---

## 3. EC2 - 仮想サーバー(=ネット越しの自分の部屋)(40分)

### 3-1. 何ができる?

「**ネット越しに使えるパソコン**」。Linuxマシンが立ち上がる。

### 3-2. 立ち上げ手順

1. EC2 コンソール → 「インスタンス起動」
2. AMI(OS): Amazon Linux 2023
3. インスタンスタイプ: `t2.micro`(無料枠)
4. キーペア作成 → `.pem` ダウンロード
5. セキュリティグループ: 22(SSH)+ 80(HTTP)+ 443(HTTPS)を開放
6. 起動

### 3-3. 接続

```bash
chmod 400 my-key.pem  # 鍵ファイルを自分だけ読める権限に
ssh -i my-key.pem ec2-user@<パブリックIP>
```

### 3-4. Pythonアプリ動かす

```bash
sudo yum install -y python3 python3-pip git
git clone https://github.com/xxx/myapp
cd myapp
pip3 install -r requirements.txt
nohup python3 app.py &  # nohup で SSH 切断後も動き続ける
```

→ **これで世界中から自分のアプリが見れる**。

### 3-5. コスト

- t2.micro: 月750時間まで無料(1年間)
- 24時間稼働で月**0円**(初年度)、以降月$10程度

> **学習用途なら無料枠で十分**。本気の本番運用は次のRDS / Lambdaへ。

---

## 4. S3 - ファイル保管庫(=容量無制限の倉庫)(30分)

### 4-1. なぜS3?

> **EC2のディスクは小さい(8〜30GB)**。画像や動画はすぐパンクする。
> S3は**容量無制限**、しかも安い($0.023/GB/月)。
> SaaS のファイルアップロード先 = S3 が定番。

### 4-2. 使い方(Pythonから)

```bash
pip install boto3  # AWS SDK for Python
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

### 4-3. コスト

- 5GB まで無料(1年)
- 以降 $0.023/GB/月(超安い)

---

## 5. Lambda - サーバーレス関数(=呼び出しベルで来る仲居さん)(40分)

### 5-1. なぜすごい?

「**コードだけアップ → 必要な時だけ動く**」。

| 項目 | EC2 | Lambda |
|---|---|---|
| 課金 | 24時間稼働分 | 実行時間だけ |
| サーバー管理 | 自分でやる | 不要 |
| 月100万実行 | $10 | **無料** |

> **イメージ:**
> EC2 = ホテルに常駐する執事(24時間給料発生)
> Lambda = ベル鳴らした時だけ来る仲居さん(来た時だけ給料)

### 5-2. ユースケース

- 定期実行(毎朝6時にデータ取得)
- Webhook 受け口
- 画像リサイズ
- API のバックエンド

### 5-3. 例:毎朝Slack通知

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

### 5-4. デプロイ手順

1. AWS Lambda コンソール → 「関数作成」
2. ランタイム: Python 3.12
3. コード貼り付け
4. 環境変数: `SLACK_WEBHOOK=xxx`
5. EventBridge で「毎朝6時」トリガー設定

→ **これで月10万円のクライアント案件**(中小企業の自動化)。

---

## 6. RDS - マネージドDB(=ホテルが管理してくれるPostgreSQL)(30分)

### 6-1. なぜ?

> **PostgreSQL を自分で運用すると面倒**。
> - バックアップ忘れ → 障害でデータ消失
> - アップデート → サーバー停止が必要
> - 高可用性 → 自前で構成するの大変
>
> **RDS は全部 AWS がやってくれる**。

### 6-2. 主な特徴

- バックアップ自動
- アップデート自動
- 高可用性
- スナップショット復元

### 6-3. 使い方

1. RDS コンソール → 「データベース作成」
2. PostgreSQL選択
3. インスタンス: `db.t3.micro`(無料枠)
4. 認証情報設定
5. パブリックアクセス: 開発時はON

### 6-4. Pythonから接続

```python
from sqlalchemy import create_engine

DB_HOST = "your-db.xxx.rds.amazonaws.com"
DB_USER = "postgres"
DB_PASS = os.getenv("DB_PASS")

engine = create_engine(f"postgresql://{DB_USER}:{DB_PASS}@{DB_HOST}:5432/mydb")
```

### 6-5. コスト

- db.t3.micro: 750時間/月 無料(1年)
- 以降月 $15-20

---

## 7. CloudFront - CDN(=世界中の支店)(15分)

### 7-1. なぜ?

「**世界中のユーザーに高速配信**」。

> **問題:** S3が東京リージョンなら、アメリカからのアクセスは遅い。
> **解決:** CloudFront が**世界中の支店(エッジ)**にキャッシュ → どこからでも爆速。

```
S3 → CloudFront → ユーザー
        (キャッシュ)
```

→ 静的サイト(GitHub Pages 代替)にも使える。

---

## 8. ECS / Fargate - コンテナ運用(=ホテルにDockerコンテナを並べる)(20分)

### 8-1. Fargate(サーバーレスコンテナ、楽)

Day 16 で作った Docker コンテナを AWS で動かす。

1. ECS クラスター作成
2. タスク定義(Dockerイメージ指定)
3. サービス起動

→ コンテナがオートスケールで動く。

### 8-2. ECR(イメージ置き場)

```bash
# AWS CLI で push
aws ecr get-login-password | docker login --username AWS --password-stdin <account>.dkr.ecr.region.amazonaws.com
docker build -t myapp .
docker tag myapp:latest <account>.dkr.ecr.region.amazonaws.com/myapp:latest
docker push <account>.dkr.ecr.region.amazonaws.com/myapp:latest
```

→ Day 16 で作った Docker イメージをそのまま運用。

---

## 9. IAM - 権限管理(=鍵の発行係)(20分)

### 9-1. なぜ重要?

「**誰が何できるか**を細かく決める」。法人案件でめちゃ大事。

> **悪い例:** ルートアカウント(全権限)で開発 → 漏洩で全リソース消滅。
> **良い例:** IAM ユーザーに「S3だけ読み書きOK」を付与 → 漏洩しても被害最小。

### 9-2. 原則

- ❌ ルートアカウントで普段作業しない
- ✅ IAM ユーザー作って必要な権限だけ
- ✅ アクセスキーは**.envに入れる、Gitに上げない**
- ✅ MFA(2段階認証)を必ず設定

### 9-3. 作り方

1. IAM コンソール → 「ユーザー作成」
2. 必要なポリシー添付(例: `AmazonS3FullAccess`)
3. アクセスキー発行
4. Pythonから:

```python
import boto3
s3 = boto3.client("s3")  # ~/.aws/credentials から自動読み込み
```

### よくある間違い

| ミス | 何が起こる | 直し方 |
|---|---|---|
| アクセスキーを Git に push | 数分で発見・悪用 | `.env` + `.gitignore` 必須 |
| ルートアカウントを毎日使う | 漏洩で全部破壊 | IAM ユーザーを作って使う |
| 権限を `*` で全許可 | 1箇所漏れたら全権限漏洩 | 最小権限の原則 |

---

## 10. CloudWatch - 監視・ログ(=監視カメラ)(15分)

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

### 10-1. アラート設定

- CPU 80%超 → Slack通知
- エラー発生 → メール通知
- 予算超過 → SMS

→ **本番運用には必須**。

---

## 11. 安いクラウド(個人/小規模)(=AWS縛りでなくてOK)

### 11-1. Render を本気で使う(60分)

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

## 13. 認定資格(おまけ)(=資格で単価UP)

| 資格 | 難易度 | 効果 |
|---|---|---|
| AWS Certified Cloud Practitioner | 易 | 入門証明 |
| Solutions Architect Associate | 中 | **本気で月100万狙うなら必須** |
| Solutions Architect Professional | 難 | 月150万圏 |

→ 単価2-3割UP、SES案件の単価交渉に使える。

---

## 14. 今日学んだ用語まとめ

| 用語 | 一言で |
|---|---|
| AWS | 世界最大のクラウド(=高級ホテル) |
| EC2 | 仮想サーバー(=自分専用の部屋) |
| S3 | 容量無制限のファイル保管庫 |
| Lambda | 必要な時だけ動く関数(=サーバーレス) |
| RDS | マネージドDB(=自動バックアップ付PostgreSQL) |
| CloudFront | 世界配信用CDN(=世界中の支店) |
| ECS | Docker コンテナ運用サービス |
| Fargate | サーバーレスコンテナ実行 |
| ECR | Dockerイメージ置き場(=AWS版Docker Hub) |
| IAM | 権限管理(=鍵の発行係) |
| CloudWatch | 監視・ログ収集 |
| Billing Alert | 課金通知(=破産防止) |
| MFA | 2段階認証(=乗っ取り防止) |
| Terraform | インフラをコードで記述するツール |
| AMI | EC2 用OSテンプレ |
| セキュリティグループ | EC2 のファイアウォール設定 |
| Render | AWSより簡単な代替クラウド |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **AWS 環境構築 / インフラ設計案件**:単価 30〜100万円(プラットフォーム例:ランサーズ/レバテック/SES)
- **既存オンプレ → AWS 移行**:単価 50〜150万円(EC2 / RDS / S3 / Lambda 構成)
- **サーバーレス基盤(Lambda + API Gateway + DynamoDB)構築**:単価 30〜80万円

### クロードへの頼み方(営業文を書かせる例)
```
SES エージェント向けに、AWS 寄りのスキルシート追記文を書いて。
Day 17 までで習得:Python / Docker / AWS(EC2 / S3 / RDS / Lambda / IAM)
稼働形態:月稼働 100〜140時間 / 月70〜90万円希望
強み:アプリ実装と AWS インフラ両方触れる(フルスタック寄り)
過去実績:Streamlit / FastAPI を EC2 にデプロイ、S3 を使ったバッチ実装、Lambda で定期実行
資格:無し(取得予定:SAA)
ポイント:小規模スタートアップで「1人インフラ係」を任せられるレベル
```

### この章だけでは足りないもの(次に学ぶべき)
- テスト品質(Day 18) / API設計(Day 19)で本番品質UP
- セキュリティ(Day 20) / 監査ログ
- Kubernetes / IaC(Day 30)で大規模本番運用へ

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
