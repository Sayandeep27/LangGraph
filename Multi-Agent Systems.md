# Multi-Agent Systems in LangGraph
---

# Introduction

Modern AI applications are becoming complex. A single LLM agent often cannot handle everything efficiently.

Instead of one large agent doing all tasks, we can create **multiple specialized agents** that work together.

This is called a **Multi-Agent System**.

LangGraph provides a powerful way to build these systems using graphs, shared state, and message passing.

This guide explains everything from basics to advanced concepts with examples and working code.

---

# What is a Multi-Agent System?

A **Multi-Agent System (MAS)** is a system where multiple AI agents:

* Work together
* Share information
* Solve different parts of a problem
* Coordinate to achieve a goal

Each agent has a specific responsibility.

### Real Life Analogy

Think of a hospital:

* Reception → collects information
* Doctor → diagnoses
* Pharmacist → gives medicine

They collaborate to help the patient.

Similarly, multi-agent systems split work between AI agents.

---

# Why Multi-Agent Systems?

## Problems with Single Agent

* Too much responsibility
* Hard to control behavior
* Difficult to scale
* Poor specialization
* Complex prompts

## Benefits of Multiple Agents

| Feature          | Benefit                                      |
| ---------------- | -------------------------------------------- |
| Specialization   | Each agent is expert in one task             |
| Modularity       | Easy to add/remove agents                    |
| Scalability      | Handles complex workflows                    |
| Reliability      | One agent failure doesn't break whole system |
| Better reasoning | Agents can verify each other                 |

---

# What is LangGraph?

LangGraph is a framework to build **stateful AI workflows** using graphs.

It allows:

* Multiple nodes (agents)
* Shared state
* Conditional routing
* Memory handling
* Multi-step reasoning

### Key Idea

Graph = Nodes + Edges + State

* Nodes → agents/functions
* Edges → control flow
* State → shared memory

Multi-agent systems are just multiple agent nodes connected together.

---

# Multi-Agent Systems in LangGraph (Core Idea)

In LangGraph:

* Each agent = a node
* Agents read/write shared state
* Graph controls communication
* Messages pass between agents

### Flow

User → Agent A → Agent B → Agent C → Output

Agents collaborate through shared state.

---

# Architecture Overview

```
User Input
   ↓
Shared State
   ↓
Agent 1 → Agent 2 → Agent 3
   ↓
Final Output
```

### Components

* State → shared data
* Nodes → agents
* Edges → communication
* Reducers → state merging

---

# Key Components

## 1. State

State stores information shared between agents.

Example:

```
from typing import TypedDict

class AgentState(TypedDict):
    task: str
    result: str
```

---

## 2. Agents as Nodes

Agents are normal Python functions.

They:

* Read state
* Process information
* Update state

```
def agent_a(state: AgentState):
    state["result"] = "Processed by Agent A"
    return state
```

---

## 3. Edges

Edges define which agent runs next.

```
graph.add_edge("agent_a", "agent_b")
```

---

## 4. Shared State Communication

Agents communicate by updating the same state.

---

# How Agents Communicate

There are three main ways:

## 1. Shared State

Agents read/write common data.

## 2. Message Passing

Agents pass messages between each other.

## 3. Supervisor Routing

One agent decides next agent.

---

# Types of Multi-Agent Patterns

| Pattern         | Description                    |
| --------------- | ------------------------------ |
| Sequential      | Agents run one after another   |
| Supervisor      | One agent controls others      |
| Collaborative   | Agents update shared state     |
| Competitive     | Agents evaluate each other     |
| Tool Specialist | Each agent uses specific tools |

---

# Example 1 — Simple Two-Agent System

## Goal

* Agent 1 → analyzes text
* Agent 2 → summarizes text

---

## Installation

```
pip install langgraph langchain openai
```

---

## Code

```
from langgraph.graph import StateGraph, START, END
from typing import TypedDict

class TextState(TypedDict):
    text: str
    analysis: str
    summary: str

# Agent 1 — analyzer

def analyze_agent(state: TextState):
    text = state["text"]
    state["analysis"] = f"Analysis of: {text}"
    return state

# Agent 2 — summarizer

def summary_agent(state: TextState):
    analysis = state["analysis"]
    state["summary"] = f"Summary: {analysis[:20]}..."
    return state

# Build graph

graph = StateGraph(TextState)

graph.add_node("analyze", analyze_agent)

graph.add_node("summarize", summary_agent)

graph.add_edge(START, "analyze")
graph.add_edge("analyze", "summarize")
graph.add_edge("summarize", END)

app = graph.compile()

result = app.invoke({"text": "LangGraph enables multi agent workflows"})
print(result)
```

---

## How It Works

1. Input goes to analyze agent
2. Analysis stored in state
3. Summarizer reads analysis
4. Final result returned

---

# Example 2 — Supervisor Agent (Router Pattern)

## Goal

* Supervisor decides which agent to call
* Math agent or Writing agent

---

## Code

```
from typing import Literal

class TaskState(TypedDict):
    task: str
    output: str

# Math agent

def math_agent(state: TaskState):
    state["output"] = "Math result"
    return state

# Writing agent

def writing_agent(state: TaskState):
    state["output"] = "Writing result"
    return state

# Supervisor decides route

def supervisor(state: TaskState) -> Literal["math", "writing"]:
    if "calculate" in state["task"]:
        return "math"
    return "writing"


graph = StateGraph(TaskState)

graph.add_node("math", math_agent)
graph.add_node("writing", writing_agent)

graph.add_conditional_edges(START, supervisor)

graph.add_edge("math", END)
graph.add_edge("writing", END)

app = graph.compile()
```

---

## What Happens

* Supervisor reads task
* Routes to correct agent
* Only one agent runs

This is very common in real systems.

---

# Example 3 — Collaborative Agents (Shared State)

## Goal

Multiple agents improve same output.

---

## Code

```
class EssayState(TypedDict):
    essay: str

# Generator

def writer(state: EssayState):
    state["essay"] = "AI is transforming industries."
    return state

# Improver

def editor(state: EssayState):
    state["essay"] += " It increases productivity."
    return state


graph = StateGraph(EssayState)

graph.add_node("writer", writer)
graph.add_node("editor", editor)

graph.add_edge(START, "writer")
graph.add_edge("writer", "editor")
graph.add_edge("editor", END)

app = graph.compile()
```

---

## Key Idea

Both agents modify same state.

---

# Example 4 — Tool Specialist Agents

## Goal

Different agents handle different tools.

---

## Code

```
class ToolState(TypedDict):
    query: str
    result: str

# Search specialist

def search_agent(state: ToolState):
    state["result"] = "Search results"
    return state

# Database specialist

def db_agent(state: ToolState):
    state["result"] = "Database results"
    return state
```

Each agent specializes in one capability.

---

# When to Use Multi-Agent Systems

Use when:

* Tasks are complex
* Need specialization
* Workflow has multiple steps
* Need verification or checking
* Building production AI systems

---

# Advantages vs Single Agent

| Single Agent        | Multi-Agent           |
| ------------------- | --------------------- |
| One large prompt    | Multiple small agents |
| Hard to debug       | Easy to debug         |
| Low control         | High control          |
| Limited scalability | Highly scalable       |

---

# Best Practices

* Keep agents specialized
* Use clear state structure
* Avoid large shared state
* Use supervisor for routing
* Log agent outputs
* Test each agent independently

---

# Summary

Multi-Agent Systems in LangGraph:

* Split intelligence across agents
* Use shared state
* Improve scalability
* Enable complex reasoning
* Provide production-level architecture

LangGraph makes building multi-agent workflows simple using graphs, nodes, and state.

---

# End of Document
