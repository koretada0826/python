# Day 19:API設計 - FastAPIでバックエンド開発

## 💡 これは何?(小学生でもわかる説明)

****他のアプリと話す窓口**を作る(FastAPI)**

### なぜ大事?
Web/モバイルアプリのバックエンドはこれ。月単価80万圏

### イメージ
**コンビニのレジ**みたいな窓口を作る。みんなここに頼みに来る

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

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

## 1. REST API とは(15分)

「**Webサービスの標準的な作り方**」。

### 例

```
GET    /users         → 全ユーザー取得
GET    /users/123     → ID 123 取得
POST   /users         → 新規作成
PUT    /users/123     → ID 123 更新
DELETE /users/123     → ID 123 削除
```

### HTTP メソッド

| メソッド | 用途 |
|---|---|
| GET | 取得(副作用なし) |
| POST | 作成 |
| PUT | 全置換 |
| PATCH | 部分更新 |
| DELETE | 削除 |

### ステータスコード

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

---

## 2. FastAPI 入門(30分)

```bash
pip install fastapi uvicorn[standard]
```

### 最小アプリ `main.py`

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def hello():
    return {"message": "Hello, FastAPI!"}

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"id": user_id, "name": "太郎"}
```

### 起動

```bash
uvicorn main:app --reload
```

→ http://localhost:8000 でアクセス
→ http://localhost:8000/docs で**自動生成APIドキュメント**(Swagger)

---

## 3. Pydantic - 型検証(30分)

入力を**自動的に検証**する。

```python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)

@app.post("/users")
def create_user(user: UserCreate):
    return {"created": user.name}
```

### 動作

- 正常: `{"name": "太郎", "email": "x@y.z", "age": 25}` → 200
- 不正: `{"age": 200}` → 422 Validation Error 自動返却

→ **入力検証を1行も書かない**で完璧。

### ネスト

```python
class Address(BaseModel):
    city: str
    zip: str

class User(BaseModel):
    name: str
    address: Address
```

---

## 4. CRUD API 完全実装(40分)

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import sessionmaker, declarative_base

# DB設定
engine = create_engine("sqlite:///./test.db")
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()

class UserDB(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    email = Column(String, unique=True)

Base.metadata.create_all(engine)

# スキーマ
class UserCreate(BaseModel):
    name: str
    email: str

class UserOut(BaseModel):
    id: int
    name: str
    email: str
    class Config:
        from_attributes = True

# DI
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

---

## 5. 認証(JWT)(40分)

```bash
pip install python-jose[cryptography] passlib[bcrypt] python-multipart
```

### ログイン → トークン発行

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from passlib.context import CryptContext
from jose import jwt, JWTError
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"
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

---

## 6. ミドルウェア / CORS(15分)

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://my-frontend.com"],
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

---

## 7. 非同期処理(asyncio)(30分)

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

---

## 8. バックグラウンドタスク(15分)

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

### 本格非同期: Celery + Redis

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

## 10. テスト(20分)

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

---

## 11. デプロイ(15分)

### Render での FastAPI

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
