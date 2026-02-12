# 🚀 Streaming in LangGraph — Complete Guide

---

## 📌 What is Streaming in LangGraph?

**Streaming in LangGraph** allows you to receive outputs from your graph **step-by-step in real time** instead of waiting for the full execution to finish.

Instead of:

```
User → Graph runs → Final result returned
```

You get:

```
User → Node executes → Partial output → Next node → Partial output → Final output
```

This makes applications:

* Faster
* More interactive
* Transparent
* Suitable for real-time UI updates

---

# 🎯 Why Streaming is Needed

## Without Streaming

* User waits for full execution
* No visibility of intermediate steps
* Slow UX for long LLM calls

## With Streaming

* See responses live
* Track graph execution
* Show progress in UI
* Better debugging
* Real-time agent systems

---

# 🧠 Core Idea

LangGraph allows you to stream:

* **State updates**
* **Node outputs**
* **LLM tokens**
* **Events during execution**

Streaming happens using:

```
graph.stream()
```

instead of:

```
graph.invoke()
```

---

# ⚡ invoke vs stream

| Feature       | invoke()          | stream()             |
| ------------- | ----------------- | -------------------- |
| Execution     | Runs fully        | Runs step-by-step    |
| Output        | Final result only | Intermediate + final |
| Real-time     | ❌ No              | ✅ Yes                |
| Debugging     | Hard              | Easy                 |
| UI updates    | ❌                 | ✅                    |
| Agent systems | Limited           | Excellent            |

---

# 🏗️ Basic Streaming Flow

```
User Input
   ↓
Graph starts
   ↓
Node 1 executes → streamed
   ↓
Node 2 executes → streamed
   ↓
Final output
```

---

# ✅ Basic Streaming Example (Step-by-Step)

## Step 1 — Install

```bash
pip install langgraph langchain-openai
```

---

## Step 2 — Import Libraries

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END
```

---

## Step 3 — Define State

```python
class MyState(TypedDict):
    text: str
```

---

## Step 4 — Create Nodes

```python
def node1(state: MyState):
    print("Node 1 running")
    return {"text": state["text"] + " → processed by node1"}


def node2(state: MyState):
    print("Node 2 running")
    return {"text": state["text"] + " → processed by node2"}
```

---

## Step 5 — Build Graph

```python
graph = StateGraph(MyState)

graph.add_node("node1", node1)
graph.add_node("node2", node2)

graph.add_edge(START, "node1")
graph.add_edge("node1", "node2")
graph.add_edge("node2", END)

app = graph.compile()
```

---

## Step 6 — Stream Execution

```python
for event in app.stream({"text": "hello"}):
    print(event)
```

---

## Output (Step-by-Step)

```
Node 1 running
{'node1': {'text': 'hello → processed by node1'}}

Node 2 running
{'node2': {'text': 'hello → processed by node1 → processed by node2'}}
```

Notice:

✔ Each node output arrives separately

✔ You see progress live

✔ You don’t wait for final result

---

# 🔄 How Streaming Works Internally

When streaming:

1. Graph starts execution
2. Node executes
3. State update emitted
4. Event returned immediately
5. Next node executes

So execution becomes **event-based**.

---

# 📦 What stream() Returns

`stream()` returns a **generator** that yields events.

Each event contains:

* Node name
* Updated state
* Execution progress

Example:

```python
{'node_name': {'updated_state'}}
```

---

# 🎯 Streaming Modes in LangGraph

LangGraph supports different streaming modes.

---

## 1️⃣ values Mode (Default)

Streams updated state values.

```python
for event in app.stream(input, stream_mode="values"):
    print(event)
```

### What you get

* Updated state after node execution
* Most commonly used mode

---

## 2️⃣ updates Mode

Streams only changes made by nodes.

```python
for event in app.stream(input, stream_mode="updates"):
    print(event)
```

### What you get

* Only modified fields
* Efficient for large states

---

## 3️⃣ debug Mode

Shows detailed execution events.

```python
for event in app.stream(input, stream_mode="debug"):
    print(event)
```

### What you get

* Node start/end
* State transitions
* Execution trace

Great for debugging.

---

# 🤖 Streaming with LLM Responses

Streaming is extremely useful with LLMs.

Instead of waiting for full response → get tokens live.

---

## Example — Chat Streaming

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(streaming=True)


def chat_node(state):
    response = llm.invoke(state["messages"])
    return {"messages": [response]}
```

Then:

```python
for event in app.stream(input):
    print(event)
```

---

# 🎨 Real-World Use Cases

## ✅ Chat applications

* Live message generation

## ✅ AI agents

* Show tool usage step-by-step

## ✅ Debugging workflows

* Trace node execution

## ✅ Monitoring pipelines

* Track progress

## ✅ Web apps

* Real-time UI updates

---

# 🆚 invoke() vs stream() vs batch()

| Method     | Purpose                           |
| ---------- | --------------------------------- |
| `invoke()` | Run once and return final result  |
| `stream()` | Run step-by-step with live output |
| `batch()`  | Run multiple inputs together      |

---

# ⚠️ Important Notes

## 1. stream() is non-blocking

Execution continues while you receive results.

---

## 2. Uses Python generator

You must iterate:

```python
for event in app.stream(input):
```

---

## 3. Best for long workflows

Short workflows may not need streaming.

---

## 4. Useful for UI frameworks

* Streamlit
* FastAPI
* WebSocket apps

---

# 🧩 Advanced Pattern — Stream + Conditional Routing

Streaming works even when graph has:

* Conditional edges
* Agent loops
* Tool calling

You see each decision step live.

---

# 🔥 Real Intuition (Very Important)

Think of streaming like:

```
YouTube Live vs Recorded Video
```

## invoke()

→ Recorded video (watch after completion)

## stream()

→ Live broadcast (watch while happening)

---

# 📊 When to Use Streaming

Use streaming when:

✔ Graph takes time

✔ Using LLMs

✔ Building chat apps

✔ Need transparency

✔ Debugging

✔ Real-time UX required

---

# ❌ When Not Needed

Avoid streaming when:

* Very small graphs
* Single fast computation
* No UI interaction needed

---

# 🧱 Complete Minimal Example (Copy-Paste Ready)

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    message: str


def step1(state: State):
    return {"message": state["message"] + " → step1"}


def step2(state: State):
    return {"message": state["message"] + " → step2"}


graph = StateGraph(State)

graph.add_node("step1", step1)
graph.add_node("step2", step2)

graph.add_edge(START, "step1")
graph.add_edge("step1", "step2")
graph.add_edge("step2", END)

app = graph.compile()

for event in app.stream({"message": "start"}):
    print(event)
```

---

# ⭐ Key Takeaways

* Streaming = real-time graph execution
* Uses `graph.stream()`
* Returns generator of events
* Best for LLM apps and agents
* Enables live UI updates
* Improves debugging

---

# ✅ Summary

**LangGraph Streaming** allows graphs to produce outputs progressively instead of waiting for full completion. It is essential for building interactive AI systems, agent workflows, and real-time applications.

It transforms graph execution from:

```
batch execution → event-driven execution
```

---

# 📚 End of README

---
