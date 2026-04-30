# Day 27:機械学習・MLOps - 「AIエンジニア」を名乗る

## 💡 これは何?(小学生でもわかる説明)

**予測モデルを作って運用する(機械学習)**

### なぜ大事?
「AI予測できます」=月単価100万圏。
今までは「AIを使う」だけだったけど、ここでは「**自分で予測モデルを作る**」段階に入る。
2026年も需要がもっとも高い領域の一つ。

### イメージ
- **機械学習モデル**=「**経験豊富な天気予報士**」(過去データから明日を予測)
- **データ**=「**過去の天気の記録**」
- **特徴量**=「**気圧・湿度・温度**などの判断材料」
- **学習**=「**予報士に過去のデータを大量に見せて訓練**する」
- **予測**=「**訓練済みの予報士に明日を聞く**」
- **scikit-learn**=「**ML の道具箱**」
- **LightGBM**=「**Kaggleでよく優勝する高性能モデル**」
- **MLflow**=「**実験ノート**」(どのレシピが何点だったか記録)
- **MLOps**=「**ML+運用**」(作って終わりじゃなく、本番で動かし続ける技術)
- **データドリフト**=「**世界が変わって予報士の知識が古くなる**」(=再学習が必要)

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 この章後の月収:**+30〜100万円**(MLエンジニア案件)
## ⏱ 所要:4時間

「AI使えます」じゃなく「**自分でモデル作って運用できます**」。
2026年も需要がもっとも高い領域の一つ。

> **なぜ高単価?**
> 「予測精度1%向上 = 売上数千万UP」みたいな世界。
> 投資対効果が大きいので、企業は喜んで月100万払う。

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

### 1-1. 流れ

```
[データ] → [特徴量] → [学習] → [評価] → [デプロイ] → [監視]
                              ↑________________(再学習)
```

> **イメージ:**
> 1. 食材を集める(データ)
> 2. 下ごしらえ(特徴量)
> 3. 料理を覚える(学習)
> 4. 試食(評価)
> 5. お店で出す(デプロイ)
> 6. 客の反応を見る(監視)
> 7. レシピを改良(再学習)

### 1-2. 主要タスク

| タスク | 例 | 代表モデル |
|---|---|---|
| **分類** | spam か否か | LogisticRegression, RandomForest, LightGBM |
| **回帰** | 価格予測 | LinearRegression, GradientBoosting |
| **クラスタリング** | 顧客セグメント分け | KMeans, DBSCAN |
| **時系列** | 売上予測 | Prophet, ARIMA |
| **異常検知** | 不正取引検出 | IsolationForest, AutoEncoder |
| **レコメンド** | おすすめ商品 | 協調フィルタリング, MF |

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
X = df.drop("target", axis=1)  # 特徴量(説明変数)
y = df["target"]               # 答え(目的変数)

# train/test 分割(必ずやる!)
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,        # 20%をテスト用に取っておく
    random_state=42,      # 同じ乱数で再現性確保
    stratify=y            # クラス比率を保つ
)

# 学習
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# 予測
y_pred = model.predict(X_test)
y_proba = model.predict_proba(X_test)[:, 1]  # 確率(0-1)

# 評価
print(classification_report(y_test, y_pred))
print(f"AUC: {roc_auc_score(y_test, y_proba):.4f}")
```

> **なぜ train/test 分割?**
> 学習に使ったデータで評価したら「カンニング」状態。
> 見たことないデータで試して初めて、本物の精度が分かる。

---

## 3. 特徴量エンジニアリング(40分)

「**データを賢いカタチに加工する**」。**精度の8割はここで決まる**。

> **イメージ:** 同じ食材でも、切り方・下味のつけ方で味が全然違う。
> ML は「**良い特徴量**」が最重要。

### 3-1. 数値変換

```python
import numpy as np

df["log_amount"] = np.log1p(df["amount"])              # 対数変換(歪んだ分布を均す)
df["amount_squared"] = df["amount"] ** 2               # 2乗(非線形を捉える)
df["amount_z"] = (df["amount"] - df["amount"].mean()) / df["amount"].std()  # 標準化
```

### 3-2. カテゴリ変換

```python
# One-Hot(=各カテゴリを0/1の列に)
df = pd.get_dummies(df, columns=["category"])

# Target Encoding(=カテゴリごとの平均ターゲット値を特徴に)
mean_target = df.groupby("category")["target"].mean()
df["category_te"] = df["category"].map(mean_target)
```

### 3-3. 日付特徴

```python
df["date"] = pd.to_datetime(df["date"])
df["year"] = df["date"].dt.year
df["month"] = df["date"].dt.month
df["dayofweek"] = df["date"].dt.dayofweek      # 0=月、6=日
df["is_weekend"] = (df["dayofweek"] >= 5).astype(int)
```

### 3-4. 集計特徴

```python
# 顧客ごとの集計を特徴量に
agg = df.groupby("user_id").agg(
    user_total=("amount", "sum"),
    user_count=("amount", "count"),
    user_avg=("amount", "mean"),
)
df = df.merge(agg, on="user_id")
```

### 3-5. 欠損値処理

```python
df["age"] = df["age"].fillna(df["age"].median())       # 中央値で埋める
df["category"] = df["category"].fillna("unknown")      # "unknown"で埋める
```

---

## 4. ⚠️ リーク防止(超重要)(20分)

「**未来の情報をうっかり使ってしまう**」のがリーク。
リークすると「学習時90%精度なのに本番20%」みたいなことが起きる。

> **イメージ:** テスト前にカンニングペーパー見て満点 → 本番でも実力ある気になるけど、実は0点。

### 4-1. NG(=リーク発生)

```python
# ❌ test も含めて Target Encoding
mean_target = df.groupby("cat")["target"].mean()  # test の target も使う
df["cat_te"] = df["cat"].map(mean_target)
```

### 4-2. OK(=train内だけで)

```python
# ✅ train 内だけで計算
mean_target = df.loc[train_idx].groupby("cat")["target"].mean()
df["cat_te"] = df["cat"].map(mean_target)
```

### 4-3. 時系列の場合

`train_test_split` ではなく**時系列split**:

```python
from sklearn.model_selection import TimeSeriesSplit
tscv = TimeSeriesSplit(n_splits=5)
# 過去で学習、未来で評価(=実運用と同じ)
```

> **なぜ時系列split?**
> 普通のsplitだと「未来データで学習、過去データで評価」になる可能性がある(=ありえない)。
> 必ず「**時間順**」にsplitする。

---

## 5. 評価指標を正しく(30分)

### 5-1. 分類

| 指標 | 用途 | たとえ話 |
|---|---|---|
| **Accuracy** | クラス均衡時のみ | 全部正解の率(=不均衡で罠) |
| **Precision** | 偽陽性減らす | spam検知:無実をspam扱いしない |
| **Recall** | 偽陰性減らす | 癌検知:癌を見逃さない |
| **F1** | Precision/Recallのバランス | 上記の調和平均 |
| **AUC** | 順位精度(不均衡OK) | ランキングの良さ |
| **Log Loss** | 確率の精度 | 予測確率がどれだけ正確か |

```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    roc_auc_score, confusion_matrix, log_loss
)
```

### 5-2. 回帰

```python
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error, r2_score
)
```

### 5-3. 不均衡データ

```python
from sklearn.utils.class_weight import compute_class_weight
weights = compute_class_weight("balanced", classes=[0,1], y=y_train)
model = RandomForestClassifier(class_weight="balanced")
```

> **不均衡データの罠:**
> 99%が正常、1%が異常のデータで「全部正常」と予測すると Accuracy 99%。
> でもこれは無意味(異常を1個も検知できてない)。
> → AUC や F1 で判定する。

---

## 6. LightGBM(実務最強)(30分)

```bash
pip install lightgbm
```

```python
import lightgbm as lgb

model = lgb.LGBMClassifier(
    n_estimators=1000,           # 木の数
    learning_rate=0.05,          # 学習率
    num_leaves=63,               # 葉の数
    min_data_in_leaf=100,        # 過学習防止
    feature_fraction=0.8,        # 特徴量サンプリング
    reg_alpha=0.1, reg_lambda=0.1,  # 正則化
)

model.fit(
    X_train, y_train,
    eval_set=[(X_val, y_val)],
    callbacks=[lgb.early_stopping(50)]  # 50回改善なしで停止
)

# 特徴量重要度(=どの特徴が効いたか)
import pandas as pd
imp = pd.DataFrame({"feature": X.columns, "importance": model.feature_importances_})
imp.sort_values("importance", ascending=False).head(20)
```

→ Kaggle・実務で**最頻**。バイナリー検証もこれを使った。

> **なぜ LightGBM 最強?**
> - 高速(=GBM系で最速クラス)
> - 高精度(=テーブルデータで最強格)
> - 扱いやすい(=チューニングが比較的楽)

---

## 7. ハイパーパラメータ最適化(20分)

> **ハイパラとは?**
> モデルの設定値(木の数、学習率など)。手動だとキリがない。
> Optuna に任せれば**自動で最適値を探してくれる**。

```bash
pip install optuna
```

```python
import optuna

def objective(trial):
    # 試したい範囲を指定
    params = {
        "n_estimators": trial.suggest_int("n_estimators", 100, 1000),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3),
        "num_leaves": trial.suggest_int("num_leaves", 15, 127),
    }
    model = lgb.LGBMClassifier(**params)
    model.fit(X_train, y_train)
    return roc_auc_score(y_val, model.predict_proba(X_val)[:, 1])

study = optuna.create_study(direction="maximize")  # AUC最大化
study.optimize(objective, n_trials=50)              # 50回試す
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
    # パラメータ記録
    mlflow.log_param("n_estimators", 1000)
    mlflow.log_param("learning_rate", 0.05)

    model.fit(X_train, y_train)
    auc = roc_auc_score(y_val, model.predict_proba(X_val)[:, 1])

    # メトリクス記録
    mlflow.log_metric("auc", auc)
    # モデル本体も保存
    mlflow.lightgbm.log_model(model, "model")
```

→ **どのパラメータでどんな精度?**を全部記録。

> **なぜ必要?**
> 100回実験すると「どれが何だっけ?」状態。
> MLflowは全部GUIで一覧化してくれる。

---

## 9. モデルAPI化(20分)

```python
# FastAPI
from fastapi import FastAPI
from pydantic import BaseModel
import joblib
import numpy as np

app = FastAPI()
model = joblib.load("model.pkl")  # 学習済みモデルをロード

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

> **なぜ劣化?**
> 世界が変わるから。コロナ前の購買予測モデルはコロナ後使えない。

### 10-1. Data Drift(=入力分布の変化)

```python
# 学習時と本番の特徴量分布を比較
from scipy.stats import ks_2samp
stat, pvalue = ks_2samp(train_amount, prod_amount)
if pvalue < 0.05:
    print("分布が変化、再学習を検討")
```

### 10-2. Prediction Drift(=出力分布の変化)

予測値の分布が変わったら通知。

ツール:**Evidently AI**, **WhyLabs** など。

---

## 11. PyTorch/TensorFlow 入門(20分)

> **scikit-learn vs PyTorch:**
> - scikit-learn = テーブルデータ向け、シンプル
> - PyTorch = 画像・音声・テキスト・LLM 用、深層学習

```bash
pip install torch
```

```python
import torch
import torch.nn as nn

class SimpleNN(nn.Module):
    def __init__(self):
        super().__init__()
        # ニューラルネット定義
        self.layers = nn.Sequential(
            nn.Linear(10, 64),    # 入力10次元 → 64次元
            nn.ReLU(),            # 活性化関数
            nn.Linear(64, 1),     # 64次元 → 出力1
            nn.Sigmoid(),         # 0-1に潰す
        )

    def forward(self, x):
        return self.layers(x)

model = SimpleNN()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.BCELoss()

# 学習ループ
for epoch in range(100):
    pred = model(X_train_tensor)
    loss = loss_fn(pred, y_train_tensor)
    optimizer.zero_grad()  # 勾配リセット
    loss.backward()        # 逆伝播
    optimizer.step()       # パラメータ更新
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

## 13. よくある間違い・エラー(15分)

| 状況 | 間違いの原因 | 直し方 |
|---|---|---|
| 学習時90%、本番20% | リーク発生 | train内だけで集計、時系列split |
| 不均衡データで Accuracy 99% | 全部多数派と予測してる | AUC・F1で評価 |
| 全特徴量を使って overfit | 過学習 | 正則化、特徴量選択、early stopping |
| 標準化していない | 距離ベース手法でバグ | `StandardScaler` 適用 |
| カテゴリを数字でそのまま | 順序を勝手に解釈 | One-Hot か Target Encoding |
| 欠損値そのまま | エラーで学習できない | fillna で埋める |
| 評価指標がバラバラ | 比較できない | プロジェクト最初に決める |
| MLflow なしで試行錯誤 | どれが何か分からなくなる | 最初から MLflow 使う |
| 本番デプロイしっぱなし | データドリフトで劣化 | 定期再学習 + 監視 |

---

## 14. 用語まとめ

| 用語 | 一言で |
|---|---|
| 特徴量 | モデルへの入力(説明変数) |
| ターゲット | 予測したい答え(目的変数) |
| train/test split | 学習用と評価用に分ける |
| 過学習(overfit) | 学習データに馴染みすぎ |
| リーク | 未来情報のうっかり混入 |
| Accuracy | 正解率 |
| Precision | 偽陽性を減らす指標 |
| Recall | 偽陰性を減らす指標 |
| AUC | 不均衡でも頑健な指標 |
| LightGBM | 高速・高精度GBMライブラリ |
| Optuna | ハイパラ自動最適化 |
| MLflow | 実験管理ツール |
| ハイパラ | モデルの設定値 |
| early stopping | 改善が止まったら停止 |
| データドリフト | 入力分布が変わる現象 |
| MLOps | ML+運用の総称 |
| PyTorch | 深層学習フレームワーク |
| 標準化 | 平均0、分散1にする |
| One-Hot | カテゴリを0/1列に変換 |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **機械学習モデル開発・ML案件**:単価 80〜300万円(プラットフォーム例:Findy Freelance/直契約/SES)
- **需要予測 / 解約予測 / 異常検知システム**:単価 80〜200万円(scikit-learn + LightGBM)
- **MLOps 基盤構築 + モデル運用**:単価 100〜300万円(MLflow + デプロイ + 監視)

### クロードへの頼み方(営業文を書かせる例)
```
ML案件の募集向けに、スキルシート + 提案文セットを書いて。
Day 27 までで習得:scikit-learn / LightGBM / Optuna / MLflow / 評価指標 / 時系列split / リーク対策
売り:単発の精度勝負ではなく、「学習〜推論〜監視〜再学習」までパイプライン化できる
過去実績:GitHubにサンプル ML プロジェクト(docker compose up で再現)
希望:単発受注の場合は 80万〜、月額契約は 50万円〜/月
注意:Kaggle 上位入賞経験は無いので、業務適用と運用フェーズで価値を出す方針
```

### この章だけでは足りないもの(次に学ぶべき)
- LLM / Agent(Day 28)
- フルスタック(Day 29)で予測結果を見せるアプリまで
- 統計・因果推論(Day 44)で「なぜそうなるか」まで説明できるレベルへ

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
