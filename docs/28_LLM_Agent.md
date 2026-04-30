# Day 28:LLM Agent・MCP・Tool use - 2026年最先端

## 💡 これは何?(小学生でもわかる説明)

**AIが自分で考えて、ツール使って、目的達成する(Agent)**

### なぜ大事?
普通のAI(ChatGPT等)=「**質問に答えるだけ**」。
Agent=「**自分で行動する**」。
例:「明日の予定教えて」→ 普通のAIは答えない。Agentはカレンダーを開いて答える。
2026年で最高単価。月150-300万円も現実。

### イメージ
- **普通のチャットAI**=「**百科事典を持った友達**」(質問に答えるだけ)
- **LLM Agent**=「**有能な秘書**」(指示するだけで、必要な情報を探し、ツールを使い、行動する)
- **Function Calling / Tool Use**=「**AIに使える道具を渡す**」
- **MCP**=「**AI とアプリを繋ぐ標準コンセント**」(USB Type-Cみたいな統一規格)
- **LangGraph**=「**Agent の流れ図**」(右に行ったらDB、左に行ったらWeb)
- **AutoGen**=「**複数AIで会議させる**」(設計者・コーダー・レビュアーが議論)
- **プロンプトエンジニアリング**=「**AIへの指示書を上手に書く技術**」
- **RAG**=「**自社データを参照して答える**」(社内マニュアルを読ませる)

### Claude Code に何を頼めばいいか
この章で覚えるのは「**何ができるか**」と「**何を頼めばいいか**」だけでOK。
具体的なコードは Claude Code に書かせる。

---

## 💰 この章後の月収:**+50〜200万円**(LLM Agent開発、最先端)
## ⏱ 所要:4時間

2026年で**もっとも単価が高い**領域。
「**Agentic AI**」「**自律エージェント**」が企業のキーワードになってる今、ここを抑えると**月150万円超**も現実。

> **なぜ最高単価?**
> - 需要爆発(=企業が「うちもAgent入れたい」と殺到)
> - 供給激少(=作れるエンジニアが世界で数千人レベル)
> - 効果絶大(=人件費月50万→Agent月3万、誰でも導入したい)

---

## ゴール

- LLM Agent の構築原理
- Function calling / Tool use
- MCP(Model Context Protocol)
- LangGraph / AutoGen
- マルチエージェント協調
- プロンプトエンジニアリング

---

## 1. LLM Agent とは(15分)

「**LLMが自分で判断して、ツールを呼び出して、目的を達成する**」。

### 1-1. 普通のチャットボット

```
ユーザー: 今日の天気は?
LLM: わかりません(知識にない)
```

### 1-2. Agent

```
ユーザー: 今日の天気は?
Agent: 「天気APIを呼ぼう」
  → tool: get_weather(location="Tokyo")
  → 結果: 晴れ、25度
LLM: 晴れて25度です。
```

→ **LLMが「外の世界」と対話**できる。

> **イメージ:**
> 普通のAI = 「お留守番AI」(家から出ない)
> Agent = 「冒険者AI」(必要なら街へ買い物に行く)

---

## 2. Function Calling(基本)(40分)

### 2-1. Anthropic Claude

```python
from anthropic import Anthropic

client = Anthropic()

# AI に渡すツールの説明
tools = [{
    "name": "get_weather",
    "description": "指定された都市の天気を取得",
    "input_schema": {
        "type": "object",
        "properties": {
            "city": {"type": "string", "description": "都市名"}
        },
        "required": ["city"]
    }
}]

def get_weather(city: str) -> dict:
    # 実際は API呼び出し
    return {"city": city, "temp": 25, "condition": "sunny"}

# 第1回呼び出し:LLMが「ツールを使う」と判断
response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "東京の天気は?"}]
)

# tool_use ブロックを取得
for block in response.content:
    if block.type == "tool_use":
        # AIが選んだツールを実行
        result = get_weather(**block.input)

        # 第2回呼び出し:結果を返してLLMに最終回答させる
        final = client.messages.create(
            model="claude-opus-4-7",
            max_tokens=1024,
            tools=tools,
            messages=[
                {"role": "user", "content": "東京の天気は?"},
                {"role": "assistant", "content": response.content},
                {"role": "user", "content": [{
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": str(result)
                }]}
            ]
        )
        print(final.content[0].text)
```

→ これが**最重要パターン**。これ書けると一気にプロ。

> **流れ:**
> 1. ユーザー質問 → AI
> 2. AI:「ツール使う必要あるな」 → ツール名+引数を返す
> 3. プログラム:ツール実行 → 結果取得
> 4. AI:結果を見て最終回答を生成

---

## 3. 複数ツール・自律ループ(40分)

```python
def agent_loop(user_message: str, max_iterations: int = 10):
    messages = [{"role": "user", "content": user_message}]

    tools = [
        {"name": "search_web", "description": "Web検索", "input_schema": {...}},
        {"name": "calculate", "description": "計算実行", "input_schema": {...}},
        {"name": "send_email", "description": "メール送信", "input_schema": {...}},
    ]

    for _ in range(max_iterations):
        response = client.messages.create(
            model="claude-opus-4-7",
            max_tokens=4096,
            tools=tools,
            messages=messages,
        )

        # stop_reason が end_turn なら終了(=AI が完了と判断)
        if response.stop_reason == "end_turn":
            return response.content[0].text

        # ツール使用なら実行して結果を返す
        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})

            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": str(result)
                    })

            messages.append({"role": "user", "content": tool_results})

    return "Max iterations reached"
```

→ Agent が自分で「次に何のツールを使うか」を判断するループ。

> **max_iterations の意味:**
> 暴走防止の上限。AIが無限ループに陥らないように10回で打ち切り。

---

## 4. MCP(Model Context Protocol)(40分)

Anthropic が**2024年末に出した標準プロトコル**。
**Claude Desktop / Cursor / Claude Code が対応**。

### 4-1. MCP とは

「**LLMと外部ツール・データソースを統一規格で繋ぐ**」プロトコル。

```
[Claude] ←→ [MCPサーバー] ←→ [DB / API / ファイルシステム]
```

> **なぜ MCP?**
> 各社が独自プロトコル作ると、ツールごとに毎回実装が必要。
> MCP に統一すれば、**1回作れば複数AIアプリで使える**(=USB Type-C と同じ哲学)。

### 4-2. 自作 MCP サーバー(Python)

```bash
pip install mcp
```

```python
# my_mcp_server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
def get_user(user_id: int) -> dict:
    """ユーザーIDから情報を取得"""
    # DB問い合わせ等
    return {"id": user_id, "name": "太郎"}

@mcp.tool()
def search_orders(user_id: int, since: str) -> list:
    """ユーザーの注文履歴"""
    return [{"order_id": 1, "amount": 1000}]

@mcp.resource("user://list")
def user_list() -> str:
    """ユーザー一覧をリソースとして提供"""
    return "ユーザー: 太郎、花子、次郎"

if __name__ == "__main__":
    mcp.run()
```

### 4-3. Claude Desktop / Code に登録

`~/.config/claude-code/config.json`:
```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/path/to/my_mcp_server.py"]
    }
  }
}
```

→ **Claude が自社DBやAPIを直接触れる**ように。
**法人案件で月100万円**取れる超有望スキル。

---

## 5. LangGraph - グラフ型ワークフロー(30分)

「**Agent を状態遷移グラフで設計**」する。LangChain チームの新作。

> **イメージ:** すごろく。
> マスごとに「何をするか」「次にどこへ行くか」を定義する。

```bash
pip install langgraph langchain-anthropic
```

```python
from typing import Annotated, TypedDict
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages
from langchain_anthropic import ChatAnthropic
from langchain_core.messages import HumanMessage

class State(TypedDict):
    # 状態(=各ノード間で受け渡されるデータ)
    messages: Annotated[list, add_messages]

llm = ChatAnthropic(model="claude-opus-4-7")

def chatbot(state: State):
    # ノードでやる処理
    return {"messages": [llm.invoke(state["messages"])]}

graph_builder = StateGraph(State)
graph_builder.add_node("chatbot", chatbot)
graph_builder.set_entry_point("chatbot")  # スタート
graph_builder.set_finish_point("chatbot") # 終了
graph = graph_builder.compile()

result = graph.invoke({"messages": [HumanMessage("こんにちは")]})
print(result["messages"][-1].content)
```

→ ノードと辺で**複雑な対話フロー**を可視化。

### 5-1. 条件分岐ノード

```python
def route(state):
    # 条件で次のノードを決める
    if "search" in state["messages"][-1].content.lower():
        return "search_agent"
    return "chat_agent"

graph_builder.add_conditional_edges("entry", route)
```

---

## 6. AutoGen - マルチエージェント(30分)

Microsoft の**複数LLM同士が会話して問題解決**するフレームワーク。

> **イメージ:** 会議室にAIを3人入れる。
> - 計画担当
> - コーダー担当
> - レビュアー担当
> 3人で議論しながらタスク完遂。

```bash
pip install autogen-agentchat autogen-ext[openai]
```

```python
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o")

planner = AssistantAgent("planner", model_client=model_client,
    system_message="プロジェクトの計画を立てる")

coder = AssistantAgent("coder", model_client=model_client,
    system_message="計画に基づいてコードを書く")

reviewer = AssistantAgent("reviewer", model_client=model_client,
    system_message="コードをレビューする")

# 順番に発言するチーム
team = RoundRobinGroupChat([planner, coder, reviewer])
result = await team.run(task="TODO アプリを設計して実装して")
```

→ **3つのエージェントが議論しながらタスク完遂**。

---

## 7. プロンプトエンジニアリング(30分)

### 7-1. Few-shot(=お手本を見せる)

```python
prompt = """
以下の例に従って分類してください。

例1:
入力: "返金してほしい"
出力: refund

例2:
入力: "ログインできない"
出力: login_issue

入力: "{user_input}"
出力:
"""
```

> **なぜ効く?**
> AIは「お手本」があると精度が劇的にUP。指示だけより例の方が伝わる。

### 7-2. Chain of Thought(CoT、思考の連鎖)

```python
prompt = """
以下の問題を**段階的に**考えて解いてください。

問題: {question}

考えるステップ:
1. まず何が問われているか確認
2. 必要な情報を整理
3. 計算
4. 答えを述べる

考え:
"""
```

> **なぜ?**
> AIは「いきなり答えろ」より「順を追って考えろ」の方が正確になる(=人間と同じ)。

### 7-3. Self-Critique(自己批判)

```python
# 第1回:回答
answer = llm.invoke(f"答えて: {question}")

# 第2回:自己批判
critique = llm.invoke(f"以下の回答を批判してください: {answer}")

# 第3回:改善
final = llm.invoke(f"批判を反映して再度答えて: {answer}\n批判: {critique}")
```

### 7-4. XML タグ構造化(Claude推奨)

```python
prompt = """
<task>
ユーザーの質問に答える
</task>

<context>
{context}
</context>

<question>
{question}
</question>

<instructions>
- 簡潔に
- 引用元を明記
- わからない時は「わかりません」
</instructions>
"""
```

> **なぜ XML?**
> Claude は XML タグで「ここからここまでがコンテキスト」と区別しやすい。

---

## 8. RAG の高度化(20分)

### 8-1. Hybrid Search

```python
# Vector検索 + キーワード検索 を組合せ
vector_results = chroma.similarity_search(query, k=5)
bm25_results = bm25.search(query, top_k=5)

# Reciprocal Rank Fusion で統合
combined = rrf_combine([vector_results, bm25_results])
```

> **なぜ Hybrid?**
> Vector(=意味検索)とキーワード(=完全一致)の両方の弱点をカバー。

### 8-2. Re-ranking

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-12-v2")
pairs = [(query, doc) for doc in candidates]
scores = reranker.predict(pairs)
top = sorted(zip(candidates, scores), key=lambda x: -x[1])[:3]
```

> **なぜ?**
> 一次検索で20件、Re-rankingで上位3件に絞る=精度UP。

### 8-3. HyDE(Hypothetical Document)

「**仮想的な答えを作ってからベクトル検索**」で精度UP。

---

## 9. LLM 評価(15分)

```python
# 評価データセット作成
test_set = [
    {"query": "返品方法は?", "expected": "返品ポリシーを確認"},
    ...
]

# LLM-as-Judge(=AIにAIの答えを採点させる)
def evaluate(query, response, expected):
    prompt = f"""
以下の応答を 0-10 で評価してください。
質問: {query}
模範: {expected}
応答: {response}
"""
    score = llm.invoke(prompt)
    return float(score)
```

> **なぜ AI が AI を評価?**
> 1万件の応答を人間が評価するのは無理。AI なら一瞬。

---

## 10. ⭐ 売れる成果物 #14:法人向け Agent システム(120分)

クロードに頼む:

```
法人向けの自律 Agent を完全に作って。

要件:
- ユーザー質問を受けて、必要なら社内DB(SQLite)を検索
- 情報不足なら Web検索ツールを呼ぶ
- 最終回答は出典付きで返す
- すべて MCP サーバーとして実装
- Claude Desktop に登録できる形

ツール:
1. search_internal_db(SQL クエリ実行、結果10件まで)
2. search_web(Tavily API使用)
3. send_email(SendGrid)
4. create_calendar_event(Google Calendar)

LangGraph でワークフロー設計、状態管理。
評価コード(LLM-as-Judge)も含める。

README に Claude Desktop での使い方を明記。
```

→ **これは月150-300万円の案件**。LLM Agentエンジニアとして名乗れる。

---

## 11. よくある間違い・エラー(15分)

| 状況 | 間違いの原因 | 直し方 |
|---|---|---|
| Agent が無限ループ | 終了条件なし | max_iterations を設定 |
| ツール実行で例外 | エラーハンドリング忘れ | try/except でAIに失敗を伝える |
| プロンプトが曖昧 | 指示不足 | XML タグ + Few-shot で構造化 |
| RAG の精度が低い | チャンク分割が悪い | チャンクサイズ調整、Hybrid + Re-ranking |
| Tool description が雑 | AIがツール選び間違える | description を詳細・例付きで |
| MCPサーバーが認識されない | パス・JSON書式ミス | configファイル確認、再起動 |
| トークン超過 | 履歴が膨らむ | 古い履歴を要約・切り捨て |
| 出力フォーマットが揺れる | 構造化指示なし | JSON Mode やスキーマ指定 |
| API課金で予算超過 | 上限なし | 月次予算アラート設定 |

---

## 12. 用語まとめ

| 用語 | 一言で |
|---|---|
| LLM | 大規模言語モデル |
| Agent | 自律的に動くAI |
| Function Calling | AIにツールを渡す機能 |
| Tool Use | Anthropicでの呼び方 |
| 自律ループ | AIが自分で考えて繰り返す |
| MCP | LLM-ツール統一規格 |
| MCPサーバー | ツールを提供する側 |
| LangGraph | グラフ型ワークフロー |
| AutoGen | マルチエージェント連携 |
| プロンプトエンジニアリング | AI への指示の上手い書き方 |
| Few-shot | お手本を例示する手法 |
| Chain of Thought | 段階的思考を促す |
| Self-Critique | AI に自己批判させる |
| RAG | 外部知識を参照する仕組み |
| Hybrid Search | 複数検索手法の組合せ |
| Re-ranking | 検索結果を再評価 |
| HyDE | 仮想答案でベクトル検索 |
| LLM-as-Judge | AIに採点させる評価 |
| Tavily | LLM 用検索 API |
| max_iterations | 暴走防止の上限 |

---

## 💼 この章まで終わったら受けられる案件 ★今最熱

### 受けられる案件(具体例)
- **LLM Agent / マルチエージェントシステム開発**:単価 100〜500万円(プラットフォーム例:Findy Freelance/直契約/SES)
- **MCP サーバー実装(Claude Desktop / Cursor 連携)**:単価 50〜200万円
- **業務自動化エージェント(Tool Use + 自律ループ)**:単価 100〜300万円(2026年現在最も需要が高い領域)

### クロードへの頼み方(営業文を書かせる例)
```
最先端 LLM Agent 案件向け、X / Note / 直営業用の3点セットを書いて。
Day 28 までで習得:Function calling / 自律ループ / MCP サーバー / LangGraph / AutoGen / プロンプトエンジニアリング
売り:2026年4月時点で「Agent を実務で組める」エンジニアは絶対数が少ない、その希少性
過去実績:GitHubに小規模 Agent デモ(MCP対応)
価格:単発 100万〜 / コンサル 時給 1.5〜3万円
ターゲット:DX に乗り遅れたくない中堅企業 / SaaS スタートアップ
トーン:過剰な煽りはせず、「これからの3年で確実に来る波」と冷静に述べる
```

### この章だけでは足りないもの(次に学ぶべき)
- フルスタック(Day 29)で UI からエージェントまで提供
- 本番運用 / K8s / IaC(Day 30)で大規模Agent運用
- Browser Agent(Day 47) / Multi-modal(Day 46)

---

## ✅ チェック

- [ ] Function calling(Anthropic SDK)書ける
- [ ] 自律ループ実装できる
- [ ] MCP サーバー作成・登録
- [ ] LangGraph で状態遷移グラフ
- [ ] AutoGen でマルチエージェント
- [ ] プロンプトエンジニアリング(Few-shot/CoT/XML)
- [ ] RAG 高度化(Hybrid/Re-ranking)
- [ ] LLM評価の概念
- [ ] **法人向け Agent システム完成**

---

## 進捗

```
Day 28: LLM Agent完了。Function calling/MCP/LangGraph/AutoGen。
2026年最先端の高単価領域(月150万圏)。
```

## 次

`29_フルスタック.md` へ。**Next.js + FastAPI フルスタック**。
