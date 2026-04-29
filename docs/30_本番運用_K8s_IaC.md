# Day 30:本番運用・Kubernetes・IaC - 月150万到達

## 💡 これは何?(小学生でもわかる説明)

****99.9%稼働するシステム**を作る・運用する**

### なぜ大事?
プロのインフラ・SRE案件。月単価150-300万円

### イメージ
**24時間稼働の工場**を設計・監視する仕事

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 この章後の月収:**+50〜200万円**(SRE/DevOps案件)
## ⏱ 所要:4時間

「**作る**」から「**動かし続ける**」へ。
本番環境を**99.9%稼働**させる技術が、SRE/DevOps領域。**月単価150-300万円**。

---

## ゴール

- Kubernetes 基本(Pod/Service/Deployment)
- Terraform で Infrastructure as Code
- 監視・ログ集約・SLO
- ブルーグリーン・カナリアデプロイ
- 障害対応・ポストモーテム

---

## 1. なぜ Kubernetes?(15分)

### 単一サーバーの限界

- 障害時: サービス全停止
- スケール: 手動で足す
- デプロイ: 1台ずつ更新

### Kubernetes 解決

- **複数ノードで冗長化**:1台死んでもOK
- **オートスケール**:負荷増で自動増設
- **ローリングアップデート**:無停止デプロイ
- **セルフヒーリング**:落ちたコンテナを自動再起動

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

### Pod(最小実行単位)

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
kubectl apply -f pod.yaml
kubectl get pods
kubectl logs my-app
kubectl exec -it my-app -- /bin/bash
kubectl delete pod my-app
```

### Deployment(冗長化・更新)

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
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
          readinessProbe:
            httpGet:
              path: /ready
              port: 8000
```

→ **3Pod 常時稼働、1個落ちても自動復旧**。

### Service(外部公開)

`service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8000
  type: LoadBalancer
```

→ **LBで負荷分散**。

---

## 5. ConfigMap / Secret(20分)

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
# Deployment 側
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

---

## 6. Ingress(URLルーティング)(20分)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
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
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

→ **CPU 70% 超えたら自動増設**(最大20Pod)。

---

## 8. Helm(K8s パッケージ管理)(20分)

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
helm install my-app ./my-chart
helm upgrade my-app ./my-chart --set image.tag=1.1
helm rollback my-app 1
```

→ **K8s 設定をテンプレート化**。プロは Helm Chart で管理。

---

## 9. Terraform - Infrastructure as Code(40分)

「**インフラをコードで管理**」。AWS/GCP/Azure 全部対応。

### インストール

```bash
brew install terraform
```

### 例:AWS S3 + CloudFront

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
terraform init
terraform plan   # 何が作られるか確認
terraform apply  # 実行
terraform destroy  # 全削除
```

→ **インフラ全部コード化**。Git管理で再現可能。

---

## 10. 監視・ログ(40分)

### Prometheus + Grafana(K8s標準)

```bash
# Prometheus + Grafana を Helm でインストール
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prom prometheus-community/kube-prometheus-stack
```

→ **CPU/メモリ/HTTP応答時間**を自動収集 → Grafana で可視化。

### Python アプリのメトリクス

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

### ログ集約 - Loki(おすすめ)

または **ELK(Elasticsearch + Logstash + Kibana)**。

### エラー追跡 - Sentry(Day 18 で既出)

---

## 11. SLO / エラーバジェット(15分)

「**99.9%稼働**を保証する」契約レベル。

### 計算

- SLO: 99.9% 稼働 → 月 43分のダウンタイム許容
- エラーバジェット使い切ったら → **新機能リリース停止、改善優先**

### SLI(指標)

- 可用性:成功リクエスト / 全リクエスト
- レイテンシ:p95 < 500ms
- エラー率:5xx < 0.1%

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

### ブルーグリーン

```
[ロードバランサ]
  ├ Blue(現行・100%)
  └ Green(新版・0%)
       ↓ 切替
  ├ Blue(0%)
  └ Green(100%)
```

→ **問題あれば即ロールバック**。

### カナリアリリース

```
新版に 1% → 10% → 50% → 100% と段階的に流す
途中でエラー率上がったら停止
```

### Argo Rollouts でカナリア自動化

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

### 障害発生時

1. **検知**(Prometheus アラート → Slack)
2. **影響範囲確認**(Grafana ダッシュボード)
3. **緊急対応**(ロールバック / スケールアップ)
4. **根本原因調査**
5. **恒久対応**

### ポストモーテム(振り返り)

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
