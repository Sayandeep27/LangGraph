# LangGraph Tools — Complete Guide

---

# What are Tools in LangGraph

## Simple Meaning

**Tools in LangGraph are external functions that the LLM can call to perform real-world tasks.**

Instead of only generating text, the LLM can:

* Do calculations
* Search information
* Query databases
* Call APIs
* Execute custom logic

Think of tools like **superpowers** you give to the LLM.

---

## Real Life Analogy

Imagine a manager (LLM):

* Manager can think and talk
* But cannot calculate large numbers quickly
* Cannot check database manually

So manager calls:

* accountant → calculation tool
* database system → retrieval tool
* internet search → search tool

Same idea in LangGraph.

---

# Why Tools are Needed

Without tools, LLM can only:

* Predict text
* Guess answers
* Hallucinate

With tools, LLM can:

* Get exact answers
* Perform actions
* Use external systems

| Without Tools          | With Tools       |
| ---------------------- | ---------------- |
| Guessing               | Real computation |
| Hallucination possible | Accurate results |
| Text only              | Real actions     |

---

# How Tools Work (Big Picture)

## Execution Flow

1. User asks something
2. LLM decides if tool is needed
3. Tool executes
4. Result returns to LLM
5. LLM produces final answer

---

## Visual Flow

```
User → LLM → Tool → Result → LLM → Answer
```

---

# Tool vs Node (Very Important)

| Node                    | Tool                    |
| ----------------------- | ----------------------- |
| Part of graph execution | External function       |
| Always runs when routed | Called only when needed |
| Graph logic             | External capability     |
| State transformer       | Real-world action       |

## Key Idea

**Nodes control flow. Tools perform actions.**

---

# How to Create a Tool

LangGraph uses LangChain tools.

---

## Basic Tool Example

```python
from langchain_core.tools import tool

@tool
def add_numbers(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b
```

---

## What Happens Here

* `@tool` → converts function into tool
* Docstring → tells LLM when to use tool
* Parameters → inputs
* Return → output

## Important Rule

**Tool description must be clear. LLM uses it to decide when to call the tool.**

---

# How to Use Tools in LangGraph

Steps:

1. Create tool
2. Bind tool to LLM
3. Add ToolNode
4. Add conditional routing

---

## Required Imports

```python
from langgraph.prebuilt import ToolNode, tools_condition
from langchain_openai import ChatOpenAI
```

---

# Full Working Example — Tool Usage

## Step 1 — Create Tool

```python
from langchain_core.tools import tool

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers"""
    return a * b
```

---

## Step 2 — Bind Tool to LLM

```python
llm = ChatOpenAI()
llm_with_tools = llm.bind_tools([multiply])
```

Now LLM can call the tool.

---

## Step 3 — Create LLM Node

```python
def chatbot(state):
    messages = state["messages"]
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}
```

---

## Step 4 — Create ToolNode

```python
tool_node = ToolNode([multiply])
```

ToolNode executes tool calls.

---

## Step 5 — Build Graph

```python
from langgraph.graph import StateGraph
from langgraph.graph.message import add_messages
from typing import TypedDict, Annotated
from langchain_core.messages import BaseMessage

class ChatState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]

builder = StateGraph(ChatState)

builder.add_node("chatbot", chatbot)
builder.add_node("tools", tool_node)

builder.set_entry_point("chatbot")
```

---

## Step 6 — Add Conditional Routing

```python
builder.add_conditional_edges(
    "chatbot",
    tools_condition
)
```

---

## Step 7 — Tool → Back to Chatbot

```python
builder.add_edge("tools", "chatbot")
```

---

## Step 8 — Compile

```python
graph = builder.compile()
```

---

# What is `tools_condition`

## Simple Definition

`tools_condition` is a prebuilt function that checks:

**"Did the LLM request a tool call?"**

Based on that, it decides where to route execution.

---

## Decision Logic

| LLM Output        | Route          |
| ----------------- | -------------- |
| Tool call present | Go to ToolNode |
| No tool call      | End execution  |

---

## Why It Exists

Without `tools_condition`, you must manually:

* inspect LLM output
* detect tool calls
* route graph

`tools_condition` does this automatically.

---

# How `tools_condition` Works Internally

## Step-by-Step

1. Reads latest AI message
2. Checks for `tool_calls`
3. If tool exists → route to `tools`
4. Else → stop

---

## Conceptual Logic

```
if ai_message.tool_calls:
    return "tools"
else:
    return END
```

---

# Flow with Tools + tools_condition

```
User Input
   ↓
Chatbot Node (LLM)
   ↓
Is tool needed?
   ↓ yes
ToolNode executes
   ↓
Return result
   ↓
Chatbot generates final answer
```

---

# Full Example — Complete Working Graph

```python
from langgraph.graph import StateGraph
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
from typing import TypedDict, Annotated
from langchain_core.messages import BaseMessage

# ------------------ TOOL ------------------

@tool
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

# ------------------ STATE ------------------

class ChatState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]

# ------------------ LLM ------------------

llm = ChatOpenAI()
llm_with_tools = llm.bind_tools([add])

# ------------------ CHAT NODE ------------------

def chatbot(state: ChatState):
    messages = state["messages"]
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}

# ------------------ TOOL NODE ------------------

tool_node = ToolNode([add])

# ------------------ GRAPH ------------------

builder = StateGraph(ChatState)

builder.add_node("chatbot", chatbot)
builder.add_node("tools", tool_node)

builder.set_entry_point("chatbot")

builder.add_conditional_edges(
    "chatbot",
    tools_condition
)

builder.add_edge("tools", "chatbot")

graph = builder.compile()
```

---

# What Happens During Execution

## Case 1 — User asks normal question

```
User: hello
→ LLM answers directly
→ tools_condition routes to END
```

---

## Case 2 — User asks calculation

```
User: add 5 and 10
→ LLM calls tool
→ ToolNode executes
→ Result returns
→ LLM formats answer
```

---

# Important Notes

## 1. Tools must have clear descriptions

LLM decides based on description.

---

## 2. Always connect tools back to chatbot

Otherwise execution stops after tool.

---

## 3. tools_condition only checks for tool calls

It does NOT execute tools.

---

## 4. ToolNode executes tools

Execution happens only inside ToolNode.

---

# Summary

## Core Concepts

* Tools = external functions for LLM
* ToolNode = executes tools
* tools_condition = routing logic
* LLM decides when to use tools

---

## One Line Memory

**LLM decides → tools_condition routes → ToolNode executes → LLM responds.**

---

# End of Document
