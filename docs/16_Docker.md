# Day 16:Docker - 環境構築・デプロイの標準

## 💡 これは何?(小学生でもわかる説明)

**アプリを「カップ麺の容器」に詰めて、お湯を注げばどこでも動く状態にする日**

### なぜ大事?
**「自分のPCでは動くんですけど…」**を撲滅。
プロの法人案件・SaaSデプロイで**必須スキル**。
Docker使えないと、面接の時点で落とされる。

### イメージ
- **Docker**=「**カップ麺の容器**」(中に麺・スープ・具材が全部入ってる)
- **Dockerfile**=「**カップ麺のレシピ**」(何を入れるか書いた紙)
- **イメージ**=「**カップ麺の完成品**」(まだお湯入れてない状態)
- **コンテナ**=「**お湯入れて食べる状態のカップ麺**」(動いてる実物)
- **Docker Hub**=「**カップ麺のスーパー**」(色んなイメージが買える)

### Claude Code に何を頼めばいいか
Dockerfile・docker-compose.yml・デプロイ手順は全部クロード。
あなたは「**何を動かしたいか**」を伝えるだけ。

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

## 1. Docker とは(=カップ麺で配る)(15分)

### 1-1. なぜDockerが必要?

> **問題:**
> 開発者A: Python 3.10 / Mac
> 開発者B: Python 3.12 / Windows
> 本番: Python 3.9 / Linux
>
> → **動作環境がバラバラ → 動かない**

#### Docker での解決

全員「**同じ箱**」を使う → どこでも同じ動作。

> **イメージ:**
> 自分の家の特製カレーをそのまま渡す → 相手の家の鍋・コンロ・調味料で味が変わる
> Docker = カレーを真空パックにして送る → 解凍するだけで全く同じ味

### 1-2. イメージ vs コンテナ

| 用語 | こども版 | 比喩 |
|---|---|---|
| **Dockerfile** | 設計図 | カップ麺のレシピ |
| **イメージ** | 完成形のテンプレ(まだ動いてない) | 棚に並んだカップ麺 |
| **コンテナ** | イメージから生まれた動く実物 | お湯入れて食べてる状態 |
| **Docker Hub** | イメージの倉庫(GitHubのDocker版) | カップ麺のスーパー |

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

> **動かないとき:** Docker Desktop アプリを起動してから再度コマンド試す。
> Macではメニューバーにクジラのアイコンが出てればOK。

---

## 3. 既存イメージを使ってみる(=スーパーで買う)(15分)

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

### よくある間違い

| ミス | 何が起こる | 直し方 |
|---|---|---|
| イメージを消さない | ディスク容量パンク(=数十GB) | 月1で `docker system prune -a` |
| `docker stop` 忘れ | バックグラウンドで動き続け重い | `docker ps` → 不要なら stop |
| `-p` 指定忘れ | ブラウザからアクセスできない | `-p 8000:8000` でポート公開 |

---

## 4. Dockerfile - 自分のアプリを箱に詰める(=レシピを書く)(40分)

### 4-1. 基本構造

プロジェクトのルートに `Dockerfile`(拡張子なし):

```dockerfile
# ベースイメージ(=土台のOS+Python環境)
FROM python:3.12-slim

# 作業ディレクトリ(=コンテナ内のフォルダ)
WORKDIR /app

# 依存ファイルだけ先にコピー(キャッシュ最適化)
COPY requirements.txt .

# パッケージインストール
RUN pip install --no-cache-dir -r requirements.txt

# アプリコードをコピー
COPY . .

# 公開ポート(情報用、実際の公開は -p で指定)
EXPOSE 8000

# 起動コマンド(コンテナ起動時に実行される)
CMD ["python", "app.py"]
```

> **なぜ requirements.txt を先にコピー?**
> Dockerは**キャッシュ**で動く。requirements.txt が変わらない限り、`pip install` の結果を再利用できる。
> コード変更だけならビルドが**爆速**になる。

### 4-2. ビルド & 起動

```bash
# イメージ作成(. は現在のフォルダ)
docker build -t myapp .

# コンテナ起動(ポート8000を公開)
docker run -p 8000:8000 myapp

# ブラウザで http://localhost:8000
```

→ **これで「自分のPCでは動く」問題が解決**。誰が使っても同じ動作。

### 4-3. Dockerfile の主な命令

| 命令 | 意味 |
|---|---|
| `FROM` | 土台のイメージ(=どのOS+環境を使うか) |
| `WORKDIR` | コンテナ内の作業場所 |
| `COPY` | ホスト → コンテナにファイルコピー |
| `RUN` | ビルド時に実行(=パッケージインストールなど) |
| `EXPOSE` | 公開ポートの宣言(情報用) |
| `CMD` | 起動時の実行コマンド |
| `ENV` | 環境変数の設定 |

---

## 5. .dockerignore(必須)(=送らないファイルリスト)(10分)

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

> **なぜ必要?**
> 何もしないと、`COPY . .` で**全ファイル**(=不要なvenvや巨大なログ)もコンテナに入る。
> 結果、イメージが10GB超えて配布できなくなる。
> .dockerignore で**必要なものだけ**入れる。

---

## 6. docker-compose - 複数コンテナ管理(=セットメニュー)(40分)

### 6-1. なぜ docker-compose?

> **Web + DB + キャッシュ**みたいな複雑構成を1ファイルで一発起動。
> Dockerfile を3つ書いて手で順番に起動するのは面倒。
> docker-compose で**コマンド1発**にまとめる。

### 6-2. `docker-compose.yml`

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

### 6-3. 操作

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

> **クライアント納品で大活躍:**
> 「`git clone` → `docker compose up`」で動く状態にしておくと、
> クライアント側のセットアップ時間がゼロ。**評価★5確定**。

---

## 7. ⭐ Streamlit/FastAPI を Docker化(=売れる作品)(40分)

### 7-1. Streamlit 例

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

> **`--server.address=0.0.0.0` のなぜ:**
> Streamlitはデフォルトで「自分のPCからしかアクセスできない」設定。
> Dockerコンテナ外(=ホスト)からアクセスするには `0.0.0.0` で開放必要。

### 7-2. FastAPI 例

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

## 8. 軽量化テクニック(=ダイエットさせる)(20分)

### 8-1. multi-stage build(=2段階ビルド)

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

> **やってること:**
> 1. ビルド用の重いイメージで `pip install`
> 2. 結果のフォルダだけ、軽いイメージにコピー
> 3. ビルド用イメージは捨てる
> → 結果イメージは「**軽量+動作に必要なものだけ**」になる

### 8-2. ベースイメージ選び

| イメージ | サイズ | 用途 |
|---|---:|---|
| `python:3.12` | ~1GB | 開発中 |
| `python:3.12-slim` | ~150MB | 本番推奨 |
| `python:3.12-alpine` | ~50MB | 超軽量(ただし C拡張で罠あり) |

→ **slim** をデフォルトに。

> **alpine の罠:**
> 軽いが、pandasやnumpyなどC拡張ライブラリのビルドが失敗しやすい。
> プロは「slim でほぼ十分、最後の最後にalpine検討」という流れ。

---

## 9. デプロイ(=世に出す)(40分)

### 9-1. 案A: Render(無料・簡単)⭐ 推奨

1. https://render.com 登録
2. 「New Web Service」
3. GitHub リポジトリを連携
4. **Dockerfile を自動検出 → デプロイ**
5. URL発行(`xxx.onrender.com`)

> **メリット:** 無料プランあり、HTTPS自動、設定がほぼゼロ。
> 個人開発者の最初の一手はこれ。

### 9-2. 案B: Fly.io(高速・安い)

```bash
brew install flyctl
fly launch  # 自動でDocker認識
fly deploy
```

### 9-3. 案C: VPS(さくら / ConoHa / Linode)

```bash
ssh user@your-server
git clone <リポジトリ>
cd project
docker compose up -d
```

→ 月500-2000円で完全自由。

### 9-4. 案D: AWS ECS / Google Cloud Run(本格)

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

## 11. 今日学んだ用語まとめ

| 用語 | 一言で |
|---|---|
| Docker | アプリを箱(コンテナ)に詰めて配る仕組み |
| Dockerfile | コンテナの設計書(レシピ) |
| イメージ | 完成形のテンプレ(まだ動いてない) |
| コンテナ | イメージから生まれた動いてる実物 |
| Docker Hub | イメージの倉庫(GitHubのDocker版) |
| WORKDIR | コンテナ内の作業ディレクトリ |
| FROM | 土台のベースイメージ指定 |
| COPY | ホスト→コンテナにファイルをコピー |
| RUN | ビルド時に実行する命令 |
| CMD | 起動時に実行する命令 |
| EXPOSE | 公開ポートの宣言 |
| .dockerignore | Dockerに送らないファイルリスト |
| docker-compose | 複数コンテナをまとめて管理 |
| multi-stage build | 2段階ビルドで軽量化 |
| slim / alpine | 軽量ベースイメージの種類 |
| Render | 無料で簡単なデプロイ先 |
| Fly.io | Docker向けの安いデプロイ先 |
| VPS | 仮想サーバ(月500-2000円で自由) |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **既存アプリの Docker 化 / 環境構築案件**:単価 5〜20万円(プラットフォーム例:ランサーズ/クラウドワークス/SES)
- **Docker Compose で開発環境セットアップ**:単価 5〜15万円(DB + バックエンド + フロントの一括セット)
- **PoC アプリを Render / Fly.io にデプロイ**:単価 5〜15万円(MVP のクラウド公開)

### クロードへの頼み方(営業文を書かせる例)
```
ランサーズの「環境構築・インフラ整備」案件への提案文を書いて。
Day 16 までで習得:Docker / Dockerfile / docker compose / Render / Fly.io デプロイ
売り:依頼者のローカル PC を汚さず、Compose 一発で開発環境を再現できる + 本番デプロイまで対応
過去実績:Streamlit / FastAPI を Docker化してデプロイしたサンプル(GitHub URL添付)
納期:3〜5日(小規模)/ 1〜2週間(マイクロサービス含む場合)
予算:8〜15万円
```

### この章だけでは足りないもの(次に学ぶべき)
- AWS / GCP(Day 17)で本格的なクラウド対応
- Kubernetes / IaC(Day 30)で大規模本番運用へ
- API設計(Day 19) / セキュリティ(Day 20) との組み合わせ

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
