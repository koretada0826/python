# Day 10:GitHub・SNS発信 - 信頼の蓄積

## 💰 直接の収入:0円(超強力な間接効果)
## 📈 効果:採用率2-3倍、単価1.5-2倍、半年後から直接依頼が来る
## ⏱ 所要:3時間

Day 9 で作ったポートフォリオを GitHub に整備し、
**SNS発信で「指名で来る」状態**を作る。

---

## ゴール

- GitHub にポートフォリオ完備
- X/Twitter 運用開始(プロフィール+初投稿)
- Note アカウント作成
- 営業文テンプレ完成
- 信頼の蓄積を開始

---

## 1. Git の超基本(15分)

### 用語

| 言葉 | 意味 |
|---|---|
| リポジトリ(repo) | プロジェクトの保管庫 |
| コミット | 変更を記録 |
| プッシュ | ローカル → GitHub |
| クローン | GitHub → ローカル |

### 基本フロー

```
コード書く
  ↓
git add .            (変更を選ぶ)
  ↓
git commit -m "..."  (記録)
  ↓
git push             (GitHubに送る)
```

---

## 2. 最初のリポジトリ(30分)

### Step 1:GitHub上で作る

1. https://github.com → 右上「+」→「New repository」
2. 名前: `python-portfolio`
3. **Public**(公開)
4. 「Add README file」のチェックは外す
5. Create repository

### Step 2:ローカルで初期化

ターミナル(プロジェクトフォルダで):

```
cd ~/portfolio
git init
```

### Step 3:.gitignore を作る

```
# .gitignore
__pycache__/
*.pyc
.env
.DS_Store
output/
*.log
node_modules/
```

→ クロードに「Python用の.gitignore作って」で完璧。

### Step 4:コミット

```
git add .
git commit -m "初コミット: ポートフォリオ"
```

### Step 5:GitHubに繋ぐ

```
git remote add origin https://github.com/<ユーザー名>/python-portfolio.git
git branch -M main
git push -u origin main
```

→ ブラウザで確認。**作品が世界に公開された**!

### 認証エラーが出たら

「Personal Access Token」を作る:
1. GitHub → Settings → Developer settings → Personal access tokens (classic)
2. Generate new token (classic)、scope: `repo`
3. パスワード欄にこのトークン貼る

---

## 3. ⭐ 重要:READMEの書き方(40分)

GitHubのトップに表示される**作品の説明書**。**ここで採用が決まる**。

### 全体README(`/README.md`)

```markdown
# Python ポートフォリオ

業務自動化・データ分析・AI連携 のツール集。

## 作品

| # | 名前 | 概要 |
|---|---|---|
| 01 | [Excel統合ツール](./01_excel_merger) | 複数Excel/CSVを自動統合 |
| 02 | [価格収集スクレイパー](./02_price_scraper) | Webから価格自動取得 |
| 03 | [AIチャットボット](./03_ai_chatbot) | Claude API + Streamlit |

## スキル

- Python 3.10+
- pandas / openpyxl / matplotlib
- requests / BeautifulSoup / Playwright
- Claude API / OpenAI API
- Streamlit

## 連絡先

- メール: ...
- Twitter: @...
- ランサーズ: ...
```

### 個別README(`/01_excel_merger/README.md`)

```markdown
# Excel統合ツール

## 概要
複数のExcel/CSVファイルを自動的に1つに統合し、月別・カテゴリ別の
サマリーシートとグラフを生成します。

## デモ

![スクショ](docs/screenshot.png)

## 機能

- フォルダ内の sales_*.csv / *.xlsx を一括読み込み
- "all" / "monthly" / "category" の3シート出力
- matplotlib によるグラフ画像 dashboard.png

## 使い方

\`\`\`bash
pip install -r requirements.txt
python3 merge_sales.py
\`\`\`

## 想定削減時間

手作業 5時間/月 → 自動化 30秒/月

## 必要環境

- Python 3.10+
- pandas, openpyxl, matplotlib

## ライセンス

MIT
```

→ クロードに「このコードのREADMEをマークダウンで書いて」で生成可。

### スクショの撮り方

- ターミナル実行画面: `Cmd + Shift + 4` → 範囲選択 → デスクトップに保存
- 出力Excelの中身: 同様
- `/01_excel_merger/docs/` に保存して README から参照

---

## 4. 全作品をまとめる構成(20分)

```
~/portfolio/
├── README.md              ← 全体紹介
├── 01_excel_merger/
│   ├── README.md
│   ├── merge_sales.py
│   ├── requirements.txt
│   ├── docs/screenshot.png
│   └── sample_data/
├── 02_price_scraper/
│   └── ...
├── 03_ai_chatbot/
│   └── ...
└── .gitignore
```

→ Day 4-6 で作った3作品を整理。

---

## 5. プロフィールページ(20分)

GitHubで `<ユーザー名>/<ユーザー名>` リポジトリを作ると、
プロフィール TOP に README が表示される。

### 例

```markdown
# 〇〇です

Pythonエンジニア / 業務自動化 / AI連携

## できること

- 📊 データ分析・レポート自動化
- 🕷 Webスクレイピング
- 🤖 AIアプリ開発(Claude/OpenAI)
- 📁 業務効率化(Excel・PDF処理)

## 作品

→ [Python ポートフォリオ](https://github.com/xxx/python-portfolio)

## 連絡

- ランサーズ: ...
- メール: ...
```

クロードに「GitHubプロフィールREADME書いて」と頼めば作ってくれる。

---

## 6. 進めるたびにコミット(15分)

毎日のフロー:

```
朝:    git pull           (他で作業してたら)
作業:  コード書く
区切り: git add . && git commit -m "メッセージ"
終了:  git push
```

### コミットメッセージ

| OK | NG |
|---|---|
| "PDF抽出機能を実装" | "更新" |
| "メール送信のバグ修正" | "fix" |
| "READMEに使い方追加" | "edit" |

→ **後で見て分かる**ようにする。

---

## 7. ⭐ X(Twitter)運用開始(40分)

**半年〜1年で直接依頼が来る**ようになる。月100万には不可欠。

### プロフィール最適化

```
🐍 Python × 業務自動化 × AI連携
✅ 月20時間の作業を5秒に
✅ 中小企業のDX支援
✅ Claude/ChatGPT API活用
🔗 ポートフォリオ: <URL>
📩 DM相談OK
```

### 投稿ネタ(毎日1つ)

| 種類 | 例 |
|---|---|
| 成果報告 | 「PDF→Excel変換ツール作った。手作業5h→30秒」 |
| 技術tips | 「pandas で複数Excel統合する3行コード」 |
| 失敗談 | 「初案件で○○ミスった話」 |
| ROI事例 | 「クライアント年間60万円削減できた」 |
| 学び | 「クロード活用の生産性5倍テク」 |

### 成長曲線

| 期間 | フォロワー | 効果 |
|---|---:|---|
| 1ヶ月 | 100 | 知り合いに認知 |
| 3ヶ月 | 500 | 案件依頼が来始める |
| 6ヶ月 | 1,000 | **月数件の直接依頼** |
| 1年 | 5,000+ | 単価が言い値で通る |

→ **継続が全て**。1日1ツイート、3ヶ月続ける。

---

## 8. Note アカウント作成(30分)

### なぜ?

- 長文で実績・ノウハウを書ける
- SEOで検索流入
- 有料記事(将来的に)で副収入

### 最初の3記事(テーマ)

1. **「未経験から12日でPythonエンジニアデビューした記録」**
2. **「クライアント年商60万円削減した自動化事例」**
3. **「Claude APIで作った社内RAG Bot構築記」**

→ クロードに「テーマ〇〇でNote記事3,000字書いて」で雛形作成可。

---

## 9. 営業文テンプレ完成(30分)

Day 11 で使う営業文を**今のうちに作っておく**。

### ランサーズ提案文(クロードに頼む)

```
ランサーズ提案文の汎用テンプレを作って。

含めるべき要素:
1. 挨拶
2. 案件への理解
3. 解決方法
4. スケジュール
5. 価格
6. ポートフォリオURL
7. 質問事項
8. 締め

トーン: 丁寧 + プロフェッショナル
800-1200字
```

→ `templates/ランサーズ提案文集.md` に既にある。流用OK。

### 直接営業のDM(クロードに頼む)

```
中小企業へのDM文を書いて。

サービス:
- Python業務自動化
- AI連携(Claude/OpenAI)
- 月額メンテ可能

提供例:
- 月15時間削減の自動化
- AI議事録要約
- 社内文書RAG Bot

トーン: 専門家として、ROI訴求
500-800字
```

---

## ✅ チェック

- [ ] GitHubに5作品push、READMEあり
- [ ] プロフィールページREADME作成
- [ ] X/Twitter プロフィール設定 + 1投稿
- [ ] Note アカウント作成
- [ ] 営業文テンプレ完成

---

## 進捗

```
Day 10: GitHub・SNS完了。
- GitHub: https://github.com/xxx/python-portfolio
- Twitter: @xxx
- Note: https://note.com/xxx
3ヶ月発信継続を約束。
```

## 次

`11_ランサーズ.md` へ。**初受注を取りに行く**。
