# Day 16:Docker - 環境構築・デプロイの標準

## 💡 これは何?(小学生でもわかる説明)

****アプリを箱に詰めて配る**仕組み**

### なぜ大事?
「自分のPCでは動くんですけど」を撲滅。プロの常識

### イメージ
**カップ麺**=箱に必要な物全部入ってる、お湯入れれば誰でも食べられる

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 この章後の月収:**+10〜30万円**(法人デプロイ案件)
## ⏱ 所要:3時間

「**自分のPCでは動くんですけど…**」を撲滅するツール。
法人案件・SaaSデプロイで**必須スキル**。

---

## ゴール

- Docker の概念わかる
- Dockerfile / docker-compose 書ける
- 自分のアプリを Docker化
- Render / AWS / VPS にデプロイ

---

## 1. Docker とは(15分)

「**アプリを箱(コンテナ)に詰めて配る仕組み**」。

### なぜ必要?

#### 問題
- 開発者A: Python 3.10 / Mac
- 開発者B: Python 3.12 / Windows
- 本番: Python 3.9 / Linux

→ **動作環境がバラバラ → 動かない**

#### Docker での解決
全員「**同じ箱**」を使う → どこでも同じ動作。

### イメージ vs コンテナ

| 用語 | こども版 |
|---|---|
| **イメージ** | 設計図(Dockerfile から作る) |
| **コンテナ** | イメージから生まれた実物 |
| **Docker Hub** | イメージの倉庫(GitHubのDocker版) |

---

## 2. インストール(15分)

1. https://www.docker.com/products/docker-desktop/
2. Mac版 ダウンロード
3. インストール → 起動
4. ターミナルで確認:

```bash
docker --version
# Docker version 24.x.x
```

---

## 3. 既存イメージを使ってみる(15分)

```bash
# Python の公式イメージで Pythonインタプリタ起動
docker run -it python:3.12

# 内部で Python が動く → exit() で抜ける
```

```bash
# 動いてるコンテナ
docker ps

# 全コンテナ(停止済みも)
docker ps -a

# 停止
docker stop <コンテナID>

# 削除
docker rm <コンテナID>

# イメージ一覧
docker images

# イメージ削除
docker rmi <イメージID>
```

---

## 4. Dockerfile - 自分のアプリを箱に詰める(40分)

プロジェクトのルートに `Dockerfile`(拡張子なし):

```dockerfile
# ベースイメージ(土台)
FROM python:3.12-slim

# 作業ディレクトリ
WORKDIR /app

# 依存ファイルだけ先にコピー(キャッシュ最適化)
COPY requirements.txt .

# パッケージインストール
RUN pip install --no-cache-dir -r requirements.txt

# アプリコードをコピー
COPY . .

# 公開ポート(情報用)
EXPOSE 8000

# 起動コマンド
CMD ["python", "app.py"]
```

### ビルド & 起動

```bash
# イメージ作成(. は現在のフォルダ)
docker build -t myapp .

# コンテナ起動(ポート8000を公開)
docker run -p 8000:8000 myapp

# ブラウザで http://localhost:8000
```

→ **これで「自分のPCでは動く」問題が解決**。誰が使っても同じ動作。

---

## 5. .dockerignore(必須)(10分)

GitとIgnoreの Docker版。

```
__pycache__/
*.pyc
.git/
.env
.venv/
venv/
node_modules/
data/
*.log
.pytest_cache/
.DS_Store
```

→ イメージを軽く保つ。

---

## 6. docker-compose - 複数コンテナ管理(40分)

「**Web + DB + キャッシュ**」みたいな複雑構成を1ファイルで。

### `docker-compose.yml`

```yaml
version: "3.9"

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=myapp
    volumes:
      - db_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  db_data:
```

### 操作

```bash
# 全部起動
docker compose up

# バックグラウンド起動
docker compose up -d

# ログ
docker compose logs -f

# 停止
docker compose down

# DB データも消す
docker compose down -v
```

→ **これで開発環境1コマンドで再現**。新人が来ても5分でセットアップ完了。

---

## 7. ⭐ Streamlit/FastAPI を Docker化(40分)

### Streamlit 例

`Dockerfile`:
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8501
CMD ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

```bash
docker build -t mystreamlit .
docker run -p 8501:8501 mystreamlit
```

### FastAPI 例

`Dockerfile`:
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

→ Day 8 で作ったSaaSをこの形式で Docker化する。

---

## 8. 軽量化テクニック(20分)

### multi-stage build

```dockerfile
# ビルドステージ(重い)
FROM python:3.12 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# 実行ステージ(軽い)
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

→ **イメージサイズ 1GB → 200MB**(配布が速い)

### ベースイメージ選び

| イメージ | サイズ | 用途 |
|---|---:|---|
| `python:3.12` | ~1GB | 開発中 |
| `python:3.12-slim` | ~150MB | 本番推奨 |
| `python:3.12-alpine` | ~50MB | 超軽量(ただし C拡張で罠あり) |

→ **slim** をデフォルトに。

---

## 9. デプロイ(40分)

### 案A: Render(無料・簡単)⭐ 推奨

1. https://render.com 登録
2. 「New Web Service」
3. GitHub リポジトリを連携
4. **Dockerfile を自動検出 → デプロイ**
5. URL発行(`xxx.onrender.com`)

### 案B: Fly.io(高速・安い)

```bash
brew install flyctl
fly launch  # 自動でDocker認識
fly deploy
```

### 案C: VPS(さくら / ConoHa / Linode)

```bash
ssh user@your-server
git clone <リポジトリ>
cd project
docker compose up -d
```

→ 月500-2000円で完全自由。

### 案D: AWS ECS / Google Cloud Run(本格)

法人案件で必要になる。**Day 17 で詳細**。

---

## 10. ⭐ 売れる成果物 #8:Docker化されたSaaS(60分)

クロードに頼む:

```
Day 8 で作った Streamlit SaaS を Docker化して。

要件:
1. Dockerfile(multi-stage、slim)
2. docker-compose.yml(Streamlit + PostgreSQL)
3. .dockerignore
4. README に「docker compose up で起動」と明記
5. Render デプロイ手順も追記

これでクライアントに「git clone → docker compose up」で
動かせる状態にする。
```

→ **法人案件で「Docker使えますか?」に YES と答えられる**。

---

## ✅ チェック

- [ ] Docker Desktop インストール完了
- [ ] `docker run python:3.12` 動かせた
- [ ] Dockerfile 書ける
- [ ] .dockerignore 設定済
- [ ] docker compose up で複数コンテナ起動
- [ ] 自分の Streamlit/FastAPI を Docker化
- [ ] Render or Fly.io でデプロイ成功
- [ ] **公開URLで自分のアプリが動いてる**

---

## 進捗

```
Day 16: Docker完了。Streamlit/FastAPI/PostgreSQL 一括Docker化。
Render デプロイ成功。URL: https://xxx.onrender.com
```

## 次

`17_AWS_クラウド.md` へ。**法人本番運用のクラウド**。
