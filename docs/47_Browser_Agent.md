# Day 47:Browser Agent - 自律的にブラウザ操作するAI

## 💡 これは何?(小学生でもわかる説明)

****AIが自分でブラウザ操作**して仕事する**

### なぜ大事?
ChatGPT Operator 級の最先端。月単価200万円

### イメージ
**バイトを雇って画面を操作させる**の自動版

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 +50-200万(2026最先端)
## ⏱ 15時間

ChatGPT Operator 級の「**AIが自分でブラウザ操作**」する技術。

---

## 1. Playwright + LLM(40分)

```python
from playwright.sync_api import sync_playwright
from anthropic import Anthropic

client = Anthropic()

def browser_agent(goal: str):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=False)
        page = browser.new_page()
        page.goto("https://google.com")
        
        for _ in range(10):
            screenshot = page.screenshot()
            html = page.content()
            
            # LLM に「次に何すべき?」聞く
            action = ask_llm(goal, screenshot, html)
            
            if action["type"] == "click":
                page.click(action["selector"])
            elif action["type"] == "type":
                page.fill(action["selector"], action["text"])
            elif action["type"] == "done":
                break
        
        browser.close()
```

---

## 2. Browser Use(OSS)(20分)

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
```

→ **自動でブラウザを開いて、検索して、結果を返す**。

---

## 3. 商用ツール(参考)

- **Claude Computer Use**(Anthropic公式)
- **ChatGPT Operator**(OpenAI)
- **Multion**

---

## 4. ⭐ 売れる成果物

```
- 競合価格チェック完全自動化(月10-30万円)
- 求人サイトから条件マッチ案件抽出(月10-20万円)
- フォーム入力代行AI(月20-50万円)
```

---

## ✅ チェック
- [ ] Playwright でブラウザ操作
- [ ] LLM でステップ判断
- [ ] Browser Use 動かせる

## 次
`48_数学.md`
