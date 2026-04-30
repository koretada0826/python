# Day 19:API設計 - FastAPIでバックエンド開発

## 💡 これは何?(小学生でもわかる説明)

**他のアプリ(Webサイトやスマホアプリ)が話しかけてくる「窓口」を作る日**

### なぜ大事?
Web/モバイルアプリの**バックエンド=API**。
ここを作れると「**バックエンドエンジニア**」を名乗れる。
法人案件で**月単価80万圏**が見えてくる。

### イメージ
- **API**=「**コンビニのレジ**」(注文窓口、決まった手順で受ける)
- **GET**=「**商品を見る**」(レジ覗くだけ、変化なし)
- **POST**=「**新規購入**」(レジに渡す)
- **PUT/PATCH**=「**返品交換**」(更新)
- **DELETE**=「**取り消し**」(削除)
- **JWT**=「**入店パス**」(ログイン後の身分証明書)
- **Pydantic**=「**注文用紙の検品係**」(変な注文は受けない)

### Claude Code に何を頼めばいいか
FastAPIコード・Pydanticスキーマ・JWT認証・テストは全部クロード。
あなたは「**何のAPIが必要か**」を伝えるだけ。

---

## 💰 この章後の月収:**+30〜80万円**(API開発案件)
## ⏱ 所要:3-4時間

「**バックエンドエンジニア**」を名乗れるレベル。
法人案件で**月80万〜の単価**取れる。

---

## ゴール

- REST API の設計原則
- FastAPI で本格 API 構築
- 認証(JWT)
- バリデーション(Pydantic)
- API ドキュメント自動生成

---

## 1. REST API とは(=Webサービスの標準的な作法)(15分)

### 1-1. なぜREST?

> **APIの呼び出し方を世界中で統一した規格**。
> 「ユーザー一覧」を見る時、どのサイトでも `GET /users` って書く。
> ルールが統一されてるから、**他人のAPIも一目で使い方分かる**。

### 1-2. 例

```
GET    /users         → 全ユーザー取得
GET    /users/123     → ID 123 取得
POST   /users         → 新規作成
PUT    /users/123     → ID 123 更新
DELETE /users/123     → ID 123 削除
```

### 1-3. HTTP メソッド

| メソッド | 用途 | 比喩 |
|---|---|---|
| GET | 取得(副作用なし) | 商品を見る |
| POST | 作成 | レジで購入 |
| PUT | 全置換 | まるごと交換 |
| PATCH | 部分更新 | 一部だけ修正 |
| DELETE | 削除 | キャンセル |

### 1-4. ステータスコード(=店員の答え方)

| 範囲 | 意味 |
|---|---|
| 2xx | 成功 |
| 3xx | リダイレクト |
| 4xx | クライアントエラー |
| 5xx | サーバーエラー |

| よく使う | 意味 |
|---|---|
| 200 OK | 成功 |
| 201 Created | 作成成功 |
| 204 No Content | 成功・返答なし |
| 400 Bad Request | 入力おかしい |
| 401 Unauthorized | 認証必要 |
| 403 Forbidden | 権限なし |
| 404 Not Found | 見つからない |
| 422 Unprocessable | バリデーションNG |
| 500 Internal Error | サーバーバグ |

> **覚え方:** 4xx は**お客の責任**、5xx は**店の責任**。

---

## 2. FastAPI 入門(=モダンなAPIフレームワーク)(30分)

### 2-1. なぜFastAPI?

| 比較 | Flask | FastAPI |
|---|---|---|
| 速さ | 普通 | **超速**(Node.js並) |
| 型ヒント | 任意 | **必須(自動検証)** |
| ドキュメント | 手動 | **自動生成(Swagger)** |
| 非同期 | 弱い | **ネイティブ対応** |

→ 2026年現在、**Pythonでバックエンド作るなら FastAPI 一択**。

### 2-2. インストール

```bash
pip install fastapi uvicorn[standard]
```

### 2-3. 最小アプリ `main.py`

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def hello():
    return {"message": "Hello, FastAPI!"}

@app.get("/users/{user_id}")
def get_user(user_id: int):  # int で型指定 → 自動的に数字以外を弾く
    return {"id": user_id, "name": "太郎"}
```

### 2-4. 起動

```bash
uvicorn main:app --reload  # --reload でコード変更時に自動再起動
```

→ http://localhost:8000 でアクセス
→ http://localhost:8000/docs で**自動生成APIドキュメント**(Swagger)

> **これがFastAPIの真骨頂:** コード書くだけで**Swagger UIが自動生成**。
> 「APIドキュメント書いて」とクライアントに言われても、URLを送るだけで終わり。

---

## 3. Pydantic - 型検証(=注文用紙の検品係)(30分)

### 3-1. なぜPydantic?

入力を**自動的に検証**する。

> **問題:** ユーザーが age に "abc" と送ってきたら?
> → 普通の Python ならエラーで500(サーバー側のバグ扱い)。
> Pydantic なら**422 Validation Error**を自動返却(=入力ミスとして処理)。

### 3-2. 基本

```python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    email: EmailStr  # メアド形式じゃないと弾く
    age: int = Field(..., ge=0, le=150)  # 0以上150以下

@app.post("/users")
def create_user(user: UserCreate):
    return {"created": user.name}
```

### 3-3. 動作

- 正常: `{"name": "太郎", "email": "x@y.z", "age": 25}` → 200
- 不正: `{"age": 200}` → 422 Validation Error 自動返却

→ **入力検証を1行も書かない**で完璧。

### 3-4. ネスト

```python
class Address(BaseModel):
    city: str
    zip: str

class User(BaseModel):
    name: str
    address: Address  # 入れ子可能
```

---

## 4. CRUD API 完全実装(=ユーザー管理API)(40分)

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import sessionmaker, declarative_base

# DB設定
engine = create_engine("sqlite:///./test.db")
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

# DBテーブル定義
class UserDB(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    email = Column(String, unique=True)

Base.metadata.create_all(engine)

# 入力スキーマ
class UserCreate(BaseModel):
    name: str
    email: str

# 出力スキーマ
class UserOut(BaseModel):
    id: int
    name: str
    email: str
    class Config:
        from_attributes = True  # SQLAlchemyのオブジェクトをそのまま変換

# Dependency Injection(DI)
def get_db():
    db = SessionLocal()
    try: yield db
    finally: db.close()

# API
from fastapi import Depends

app = FastAPI()

@app.post("/users", response_model=UserOut, status_code=201)
def create_user(user: UserCreate, db = Depends(get_db)):
    db_user = UserDB(name=user.name, email=user.email)
    db.add(db_user); db.commit(); db.refresh(db_user)
    return db_user

@app.get("/users", response_model=list[UserOut])
def list_users(db = Depends(get_db)):
    return db.query(UserDB).all()

@app.get("/users/{user_id}", response_model=UserOut)
def get_user(user_id: int, db = Depends(get_db)):
    user = db.query(UserDB).filter(UserDB.id == user_id).first()
    if not user:
        raise HTTPException(404, "Not found")
    return user

@app.put("/users/{user_id}", response_model=UserOut)
def update_user(user_id: int, user: UserCreate, db = Depends(get_db)):
    db_user = db.query(UserDB).filter(UserDB.id == user_id).first()
    if not db_user: raise HTTPException(404)
    db_user.name = user.name; db_user.email = user.email
    db.commit(); db.refresh(db_user)
    return db_user

@app.delete("/users/{user_id}", status_code=204)
def delete_user(user_id: int, db = Depends(get_db)):
    db_user = db.query(UserDB).filter(UserDB.id == user_id).first()
    if not db_user: raise HTTPException(404)
    db.delete(db_user); db.commit()
```

→ **これだけで本格 API**。

> **`Depends(get_db)` は何?**
> Dependency Injection(=依存性注入)。
> 「テスト時はモックDBに差し替える」みたいなことが楽になる。

---

## 5. 認証(JWT)(=入店パスの仕組み)(40分)

### 5-1. JWTって?

> **JSON Web Token**=「**改ざん不可能な入店パス**」。
> ログイン成功時に発行 → 以降のAPI呼び出しに添付 → サーバーが本人確認。

```bash
pip install python-jose[cryptography] passlib[bcrypt] python-multipart
```

### 5-2. ログイン → トークン発行

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from passlib.context import CryptContext
from jose import jwt, JWTError
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"  # 本番は環境変数で
ALGORITHM = "HS256"

pwd_context = CryptContext(schemes=["bcrypt"])
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="login")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

def create_token(data: dict, expires_delta: timedelta = None) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

@app.post("/login")
def login(form: OAuth2PasswordRequestForm = Depends(), db = Depends(get_db)):
    user = db.query(UserDB).filter(UserDB.email == form.username).first()
    if not user or not verify_password(form.password, user.password_hash):
        raise HTTPException(401, "認証失敗")
    token = create_token({"sub": str(user.id)})
    return {"access_token": token, "token_type": "bearer"}

# 認証必要なエンドポイント
def get_current_user(token: str = Depends(oauth2_scheme), db = Depends(get_db)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = int(payload.get("sub"))
    except JWTError:
        raise HTTPException(401, "Invalid token")
    user = db.query(UserDB).filter(UserDB.id == user_id).first()
    if not user: raise HTTPException(401)
    return user

@app.get("/me")
def me(current_user: UserDB = Depends(get_current_user)):
    return {"name": current_user.name}
```

→ **これでログイン制御付き API 完成**。

### よくある間違い

| ミス | 何が起こる | 直し方 |
|---|---|---|
| SECRET_KEY をハードコード | 流出で全トークン偽造可 | 環境変数経由(.env) |
| パスワードを平文保存 | DB漏洩で全パス露出 | bcrypt でハッシュ化 |
| トークン有効期限なし | 半永久的に乗っ取られる | `exp` 設定(15分〜1時間) |

---

## 6. ミドルウェア / CORS(=フロントから呼べるように)(15分)

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://my-frontend.com"],  # 信頼するドメイン
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# リクエストログ
import time
@app.middleware("http")
async def log_requests(request, call_next):
    start = time.time()
    response = await call_next(request)
    duration = time.time() - start
    print(f"{request.method} {request.url.path} - {duration:.2f}s")
    return response
```

> **CORSとは:**
> ブラウザのセキュリティ機能。「**別ドメインからのAPI呼び出しを禁止**」がデフォルト。
> CORS設定で「このドメインからは許可する」を明示する。

---

## 7. 非同期処理(asyncio)(=並列で速くする)(30分)

```python
@app.get("/slow")
async def slow():
    import asyncio
    await asyncio.sleep(2)
    return {"done": True}

# 並列処理
import httpx

@app.get("/aggregate")
async def aggregate():
    async with httpx.AsyncClient() as client:
        results = await asyncio.gather(
            client.get("https://api1.com"),
            client.get("https://api2.com"),
            client.get("https://api3.com"),
        )
    return [r.json() for r in results]
```

→ **3つのAPI同時呼び出し**で3倍速。

> **なぜ async?**
> 通常: 1秒×3 = 3秒
> async: 並列で1秒
> ユーザー体感速度が3倍。

---

## 8. バックグラウンドタスク(=裏で処理させる)(15分)

```python
from fastapi import BackgroundTasks

def send_email(email: str, body: str):
    # 重い処理
    ...

@app.post("/notify")
def notify(email: str, bg: BackgroundTasks):
    bg.add_task(send_email, email, "Hello")
    return {"status": "queued"}
```

→ レスポンス即返す + 裏で処理。

### 8-1. 本格非同期: Celery + Redis

長時間処理は Celery に任せる(後日学習)。

---

## 9. ファイルアップロード(15分)

```python
from fastapi import File, UploadFile

@app.post("/upload")
async def upload(file: UploadFile = File(...)):
    contents = await file.read()
    with open(f"uploads/{file.filename}", "wb") as f:
        f.write(contents)
    return {"filename": file.filename, "size": len(contents)}
```

---

## 10. テスト(=API 用 pytest)(20分)

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_get_user():
    response = client.get("/users/1")
    assert response.status_code == 200
    assert response.json()["name"] == "太郎"

def test_create_user():
    response = client.post("/users", json={"name": "花子", "email": "h@y.z"})
    assert response.status_code == 201
```

> **TestClient のすごさ:**
> 実際にサーバー起動せず、メモリ内でAPIをテスト。**爆速**。

---

## 11. デプロイ(15分)

### 11-1. Render での FastAPI

`Dockerfile`:
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

→ Render に push → 公開URL発行 → 完了。

---

## 12. ⭐ 売れる成果物 #10:本格REST API(60分)

クロードに頼む:

```
FastAPI で「TODOアプリ」のREST APIを作って。

機能:
- ユーザー登録・ログイン(JWT)
- TODO の CRUD
- ユーザーごとに分離(他人のTODO見れない)
- ページング(?page=1&size=10)
- 検索(?q=キーワード)
- タグでフィルタ

要件:
- SQLAlchemy + PostgreSQL
- pytest テスト30件以上
- Pydantic 完全活用
- README に curl コマンド例
- Docker 化

これを納品物として完成させて。
```

→ **これで「APIエンジニア」として通用**。月額10万〜の保守契約取れる。

---

## 13. 今日学んだ用語まとめ

| 用語 | 一言で |
|---|---|
| API | アプリ同士が話す窓口 |
| REST | API設計の世界標準ルール |
| HTTP メソッド | GET/POST/PUT/PATCH/DELETE |
| ステータスコード | 200=成功、404=見つからない、500=サーバーエラー |
| FastAPI | モダンなPython製APIフレームワーク |
| Uvicorn | FastAPI を動かすサーバー |
| Swagger UI | 自動生成APIドキュメント(/docs) |
| Pydantic | 入力検証ライブラリ(注文用紙の検品係) |
| BaseModel | Pydantic のスキーマ定義クラス |
| CRUD | Create / Read / Update / Delete |
| Depends | Dependency Injection の仕組み |
| JWT | 改ざん不可能な認証トークン |
| bcrypt | パスワードハッシュ化アルゴリズム |
| CORS | クロスオリジン制御(他ドメインAPI呼び出し許可) |
| async / await | 非同期並列処理 |
| BackgroundTasks | レスポンス後に裏で実行する処理 |
| TestClient | FastAPI 専用のテストクライアント |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **REST API 新規設計・実装**:単価 30〜100万円(プラットフォーム例:ランサーズ/レバテック/直契約)
- **既存システムへの API 追加(連携用)**:単価 30〜80万円
- **マイクロサービス化のための API 切り出し**:単価 50〜150万円(Day 35 の前哨戦)

### クロードへの頼み方(営業文を書かせる例)
```
ランサーズの「FastAPI で REST API 構築」案件への提案文を書いて。
Day 19 までで習得:FastAPI / Pydantic / SQLAlchemy / JWT認証 / CORS / async / OpenAPI 仕様書自動生成
売り:OpenAPI 仕様書 + 認証 + テスト + Docker 一式で納品
過去実績:Streamlit と組み合わせた API デモ(GitHub URL)
納期:2〜3週間
予算:50〜80万円
ポイント:納品時に「フロントエンド側がそのまま叩けるドキュメント」まで揃える
```

### この章だけでは足りないもの(次に学ぶべき)
- セキュリティ(Day 20)で API の脆弱性対策
- チーム開発・PR文化(Day 21)
- マイクロサービス / gRPC(Day 35)

---

## ✅ チェック

- [ ] FastAPI で REST API 作れる
- [ ] Pydantic でバリデーション書ける
- [ ] SQLAlchemy でDB連携
- [ ] JWT 認証実装
- [ ] CORS / Middleware 理解
- [ ] async / await 理解
- [ ] テスト書ける
- [ ] Swagger で API ドキュメント自動生成
- [ ] **本格REST APIをポートフォリオに追加**

---

## 進捗

```
Day 19: API設計完了。FastAPI/Pydantic/JWT/SQLAlchemy 一通り。
TODOアプリ API 完成。月80万圏のスキル装備。
```

## 次

`20_セキュリティ.md` へ。**法人案件で必ず聞かれる**。
