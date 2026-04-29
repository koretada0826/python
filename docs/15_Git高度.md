# Day 15:Git 高度 - チーム開発の作法

## 💰 この章後の月収:**+5〜20万円**(チーム案件追加)
## ⏱ 所要:2-3時間

「**Git使えます**」と「**Git で実務こなせます**」は別物。
SES や法人案件は**ブランチ運用**ができないと話にならない。

---

## ゴール

- branch / merge / rebase 使える
- Pull Request の出し方
- conflict 解決
- GitHub Issues / Projects 活用
- コミット作法

---

## 1. ブランチ - 並行開発の基本(30分)

### なぜ必要?

```
main(本番) ─────────●──────────────●────────────●
                    \              \           /
                     feature-1      feature-2
                       (login)       (search)
```

→ **本番に影響なく**新機能を作れる。

### 操作

```bash
# 現在のブランチ確認
git branch

# 新ブランチ作成 + 切替
git checkout -b feature/login
# or
git switch -c feature/login

# 作業 → コミット
git add .
git commit -m "ログイン機能追加"

# main に戻る
git switch main

# main に取り込む
git merge feature/login

# ブランチ削除
git branch -d feature/login
```

### 命名規則(プロは守る)

| プレフィックス | 用途 | 例 |
|---|---|---|
| `feature/` | 新機能 | `feature/login` |
| `fix/` | バグ修正 | `fix/login-bug` |
| `hotfix/` | 緊急修正 | `hotfix/security` |
| `refactor/` | リファクタ | `refactor/api-clean` |
| `docs/` | ドキュメント | `docs/readme-update` |

---

## 2. Pull Request(PR)- レビュー文化(40分)

### 流れ

```
1. branch作成
2. 作業 + commit
3. push to GitHub
4. PR作成(GitHubのUI)
5. レビュー → 修正 → 承認
6. main にマージ
```

### コマンド

```bash
git switch -c feature/new

# 作業
git add .
git commit -m "新機能X追加"

# 初push(リモートにブランチ作る)
git push -u origin feature/new

# 以後の push
git push
```

### GitHub UI で PR

1. push 後、GitHub に「Compare & pull request」ボタン
2. タイトル: `[Feature] 新機能X追加`
3. 本文(テンプレ):

```markdown
## 概要
〇〇を実装しました。

## 変更内容
- ファイル A: 機能追加
- ファイル B: バグ修正

## 動作確認
- ✅ ローカルで動作確認済み
- ✅ テスト追加(test_xxx.py)

## スクショ
(あれば貼る)

## レビュアーへ
- 〇〇の設計について意見ほしい
- ✕✕の命名で迷った
```

→ **PR テンプレ**は `.github/PULL_REQUEST_TEMPLATE.md` に保存しておくと自動表示。

### コードレビューを受ける作法

- ✅ 指摘には**素直に対応**
- ✅ 反論する時は**理由を書く**
- ✅ 修正後は「修正しました」とコメント
- ❌ 感情的にならない

---

## 3. conflict(コンフリクト)解決(30分)

### 起きるシーン

`main` で同じファイルが変更され、自分の branch とぶつかる。

```bash
git switch main
git pull
git switch feature/login
git merge main
# CONFLICT (content): Merge conflict in app.py
```

### ファイルの中身

```python
<<<<<<< HEAD
print("自分の変更")
=======
print("他人の変更")
>>>>>>> main
```

### 解決手順

1. ファイルを開いて `<<<` `===` `>>>` を全部消す
2. **両方の意図を満たす**コードに書き直す
3. コミット

```bash
git add app.py
git commit -m "resolve conflict"
```

### コンフリクト避けるコツ

- **branch を長く生かさない**(早めにマージ)
- **小さい単位で push & PR**
- 他人が触ってるファイルに触る前に**Slack等で一声**

---

## 4. rebase vs merge(20分)

### merge

```
main:    A───B───C───────M
              \         /
feature:       D───E
```
→ コミット履歴がそのまま残る。

### rebase

```
main:    A───B───C
                  \
feature:           D'───E'
```
→ **直線的な履歴**になる。

### コマンド

```bash
git switch feature/login
git rebase main
# conflict 出たら解決 → git rebase --continue
```

### 使い分け

- **個人開発**: 好きな方
- **チーム開発**: 多くは「**main へ merge、ブランチ内では rebase**」
- **絶対NG**: 公開済みのコミットを rebase(チーム混乱)

---

## 5. よく使う高度コマンド(20分)

```bash
# 直前のコミットを修正
git commit --amend

# 過去のコミットを書き換え(対話的)
git rebase -i HEAD~3

# 一時退避(まだコミットしたくない)
git stash
git stash pop

# 特定のコミットだけ別branchに取り込む
git cherry-pick <コミットハッシュ>

# 他人の変更を取り込む(pull の安全版)
git fetch origin
git merge origin/main

# ファイルだけ過去の状態に戻す
git checkout <ハッシュ> -- app.py

# 直前の変更を取り消す(コミット前)
git restore app.py

# ブランチ削除(ローカル & リモート)
git branch -d feature/old
git push origin --delete feature/old

# 履歴をきれいに見る
git log --oneline --graph --all
```

---

## 6. .gitignore のプロ仕様(10分)

```
# Python
__pycache__/
*.pyc
*.pyo
.Python
venv/
.venv/
env/

# 環境変数
.env
.env.local
.env.*.local

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# ログ
*.log
logs/

# テスト
.pytest_cache/
.coverage
htmlcov/

# ビルド
dist/
build/
*.egg-info/

# 機密
secrets/
credentials.json

# データ(大きい)
*.csv
*.xlsx
data/
output/
```

→ クロードに「Python用の.gitignore作って」で完璧版もらえる。

---

## 7. GitHub Issues - タスク管理(15分)

PRと連携して使う。

### Issue 作成例

```
タイトル: ログイン後のリダイレクト先がおかしい

内容:
## 再現手順
1. /login にアクセス
2. ID/Pass入力
3. 送信
4. /home に飛ぶはずが /login に戻る

## 期待動作
/home に遷移

## 環境
- Python 3.12
- Flask 3.0
```

### PR で Issue を閉じる

PR の本文に書く:
```
Closes #42
```

→ PR がマージされると Issue 自動クローズ。

---

## 8. GitHub Projects - カンバン(15分)

Trello/Notion みたいなボード。GitHub内に組み込み。

```
Todo → In Progress → Review → Done
```

各カードを Issue / PR と連携。

→ **法人案件で「タスク管理は?」と聞かれたら GitHub Projects と答える**。

---

## 9. CI/CD 入門(GitHub Actions)(30分)

「**push したら自動でテスト実行**」する仕組み。

### `.github/workflows/test.yml`

```yaml
name: Test

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: pytest
```

→ push する度に GitHub が自動でテスト実行 → 失敗したら通知。

### 効果

- バグの早期発見
- PR レビューが楽(緑チェック確認だけ)
- **「CI/CD 経験あり」がスキル欄に書ける**

---

## 10. 実務でやらかさない7つの作法

1. **本番ブランチ(main)に直接 push しない**
2. **commit メッセージは何の変更か分かる文に**
3. **公開済みコミットを rebase しない**
4. **大きなファイルを push しない**(.gitignoreに入れる)
5. **`.env` を絶対にコミットしない**
6. **`git push --force` は自分のブランチだけ**
7. **conflict は焦らず1ファイルずつ解決**

---

## ✅ チェック

- [ ] branch 作成・切替・削除できる
- [ ] PR 出せる(自分のリポジトリで練習)
- [ ] conflict 解決経験
- [ ] rebase / merge の違いわかる
- [ ] .gitignore プロ仕様
- [ ] Issue / Projects 触った
- [ ] GitHub Actions で CI 動いた
- [ ] 7作法理解

---

## 進捗

```
Day 15: Git高度完了。branch/PR/rebase/conflict/CI 理解。
SES/チーム案件への準備完了。
```

## 次

`16_Docker.md` へ。**環境構築・デプロイの標準ツール**。
