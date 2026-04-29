# Day 27:機械学習・MLOps - 「AIエンジニア」を名乗る

## 💰 この章後の月収:**+30〜100万円**(MLエンジニア案件)
## ⏱ 所要:4時間

「AI使えます」じゃなく「**自分でモデル作って運用できます**」。
2026年も需要がもっとも高い領域の一つ。

---

## ゴール

- scikit-learn でMLモデル構築
- 特徴量エンジニアリング
- 評価指標を正しく使える
- モデルをAPIで提供
- MLflow で実験管理
- A/Bテスト・モニタリング

---

## 1. ML の全体像(15分)

```
[データ] → [特徴量] → [学習] → [評価] → [デプロイ] → [監視]
                              ↑________________(再学習)
```

### 主要タスク

- **分類**(spam か否か):LogisticRegression, RandomForest, LightGBM
- **回帰**(価格予測):LinearRegression, GradientBoosting
- **クラスタリング**(顧客セグメント):KMeans, DBSCAN
- **時系列**(売上予測):Prophet, ARIMA
- **異常検知**:IsolationForest, AutoEncoder
- **レコメンド**:協調フィルタリング, MF, NeuralCF

---

## 2. scikit-learn 基本(40分)

```bash
pip install scikit-learn pandas matplotlib
```

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, roc_auc_score
import pandas as pd

df = pd.read_csv("data.csv")
X = df.drop("target", axis=1)
y = df["target"]

# train/test 分割(必ず)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 学習
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# 予測
y_pred = model.predict(X_test)
y_proba = model.predict_proba(X_test)[:, 1]

# 評価
print(classification_report(y_test, y_pred))
print(f"AUC: {roc_auc_score(y_test, y_proba):.4f}")
```

---

## 3. 特徴量エンジニアリング(40分)

「**データを賢いカタチに加工する**」。**精度の8割はここで決まる**。

### 数値

```python
import numpy as np

df["log_amount"] = np.log1p(df["amount"])  # 対数変換
df["amount_squared"] = df["amount"] ** 2
df["amount_z"] = (df["amount"] - df["amount"].mean()) / df["amount"].std()
```

### カテゴリ

```python
# One-Hot
df = pd.get_dummies(df, columns=["category"])

# Target Encoding(リーク注意)
mean_target = df.groupby("category")["target"].mean()
df["category_te"] = df["category"].map(mean_target)
```

### 日付

```python
df["date"] = pd.to_datetime(df["date"])
df["year"] = df["date"].dt.year
df["month"] = df["date"].dt.month
df["dayofweek"] = df["date"].dt.dayofweek
df["is_weekend"] = (df["dayofweek"] >= 5).astype(int)
```

### 集計特徴

```python
# 顧客ごとの集計を結合
agg = df.groupby("user_id").agg(
    user_total=("amount", "sum"),
    user_count=("amount", "count"),
    user_avg=("amount", "mean"),
)
df = df.merge(agg, on="user_id")
```

### 欠損値

```python
df["age"] = df["age"].fillna(df["age"].median())
df["category"] = df["category"].fillna("unknown")
```

---

## 4. ⚠️ リーク防止(超重要)(20分)

「**未来の情報をうっかり使ってしまう**」のがリーク。バイナリー検証で苦しんだやつ。

### NG

```python
# ❌ test も含めて Target Encoding
mean_target = df.groupby("cat")["target"].mean()  # test の target も使う
df["cat_te"] = df["cat"].map(mean_target)
```

### OK

```python
# ✅ train 内だけで計算
mean_target = df.loc[train_idx].groupby("cat")["target"].mean()
df["cat_te"] = df["cat"].map(mean_target)
```

### 時系列の場合

`train_test_split` ではなく**時系列split**:

```python
from sklearn.model_selection import TimeSeriesSplit
tscv = TimeSeriesSplit(n_splits=5)
```

---

## 5. 評価指標を正しく(30分)

### 分類

| 指標 | 用途 |
|---|---|
| **Accuracy** | クラス均衡時のみ |
| **Precision** | 偽陽性減らす(spam検知) |
| **Recall** | 偽陰性減らす(癌検知) |
| **F1** | Precision/Recallのバランス |
| **AUC** | 順位精度(不均衡OK) |
| **Log Loss** | 確率の精度 |

```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    roc_auc_score, confusion_matrix, log_loss
)
```

### 回帰

```python
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error, r2_score
)
```

### 不均衡データ

```python
from sklearn.utils.class_weight import compute_class_weight
weights = compute_class_weight("balanced", classes=[0,1], y=y_train)
model = RandomForestClassifier(class_weight="balanced")
```

---

## 6. LightGBM(実務最強)(30分)

```bash
pip install lightgbm
```

```python
import lightgbm as lgb

model = lgb.LGBMClassifier(
    n_estimators=1000,
    learning_rate=0.05,
    num_leaves=63,
    min_data_in_leaf=100,
    feature_fraction=0.8,
    reg_alpha=0.1, reg_lambda=0.1,
)

model.fit(
    X_train, y_train,
    eval_set=[(X_val, y_val)],
    callbacks=[lgb.early_stopping(50)]
)

# 特徴量重要度
import pandas as pd
imp = pd.DataFrame({"feature": X.columns, "importance": model.feature_importances_})
imp.sort_values("importance", ascending=False).head(20)
```

→ Kaggle・実務で**最頻**。バイナリー検証もこれを使った。

---

## 7. ハイパーパラメータ最適化(20分)

```bash
pip install optuna
```

```python
import optuna

def objective(trial):
    params = {
        "n_estimators": trial.suggest_int("n_estimators", 100, 1000),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3),
        "num_leaves": trial.suggest_int("num_leaves", 15, 127),
    }
    model = lgb.LGBMClassifier(**params)
    model.fit(X_train, y_train)
    return roc_auc_score(y_val, model.predict_proba(X_val)[:, 1])

study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=50)
print(study.best_params)
```

---

## 8. MLflow - 実験管理(30分)

```bash
pip install mlflow
mlflow ui  # http://localhost:5000
```

```python
import mlflow

mlflow.set_experiment("churn_prediction")

with mlflow.start_run():
    mlflow.log_param("n_estimators", 1000)
    mlflow.log_param("learning_rate", 0.05)
    
    model.fit(X_train, y_train)
    auc = roc_auc_score(y_val, model.predict_proba(X_val)[:, 1])
    
    mlflow.log_metric("auc", auc)
    mlflow.lightgbm.log_model(model, "model")
```

→ **どのパラメータでどんな精度?**を全部記録。

---

## 9. モデルAPI化(20分)

```python
# FastAPI
from fastapi import FastAPI
from pydantic import BaseModel
import joblib
import numpy as np

app = FastAPI()
model = joblib.load("model.pkl")

class Features(BaseModel):
    age: int
    amount: float
    category: int

@app.post("/predict")
def predict(f: Features):
    X = np.array([[f.age, f.amount, f.category]])
    proba = float(model.predict_proba(X)[0, 1])
    return {"churn_probability": proba}
```

→ **モデルを REST API として公開**。クライアントは HTTP で叩くだけ。

---

## 10. モデルモニタリング(15分)

本番で**精度が劣化していくのを検知**。

### Data Drift

```python
# 学習時と本番の特徴量分布を比較
from scipy.stats import ks_2samp
stat, pvalue = ks_2samp(train_amount, prod_amount)
if pvalue < 0.05:
    print("分布が変化、再学習を検討")
```

### Prediction Drift

```python
# 予測値の分布が変わったら通知
```

ツール:**Evidently AI**, **WhyLabs** など。

---

## 11. PyTorch/TensorFlow 入門(20分)

```bash
pip install torch
```

```python
import torch
import torch.nn as nn

class SimpleNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.layers = nn.Sequential(
            nn.Linear(10, 64),
            nn.ReLU(),
            nn.Linear(64, 1),
            nn.Sigmoid(),
        )
    
    def forward(self, x):
        return self.layers(x)

model = SimpleNN()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.BCELoss()

# 学習
for epoch in range(100):
    pred = model(X_train_tensor)
    loss = loss_fn(pred, y_train_tensor)
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

→ 画像・音声・テキストの本格DLは PyTorch が標準。

---

## 12. ⭐ 売れる成果物 #13:解約予測モデル + API + ダッシュボード(120分)

クロードに頼む:

```
顧客解約予測の MLプロジェクトを完全に作って。

データ:
- 顧客マスタ + 利用履歴(模擬データ自動生成)
- ターゲット: 翌月解約する/しない

実装:
1. データ生成スクリプト(realistic データ 10,000件)
2. EDA(探索的分析)Jupyter Notebook
3. 特徴量エンジニアリング(集計・時系列)
4. LightGBM + Optuna でハイパラ最適化
5. MLflow で実験管理
6. FastAPI でモデル提供 (/predict)
7. Streamlit ダッシュボード(顧客リスト表示・解約確率)
8. 単体テスト(pytest)
9. Dockerfile / docker-compose.yml

README に「docker compose up」で動く完全版で。
```

→ **これがあれば「MLエンジニア」名乗れる**。月単価100万圏。

---

## ✅ チェック

- [ ] scikit-learn でモデル学習
- [ ] 特徴量エンジニアリングの基本
- [ ] リーク対策・時系列split
- [ ] 評価指標を用途別に使える
- [ ] LightGBM + Optuna でチューニング
- [ ] MLflow で実験管理
- [ ] モデル → FastAPI で公開
- [ ] PyTorch の存在を知ってる
- [ ] **解約予測プロジェクト完成**

---

## 進捗

```
Day 27: ML/MLOps完了。LightGBM/Optuna/MLflow/FastAPI/PyTorch。
MLエンジニア・データサイエンス案件対応(月100万圏)。
```

## 次

`28_LLM_Agent.md` へ。**最先端 LLM Agent / MCP / Tool use**。
