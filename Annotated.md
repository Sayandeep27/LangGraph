# LangGraph — `Annotated` Explained (Complete Guide)

---

## Overview

In **LangGraph**, `Annotated` is used to add extra meaning (metadata or behavior) to a state field — especially to tell LangGraph how values should be updated or merged when multiple nodes modify the same state.

It comes from Python’s typing system:

```python
from typing import Annotated
```

---

## Think of it like:

> **"This variable has a type + special rule about how it behaves."**

---

# What is Annotated in LangGraph (Simple Meaning)

## Normal type hint

```python
count: int
```

→ Just says `count` is an integer.

---

## Annotated type hint

```python
count: Annotated[int, operator.add]
```

→ Says:

* count is an integer
* If multiple nodes update it → add the values together

---

## So Annotated =

```
type + extra instruction
```

---

# Why LangGraph Uses Annotated

LangGraph workflows often have:

* multiple nodes running
* multiple updates to the same state variable
* parallel execution
* merging outputs

So LangGraph needs to know:

> **If two nodes update the same field → what should happen?**

---

## Options could be:

* overwrite?
* add?
* append?
* merge?

`Annotated` tells LangGraph the rule.

---

# Basic Syntax in LangGraph

## Structure

```python
field_name: Annotated[data_type, update_rule]
```

---

## Parts

| Part        | Meaning                    |
| ----------- | -------------------------- |
| data_type   | Type of value              |
| update_rule | How updates should combine |

---

# Example 1 — Without Annotated (Default Behavior)

```python
from typing import TypedDict

class State(TypedDict):
    messages: list
```

## Behavior

* Last update wins
* Previous value replaced

```
["hello"] → ["bye"]
```

---

# Example 2 — Using Annotated (Append Behavior)

```python
from typing import Annotated, TypedDict
import operator

class State(TypedDict):
    messages: Annotated[list, operator.add]
```

## Behavior

* Values get combined

```
["hello"] + ["bye"]
= ["hello", "bye"]
```

---

## Very useful for:

* chat history
* logs
* accumulated results

---

# Real LangGraph Example (Important)

This is very commonly used.

```python
from typing import Annotated, TypedDict
from langgraph.graph import StateGraph
import operator

class ChatState(TypedDict):
    messages: Annotated[list, operator.add]
```

---

## What happens

If two nodes return:

```
Node A → ["Hi"]
Node B → ["How are you?"]
```

---

## Final state becomes:

```
["Hi", "How are you?"]
```

Instead of overwriting.

---

# How It Works Internally (Concept)

LangGraph stores:

```
(messages, operator.add)
```

---

## When updates happen:

```
new_value = operator.add(old_value, node_output)
```

So it follows the rule you defined.

---

# Common Update Rules Used with Annotated

## 1. Add numbers

```python
score: Annotated[int, operator.add]
```

```
10 + 5 = 15
```

---

## 2. Append lists (most common)

```python
items: Annotated[list, operator.add]
```

```
["a"] + ["b"] = ["a", "b"]
```

---

## 3. Custom function

You can define your own merge logic.

```python
def merge_dict(a, b):
    return {**a, **b}

data: Annotated[dict, merge_dict]
```

---

# When You Should Use Annotated in LangGraph

## Use it when:

* multiple nodes update same state
* you want accumulation
* parallel branches exist
* chat history storage
* logging results
* combining outputs

---

## Do NOT use when:

* only one node updates value
* simple overwrite is fine

---

# Simple Real-World Analogy

Imagine a shared notebook.

## Without Annotated

Last person writing → erases previous content.

## With Annotated

Each person → adds their notes.

---

# Difference: Normal vs Annotated

| Feature                   | Normal Type | Annotated |
| ------------------------- | ----------- | --------- |
| Defines data type         | Yes         | Yes       |
| Defines update behavior   | No          | Yes       |
| Used for merging state    | No          | Yes       |
| Needed for parallel nodes | No          | Yes       |

---

# One-Line Definition (Interview Ready)

> **In LangGraph, Annotated is used to specify how state variables should be merged or updated when multiple nodes modify the same state.**

---
