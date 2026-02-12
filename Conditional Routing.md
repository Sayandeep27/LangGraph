# Conditional Routing in LangGraph

---

## Overview

**Conditional routing in LangGraph** allows workflows to dynamically decide which node to execute next based on specific conditions. Instead of a fixed execution flow, LangGraph enables intelligent decision-making similar to `if–else` logic in programming.

This is especially useful when building:

* AI agents
* LLM workflows
* Multi-step pipelines
* Validation systems
* Tool selection logic
* Classification pipelines
* Error handling flows

---

## Key Concepts

### What is Conditional Routing?

Conditional routing allows a graph to choose different execution paths based on the current state.

### Fixed Flow (No Decision)

```
START → A → B → END
```

### Conditional Flow (Dynamic Decision)

```
START → Decision
           ├── Path A
           └── Path B
```

---

## Real-World Use Cases

| Use Case            | Description                                  |
| ------------------- | -------------------------------------------- |
| Intent Detection    | Route user queries to different handlers     |
| Document Validation | Process valid documents, reject invalid ones |
| Tool Selection      | AI agent chooses correct tool                |
| Classification      | Route based on predicted class               |
| Error Handling      | Retry or fallback logic                      |
| RAG Systems         | Route queries to different retrievers        |

---

## Core Components of Conditional Routing

Conditional routing in LangGraph requires three components:

### 1. State

Shared data container that flows through nodes.

### 2. Decision Function (Router)

Determines which node should execute next.

### 3. Routing Map

Maps decision outputs to target nodes.

---

## Installation

```bash
pip install langgraph
```

---

## Step-by-Step Implementation

This example demonstrates **BMI classification using conditional routing**.

---

## Step 1 — Define State

```python
from typing import TypedDict

class BMIState(TypedDict):
    weight_kg: float
    height_m: float
    bmi: float
    category: str
```

---

## Step 2 — Create Processing Nodes

Nodes modify or update the state.

### Calculate BMI

```python
def calculate_bmi(state: BMIState):
    weight = state["weight_kg"]
    height = state["height_m"]

    bmi = weight / (height ** 2)
    state["bmi"] = round(bmi, 2)

    return state
```

### Underweight Handler

```python
def underweight_node(state: BMIState):
    state["category"] = "Underweight"
    return state
```

### Normal Handler

```python
def normal_node(state: BMIState):
    state["category"] = "Normal"
    return state
```

### Overweight Handler

```python
def overweight_node(state: BMIState):
    state["category"] = "Overweight"
    return state
```

---

## Step 3 — Create Decision Function (Router)

The router checks the state and returns a label.

```python
def bmi_router(state: BMIState):
    bmi = state["bmi"]

    if bmi < 18.5:
        return "underweight"
    elif bmi < 25:
        return "normal"
    else:
        return "overweight"
```

---

## Step 4 — Build the Graph

```python
from langgraph.graph import StateGraph, END

builder = StateGraph(BMIState)

builder.add_node("calculate", calculate_bmi)
builder.add_node("underweight", underweight_node)
builder.add_node("normal", normal_node)
builder.add_node("overweight", overweight_node)

builder.set_entry_point("calculate")
```

---

## Step 5 — Add Conditional Routing

```python
builder.add_conditional_edges(
    "calculate",
    bmi_router,
    {
        "underweight": "underweight",
        "normal": "normal",
        "overweight": "overweight"
    }
)
```

---

## Step 6 — Connect Nodes to END

```python
builder.add_edge("underweight", END)
builder.add_edge("normal", END)
builder.add_edge("overweight", END)
```

---

## Step 7 — Compile and Run

```python
graph = builder.compile()

result = graph.invoke({
    "weight_kg": 70,
    "height_m": 1.75
})

print(result)
```

---

## Execution Flow

```
START
   ↓
calculate_bmi
   ↓
bmi_router decision
   ├── underweight → underweight_node → END
   ├── normal → normal_node → END
   └── overweight → overweight_node → END
```

---

## How `add_conditional_edges()` Works

1. A node completes execution.
2. Router function evaluates state.
3. Router returns a label.
4. Label maps to a node.
5. Next node executes.

---

## Conditional vs Normal Edges

| Feature        | Normal Edge          | Conditional Edge          |
| -------------- | -------------------- | ------------------------- |
| Flow           | Fixed                | Dynamic                   |
| Decision Logic | No                   | Yes                       |
| Syntax         | `add_edge()`         | `add_conditional_edges()` |
| Use Case       | Sequential workflows | Intelligent routing       |

---

## Important Rules

* Router must return a string label.
* Label must exist in routing map.
* Router reads state but should not modify it.
* One node can route to multiple nodes.

---

## Advanced Routing Patterns

* Multi-level routing
* AI agent tool selection
* Error retry workflows
* Validation pipelines
* Multi-agent systems
* Parallel + conditional execution

---

## Mental Model

```
Node = Worker
Router = Decision Manager
State = Shared Memory
```

---

## When to Use Conditional Routing

Use conditional routing when your workflow requires dynamic decisions based on data, user input, or model outputs.

---

## License

This project is for educational purposes.

---
