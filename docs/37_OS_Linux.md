# Day 37:OS・Linux・systemd・プロセス管理

## 💡 これは何?(小学生でもわかる説明)

**Linux サーバー(本番環境の王様)を自分で操れるようになる日**

### なぜ大事?
本番サーバーの**90%は Linux**。世界中のWebサービス・銀行・ECサイトが Linux で動いている。

普通のエンジニア = ローカル(Mac/Win)で開発 → デプロイは「インフラ屋に任せる」
**Linux 触れるエンジニア** = 本番サーバーまで自分で見れる → 単価+20万

なぜ単価が上がる?
- 本番障害が起きた時、**深夜に呼ばれて対応できる**
- インフラ屋を雇わずに済む(企業のコスト削減)
- 開発から運用まで一人で回せる(=フルスタック)

### イメージ
- **OS**=「**車のエンジンと運転席**」
  - スマホ・PCも内部はOSが動かしてる
- **Mac/Win**=「**普通乗用車**」(免許あれば誰でも乗れる)
- **Linux**=「**重機・トラック**」(操作覚えると現場で重宝される)
- **systemd**=「**お店のシャッター係**」(朝開けて夜閉める、勝手に再起動)
- **cron**=「**毎日決まった時間に来る配達員**」
- **nginx**=「**受付係**」(来客を適切な部屋に案内)

> スマホもパソコンも**OS**(基本ソフト)が動かしてる。
> サーバーの世界では**Linux**が王様。「Linux 触れる」=「サーバー触れる」。

### Claude Code に何を頼めばいいか
ここで覚えるのは「**コマンド名**」「**何のために使うか**」。
細かい構文・設定ファイルは Claude Code に書かせる。

---

## 💰 +20-50万(SRE/インフラ)
## ⏱ 20時間

サーバーが**Linux** = 9割。**Linux 触れます**で単価+20万。

> **なぜそんなに上がる?**
> Web系企業ほぼ全社が Linux サーバー。
> 「本番障害対応できる人」は希少なので**夜間オンコール手当**で月+5万も普通。

---

## ゴール

- 必須Linuxコマンド30個を使える
- systemd でアプリを常駐サービス化できる
- cron で定期実行ができる
- SSH 鍵認証でサーバー接続
- nginx + Let's Encrypt で HTTPS化
- ログ調査・パフォーマンス調査ができる

---

## 1. シェル・コマンド(必修30個)(40分)

### 1-1. ファイル操作

```bash
# 一覧・移動・現在地
ls -la             # ファイル一覧(隠しファイル含む)
cd /path           # ディレクトリ移動
pwd                # 現在地表示

# 作る・消す・コピー
mkdir -p a/b/c     # 階層フォルダ作成
rm -rf folder      # 削除(危険、確認なし)
cp src dst         # コピー
mv old new         # 移動・リネーム

# 検索・閲覧
find . -name "*.py"       # ファイル検索
cat file                  # 中身全部表示
less file                 # ページ単位表示(qで終了)
head -20 file             # 先頭20行
tail -f log.txt           # リアルタイムで末尾追跡
```

### 1-2. テキスト処理(エンジニアの命)

```bash
grep "error" log              # 文字検索
sed -i 's/a/b/g' f            # 置換(in-place)
awk '{print $1}'              # 1列目だけ取り出す
sort | uniq -c | sort -rn     # 集計(出現回数ランキング)
```

> **集計の魔法:**
> ログから「アクセス多いIP上位10」を1行で出せる。
> ```
> awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
> ```

### 1-3. プロセス管理

```bash
ps aux | grep python          # Pythonプロセス一覧
top                            # CPU/メモリ使用率
htop                           # top のきれい版
kill -9 PID                    # プロセス強制終了
nohup python app.py &          # 終了後も実行継続
```

### 1-4. ネットワーク

```bash
curl -i https://...            # HTTP リクエスト
wget URL                       # ファイルダウンロード
ssh user@host                  # サーバー接続
scp file user@host:/path       # ファイル転送
ping host                      # 疎通確認
```

### 1-5. パイプ・リダイレクト(超重要)

```bash
cmd1 | cmd2        # cmd1の出力をcmd2の入力へ(=ホースで繋ぐ)
cmd > file         # 結果をファイルに上書き
cmd >> file        # 結果をファイルに追記
cmd 2>&1           # エラー出力も標準出力に流す
cmd 2>/dev/null    # エラーは捨てる
```

> **イメージ:** UNIXパイプは「**水道のホース**」。
> 1つのコマンドの出力を、次のコマンドの入力にホースで繋ぐ。
> これでシンプルなコマンドを組み合わせて巨大な処理ができる。

### 1-6. 権限

```bash
chmod +x script.sh        # 実行権限付与
chown user:group file     # 所有者変更
sudo cmd                  # 管理者権限で実行
```

→ **これだけで90%の作業ができる**。

---

## 2. systemd(サービス管理)(30分)

### 2-1. なぜ必要?

`python app.py` で起動したアプリは、
- ログアウトしたら**死ぬ**
- サーバー再起動したら**死ぬ**
- クラッシュしても**自動復活しない**

→ これを解決するのが **systemd**。
「**お店のシャッター係**」のように、自動で起動・監視・再起動してくれる。

### 2-2. サービス化

`/etc/systemd/system/myapp.service`:
```ini
[Unit]
Description=My App
After=network.target          # ネットワーク準備後に起動

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/app
ExecStart=/usr/bin/python /home/ubuntu/app/main.py
Restart=on-failure            # 失敗したら再起動
RestartSec=10                 # 10秒待ってから再起動
Environment="API_KEY=xxx"

[Install]
WantedBy=multi-user.target
```

### 2-3. 操作コマンド

```bash
sudo systemctl daemon-reload       # 設定変更後に必須
sudo systemctl start myapp         # 起動
sudo systemctl stop myapp          # 停止
sudo systemctl restart myapp       # 再起動
sudo systemctl enable myapp        # 起動時自動実行
sudo systemctl status myapp        # 状態確認
sudo journalctl -u myapp -f        # ログ追跡(リアルタイム)
```

→ **本番サーバーで Pythonアプリを常駐させる**標準。

---

## 3. cron(定期実行)(15分)

### 3-1. 設定

```bash
crontab -e
```

### 3-2. 書式

```bash
# 分 時 日 月 曜日 コマンド

# 毎分実行
* * * * * /usr/bin/python /home/ubuntu/script.py

# 毎日朝6時(ログも保存)
0 6 * * * /usr/bin/python /home/ubuntu/daily.py >> /var/log/daily.log 2>&1

# 月曜の朝9時
0 9 * * 1 /usr/bin/python /home/ubuntu/weekly.py
```

> **書式の覚え方:** 左から **「分・時・日・月・曜日」**。
> `*` は「全部」、数字で固定。

→ Day 6 の**ライブ予測スクリプト**を毎分実行とかに使う。

### 3-3. よくある罠

| 罠 | 対処 |
|---|---|
| 環境変数が無い | `crontab -e` の冒頭で `PATH=` を明示 |
| Pythonのパスが違う | `which python` の絶対パスで書く |
| ログが残らない | `>> /path/to/log 2>&1` を必ず付ける |

---

## 4. ファイル権限(15分)

### 4-1. 読み方

```bash
ls -l
# -rwxr-xr-x  user group  ...
# rwx | r-x | r-x
# 所有者 | グループ | その他全員
#  r=read, w=write, x=execute
```

### 4-2. 数字での指定

```bash
chmod 755 file   # rwxr-xr-x  (自分:読書実行、他:読実行)
chmod 644 file   # rw-r--r--  (自分:読書、他:読のみ)
chmod 600 .env   # rw-------  (自分だけ読書、秘密ファイル)

chown user:group file   # 所有者・グループ変更
```

> **計算式:** r=4, w=2, x=1 を足す
> - 7 = rwx(全部)
> - 6 = rw-(読書)
> - 5 = r-x(読実行)
> - 4 = r--(読のみ)

→ `.env` は **600**(自分だけ読める)が必須。これを公開すると**APIキー流出事故**。

---

## 5. SSH 鍵認証(20分)

### 5-1. なぜ鍵認証?

- パスワードは**総当たり攻撃で破られる**
- 鍵認証なら**実質破られない**(ed25519 は量子コンピュータが来るまで安全)

### 5-2. 設定手順

```bash
# 1. 鍵生成
ssh-keygen -t ed25519 -C "your_email@example.com"

# 2. 公開鍵をサーバーに登録
ssh-copy-id user@host

# 3. 接続(パスワード不要)
ssh user@host

# 4. Config で別名・短縮設定
cat >> ~/.ssh/config <<EOF
Host myserver
  HostName 1.2.3.4
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519
EOF

ssh myserver  # 短く繋がる
```

→ パスワード認証は**廃止**するのが本番セキュリティ。

### 5-3. 鍵の使い分け

| 鍵 | 用途 |
|---|---|
| 秘密鍵 (`id_ed25519`) | 自分だけが持つ、絶対に公開しない |
| 公開鍵 (`id_ed25519.pub`) | サーバーに登録、公開してOK |

> **絶対やってはいけない:**
> 秘密鍵を GitHub にコミット。即時アカウント乗っ取られる。

---

## 6. nginx(リバースプロキシ)(20分)

### 6-1. なぜ必要?

- 1サーバーで複数アプリを動かしたい
- HTTPS 終端をしたい
- 静的ファイル配信を高速化したい
- アクセス制限・ロードバランス

### 6-2. 設定例

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:8000;       # FastAPIへ転送
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        alias /var/www/static/;                 # 静的ファイル直接配信
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t                       # 文法チェック
sudo systemctl reload nginx         # 反映
```

→ 1ホストで複数アプリ・SSL終端・静的ファイル配信。

### 6-3. Let's Encrypt SSL(無料HTTPS)

```bash
sudo certbot --nginx -d api.example.com
```

→ 無料の SSL 証明書、**3ヶ月毎自動更新**。

> **HTTPS必須の時代:**
> Chromeが「HTTPSじゃないサイト」を**警告表示**する。
> SEO的にも、信頼性的にもHTTPS必須。

---

## 7. ログ調査(15分)

### 7-1. 主要ログ

```bash
# システムログ
sudo journalctl -xe                # 最新の重要ログ
sudo journalctl -u nginx -f        # nginx だけリアルタイム

# アプリログ
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# 大量ログから抽出
grep -i error /var/log/syslog | head -20
zcat /var/log/syslog.1.gz | grep error    # 圧縮ログも検索
```

### 7-2. 障害対応の鉄則

| 順番 | やること |
|---|---|
| 1 | `journalctl -xe` で最新エラー確認 |
| 2 | `top` / `htop` で負荷確認 |
| 3 | `df -h` でディスク空き確認 |
| 4 | `systemctl status` で関連サービス確認 |
| 5 | アプリログ追跡 |

---

## 8. パフォーマンス調査(15分)

```bash
top              # CPU/メモリ使用率(リアルタイム)
htop             # top のきれい版・操作しやすい
free -h          # メモリ詳細
df -h            # ディスク容量
du -sh *         # ディレクトリ別サイズ
iostat -x 1      # ディスクIO(1秒ごと)
netstat -tlnp    # ポート開放確認(古いコマンド)
ss -tlnp         # 新版(高速)
```

→ 障害時に**何が原因か**5分で切り分け。

> **障害切り分けの目安:**
> - CPU 100% → アプリの無限ループ・重い処理
> - メモリ枯渇 → メモリリーク・キャッシュ過多
> - ディスク満杯 → ログ肥大・古いデータ
> - ポート閉鎖 → サービス停止・nginx設定ミス

---

## 9. Docker on Linux(15分)

```bash
# Ubuntu に Docker 入れる
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER       # sudo無しでdocker使えるようにする
# 一旦ログアウト→再ログイン

docker run -d -p 80:80 nginx        # nginxコンテナ起動
docker ps                            # 起動中コンテナ
docker logs container_id             # ログ
docker exec -it container_id /bin/bash   # コンテナの中に入る
```

---

## 10. ⭐ 課題

```bash
1. AWS EC2(Ubuntu)立ち上げ → SSH 鍵で接続
2. Day 19 の FastAPI を動かす
3. systemd でサービス化(再起動でも自動起動)
4. nginx + Let's Encrypt で HTTPS化
5. ログを journalctl で確認
6. 不要プロセス kill -9 で殺す経験
```

---

## 11. よくある間違い・エラー表

| 症状 | 原因 | 対処 |
|---|---|---|
| `Permission denied` | 権限不足 | `sudo` または `chmod` |
| `command not found` | PATH に無い | 絶対パスで指定、または `which` で確認 |
| systemd で起動失敗 | 設定ミス、パス相対 | `journalctl -u myapp -xe` で確認 |
| cron が動かない | 環境変数がない | `crontab -e` で `PATH=...` 明示 |
| nginx 502 Bad Gateway | バックエンド落ちてる | アプリの稼働確認 |
| `disk full` | ログ肥大 | `du -sh *` で犯人特定、ログローテーション |
| SSH接続できない | 鍵権限600じゃない | `chmod 600 ~/.ssh/id_ed25519` |
| `kill -9` でも消えない | カーネルレベルでハング | サーバー再起動 |

---

## 12. 今日学んだ用語まとめ

| 用語 | 一言で |
|---|---|
| シェル | コマンドを打つ画面(bash, zsh) |
| パイプ `\|` | コマンド出力を次の入力に渡す |
| systemd | サービスを常駐管理する仕組み |
| journalctl | systemd のログを見るコマンド |
| cron | 定期実行スケジューラ |
| chmod | 権限変更 |
| chown | 所有者変更 |
| SSH | 安全なリモート接続 |
| 鍵認証 | パスワード代わりの暗号鍵で認証 |
| nginx | 高速Webサーバー・リバースプロキシ |
| リバースプロキシ | 来客を適切な部屋に案内する係 |
| Let's Encrypt | 無料SSL証明書サービス |
| certbot | Let's Encrypt 自動取得ツール |
| top / htop | CPU・メモリ監視 |
| netstat / ss | ポート確認 |

---

## 💼 この章まで終わったら受けられる案件

### 受けられる案件(具体例)
- **サーバー管理・SRE 案件**:月単価 60〜120万円(プラットフォーム例:レバテック/Findy Freelance/SES)
- **VPS / オンプレ運用代行**:単価 5〜30万円/月(継続契約)
- **Linux サーバートラブルシューティング**:時給 8,000〜15,000円

### クロードへの頼み方(営業文を書かせる例)
```
SES エージェント向け、SRE / インフラ寄りスキルシート文を書いて。
Day 37 までで習得:Linux コマンド / systemd / cron / SSH鍵 / nginx + SSL / ログ追跡
稼働:月稼働 100〜140時間 / 月70〜100万円
売り:アプリ開発(Python)経験もあり、運用と開発を行き来できる
過去実績:VPS 上に Django / FastAPI を本番運用したサンプル
注意:大規模(数百ノード)経験は限定的、中堅規模を中心に伴走したい旨を率直に
```

### この章だけでは足りないもの(次に学ぶべき)
- ネットワーク(Day 38) / セキュリティ深掘り(Day 39)
- テスト戦略(Day 40)
- 大規模本番運用(Day 30 K8s / IaC)

---

## ✅ チェック

- [ ] Linux コマンド30個使える
- [ ] systemd でサービス化
- [ ] cron で定期実行設定
- [ ] SSH 鍵認証ログイン
- [ ] nginx + SSL 設定
- [ ] ログ追跡できる
- [ ] **EC2 で本番運用経験**

## 次
`38_ネットワーク.md` で **TCP/IP・HTTP・DNSの仕組み** を学ぶ。
