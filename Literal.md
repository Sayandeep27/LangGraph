# Python `Literal` — Complete Guide

---

## ✅ What is Literal in Python?

**Literal** is a type hint from the `typing` module that means:

> This variable can only have specific fixed values.

It restricts a variable to **exact values**, not just a type.

---

## 📌 Basic Example

```python
from typing import Literal

def set_mode(mode: Literal["train", "test", "eval"]):
    print(f"Mode is {mode}")
```

Now:

```python
set_mode("train")   # ✅ allowed
set_mode("test")    # ✅ allowed
set_mode("hello")   # ❌ type checker error
```

### Here:

```python
Literal["train", "test", "eval"]
```

### Means:

`mode` must be exactly one of these 3 strings.

---

## 🔎 Difference Between `str` and `Literal`

| Feature           | `str` (Normal Type Hint) | `Literal`                        |
| ----------------- | ------------------------ | -------------------------------- |
| Meaning           | Any string allowed       | Only specific values allowed     |
| Restriction Level | Loose                    | Strict                           |
| Safety            | Low                      | High                             |
| Example           | `mode: str`              | `mode: Literal["train", "test"]` |

### Normal Type Hint

```python
mode: str
```

Means:

Any string is allowed.

### Literal Type Hint

```python
mode: Literal["train", "test"]
```

Means:

Only `"train"` or `"test"` allowed.

---

## 🧠 Why Is It Useful?

* Prevents invalid values
* Makes code safer
* Improves autocomplete in editors
* Very useful in state machines (like LangGraph)

---

## 🚀 Literal in LangGraph (Important for You)

In LangGraph, you often see:

```python
from typing import Literal

def route(state) -> Literal["node_a", "node_b"]:
    if state["score"] > 5:
        return "node_a"
    return "node_b"
```

### This tells LangGraph:

This function will only return either `"node_a"` or `"node_b"`.

So it can validate routing at compile time.

---

## 🔥 Another Example with Numbers

```python
def get_rating(level: Literal[1, 2, 3]):
    return level
```

Only `1`, `2`, or `3` allowed.

---

## 📌 Important

**Literal is:**

* Only for type hints
* Checked by static type checkers (like mypy, VSCode)
* Not enforced at runtime automatically

If you want runtime enforcement → use **Pydantic**.

---

## 🆚 Literal vs Enum (Common Confusion)

You could also use:

```python
from enum import Enum
```

### Enum is better when:

* You want named constants
* You want runtime safety

### Literal is:

* Lighter
* Used mostly for typing

---

## 🧠 In One Line

> **Literal means:**
>
> **"This variable must be exactly one of these specific values."**

---
