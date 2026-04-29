# Day 37:OS・Linux・systemd・プロセス管理

## 💰 +20-50万(SRE/インフラ)
## ⏱ 20時間

サーバーが**Linux** = 9割。**Linux 触れます**で単価+20万。

---

## 小学生説明

スマホもパソコンも**OS**(基本ソフト)が動かしてる。
サーバーの世界では**Linux**が王様。「Linux 触れる」=「サーバー触れる」。

---

## 1. シェル・コマンド(必修30個)(40分)

```bash
# ファイル
ls -la             # 一覧
cd /path           # 移動
pwd                # 現在地
mkdir -p a/b/c     # フォルダ作成
rm -rf folder      # 削除(危険)
cp src dst         # コピー
mv old new         # 移動・リネーム
find . -name "*.py"  # 検索
cat file           # 中身表示
less file          # ページ表示(q で終了)
head -20 file      # 先頭20行
tail -f log.txt    # リアルタイム末尾
grep "error" log   # 検索
sed -i 's/a/b/g' f # 置換
awk '{print $1}'   # 1列目
sort | uniq -c | sort -rn  # 集計

# プロセス
ps aux | grep python
top                # CPU/メモリ
htop               # きれい版
kill -9 PID
nohup python app.py &  # 終了後も実行継続

# ネットワーク
curl -i https://...
wget URL
ssh user@host
scp file user@host:/path
ping host

# パイプ・リダイレクト
cmd1 | cmd2        # cmd1の出力をcmd2に
cmd > file         # 上書き
cmd >> file        # 追記
cmd 2>&1           # エラーも標準出力に

# 権限
chmod +x script.sh
chown user:group file
sudo cmd           # 管理者権限
```

→ **これだけで90%の作業ができる**。

---

## 2. systemd(サービス管理)(30分)

### サービス化

`/etc/systemd/system/myapp.service`:
```ini
[Unit]
Description=My App
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/app
ExecStart=/usr/bin/python /home/ubuntu/app/main.py
Restart=on-failure
RestartSec=10
Environment="API_KEY=xxx"

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start myapp
sudo systemctl enable myapp     # 起動時自動実行
sudo systemctl status myapp
sudo journalctl -u myapp -f     # ログ追跡
```

→ **本番サーバーで Pythonアプリを常駐させる**標準。

---

## 3. cron(定期実行)(15分)

```bash
crontab -e

# 毎分
* * * * * /usr/bin/python /home/ubuntu/script.py

# 毎日朝6時
0 6 * * * /usr/bin/python /home/ubuntu/daily.py >> /var/log/daily.log 2>&1

# 月曜の朝9時
0 9 * * 1 ...
```

→ Day 6 の**ライブ予測スクリプト**を毎分実行とかに使う。

---

## 4. ファイル権限(15分)

```bash
ls -l
# -rwxr-xr-x  user group  ...
# rwx | r-x | r-x
# 所有者 | グループ | 全員

chmod 755 file   # rwxr-xr-x
chmod 644 file   # rw-r--r--
chmod 600 .env   # rw------- (秘密)

chown user:group file
```

→ `.env` は **600**(自分だけ読める)が必須。

---

## 5. SSH 鍵認証(20分)

```bash
# 鍵生成
ssh-keygen -t ed25519 -C "your_email@example.com"

# 公開鍵をサーバーに登録
ssh-copy-id user@host

# 接続(パスワード不要)
ssh user@host

# Config(別名・短縮)
cat >> ~/.ssh/config <<EOF
Host myserver
  HostName 1.2.3.4
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519
EOF

ssh myserver  # 短く
```

→ パスワード認証は**廃止**するのが本番セキュリティ。

---

## 6. nginx(リバースプロキシ)(20分)

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /static/ {
        alias /var/www/static/;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

→ 1ホストで複数アプリ・SSL終端・静的ファイル配信。

### Let's Encrypt SSL

```bash
sudo certbot --nginx -d api.example.com
```

→ 無料の SSL 証明書、3ヶ月毎自動更新。

---

## 7. ログ調査(15分)

```bash
# システムログ
sudo journalctl -xe
sudo journalctl -u nginx -f
tail -f /var/log/nginx/access.log

# 大量ログから抽出
grep -i error /var/log/syslog | head -20
zcat /var/log/syslog.1.gz | grep error
```

---

## 8. パフォーマンス調査(15分)

```bash
top              # CPU/メモリ
htop             # きれい版
free -h          # メモリ詳細
df -h            # ディスク
du -sh *         # ディレクトリ別サイズ
iostat -x 1      # ディスクIO
netstat -tlnp    # ポート開放確認
ss -tlnp         # 新版
```

→ 障害時に**何が原因か**5分で切り分け。

---

## 9. Docker on Linux(15分)

```bash
# Ubuntu に Docker 入れる
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
# 一旦ログアウト

docker run -d -p 80:80 nginx
docker ps
docker logs container_id
docker exec -it container_id /bin/bash
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

## ✅ チェック

- [ ] Linux コマンド30個使える
- [ ] systemd でサービス化
- [ ] cron で定期実行設定
- [ ] SSH 鍵認証ログイン
- [ ] nginx + SSL 設定
- [ ] ログ追跡できる
- [ ] **EC2 で本番運用経験**

## 次
`38_ネットワーク.md`
