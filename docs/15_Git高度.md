# Day 15:Git 高度 - チーム開発の作法

## 💡 これは何?(小学生でもわかる説明)

**チームで同じコードを編集しても壊れない仕組み(branch・PR・merge)を覚える日**

### なぜ大事?
1人開発 → 月60万、**チーム参加できる → 月100万**。
SES や法人案件は「ブランチ運用ができない人」とは契約しない。
**Gitの作法を知ってるかどうかで応募できる案件の数が3倍違う**。

### イメージ
- **ブランチ**=「**作業用の別世界(平行宇宙)**」(本物に影響なく試せる)
- **merge**=「**別世界の成果を本物に取り込む**」
- **rebase**=「**歴史を書き直して綺麗にする**」(プロが好む)
- **PR**=「**先生に提出して赤ペンチェックを受ける**」(レビュー文化)
- **conflict**=「**書類の同じ行を2人が違うこと書いた状態**」(直し方がある)

### Claude Code に何を頼めばいいか
コミットメッセージ・PR本文・conflict解決・CI設定は全部クロード。
あなたは「**何を作業するか**」「**いつ統合するか**」を判断するだけ。

---

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

## 1. ブランチ - 並行開発の基本(=平行宇宙を作る)(30分)

### 1-1. なぜ必要?

> **本番(main)を直接触ると、バグが出た瞬間にサービス停止**。
> 別の世界(=ブランチ)で作業して、完成したものだけ本番に取り込めば安全。

```
main(本番) ─────────●──────────────●────────────●
                    \              \           /
                     feature-1      feature-2
                       (login)       (search)
```

→ **本番に影響なく**新機能を作れる。

### 1-2. 操作

```bash
# 現在のブランチ確認
git branch

# 新ブランチ作成 + 切替
git checkout -b feature/login
# or(新しい書き方)
git switch -c feature/login

# 作業 → コミット
git add .
git commit -m "ログイン機能追加"

# main に戻る
git switch main

# main に取り込む(merge)
git merge feature/login

# 用済みブランチを削除
git branch -d feature/login
```

### 1-3. 命名規則(プロは守る)

| プレフィックス | 用途 | 例 |
|---|---|---|
| `feature/` | 新機能 | `feature/login` |
| `fix/` | バグ修正 | `fix/login-bug` |
| `hotfix/` | 緊急修正 | `hotfix/security` |
| `refactor/` | リファクタ | `refactor/api-clean` |
| `docs/` | ドキュメント | `docs/readme-update` |

> **なぜ命名規則?**
> 100個のブランチがある時、`feature/` を見れば「あ、新機能か」と一発で分かる。
> プロのチームは**ブランチ名を見るだけで内容が分かる**。

### よくある間違い

| ミス | 何がダメ | 直し方 |
|---|---|---|
| `main` で直接作業 | 巻き戻し不可、本番事故 | 必ず `git switch -c feature/xxx` から |
| ブランチ名が日本語 | 文字化け・サーバー側で問題 | 英語(feature/login)で統一 |
| 1ブランチで100コミット | レビュー不可、巻き戻せない | 1機能=1ブランチ、PR後すぐ削除 |

---

## 2. Pull Request(PR)- レビュー文化(=赤ペン先生に見せる)(40分)

### 2-1. なぜPR?

> **他人の目で「本当に大丈夫?」をチェックしてもらう仕組み**。
> 1人だと気づかないバグも、レビューで「ここ変じゃない?」と指摘される。
> **PRなしで本番反映するチームは、ほぼ存在しない**。

### 2-2. 流れ

```
1. branch作成
2. 作業 + commit
3. push to GitHub
4. PR作成(GitHubのUI)
5. レビュー → 修正 → 承認
6. main にマージ
```

### 2-3. コマンド

```bash
git switch -c feature/new

# 作業
git add .
git commit -m "新機能X追加"

# 初push(リモートにブランチ作る、-uで紐付け)
git push -u origin feature/new

# 以後の push
git push
```

### 2-4. GitHub UI で PR

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

### 2-5. コードレビューを受ける作法

| やる | やらない |
|---|---|
| ✅ 指摘には**素直に対応** | ❌ 感情的になる |
| ✅ 反論する時は**理由を書く** | ❌ 「○○が悪い」と人格批判 |
| ✅ 修正後は「修正しました」とコメント | ❌ 無言でforce push |

> **指摘=攻撃ではない**。コードを良くする提案。
> プロほど「指摘してくれてありがとう」のスタンス。

---

## 3. conflict(コンフリクト)解決(=書類の同じ行で揉めた時)(30分)

### 3-1. 起きるシーン

`main` で同じファイルが変更され、自分の branch とぶつかる。

```bash
git switch main
git pull
git switch feature/login
git merge main
# CONFLICT (content): Merge conflict in app.py
```

### 3-2. ファイルの中身

```python
<<<<<<< HEAD
print("自分の変更")
=======
print("他人の変更")
>>>>>>> main
```

> **読み方:**
> - `<<<<<<< HEAD` から `=======` まで → **自分の変更**
> - `=======` から `>>>>>>> main` まで → **相手の変更**
> - 両方の意図を満たすコードに書き直して、`<<<` 系の記号は全部消す

### 3-3. 解決手順

1. ファイルを開いて `<<<` `===` `>>>` を全部消す
2. **両方の意図を満たす**コードに書き直す
3. コミット

```bash
git add app.py
git commit -m "resolve conflict"
```

> **困ったらクロードに見せる:**
> ```
> このコンフリクトを解決して:
> <ファイル中身そのままコピペ>
> ```

### 3-4. コンフリクト避けるコツ

- **branch を長く生かさない**(早めにマージ)
- **小さい単位で push & PR**
- 他人が触ってるファイルに触る前に**Slack等で一声**

---

## 4. rebase vs merge(=履歴の作り方2種類)(20分)

### 4-1. merge(=2本の道がY字で合流)

```
main:    A───B───C───────M
              \         /
feature:       D───E
```
→ コミット履歴がそのまま残る(分岐がそのまま記録)。

### 4-2. rebase(=自分の歴史を最新の上に並べ直す)

```
main:    A───B───C
                  \
feature:           D'───E'
```
→ **直線的な履歴**になる(分岐が消える)。

### 4-3. コマンド

```bash
git switch feature/login
git rebase main
# conflict 出たら解決 → git rebase --continue
```

### 4-4. 使い分け

- **個人開発**: 好きな方
- **チーム開発**: 多くは「**main へ merge、ブランチ内では rebase**」
- **絶対NG**: 公開済みのコミットを rebase(チーム混乱)

> **覚え方:**
> - merge = 履歴が複雑だが安全
> - rebase = 履歴がきれいだが危険(=押し付け系)

---

## 5. よく使う高度コマンド(=便利な道具箱)(20分)

```bash
# 直前のコミットを修正(コミットメッセージ修正・ファイル追加忘れ等)
git commit --amend

# 過去のコミットを書き換え(対話的)
git rebase -i HEAD~3

# 一時退避(まだコミットしたくない)
git stash         # 退避
git stash pop     # 戻す

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

> **stash の使いどころ:**
> 「コミットしたくない一時的な実験中」に、急に main で別作業が必要になった時。
> stash で退避 → main で作業 → 戻ってきて pop で復帰。

---

## 6. .gitignore のプロ仕様(=漏れちゃ困るリスト)(10分)

```
# Python
__pycache__/
*.pyc
*.pyo
.Python
venv/
.venv/
env/

# 環境変数(=絶対に上げてはいけない機密)
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

> **特に重要:**
> - `.env` を上げると **API Keyが流出 → 数分で悪用される**
> - `*.csv` を上げると個人情報漏洩リスク
> - `__pycache__` はリポジトリ容量を無駄に増やす

---

## 7. GitHub Issues - タスク管理(=作業チケット)(15分)

PRと連携して使う。

### 7-1. Issue 作成例

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

### 7-2. PR で Issue を閉じる

PR の本文に書く:
```
Closes #42
```

→ PR がマージされると Issue 自動クローズ。

> **メリット:** Issueとコードが紐付くので「いつ・どのコミットでこのバグが直ったか」が後から追える。

---

## 8. GitHub Projects - カンバン(=タスクボード)(15分)

Trello/Notion みたいなボード。GitHub内に組み込み。

```
Todo → In Progress → Review → Done
```

各カードを Issue / PR と連携。

→ **法人案件で「タスク管理は?」と聞かれたら GitHub Projects と答える**。

---

## 9. CI/CD 入門(GitHub Actions)(=自動味見ロボット)(30分)

「**push したら自動でテスト実行**」する仕組み。

### 9-1. なぜ必要?

> **人間は忘れる**。テスト実行を毎回手でやるとサボる。
> CIなら**push する度に自動で実行**、失敗したら通知が飛ぶ。

### 9-2. 設定ファイル `.github/workflows/test.yml`

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

### 9-3. 効果

- バグの早期発見
- PR レビューが楽(緑チェック確認だけ)
- **「CI/CD 経験あり」がスキル欄に書ける**

---

## 10. 実務でやらかさない7つの作法(=禁忌の数々)

| # | 作法 | やらかすとどうなる |
|---|---|---|
| 1 | 本番ブランチ(main)に直接 push しない | 本番事故、巻き戻し不可 |
| 2 | commit メッセージは何の変更か分かる文に | レビュー不能、後で追えない |
| 3 | 公開済みコミットを rebase しない | チーム全員の履歴がぶっ壊れる |
| 4 | 大きなファイルを push しない | リポジトリ容量爆発 |
| 5 | `.env` を絶対にコミットしない | API Key 流出、数分で悪用 |
| 6 | `git push --force` は自分のブランチだけ | 他人のコミットが消滅 |
| 7 | conflict は焦らず1ファイルずつ解決 | コード破壊、戻せない事故 |

> **特に5番**: `.env`を上げた瞬間にOpenAI/Anthropicのキーが流出。
> ボットが秒で発見し、$10,000規模の請求が来る事例あり。
> Git履歴に1度入ったら**BFG Repo-Cleanerで完全削除**+**キー即無効化**。

---

## 11. 今日学んだ用語まとめ

| 用語 | 一言で |
|---|---|
| ブランチ | 作業用の別世界(平行宇宙) |
| main | 本番ブランチ(直接触らない) |
| commit | 変更を保存する単位 |
| push | リモート(GitHub)に送る |
| pull | リモートから取り込む |
| merge | 2本のブランチを合流させる |
| rebase | 自分の履歴を最新の上に並べ直す |
| Pull Request(PR) | レビュー依頼の仕組み |
| conflict | 同じ行を2人が違うこと書いた状態 |
| .gitignore | 上げないファイルを書くリスト |
| Issue | 作業チケット |
| GitHub Actions | 自動テスト・自動デプロイ(CI/CD) |
| CI/CD | 継続的インテグレーション/デリバリー |
| stash | 一時的に変更を退避する |
| cherry-pick | 特定のコミットだけ取り込む |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **チーム開発参画(SES / 業務委託)**:月単価 60〜90万円(プラットフォーム例:レバテック / ITプロパートナーズ / Midworks)
- **既存リポジトリの整理・GitHub Actions セットアップ**:単価 5〜20万円(CI / CD 導入支援)
- **OSS 風に GitHub 公開する社内ツール開発**:単価 10〜30万円(Issue / PR ベースで対応)

### クロードへの頼み方(営業文を書かせる例)
```
SES エージェント(レバテック等)に提出するスキルシート(A4 2枚)の文面を書いて。
Day 15 までで習得:Python / pandas / Web開発(Streamlit) / DB(SQL / SQLAlchemy) / Git高度(branch / PR / rebase / conflict解決) / GitHub Actions
稼働形態:週3〜5日 / 月60〜80万円希望 / リモート歓迎
強み:1人でMVPまで作れる + チーム開発のお作法(PRレビュー / CI)を理解
過去実績:GitHubポートフォリオURL貼る
弱み:大規模開発経験は浅いので、その分PRごとの丁寧なレビュー対応で補う
```

### この章だけでは足りないもの(次に学ぶべき)
- Docker(Day 16)で環境を統一しチーム開発に入りやすく
- AWS(Day 17)で本番環境まで触れるエンジニアへ
- API設計・テスト品質(Day 18〜19)で月単価70万→90万へ

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
