# ChatState in LangGraph — Complete Guide

---

## Overview

This guide explains the following LangGraph state definition in a **simple and clear way**:

```python
class ChatState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]
```

This README covers:

* What `TypedDict` is
* What `messages` represents
* What `list[BaseMessage]` means
* What `Annotated[..., add_messages]` does
* Why `add_messages` is needed
* How LangGraph merges messages
* Full working example (step-by-step)

---

# Table of Contents

1. [What is TypedDict?](#what-is-typeddict)
2. [What is messages?](#what-is-messages)
3. [What is list[BaseMessage]?](#what-is-listbasemessage)
4. [What is Annotated[..., add_messages]?](#what-is-annotated-add_messages)
5. [How Everything Works Together](#how-everything-works-together)
6. [Full Example (Complete Workflow)](#full-example-complete-workflow)
7. [Without add_messages vs With add_messages](#without-add_messages-vs-with-add_messages)
8. [Real Life Analogy](#real-life-analogy)
9. [One Line Summary](#one-line-summary)

---

# What is TypedDict?

## Simple Meaning

`TypedDict` defines the **structure of a dictionary**.

It tells Python:

* What keys must exist
* What type of values each key stores

It is like defining rules for a dictionary.

---

## Normal Dictionary (No Rules)

```python
data = {
    "name": "Sam",
    "age": 25
}
```

Anything can be stored.

---

## TypedDict (With Rules)

```python
from typing import TypedDict

class Person(TypedDict):
    name: str
    age: int
```

Now Python expects:

```python
p: Person = {"name": "Sam", "age": 25}
```

---

## Key Idea

```
TypedDict = Dictionary with predefined structure
```

---

# What is messages?

```python
messages: ...
```

This means:

* The state must contain a key called `"messages"`
* It stores conversation history

---

## State Structure

```python
{
  "messages": ...
}
```

---

# What is list[BaseMessage]?

## Meaning

```
messages = list of message objects
```

---

## What is BaseMessage?

In LangChain, all message types inherit from `BaseMessage`.

| Message Type  | Purpose               |
| ------------- | --------------------- |
| HumanMessage  | User input            |
| AIMessage     | AI response           |
| SystemMessage | Instructions to model |

---

## Example

```python
from langchain_core.messages import HumanMessage, AIMessage

HumanMessage(content="Hello")
AIMessage(content="Hi!")
```

---

## So This Means

```python
list[BaseMessage]
```

Example structure:

```python
[
  HumanMessage("Hello"),
  AIMessage("Hi!")
]
```

---

# What is Annotated[..., add_messages]?

## Most Important Part

```python
Annotated[list[BaseMessage], add_messages]
```

This tells LangGraph **how to update state values**.

---

## Problem Without It

Normally when state updates:

```
old value → replaced by new value
```

Chat history would disappear.

---

## What add_messages Does

```
new messages → appended to existing messages
```

It performs:

```
old_messages + new_messages
```

---

## Why Needed

Chat systems need memory:

```
User message
AI reply
User message
AI reply
```

History must persist.

---

# How Everything Works Together

This definition:

```python
class ChatState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]
```

means:

* State is a structured dictionary
* It must contain `messages`
* Messages store chat history
* New messages are automatically appended

---

# Full Example (Complete Workflow)

## Step 1 — Install Dependencies

```bash
pip install langgraph langchain langchain-core
```

---

## Step 2 — Define Chat State

```python
from typing import TypedDict, Annotated
from langchain_core.messages import BaseMessage
from langgraph.graph.message import add_messages

class ChatState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]
```

---

## Step 3 — Create Initial State

```python
state = {
    "messages": []
}
```

---

## Step 4 — User Sends Message

```python
from langchain_core.messages import HumanMessage

update = {
    "messages": [HumanMessage(content="Hello")]
}
```

LangGraph automatically merges:

```
[] + ["Hello"]
```

State becomes:

```python
{
  "messages": [HumanMessage(content="Hello")]
}
```

---

## Step 5 — AI Responds

```python
from langchain_core.messages import AIMessage

update = {
    "messages": [AIMessage(content="Hi! How can I help?")]
}
```

LangGraph merges again:

```
["Hello"] + ["Hi! How can I help?"]
```

Final state:

```python
{
  "messages": [
      HumanMessage(content="Hello"),
      AIMessage(content="Hi! How can I help?")
  ]
}
```

---

# Without add_messages vs With add_messages

## Without add_messages

| Step               | Result               |
| ------------------ | -------------------- |
| User sends message | Stored               |
| AI replies         | User message deleted |

---

## With add_messages

| Step               | Result               |
| ------------------ | -------------------- |
| User sends message | Stored               |
| AI replies         | Both messages stored |

---

# Real Life Analogy

Think of a WhatsApp chat.

Without `add_messages`:

```
New message → old chat deleted
```

With `add_messages`:

```
New message → added to chat history
```

---

# One Line Summary

```
messages: Annotated[list[BaseMessage], add_messages]
```

means:

```
Store chat history and automatically append new messages instead of replacing old ones.
```

---

# License

This project is for educational purposes.
