# Agent Loops & Cycles in LangGraph

---

# 1. What is an Agent in LangGraph?

An **Agent** is a system that:

* Thinks
* Decides
* Acts
* Observes results
* Repeats until goal achieved

Unlike normal programs:

```
input → process → output → end
```

Agents work like:

```
think → act → observe → think again → repeat
```

This repeating behavior requires **loops**.

---

# 2. What is a Loop?

A **Loop** means:

> The graph runs the same steps again and again until a condition is met.

In LangGraph:

* Nodes can connect back to previous nodes
* Execution repeats
* Stops when condition says "done"

### Simple Idea

```
Ask → Check → Fix → Check → Fix → Done
```

---

# 3. What is a Cycle in LangGraph?

A **Cycle** is the structure that creates loops.

## Definition

A **cycle** happens when:

```
Node A → Node B → Node C → Node A
```

The graph has a circular path.

### Important

* **Cycle = graph structure**
* **Loop = repeated execution caused by cycle**

---

# 4. Why Agent Loops Are Needed

LLM agents rarely solve tasks in one step.

They must:

* Try
* Check
* Correct
* Retry

### Example Problems

* Answer refinement
* Tool usage
* Error correction
* Planning
* Self evaluation

Without loops:

```
LLM runs once → stops
```

With loops:

```
LLM improves answer repeatedly
```

---

# 5. How Loops Work Internally

LangGraph uses:

* State
* Conditional routing
* Cyclic edges

## Execution Steps

### Step 1 — Node updates state

```
state → node → new state
```

### Step 2 — Router checks state

```
if done → END
else → run again
```

### Step 3 — Graph follows edge

```
node → router → node again
```

### Step 4 — Repeat

---

# 6. Simple Flow Comparison

## Normal Graph (No Loop)

```
START → A → B → END
```

Runs once.

---

## Graph with Cycle

```
START → A → B
          ↑   ↓
          ← C
```

Runs repeatedly.

---

# 7. Basic Agent Loop Example

## Goal

Improve text until it becomes "good".

---

## Step 1 — Define State

```python
from typing import TypedDict

class AgentState(TypedDict):
    text: str
    quality: str
```

---

## Step 2 — Generator Node

```python
def generate(state: AgentState):
    state["text"] += " improved"
    return state
```

---

## Step 3 — Checker Node

```python
def check_quality(state: AgentState):
    if len(state["text"]) > 30:
        state["quality"] = "good"
    else:
        state["quality"] = "bad"
    return state
```

---

## Step 4 — Router

```python
def should_continue(state: AgentState):
    if state["quality"] == "good":
        return "end"
    return "retry"
```

---

## Step 5 — Build Graph with Cycle

```python
from langgraph.graph import StateGraph, END

builder = StateGraph(AgentState)

builder.add_node("generate", generate)
builder.add_node("check", check_quality)

builder.set_entry_point("generate")

builder.add_edge("generate", "check")

builder.add_conditional_edges(
    "check",
    should_continue,
    {
        "retry": "generate",
        "end": END
    }
)

graph = builder.compile()
```

---

## Execution Flow

```
generate → check → generate → check → ... → END
```

This is an **agent loop**.

---

# 8. Tool‑Using Agent Loop (Real Agent Behavior)

Real agents:

* Think
* Decide tool
* Execute tool
* Observe result
* Repeat

---

## State

```python
class ToolState(TypedDict):
    question: str
    answer: str
    need_tool: bool
```

---

## LLM Decide Node

```python
def agent(state: ToolState):
    if "weather" in state["question"]:
        state["need_tool"] = True
    else:
        state["answer"] = "Direct answer"
        state["need_tool"] = False
    return state
```

---

## Tool Node

```python
def tool(state: ToolState):
    state["answer"] = "Weather is 25°C"
    return state
```

---

## Router

```python
def route(state: ToolState):
    if state["need_tool"]:
        return "tool"
    return "end"
```

---

## Graph

```python
builder = StateGraph(ToolState)

builder.add_node("agent", agent)
builder.add_node("tool", tool)

builder.set_entry_point("agent")

builder.add_conditional_edges(
    "agent",
    route,
    {
        "tool": "tool",
        "end": END
    }
)

builder.add_edge("tool", "agent")

graph = builder.compile()
```

---

## Flow

```
agent → tool → agent → tool → ... → END
```

This mimics real LLM agents.

---

# 9. Conditional Loop Control

Loops must always have a stop condition.

## Common Stop Conditions

* Quality threshold reached
* Max iterations
* Tool finished
* Answer validated
* Error fixed

---

## Max Iteration Example

```python
class LoopState(TypedDict):
    count: int
```

```python
def should_continue(state: LoopState):
    if state["count"] > 5:
        return "end"
    return "retry"
```

---

# 10. Loop vs Normal Workflow

| Feature         | Normal Graph | Agent Loop |
| --------------- | ------------ | ---------- |
| Execution       | Once         | Repeated   |
| Structure       | Linear       | Cyclic     |
| Decision making | Minimal      | Continuous |
| Adaptability    | Low          | High       |
| Intelligence    | Low          | High       |

---

# 11. Real World Analogy

## Human Problem Solving

```
Try solving math
Check answer
Wrong → try again
Correct → stop
```

This is a loop.

---

## Washing Machine

```
Wash → Check dirt → Wash again → Done
```

---

# 12. Best Practices

## Always Add Stop Condition

Prevents infinite loops.

---

## Track Iterations

```
state["count"] += 1
```

---

## Keep State Small

Avoid memory explosion.

---

## Make Routing Clear

Easy debugging.

---

# 13. Common Mistakes

## No Stop Condition

Graph never ends.

---

## Infinite Tool Calls

Agent repeatedly calls tool.

---

## State Not Updated

Loop never changes behavior.

---

## Wrong Conditional Mapping

Edges never reached.

---

# 14. Summary

## Core Idea

```
Cycle → enables loop
Loop → enables agent behavior
```

---

## Key Concepts

* Cycles create repeated execution
* Loops enable thinking agents
* Conditional routing controls loops
* State drives decisions
* Stop condition is mandatory

---

# You Now Understand

* Agent loops
* Cycles
* Conditional routing
* Iterative execution
* Tool calling agents
* Loop control

---

**End of Guide**
