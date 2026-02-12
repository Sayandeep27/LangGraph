# LangGraph Subgraphs — Complete Beginner Guide

---

# What are Subgraphs in LangGraph?

## Simple Meaning

A **Subgraph** in LangGraph is a **graph inside another graph**.

It helps you:

* Break large workflows into smaller reusable parts
* Organize complex pipelines
* Reuse logic multiple times
* Keep code clean and modular

Think of it like:

```
Main Graph = Big Machine
Subgraph = Small Machine inside it
```

The main graph calls the subgraph → subgraph runs → returns result → main graph continues.

---

# Why Use Subgraphs?

## Problem Without Subgraphs

When building large AI workflows:

* Code becomes messy
* Nodes become too large
* Hard to reuse logic
* Difficult to maintain

## Solution → Subgraphs

Subgraphs allow:

* Code reuse
* Cleaner structure
* Separation of logic
* Modular design

---

# Real Life Analogy

## Making Tea ☕

Main process:

1. Boil water
2. Prepare tea mixture
3. Serve tea

But **prepare tea mixture** itself has steps:

* Add tea
* Add sugar
* Add milk

Instead of writing these steps in main process → we make a **sub-process**.

That sub-process = **Subgraph**.

---

# How Subgraphs Work Internally

## Flow

```
START
   ↓
Main Graph Node
   ↓
Call Subgraph
   ↓
Subgraph Executes
   ↓
Return Output
   ↓
Main Graph Continues
   ↓
END
```

---

# Basic Components of Subgraphs

| Component   | Meaning                                     |
| ----------- | ------------------------------------------- |
| State       | Shared data between main graph and subgraph |
| Subgraph    | Smaller graph with its own nodes            |
| Compile     | Convert graph into runnable object          |
| Add as Node | Subgraph becomes node in main graph         |

---

# Important Concept (VERY IMPORTANT)

## Subgraph = Runnable

When you compile a graph:

```
subgraph = builder.compile()
```

It becomes a **runnable object**.

This runnable can be used like a node inside another graph.

This is the key idea behind subgraphs.

---

# Example 1 — Basic Subgraph Example (Step-by-Step)

## Goal

We will build:

* Subgraph → calculates square
* Main graph → uses subgraph

---

# Step 1 — Install Libraries

```python
pip install langgraph
```

---

# Step 2 — Import Libraries

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict
```

---

# Step 3 — Define State

State is shared between main graph and subgraph.

```python
class NumberState(TypedDict):
    number: int
    result: int
```

Explanation:

* number → input value
* result → output

---

# Step 4 — Create Subgraph Nodes

## Node to calculate square

```python
def square_node(state: NumberState):
    num = state["number"]
    return {"result": num * num}
```

---

# Step 5 — Build Subgraph

```python
sub_builder = StateGraph(NumberState)

sub_builder.add_node("square", square_node)

sub_builder.add_edge(START, "square")
sub_builder.add_edge("square", END)

square_subgraph = sub_builder.compile()
```

What happened here:

* Created small graph
* Added node
* Connected START → node → END
* Compiled graph

Now `square_subgraph` is runnable.

---

# Step 6 — Create Main Graph

```python
main_builder = StateGraph(NumberState)

# add subgraph as node
main_builder.add_node("square_process", square_subgraph)

main_builder.add_edge(START, "square_process")
main_builder.add_edge("square_process", END)

main_graph = main_builder.compile()
```

Important:

```
main_builder.add_node("square_process", square_subgraph)
```

Subgraph behaves exactly like a node.

---

# Step 7 — Run Graph

```python
result = main_graph.invoke({"number": 5})
print(result)
```

Output:

```
{'number': 5, 'result': 25}
```

---

# What Happened Internally

```
Main Graph
   ↓
Calls Subgraph
   ↓
Square Node Executes
   ↓
Returns Result
```

---

# Example 2 — Subgraph with Multiple Nodes

## Goal

Subgraph will:

1. Add 10
2. Multiply by 2

---

# Step 1 — State

```python
class CalcState(TypedDict):
    value: int
```

---

# Step 2 — Nodes

```python
def add_ten(state: CalcState):
    return {"value": state["value"] + 10}


def multiply_two(state: CalcState):
    return {"value": state["value"] * 2}
```

---

# Step 3 — Build Subgraph

```python
calc_builder = StateGraph(CalcState)

calc_builder.add_node("add", add_ten)
calc_builder.add_node("multiply", multiply_two)

calc_builder.add_edge(START, "add")
calc_builder.add_edge("add", "multiply")
calc_builder.add_edge("multiply", END)

calc_subgraph = calc_builder.compile()
```

Flow:

```
START → add → multiply → END
```

---

# Step 4 — Use in Main Graph

```python
main_builder = StateGraph(CalcState)

main_builder.add_node("calculation", calc_subgraph)

main_builder.add_edge(START, "calculation")
main_builder.add_edge("calculation", END)

main_graph = main_builder.compile()

print(main_graph.invoke({"value": 5}))
```

Output:

```
{'value': 30}
```

Why?

```
5 + 10 = 15
15 * 2 = 30
```

---

# Example 3 — Real AI Use Case (LLM Pipeline)

## Real Scenario

Main graph:

* Generate answer
* Evaluate answer

Evaluation logic → subgraph.

---

## State

```python
class AIState(TypedDict):
    text: str
    score: int
```

---

## Subgraph — Evaluation Pipeline

```python
def evaluate_text(state: AIState):
    text = state["text"]
    score = len(text)
    return {"score": score}


eval_builder = StateGraph(AIState)

eval_builder.add_node("evaluate", evaluate_text)
eval_builder.add_edge(START, "evaluate")
eval_builder.add_edge("evaluate", END)

eval_subgraph = eval_builder.compile()
```

---

## Main Graph

```python
main_builder = StateGraph(AIState)

main_builder.add_node("evaluation", eval_subgraph)

main_builder.add_edge(START, "evaluation")
main_builder.add_edge("evaluation", END)

main_graph = main_builder.compile()

print(main_graph.invoke({"text": "Hello world"}))
```

---

# Key Rules of Subgraphs

## Rule 1 — Same State Structure

Main graph and subgraph must share compatible state fields.

---

## Rule 2 — Compile First

Always compile subgraph before using.

```
subgraph = builder.compile()
```

---

## Rule 3 — Subgraph Acts Like Node

```
main_builder.add_node("name", subgraph)
```

---

## Rule 4 — State Flows Automatically

No need to manually pass variables.

---

# When Should You Use Subgraphs?

Use when:

* Workflow is large
* Repeating logic exists
* Multi-step processing needed
* LLM pipelines
* Evaluation loops
* Tool pipelines
* Agent workflows

---

# Advantages of Subgraphs

| Advantage    | Benefit                   |
| ------------ | ------------------------- |
| Reusable     | Write once use many times |
| Modular      | Clean structure           |
| Maintainable | Easy debugging            |
| Scalable     | Works for large pipelines |
| Organized    | Less messy code           |

---

# Subgraph vs Normal Node

| Feature                 | Node    | Subgraph   |
| ----------------------- | ------- | ---------- |
| Complexity              | Simple  | Multi-step |
| Reusable                | Limited | High       |
| Internal Flow           | No      | Yes        |
| Contains Multiple Nodes | No      | Yes        |

---

# Common Beginner Mistakes

## ❌ Using uncompiled subgraph

Wrong:

```python
main_builder.add_node("x", sub_builder)
```

Correct:

```python
main_builder.add_node("x", sub_builder.compile())
```

---

## ❌ State mismatch

Main and subgraph states must align.

---

# Visual Architecture

```
MAIN GRAPH

START
  ↓
Node A
  ↓
Subgraph
   ├─ Node 1
   ├─ Node 2
   └─ Node 3
  ↓
Node B
  ↓
END
```

---

# Summary (Must Remember)

* Subgraph = graph inside graph
* Used for modular workflows
* Compile subgraph first
* Add to main graph like node
* State flows automatically
* Helps build complex AI systems

---

# You Now Understand

* What subgraphs are
* Why they exist
* How they work
* How to build them
* Real examples
* Best practices

---

**End of Guide**
