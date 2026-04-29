# Day 28:LLM Agent・MCP・Tool use - 2026年最先端

## 💰 この章後の月収:**+50〜200万円**(LLM Agent開発、最先端)
## ⏱ 所要:4時間

2026年で**もっとも単価が高い**領域。
「**Agentic AI**」「**自律エージェント**」が企業のキーワードになってる今、ここを抑えると**月150万円超**も現実。

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

### 普通のチャットボット
```
ユーザー: 今日の天気は?
LLM: わかりません(知識にない)
```

### Agent
```
ユーザー: 今日の天気は?
Agent: 「天気APIを呼ぼう」
  → tool: get_weather(location="Tokyo")
  → 結果: 晴れ、25度
LLM: 晴れて25度です。
```

→ **LLMが「外の世界」と対話**できる。

---

## 2. Function Calling(基本)(40分)

### Anthropic Claude

```python
from anthropic import Anthropic

client = Anthropic()

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
        
        # stop_reason が end_turn なら終了
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

---

## 4. MCP(Model Context Protocol)(40分)

Anthropic が**2024年末に出した標準プロトコル**。
**Claude Desktop / Cursor / Claude Code が対応**。

### MCP とは

「**LLMと外部ツール・データソースを統一規格で繋ぐ**」プロトコル。

```
[Claude] ←→ [MCPサーバー] ←→ [DB / API / ファイルシステム]
```

### 自作 MCP サーバー(Python)

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

### Claude Desktop / Code に登録

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
    messages: Annotated[list, add_messages]

llm = ChatAnthropic(model="claude-opus-4-7")

def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}

graph_builder = StateGraph(State)
graph_builder.add_node("chatbot", chatbot)
graph_builder.set_entry_point("chatbot")
graph_builder.set_finish_point("chatbot")
graph = graph_builder.compile()

result = graph.invoke({"messages": [HumanMessage("こんにちは")]})
print(result["messages"][-1].content)
```

→ ノードと辺で**複雑な対話フロー**を可視化。

### 条件分岐ノード

```python
def route(state):
    if "search" in state["messages"][-1].content.lower():
        return "search_agent"
    return "chat_agent"

graph_builder.add_conditional_edges("entry", route)
```

---

## 6. AutoGen - マルチエージェント(30分)

Microsoft の**複数LLM同士が会話して問題解決**するフレームワーク。

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

team = RoundRobinGroupChat([planner, coder, reviewer])
result = await team.run(task="TODO アプリを設計して実装して")
```

→ **3つのエージェントが議論しながらタスク完遂**。

---

## 7. プロンプトエンジニアリング(30分)

### Few-shot

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

### Chain of Thought(CoT)

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

### Self-Critique

```python
# 第1回:回答
answer = llm.invoke(f"答えて: {question}")

# 第2回:自己批判
critique = llm.invoke(f"以下の回答を批判してください: {answer}")

# 第3回:改善
final = llm.invoke(f"批判を反映して再度答えて: {answer}\n批判: {critique}")
```

### XML タグ構造化(Claude推奨)

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

---

## 8. RAG の高度化(20分)

### Hybrid Search

```python
# Vector検索 + キーワード検索 を組合せ
vector_results = chroma.similarity_search(query, k=5)
bm25_results = bm25.search(query, top_k=5)

# Reciprocal Rank Fusion で統合
combined = rrf_combine([vector_results, bm25_results])
```

### Re-ranking

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-12-v2")
pairs = [(query, doc) for doc in candidates]
scores = reranker.predict(pairs)
top = sorted(zip(candidates, scores), key=lambda x: -x[1])[:3]
```

### HyDE(Hypothetical Document)

「**仮想的な答えを作ってからベクトル検索**」で精度UP。

---

## 9. LLM 評価(15分)

```python
# 評価データセット作成
test_set = [
    {"query": "返品方法は?", "expected": "返品ポリシーを確認"},
    ...
]

# LLM-as-Judge
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
