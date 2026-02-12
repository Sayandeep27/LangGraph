# LangGraph — `thread_id` and Session Memory (Complete Guide)

---

# What is Memory in LangGraph?

## Simple Meaning

**Memory in LangGraph = Ability of the graph to remember past interactions.**

It allows:

* Conversation continuity
* Multi-turn chat
* Context awareness
* User session tracking

Without memory:

```
Every request = fresh start
```

With memory:

```
Graph remembers previous messages
```

---

# What is Session Memory?

## Definition

**Session Memory = Memory stored for one specific conversation session.**

Each user or conversation gets its own memory.

---

## Real Life Example

Think of ChatGPT:

* You ask a question
* It remembers previous messages
* Conversation continues

Each chat window = one session.

That is **session memory**.

---

## Key Idea

```
Different users → different memory
Same user session → same memory
```

---

# What is `thread_id`?

## Definition

**`thread_id` is a unique identifier that tells LangGraph which session memory to use.**

It separates conversations.

---

## Simple Meaning

```
thread_id = session name
```

It tells:

```
"Store and load memory for THIS conversation"
```

---

## Real World Analogy

Imagine a hospital:

* Each patient has a patient ID
* Doctor loads patient history using ID

Here:

* Patient ID → `thread_id`
* Patient history → memory

---

# Why `thread_id` is Needed

## Problem Without It

Without `thread_id`:

* All users share same memory
* Conversations mix
* Context breaks

Example problem:

```
User A: My name is John
User B: What is my name?
AI: John (WRONG)
```

---

## Solution

```
thread_id isolates memory
```

Each conversation becomes independent.

---

# How `thread_id` Works Internally

## Core Idea

LangGraph stores memory like:

```
memory_store = {
    "thread_1": conversation_data,
    "thread_2": conversation_data
}
```

When you send:

```python
config={"configurable": {"thread_id": "user_1"}}
```

LangGraph:

1. Finds memory for `user_1`
2. Loads previous state
3. Runs graph
4. Saves updated state

---

# Session Memory Flow

```
User Request
     ↓
thread_id received
     ↓
Load session memory
     ↓
Run graph
     ↓
Update memory
     ↓
Save memory
```

---

# Without vs With `thread_id`

## Without

```python
graph.invoke(input)
```

Result:

* No memory
* Stateless execution

---

## With

```python
graph.invoke(
    input,
    config={"configurable": {"thread_id": "user_1"}}
)
```

Result:

* Persistent memory
* Session continuity

---

# Checkpointer Concept

## Definition

A **checkpointer** is responsible for saving and loading session state.

It enables memory persistence.

---

## Built-in Option

### MemorySaver

```
from langgraph.checkpoint.memory import MemorySaver
```

Stores memory in RAM.

---

## Why Checkpointer Needed

Without checkpointer:

```
No session memory
```

With checkpointer:

```
thread_id → stored state
```

---

# How Session Memory is Stored

## Structure

```
{
  thread_id: {
    state_values
  }
}
```

Example:

```
{
 "user_1": {"messages": [...]},
 "user_2": {"messages": [...]}
}
```

---

# Step-by-Step Working

## Step 1 — Create State

```python
from typing import TypedDict

class ChatState(TypedDict):
    messages: list
```

---

## Step 2 — Create Node

```python
def chat_node(state: ChatState):
    messages = state["messages"]
    response = llm.invoke(messages)
    return {"messages": [response]}
```

---

## Step 3 — Create Graph

```python
from langgraph.graph import StateGraph

graph = StateGraph(ChatState)
```

---

## Step 4 — Add Memory

```python
from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()
app = graph.compile(checkpointer=memory)
```

---

## Step 5 — Use thread_id

```python
app.invoke(
    {"messages": ["Hello"]},
    config={"configurable": {"thread_id": "user_1"}}
)
```

---

# Complete Code Example

```python
from typing import TypedDict
from langgraph.graph import StateGraph
from langgraph.checkpoint.memory import MemorySaver

# 1. State
class ChatState(TypedDict):
    messages: list

# 2. Node
def chat_node(state: ChatState):
    messages = state["messages"]
    response = "AI Response"  # simulate LLM
    return {"messages": messages + [response]}

# 3. Graph
builder = StateGraph(ChatState)
builder.add_node("chat", chat_node)
builder.set_entry_point("chat")
builder.set_finish_point("chat")

# 4. Memory
memory = MemorySaver()
app = builder.compile(checkpointer=memory)

# 5. Session 1
app.invoke(
    {"messages": ["Hello"]},
    config={"configurable": {"thread_id": "user_1"}}
)

# 6. Session 2
app.invoke(
    {"messages": ["Hi"]},
    config={"configurable": {"thread_id": "user_2"}}
)
```

---

# Multi-User Scenario Example

## Two Users

```
User A → thread_id="A"
User B → thread_id="B"
```

Memory stored separately.

---

## Result

```
A remembers A history
B remembers B history
```

---

# Common Mistakes

## 1. Not passing thread_id

Result:

```
No session memory
```

---

## 2. Same thread_id for all users

Result:

```
Mixed conversations
```

---

## 3. No checkpointer

Result:

```
Memory not saved
```

---

# Best Practices

* Always use unique `thread_id` per user
* Use database checkpointer for production
* Never share thread_id between users
* Always enable memory for chat apps

---

# Summary

## In One Line

```
thread_id = session identifier
Session memory = stored conversation for that thread
```

---

## Final Understanding

* Session memory stores conversation state
* thread_id identifies session
* Checkpointer saves state
* Each thread has separate memory
* Enables real chat experience

---

**End of Document**
