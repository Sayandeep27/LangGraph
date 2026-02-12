# LangGraph — Checkpointing & Persistence (Complete Guide)
---

# What is Checkpointing?

**Checkpointing means saving the current state of a graph while it is running so it can be restored later.**

It stores:

* state values
* conversation history
* intermediate results
* progress of execution

So if execution stops, you can resume from where it stopped.

---

## Simple Meaning

Checkpoint = **Saved progress of a workflow**.

Just like:

* Saving a game
* Saving a document draft
* Saving a video editing project

---

# Why Checkpointing is Needed

Without checkpointing:

* State disappears after execution
* Cannot resume work
* No memory across runs
* No long conversations

With checkpointing:

* Resume execution anytime
* Maintain conversation memory
* Store workflow progress
* Build production systems

---

# What is Persistence?

**Persistence means storing data permanently so it survives program restarts.**

Examples:

* Database storage
* Disk storage
* File storage

---

## Simple Meaning

Persistence = **Data stays even if program stops**.

---

# Checkpointing vs Persistence

| Feature   | Checkpointing       | Persistence          |
| --------- | ------------------- | -------------------- |
| Purpose   | Save workflow state | Store data long term |
| When used | During execution    | Across sessions      |
| Storage   | Memory / DB         | Disk / DB            |
| Use case  | Resume execution    | Long term storage    |

In LangGraph:

* Checkpointing → saves graph state
* Persistence → keeps checkpoints stored permanently

---

# How Checkpointing Works in LangGraph

LangGraph saves the **graph state after each step**.

Flow:

1. Node runs
2. State updates
3. Checkpointer saves state
4. Next node runs

If program crashes → resume from saved state.

---

# What is a Checkpointer?

A **checkpointer** is a component that saves and loads graph state.

It decides:

* Where state is stored
* How state is retrieved
* How conversations persist

---

## Think of it like

* RAM → MemorySaver
* File → SQLite
* Cloud database → Postgres

---

# Types of Checkpointers in LangGraph

---

# 1. MemorySaver (In‑Memory Checkpointer)

## What it is

Stores state in RAM.

---

## Characteristics

* Very fast
* Temporary storage
* Lost when program stops
* Used for testing

---

## Example

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph

checkpointer = MemorySaver()

app = graph.compile(checkpointer=checkpointer)
```

---

## Behavior

* Works only while program runs
* Restart → data lost

---

## Use Case

* Learning
* Testing
* Temporary workflows

---

# 2. SQLite Checkpointer

## What it is

Stores checkpoints in a local SQLite database.

---

## Characteristics

* Persistent storage
* Survives restarts
* Stored in local file
* Production ready for small apps

---

## Example

```python
from langgraph.checkpoint.sqlite import SqliteSaver

checkpointer = SqliteSaver.from_conn_string("sqlite:///checkpoint.db")

app = graph.compile(checkpointer=checkpointer)
```

---

## Behavior

* State saved to file
* Can resume later

---

## Use Case

* Chatbots
* Local apps
* Small production systems

---

# 3. Postgres Checkpointer

## What it is

Stores checkpoints in PostgreSQL database.

---

## Characteristics

* Highly scalable
* Distributed systems
* Production ready
* Multi‑user support

---

## Example

```python
from langgraph.checkpoint.postgres import PostgresSaver

checkpointer = PostgresSaver.from_conn_string(
    "postgresql://user:pass@localhost/db"
)

app = graph.compile(checkpointer=checkpointer)
```

---

## Use Case

* Enterprise apps
* Production agents
* Multi‑user systems

---

# Step‑by‑Step Example (Without Checkpointing)

---

## State

```python
from typing import TypedDict

class CounterState(TypedDict):
    count: int
```

---

## Node

```python
def increment(state: CounterState):
    return {"count": state["count"] + 1}
```

---

## Graph

```python
from langgraph.graph import StateGraph

builder = StateGraph(CounterState)
builder.add_node("inc", increment)
builder.set_entry_point("inc")

app = builder.compile()
```

---

## Run

```python
app.invoke({"count": 0})
```

---

## Problem

* Next run starts from 0 again
* No memory

---

# Step‑by‑Step Example (With Checkpointing)

---

## Add Checkpointer

```python
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()

app = builder.compile(checkpointer=checkpointer)
```

---

## Run with Thread ID

```python
config = {"configurable": {"thread_id": "user1"}}

app.invoke({"count": 0}, config=config)
app.invoke({}, config=config)
```

---

## Result

* Second run continues previous state

---

# How Thread IDs Work

Thread ID = **unique session identifier**.

It tells LangGraph:

* Which conversation
* Which workflow state
* Which user session

---

## Example

```python
config = {"configurable": {"thread_id": "user_123"}}
```

---

## Why Important

* Multiple users
* Multiple sessions
* Independent memory

---

# Real‑World Analogy

---

## Google Docs

* Auto save → checkpointing
* Document stored in cloud → persistence

---

## Video Game

* Save game → checkpoint
* Game file on disk → persistence

---

# When to Use Which Checkpointer

| Scenario          | Recommended |
| ----------------- | ----------- |
| Learning          | MemorySaver |
| Testing           | MemorySaver |
| Small app         | SQLite      |
| Local chatbot     | SQLite      |
| Production system | Postgres    |
| Multi‑user app    | Postgres    |

---

# Key Takeaways

* Checkpointing saves workflow progress
* Persistence keeps data permanently
* Checkpointer manages storage
* MemorySaver → temporary
* SQLite → local persistence
* Postgres → production scale
* Thread ID manages sessions

---

# End of Guide
