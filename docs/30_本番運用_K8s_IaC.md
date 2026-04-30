# Day 30:本番運用・Kubernetes・IaC - 月150万到達

## 💡 これは何?(小学生でもわかる説明)

**99.9%稼働するシステムを作って運用する**

### なぜ大事?
ここまで学んだスキルで「**動くもの**」は作れる。
でも、本物の仕事は「**24時間365日落ちないシステム**」。
ここを支える職人=SRE/DevOps。月単価150-300万円。

### イメージ
- **Kubernetes(K8s)**=「**指揮者**」(複数のサーバーを統率して、1つのオーケストラのように動かす)
- **Pod**=「**1人の演奏者**」(コンテナを動かす最小単位)
- **Deployment**=「**演奏者の配置計画**」(常時3人配置、1人倒れたら補充)
- **Service**=「**ステージへの入口**」(外からどう演奏者にアクセスするか)
- **Ingress**=「**会場の受付**」(URLで適切な演奏者へ案内)
- **Helm**=「**演奏会の設定テンプレ**」(よくある構成を一発デプロイ)
- **Terraform**=「**インフラの設計図をコードで書く**」(AWSリソースをファイルで管理)
- **IaC**=「**Infrastructure as Code**」(インフラをGit管理できる)
- **Prometheus**=「**心拍計**」(システムの生死を常時監視)
- **Grafana**=「**心拍計の画面**」(かっこいいダッシュボード)
- **SLO**=「**99.9%稼働を約束する目標**」
- **ブルーグリーンデプロイ**=「**新旧の2台を並行で持ち、瞬時に切替**」

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 この章後の月収:**+50〜200万円**(SRE/DevOps案件)
## ⏱ 所要:4時間

「**作る**」から「**動かし続ける**」へ。
本番環境を**99.9%稼働**させる技術が、SRE/DevOps領域。**月単価150-300万円**。

> **なぜ最高単価?**
> 開発者は山ほどいる。でも「本番を24時間支える人」は希少。
> 1分のダウン = 数百万の損失、という会社にとって、SRE は**保険**であり**命綱**。

---

## ゴール

- Kubernetes 基本(Pod/Service/Deployment)
- Terraform で Infrastructure as Code
- 監視・ログ集約・SLO
- ブルーグリーン・カナリアデプロイ
- 障害対応・ポストモーテム

---

## 1. なぜ Kubernetes?(15分)

### 1-1. 単一サーバーの限界

| 問題 | 単一サーバー時の症状 |
|---|---|
| 障害時 | サービス全停止 |
| スケール | 手動で足す(時間かかる) |
| デプロイ | 1台ずつ更新(=ダウンタイム発生) |

### 1-2. Kubernetes 解決

| 機能 | 効果 | たとえ |
|---|---|---|
| **複数ノードで冗長化** | 1台死んでもOK | 飛行機のエンジンが2基ある |
| **オートスケール** | 負荷増で自動増設 | 客が増えたら店員自動補充 |
| **ローリングアップデート** | 無停止デプロイ | 客いる間に厨房を入れ替え |
| **セルフヒーリング** | 落ちたコンテナを自動再起動 | 倒れた店員を自動で蘇生 |

→ **クラウド時代のデファクト**。

---

## 2. Docker → K8s への流れ(10分)

```
[Docker] = 1コンテナを動かす
   ↓
[Docker Compose] = 複数コンテナを1ホストで
   ↓
[Kubernetes] = 複数コンテナを複数ホストで
```

> **イメージ:**
> - Docker = 1人で家事
> - Compose = 家族で家事分担
> - K8s = 大企業の総務部(複数オフィス管理)

---

## 3. ローカル K8s 環境(20分)

```bash
# Docker Desktop の Kubernetes を有効化
# Settings → Kubernetes → Enable Kubernetes

# 確認
kubectl version
kubectl get nodes
```

代替:
- **kind**(Kubernetes in Docker)
- **minikube**

---

## 4. Pod / Deployment / Service(40分)

### 4-1. Pod(最小実行単位)

> **Pod とは?**
> 1つ以上のコンテナをまとめた最小単位。普通は1Pod=1コンテナ。

`pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: python:3.12-slim
      command: ["python", "-m", "http.server", "8000"]
      ports:
        - containerPort: 8000
```

```bash
kubectl apply -f pod.yaml      # 反映
kubectl get pods               # 一覧
kubectl logs my-app            # ログ
kubectl exec -it my-app -- /bin/bash  # 入って中で操作
kubectl delete pod my-app      # 削除
```

### 4-2. Deployment(冗長化・更新)

> **Deployment とは?**
> 「**Pod を何個・どう動かすか**」の管理者。落ちたら自動再起動。

`deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3  # 3つのPodを常時稼働
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: my-app:1.0
          resources:
            limits:
              memory: 512Mi
              cpu: 500m
          # 死活監視:応答なければ再起動
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
          # 受付OKチェック:準備完了まで負荷分散から除外
          readinessProbe:
            httpGet:
              path: /ready
              port: 8000
```

→ **3Pod 常時稼働、1個落ちても自動復旧**。

### 4-3. Service(外部公開)

> **Service とは?**
> 複数のPodへの「**入り口**」。Pod が増減しても URL は変わらない。

`service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app  # この Label のPodへ転送
  ports:
    - port: 80
      targetPort: 8000
  type: LoadBalancer  # 負荷分散つき
```

→ **LBで負荷分散**。

---

## 5. ConfigMap / Secret(20分)

> **ConfigMap**=「設定値の置き場」(普通の設定)
> **Secret**=「機密の置き場」(APIキー・DBパスワードなど)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_URL: "postgresql://..."
  LOG_LEVEL: "info"
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  API_KEY: "sk-xxxxx"
```

```yaml
# Deployment 側で参照
env:
  - name: DATABASE_URL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: DATABASE_URL
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: API_KEY
```

> **なぜ分ける?**
> Secret は base64 + RBAC で保護。コードに秘密を直書きしない=漏洩防止。

---

## 6. Ingress(URLルーティング)(20分)

> **Ingress とは?**
> ドメインで複数サービスを振り分ける受付。SSL対応も。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt  # 自動でSSL証明書取得
spec:
  tls:
    - hosts: [api.example.com]
      secretName: tls-cert
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-svc
                port:
                  number: 80
```

→ **ドメインで複数サービスを振り分け**+SSL自動。

---

## 7. オートスケーリング(HPA)(15分)

> **HPA とは?**
> Horizontal Pod Autoscaler。負荷に応じてPodを自動増減。

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2     # 最低2Pod
  maxReplicas: 20    # 最大20Pod
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70  # CPU 70%超えたら増設
```

→ **CPU 70% 超えたら自動増設**(最大20Pod)。

---

## 8. Helm(K8s パッケージ管理)(20分)

> **Helm とは?**
> K8s 設定の「**テンプレート集**」。よく使う構成を1コマンドでインストール。
> Pythonでいう pip。

```bash
brew install helm
helm create my-chart
```

`my-chart/values.yaml`:
```yaml
image:
  repository: my-app
  tag: "1.0"
replicaCount: 3
ingress:
  enabled: true
  host: api.example.com
```

```bash
helm install my-app ./my-chart                      # インストール
helm upgrade my-app ./my-chart --set image.tag=1.1  # 更新
helm rollback my-app 1                              # 元に戻す
```

→ **K8s 設定をテンプレート化**。プロは Helm Chart で管理。

---

## 9. Terraform - Infrastructure as Code(40分)

「**インフラをコードで管理**」。AWS/GCP/Azure 全部対応。

> **イメージ:** GUI で AWS をポチポチクリックする代わりに、
> ファイルにインフラ構成を書いて `terraform apply` で一括作成。
> Git管理できる=人為ミス減、再現可能。

### 9-1. インストール

```bash
brew install terraform
```

### 9-2. 例:AWS S3 + CloudFront

`main.tf`:
```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" {
  region = "ap-northeast-1"
}

resource "aws_s3_bucket" "site" {
  bucket = "my-static-site"
}

resource "aws_s3_bucket_website_configuration" "site" {
  bucket = aws_s3_bucket.site.id
  index_document { suffix = "index.html" }
}

resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = aws_s3_bucket.site.bucket_regional_domain_name
    origin_id   = "S3-${aws_s3_bucket.site.id}"
  }
  enabled             = true
  default_root_object = "index.html"

  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.site.id}"
    viewer_protocol_policy = "redirect-to-https"
    forwarded_values {
      query_string = false
      cookies { forward = "none" }
    }
  }

  restrictions {
    geo_restriction { restriction_type = "none" }
  }

  viewer_certificate { cloudfront_default_certificate = true }
}

output "url" {
  value = aws_cloudfront_distribution.cdn.domain_name
}
```

```bash
terraform init       # 初期化
terraform plan       # 何が作られるか確認(=シミュレーション)
terraform apply      # 実行
terraform destroy    # 全削除
```

→ **インフラ全部コード化**。Git管理で再現可能。

> **なぜ IaC?**
> - 手動構築 = 「太郎さんしか分からない設定」になりがち
> - IaC = ファイル見れば誰でも分かる、Git で履歴管理

---

## 10. 監視・ログ(40分)

### 10-1. Prometheus + Grafana(K8s標準)

> **Prometheus**=「全Podから定期的に状態を集める収集役」
> **Grafana**=「集めた数字をかっこいいグラフで見せる役」

```bash
# Prometheus + Grafana を Helm でインストール
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prom prometheus-community/kube-prometheus-stack
```

→ **CPU/メモリ/HTTP応答時間**を自動収集 → Grafana で可視化。

### 10-2. Python アプリのメトリクス

```bash
pip install prometheus-fastapi-instrumentator
```

```python
from fastapi import FastAPI
from prometheus_fastapi_instrumentator import Instrumentator

app = FastAPI()
Instrumentator().instrument(app).expose(app)
# → /metrics エンドポイント自動公開
```

### 10-3. ログ集約 - Loki(おすすめ)

または **ELK(Elasticsearch + Logstash + Kibana)**。

> **なぜ集約?**
> 100Podある環境で、Pod ごとにログ見るのは無理。
> 1ヶ所に集めて検索できると神。

### 10-4. エラー追跡 - Sentry(Day 18 で既出)

---

## 11. SLO / エラーバジェット(15分)

「**99.9%稼働**を保証する」契約レベル。

### 11-1. 計算

- SLO: 99.9% 稼働 → 月 43分のダウンタイム許容
- エラーバジェット使い切ったら → **新機能リリース停止、改善優先**

| SLO | 月の許容ダウンタイム |
|---|---|
| 99% | 7時間 |
| 99.9% | 43分 |
| 99.99% | 4分 |

### 11-2. SLI(指標)

- 可用性:成功リクエスト / 全リクエスト
- レイテンシ:p95 < 500ms
- エラー率:5xx < 0.1%

> **エラーバジェットの考え方:**
> 「100%目指すと開発が止まる」。
> 「あと43分まで落ちてOK」と決めると、新機能リリースを攻められる。

---

## 12. CI/CD パイプライン(30分)

`.github/workflows/deploy.yml`:
```yaml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install -r requirements.txt
      - run: pytest
      - run: ruff check .
      - run: mypy .

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t my-app:${{ github.sha }} .
      - run: docker push registry.example.com/my-app:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: azure/k8s-deploy@v5
        with:
          manifests: k8s/
          images: my-app:${{ github.sha }}
```

→ **push → 自動テスト → ビルド → 本番反映**。

---

## 13. ブルーグリーン・カナリアデプロイ(20分)

### 13-1. ブルーグリーン

> **イメージ:** 同じ家を2軒持つ。1軒に住みながら、もう1軒を改装。
> 改装完了したら住所を切り替え。問題あれば元に戻す。

```
[ロードバランサ]
  ├ Blue(現行・100%)
  └ Green(新版・0%)
       ↓ 切替
  ├ Blue(0%)
  └ Green(100%)
```

→ **問題あれば即ロールバック**。

### 13-2. カナリアリリース

> **由来:** 昔、炭鉱でカナリアを連れて入った(危険ガスを先に検知)。
> 新版をまず一部のユーザーに見せて、問題ないか確認しながら拡大。

```
新版に 1% → 10% → 50% → 100% と段階的に流す
途中でエラー率上がったら停止
```

### 13-3. Argo Rollouts でカナリア自動化

```yaml
strategy:
  canary:
    steps:
      - setWeight: 10
      - pause: { duration: 10m }
      - setWeight: 50
      - pause: { duration: 30m }
      - setWeight: 100
```

---

## 14. 障害対応・ポストモーテム(15分)

### 14-1. 障害発生時

1. **検知**(Prometheus アラート → Slack)
2. **影響範囲確認**(Grafana ダッシュボード)
3. **緊急対応**(ロールバック / スケールアップ)
4. **根本原因調査**
5. **恒久対応**

### 14-2. ポストモーテム(振り返り)

```markdown
## インシデント: API 5xx 急増
- 日時: 2026-04-30 14:00 - 14:35
- 影響: ユーザー約 5000人にエラー画面
- 原因: DB接続プール枯渇
- 解決: プールサイズ増 + コネクション漏れ修正
- 再発防止: 接続数アラート追加
- 学び: 個別実装でなくコネクション管理を共通化
```

→ **責任追及じゃなく仕組み改善**にフォーカス。

> **Blameless Postmortem(無責任追及振り返り):**
> 「誰が悪い」ではなく「**仕組みのどこが悪い**」を議論する文化。
> 個人を責めると次から隠蔽される=問題悪化。

---

## 15. ⭐ 売れる成果物 #16:Production級 K8s + IaC プロジェクト(120分)

クロードに頼む:

```
Production レベルの完全なシステムを Terraform + Kubernetes で構築する設計&コードを作って。

要件:
- AWS EKS で K8s クラスター
- VPC/Subnet/ALB/Route53 の設定込み
- アプリ:FastAPI(Day 19のTODO API使用)
- DB:RDS PostgreSQL(Multi-AZ)
- Cache:ElastiCache Redis
- Object Storage:S3
- CDN:CloudFront
- Secrets:AWS Secrets Manager
- 監視:Prometheus + Grafana(Helm)
- ログ:Loki
- CI/CD:GitHub Actions

成果物:
- terraform/ ディレクトリ(modules/, environments/dev,prod)
- k8s/ ディレクトリ(Helm Chart)
- .github/workflows/(test/build/deploy)
- README に手順書
- アーキテクチャ図(Mermaid)
- 月額コスト試算

これを「中堅 SaaS 企業の本番」想定で構築。
```

→ **これで月単価200-300万円のSRE案件**が取れる。

---

## 16. よくある間違い・エラー(15分)

| 状況 | 間違いの原因 | 直し方 |
|---|---|---|
| Pod が CrashLoopBackOff | アプリ側エラー | `kubectl logs` で確認、修正 |
| Service にアクセスできない | Label セレクタ不一致 | Pod と Service の Label を合わせる |
| Secret をコードに直書き | 漏洩リスク | Secret リソースまたはAWS Secrets Manager |
| HPA で増えすぎ | maxReplicas 上限なし | maxReplicas 設定必須 |
| Terraform で本番破壊 | 適当に apply | plan で必ず確認、本番は別 state |
| `terraform apply` で動作変更 | tfstate がローカル | S3 + DynamoDB でロック付き共有 |
| デプロイで全Pod同時更新 | rolling 設定なし | maxUnavailable / maxSurge 設定 |
| ログ消える | 永続化なし | Loki / S3 へ転送 |
| 監視アラートだけ | 復旧手順がない | Runbook を整備 |
| ポストモーテムで人責める | 心理的安全性低い | Blameless 文化を浸透 |

---

## 17. 用語まとめ

| 用語 | 一言で |
|---|---|
| Kubernetes(K8s) | コンテナオーケストレーター |
| Pod | 最小実行単位(=コンテナのまとまり) |
| Deployment | Pod 管理者 |
| Service | Pod への入口 |
| Ingress | URL ルーティング |
| ConfigMap | 設定値置き場 |
| Secret | 機密情報置き場 |
| HPA | Pod 自動増減 |
| Helm | K8s パッケージ管理 |
| IaC | Infrastructure as Code |
| Terraform | IaC の代表ツール |
| tfstate | Terraform の状態ファイル |
| Prometheus | メトリクス収集 |
| Grafana | 可視化ダッシュボード |
| Loki | ログ集約 |
| SLO | サービス品質目標 |
| SLI | SLO 測定指標 |
| エラーバジェット | 許容ダウンタイム枠 |
| ブルーグリーン | 2環境切替デプロイ |
| カナリア | 段階リリース |
| ロールバック | 元バージョンに戻す |
| ポストモーテム | 障害振り返り |
| Blameless | 個人を責めない文化 |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **K8s / IaC を活用した SRE 案件(SES)**:月単価 80〜150万円(プラットフォーム例:レバテック/Findy Freelance/直契約)
- **インフラ構築・移行案件(オンプレ → K8s)**:単価 100〜500万円
- **Terraform による IaC 化・運用自動化**:単価 50〜200万円

### クロードへの頼み方(営業文を書かせる例)
```
SRE / インフラエンジニア向け案件への提案文を書いて。
Day 30 までで習得:Docker / K8s / Helm / Terraform / GitHub Actions / Prometheus / Grafana
稼働:月140時間 / 月100〜130万円希望
売り:アプリ実装(Python / FastAPI)も書けるので、開発チームと密に連携できるSRE
過去実績:GitHubに「中規模SaaS本番想定」のサンプル(Helm + Terraform + 監視一式)
注意:超大規模(1000ノード級)未経験。中堅以下の会社でリードしたい旨を率直に書く
```

### この章だけでは足りないもの(次に学ぶべき)
- アルゴリズム(Day 31)で更にパフォ最適化へ
- アーキテクチャ設計(Day 34) / マイクロサービス(Day 35)
- セキュリティ深掘り(Day 39) / ネットワーク(Day 38)

---

## ✅ チェック

- [ ] kubectl で Pod / Deployment / Service 操作
- [ ] ConfigMap / Secret で設定管理
- [ ] Ingress で SSL付きルーティング
- [ ] HPA でオートスケール
- [ ] Helm Chart 作成
- [ ] Terraform で AWS リソース作成
- [ ] Prometheus + Grafana でメトリクス監視
- [ ] SLO/エラーバジェット理解
- [ ] CI/CD パイプライン構築
- [ ] ブルーグリーン or カナリア理解
- [ ] **Production級 IaC プロジェクト完成**

---

## 進捗

```
Day 30: K8s/IaC/監視 完了。
Production運用・SREエンジニア案件対応(月150-300万圏)。
全30日完走可能。
```

## 次

`23_最終_独立宣言.md` を読み返して**月100万への最終確認**。
そして **365日継続で月150-300万** へ。
