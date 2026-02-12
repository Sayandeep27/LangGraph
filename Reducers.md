# LangGraph Reducers — Complete Guide

---


```python
class ChatState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]
    count: Annotated[int, operator.add]
```

---

# What is a Reducer in LangGraph?

A **reducer** is a function that decides:

> **How a state value should be updated when multiple nodes write to the same key.**

In simple words:

A reducer defines **how to merge old state + new state**.

---

## Simple Definition

```
Reducer = Rule for merging state updates
```

---

# Why Reducers Are Needed

LangGraph workflows run like a graph:

```
START
  ├── Node A
  └── Node B
```

Multiple nodes may update the same state variable.

Example:

* Node A → adds "Hello"
* Node B → adds "World"

Now the system must decide:

* Overwrite?
* Append?
* Add?
* Keep both?

**Reducer decides this behavior.**

---

# Default Behavior (Without Reducer)

If no reducer is defined:

```
Last value wins (overwrite)
```

### Example

```python
from typing import TypedDict

class State(TypedDict):
    count: int
```

```
Node A → {"count": 1}
Node B → {"count": 2}

Final count = 2
```

Because the value gets overwritten.

---

# How Reducers Work Internally

A reducer is a function:

```
(old_value, new_value) → merged_value
```

LangGraph calls this function whenever a state key receives multiple updates.

---

# How to Define Reducers in LangGraph

Reducers are defined using `Annotated` from Python typing.

```python
from typing import Annotated
```

Syntax:

```python
state_key: Annotated[type, reducer_function]
```

---

# Example 1 — Number Addition Using Reducer

## State Definition

```python
from typing import TypedDict, Annotated
import operator

class State(TypedDict):
    count: Annotated[int, operator.add]
```

## What Happens

```
old_value + new_value
```

## Node Updates

```
Node A → count = 1
Node B → count = 2
```

## Final Result

```
count = 3
```

---

# Example 2 — List Appending (Chat Messages)

## State Definition

```python
from typing import TypedDict, Annotated
from operator import add

class ChatState(TypedDict):
    messages: Annotated[list, add]
```

## Behavior

Lists are combined.

```
["Hello"] + ["Hi"] → ["Hello", "Hi"]
```

---

# VVI EXAMPLE — IMPORTANT (Do Not Skip)

## State Definition

```python
class ChatState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]
    count: Annotated[int, operator.add]
```

---

## What Are Reducers Here?

In this state definition, **both fields use reducers**.

| State Key | Type              | Reducer Used | What It Does                |
| --------- | ----------------- | ------------ | --------------------------- |
| messages  | list[BaseMessage] | add_messages | Merges chat messages safely |
| count     | int               | operator.add | Adds numbers                |

---

## 1. Reducer: `add_messages`

### What it is

`add_messages` is a special reducer provided by LangGraph for chat systems.

### What it does

It:

* Appends new messages
* Maintains conversation history
* Avoids message overwriting
* Handles message ordering
* Prevents duplicate message conflicts

### How it works conceptually

```
old_messages + new_messages → combined_messages
```

### Example

```
Existing → ["Hello"]
New → ["How are you?"]

Final → ["Hello", "How are you?"]
```

### Why not use normal `add`?

Because chat messages:

* Have roles (user, AI, system)
* Have metadata
* Require proper merging
* Must preserve order

`add_messages` handles all this automatically.

---

## 2. Reducer: `operator.add`

### What it is

Python's built-in addition function.

### What it does

```
old_count + new_count
```

### Example

```
Existing → 5
New → 3

Final → 8
```

So `count` keeps increasing instead of being replaced.

---

# Visual Understanding

## Without Reducer

```
count = 1 → overwritten → 2
```

## With Reducer

```
count = 1 + 2 = 3
```

---

# Built-in Reducers (Common Ones)

| Reducer        | Purpose              |
| -------------- | -------------------- |
| operator.add   | Add numbers or lists |
| add_messages   | Merge chat messages  |
| Custom reducer | Any custom logic     |

---

# Custom Reducer Example

You can define your own merging rule.

## Keep Maximum Value Only

```python
def max_reducer(old, new):
    return max(old, new)

class State(TypedDict):
    score: Annotated[int, max_reducer]
```

Now the state always keeps the highest value.

---

# Real-World Use Cases

Reducers are commonly used in:

* LLM chat history management
* Parallel workflow execution
* Multi-agent systems
* Accumulating results
* Logging
* Metrics tracking
* Streaming responses

---

# Key Takeaways

* Reducer defines how state updates merge.
* Default behavior is overwrite.
* Reducers prevent data loss.
* Important for parallel node execution.
* Defined using `Annotated`.

---

# One-Line Summary

> **Reducer in LangGraph is a function that defines how state values are merged when multiple nodes update the same key.**

---

# End of Guide
