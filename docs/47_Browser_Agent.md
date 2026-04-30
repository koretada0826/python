# Day 47:Browser Agent - 自律的にブラウザ操作するAI

## 💡 これは何?(小学生でもわかる説明)

**AIが自分でブラウザ操作**して仕事する

### なぜ大事?
ChatGPT Operator 級の最先端。月単価200万円。

普通のスクレイピングは「**プログラマが手順を書く**」必要があった。
Browser Agent は「**目的だけ**伝えると、AIが画面を見ながら自分で操作する」。

「アマゾンで Switch を検索してTOP3の価格教えて」
→ AIが**自分でアマゾンを開き・検索し・読み取って報告**してくれる。

これができれば、**人間がやってる単純作業**ほぼ全部置き換えられる。

### イメージ
- **Browser Agent** = 「**ブラウザの中で動く新人バイト**」
  - 画面のスクショを見る
  - 「次にどこをクリック?」をLLMに聞く
  - クリック・入力する
  - 目標達成までこれを繰り返す

### 比喩でひとこと
今までの自動化が「**マクロ(同じ手順を繰り返す)**」だったのが、
Browser Agent は「**現場で判断するバイト**」。
画面が変わっても**自分で考えて対応**する。
だから保守も**ほぼ不要**で、案件は契約継続しやすい。

### Claude Code に何を頼めばいいか
- 「Playwright で Google に行って X を検索して結果取ってくる Agent」
- 「browser-use で求人サイトから条件マッチ案件抽出」
- 「画面崩れに強いリトライ込みのフォーム入力Bot」
こういう粒度で頼める。

---

## 💰 +50-200万(2026最先端)
## ⏱ 15時間

ChatGPT Operator 級の「**AIが自分でブラウザ操作**」する技術。
**できる人がほぼいない**ので、新領域の単価最先端。

### 案件単価感

| 案件 | 単価 |
|---|---|
| 競合価格チェック完全自動化 | 月10-30万 |
| 求人サイト自動巡回・抽出 | 月10-20万 |
| フォーム入力代行AI | 月20-50万 |
| 業務システムの自動操作 | 月50-200万 |
| 在庫・予約サイト自動巡回 | 月20-50万 |

---

## ゴール

- **Playwright + LLM** の基本構造が組める
- **browser-use** で素早くPoCを作れる
- **Claude Computer Use** や ChatGPT Operator の存在を知る
- 安定運用のための**リトライ・ロギング**を設計できる

---

## 1. なぜ普通のスクレイピングじゃダメ?

| 観点 | 普通のスクレイピング | Browser Agent |
|---|---|---|
| 仕組み | DOM のセレクタ指定 | 画面を見て判断 |
| 画面崩れ | **すぐ壊れる** | 強い |
| ログイン突破 | 個別実装が必要 | LLM が考える |
| メンテ頻度 | 月1〜週1で修正 | ほぼ不要 |
| 開発時間 | サイトごとに数日 | 数時間 |
| **単価** | 10-30万 | **50-200万** |

> **比喩:** 普通のスクレイピング = 「**地図を渡されたバイト**」(地図が変わると迷子)
> Browser Agent = 「**頭が良いバイト**」(地図がなくても自分で目的地に着く)

---

## 2. Playwright + LLM(40分)

### 2-1. Playwright とは

> **イメージ:** ブラウザを**Pythonから操作する**ライブラリ。
> Microsoft 製で、Chrome / Edge / Firefox / Safari に対応。

```bash
pip install playwright
playwright install
```

### 2-2. ベースの構造

```python
from playwright.sync_api import sync_playwright
from anthropic import Anthropic

client = Anthropic()

def browser_agent(goal: str):
    with sync_playwright() as p:
        # ブラウザ起動(headless=False で目で見える)
        browser = p.chromium.launch(headless=False)
        page = browser.new_page()
        page.goto("https://google.com")
        
        # 最大10ステップ動かす
        for step in range(10):
            # 1. 画面のスクショとHTMLを取得
            screenshot = page.screenshot()
            html = page.content()
            
            # 2. 「次に何すべき?」をLLMに聞く
            action = ask_llm(goal, screenshot, html)
            
            # 3. 指示通り操作
            if action["type"] == "click":
                page.click(action["selector"])
            elif action["type"] == "type":
                page.fill(action["selector"], action["text"])
            elif action["type"] == "scroll":
                page.mouse.wheel(0, 500)
            elif action["type"] == "done":
                break
        
        browser.close()
```

### 2-3. ask_llm の中身

```python
def ask_llm(goal, screenshot, html):
    """LLMに次のアクションをJSONで返してもらう"""
    response = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=512,
        messages=[{
            "role": "user",
            "content": [
                {"type": "image", "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": base64.b64encode(screenshot).decode()
                }},
                {"type": "text", "text": f"""
目的: {goal}

現在のHTML(抜粋): {html[:3000]}

次のアクションをJSONで返して:
{{"type": "click", "selector": "..."}}
{{"type": "type", "selector": "...", "text": "..."}}
{{"type": "scroll"}}
{{"type": "done"}}
"""}
            ]
        }]
    )
    return json.loads(response.content[0].text)
```

→ これだけで **AI が自分でブラウザを動かす**。

---

## 3. Browser Use(OSS)(20分)

### 3-1. browser-use とは

> **イメージ:** 上の Playwright + LLM を**ライブラリ化**したもの。
> 数行で Browser Agent が動く。

```bash
pip install browser-use
```

```python
from browser_use import Agent
from langchain_anthropic import ChatAnthropic

agent = Agent(
    task="amazon.co.jp で 'Switch' を検索して、価格TOP3 を教えて",
    llm=ChatAnthropic(model="claude-opus-4-7"),
)
result = agent.run()
print(result)
```

→ **自動でブラウザを開いて、検索して、結果を返す**。
プロトタイプには **これが一番速い**。

### 3-2. 使い分け

| 場面 | 使うもの |
|---|---|
| とりあえず試したい | **browser-use** |
| 細かく制御したい | **Playwright + LLM** 自前 |
| 商用で安定運用 | Computer Use / Operator |

---

## 4. 商用ツール(参考)

| ツール | 提供 | 特徴 |
|---|---|---|
| **Claude Computer Use** | Anthropic公式 | 画面全体を操作可能、最強 |
| **ChatGPT Operator** | OpenAI | コンシューマ向け |
| **Multion** | Multion AI | 早期から提供 |

→ 自社で1から作るより、**商用APIを組み込む方が速い場合もある**。

---

## 5. 安定運用の鉄則

### 5-1. 必ず必要な要素

| 要素 | 理由 |
|---|---|
| **リトライ** | 画面読み込み失敗・LLM一時エラー対策 |
| **タイムアウト** | 無限ループ防止(最大20ステップなど) |
| **ロギング** | 失敗時の原因特定 |
| **スクショ保存** | デバッグ用に各ステップ画像 |
| **エラー検知** | 「想定外画面」に出たら停止 |
| **CAPTCHA対策** | 検出したら人間に通知 |

### 5-2. リトライ実装例

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10)
)
def safe_click(page, selector):
    page.click(selector, timeout=5000)
```

---

## 6. ⭐ 売れる成果物

```
案件A: 競合価格チェック完全自動化(月10-30万円)
  - 毎朝6時に競合5社サイト巡回
  - 商品ごとに価格スプレッドシート更新
  - Slack に変動アラート

案件B: 求人サイトから条件マッチ案件抽出(月10-20万円)
  - キーワード・年収・勤務地で絞り込み
  - 新着のみ毎朝メール送信
  - 重複排除

案件C: フォーム入力代行AI(月20-50万円)
  - エクセル → 各種申込フォーム自動入力
  - エラー時はスクショ + Slack通知
  - 人間が承認後に最終Submit

案件D: 業務システムの自動操作(月50-200万円)
  - レガシーシステムの定型業務
  - 在庫・受注・発注の同期
  - 月次レポート自動生成
```

---

## 7. よくあるエラー・落とし穴

| 症状 | 原因 | 直し方 |
|---|---|---|
| 操作が早すぎて反応しない | 画面遷移待ちなし | `wait_for_load_state` |
| Captcha で止まる | Bot検出 | 人間にエスカレート/別IP |
| LLM が間違った要素クリック | スクショ解像度低い | フル解像度で送る |
| ループが終わらない | done 判定なし | 最大ステップ数で強制終了 |
| 料金が爆発 | LLM 呼びすぎ | キャッシュ/HTML 圧縮 |

---

## 8. ⭐ 課題

```
1. browser-use で「Amazon でランキング1位の商品名と価格を取得」を実装
2. Playwright で「ログイン→検索→CSV ダウンロード」の Agent を作成
3. リトライ・ロギング・タイムアウト込みで本番品質に
4. ヘッドレス + cron で毎朝自動実行
5. Slack に結果通知
```

---

## 9. 用語まとめ表

| 用語 | 一言で |
|---|---|
| Browser Agent | AIが自分でブラウザ操作する仕組み |
| Playwright | Pythonでブラウザを操作するライブラリ |
| browser-use | Browser Agent のOSSフレームワーク |
| Claude Computer Use | Anthropic公式の画面操作API |
| ChatGPT Operator | OpenAI のブラウザ操作AI |
| Headless | 画面表示なしで動かすモード |
| セレクタ | クリック対象を指定する記法 (CSS/XPath) |
| Captcha | Bot検出 / 人間判定 |
| リトライ | 失敗時の再試行 |
| バックオフ | 指数的に待ち時間を増やすリトライ |

---

## 💼 この章まで終わったら受けられる案件 ★最先端

### 受けられる案件(具体例)
- **Browser Agent / Operator 系の自動化システム開発**:単価 100〜300万円(プラットフォーム例:Findy Freelance/直契約)
- **業務 SaaS 操作の完全自動化(代行ロボット)**:単価 80〜250万円
- **Browser Agent コンサル + PoC 開発**:単価 100〜300万円(2026年急速に需要拡大中)

### クロードへの頼み方(営業文を書かせる例)
```
最先端領域、Browser Agent / Operator 系の提案文を書いて。
Day 47 までで習得:Playwright / LLM ステップ判断 / browser-use / リトライ / バックオフ / スクショデバッグ
売り:「人間のクリック作業をAIに任せる」レベルの自動化、社内SaaSにAPIが無くても操作できる
価格:PoC 80万〜 / 本番運用 200万〜 / 月額保守 10〜30万円
ターゲット:大量のSaaS操作(ECモール出品、求人媒体管理、銀行操作)を抱える企業
注意:Captcha や利用規約に触れる場合の合法性を必ず明記
800字、過剰な煽りは避けて2026年現在の現実的水準で書く
```

### この章だけでは足りないもの(次に学ぶべき)
- 数学(Day 48)で更にAI案件単価UP
- 英語・海外案件(Day 49) → Upwork で時給 $50〜150
- 最速100万ロードマップ(Day 50)で総合戦略を立てる

---

## ✅ チェック
- [ ] Playwright でブラウザ操作できた
- [ ] LLM でステップ判断する Agent を作れた
- [ ] browser-use を動かせた
- [ ] リトライ・タイムアウト・ロギング込みで作った
- [ ] スクショ保存でデバッグできる
- [ ] 自動実行(cron)で運用できた

## 次
`48_数学.md` で**ML の中身を読める**ようにする。
