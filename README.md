# Week 6 -> Day 1 -> AI Agents in n8n

---

## Table of Contents

1. [What Are AI Agents on n8n?](#1-what-are-ai-agents-on-n8n)
2. [AI Agents vs AI Automations](#2-ai-agents-vs-ai-automations)
3. [AI Agent Architecture](#3-ai-agent-architecture)
4. [Types of AI Agents in n8n](#4-types-of-ai-agents-in-n8n)
5. [Memory in n8n AI Agents](#5-memory-in-n8n-ai-agents)

---

## 1. What Are AI Agents on n8n?

### Definition

An AI Agent in n8n is an **autonomous, decision-making workflow component** that uses a Large Language Model (LLM) as its reasoning engine to dynamically determine which actions to take in response to user input. Unlike a standard LLM call that simply returns text, an AI Agent is given a set of **tools** — and it decides *which* tools to use, *in what order*, and *how to interpret* the results before responding.

n8n implements AI Agents through its **AI Agent node**, which is built on top of the [LangChain](https://js.langchain.com/) JavaScript framework. This gives you access to sophisticated agent behavior through a visual, drag-and-drop interface — no code required for most use cases.

### How It Works at a High Level

```
┌─────────────────────────────────────────────────────┐
│                   n8n WORKFLOW                      │
│                                                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────────┐   │
│  │  Chat    │──▶│ AI Agent │───▶│  Output /    │   │
│  │  Trigger │    │  Node    │    │  Response    │   │
│  └──────────┘    └────┬─────┘    └──────────────┘   │
│                       │                             │
│              ┌────────┼────────┐                    │
│              │        │        │                    │
│          ┌───▼──┐ ┌───▼──┐ ┌──▼───┐                 │
│          │ LLM  │ │Tools │ │Memory│                 │
│          │Model │ │      │ │      │                 │
│          └──────┘ └──────┘ └──────┘                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### The Four Pillars of an n8n AI Agent

| Pillar | n8n Component | Purpose |
|---|---|---|
| **Trigger** | Chat Trigger / Webhook | Entry point — receives user input |
| **Brain** | AI Agent Node + LLM | Reasoning, planning, and decision-making |
| **Tools** | Tool sub-nodes (HTTP Request, Code, Calculator, etc.) | Actions the agent can perform |
| **Memory** | Memory sub-nodes (Simple Memory, Postgres, Redis) | Conversational context retention |

### Example: A Simple Customer Support Agent

```
Chat Trigger  ──▶  AI Agent  ──▶  Response
                     │
            ┌────────┼────────┐
            ▼        ▼        ▼
        Google     Check     Create
        Gemini     Order     Support
        (LLM)     Status    Ticket
                  (Tool)    (Tool)
```

In this example, when a customer says *"Where is my order #4521?"*, the AI Agent:

1. **Receives** the message via the Chat Trigger
2. **Reasons** about which tool to use (selects "Check Order Status")
3. **Executes** the tool with the extracted order ID
4. **Formulates** a natural language response with the result
5. **Returns** the answer to the customer

The agent makes this decision autonomously — you don't write IF/ELSE logic for every possible question.

---

## 2. AI Agents vs AI Automations

This is one of the most critical distinctions to understand when building on n8n. Although both live inside the n8n workflow canvas, they operate on fundamentally different principles.

### Core Difference

| Dimension | AI Automation (Workflow) | AI Agent |
|---|---|---|
| **Decision Logic** | Fixed, predefined paths | Dynamic, LLM-driven decisions |
| **Flow Control** | Deterministic (IF/THEN/ELSE) | Probabilistic (model reasoning) |
| **Adaptability** | Handles only anticipated scenarios | Handles novel, unexpected inputs |
| **Tool Selection** | You define which tool runs when | Agent chooses which tool to call |
| **Iteration** | Single pass through the workflow | Multiple reasoning loops possible |
| **Complexity Ceiling** | Limited by how many branches you build | Limited by model capability + tools |
| **Predictability** | Very high — same input → same output | Lower — same input may vary slightly |
| **Cost per Run** | Usually lower (fewer API calls) | Higher (multiple LLM calls per run) |

### Visual Comparison

**AI Automation (Traditional Workflow):**
```
Trigger ──▶ IF condition ──▶ Action A
                 │
                 └──▶ Action B ──▶ Action C
```
Every path is hardcoded. The workflow author must anticipate all possibilities.

**AI Agent:**
```
Trigger ──▶ AI Agent ──◀──▶ Tool A
                 │◀──▶ Tool B
                 │◀──▶ Tool C
                 ▼
            Response
```
The agent dynamically selects and calls tools in a loop until it has enough information to respond.

### When to Use Each

#### Choose AI Automation When:
- The task is repetitive and well-defined (e.g., "when a form is submitted, add a row to Google Sheets")
- Predictability is critical (e.g., financial transactions, compliance workflows)
- You need to minimize API costs
- The input/output format is always the same

#### Choose AI Agents When:
- The input is natural language and unpredictable
- The task requires judgment or interpretation
- Multiple tools might be needed depending on context
- You want to handle follow-up questions or multi-turn conversations

### Hybrid Approach: The Best of Both Worlds

n8n's real power comes from **combining** deterministic automation with AI agents. For example:

```
Webhook Trigger
      │
      ▼
  Classify Email (AI)
      │
  ┌───┼───────────┐
  ▼   ▼           ▼
Urgent  Normal   Spam
  │      │         │
  ▼      ▼         ▼
AI     Auto-      Delete
Agent  respond    (Auto)
(full  (Template)
 reasoning)
```

Here, a quick AI classification step routes emails, but only truly complex cases go to the full AI Agent — saving cost and increasing reliability.

---

## 3. AI Agent Architecture

Every AI agent, whether theoretical or built in n8n, follows a three-layer architecture: **Perception → Brain → Action**. Understanding this architecture is essential for designing effective agents.

### 3.1 Perception Layer (Sensors / Input)

The Perception layer is how the agent receives information about its environment. In n8n, this is handled by **trigger nodes** and **input processing**.

#### Input Types in n8n

| Input Source | n8n Node | Data Type |
|---|---|---|
| User chat message | Chat Trigger | Natural language text |
| Incoming webhook | Webhook Node | JSON payload |
| Scheduled event | Cron / Schedule Trigger | Time-based activation |
| Email arrival | Email Trigger (IMAP) | Email content + attachments |
| Messaging platform | Telegram / Slack Trigger | Text, images, voice |
| Database change | Postgres / MongoDB Trigger | Structured data |
| File upload | Form Trigger | Documents, images |

#### Architecture Diagram — Perception Layer

```
┌──────────────────────── PERCEPTION LAYER ───────────────────────┐
│                                                                 │
│   External World                                                │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│   │ User     │  │ Webhook  │  │ Email    │  │ Telegram │        │
│   │ Chat     │  │ Request  │  │ Message  │  │ Message  │        │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
│        │             │             │             │              │
│        └─────────────┴──────┬──────┴─────────────┘              │
│                             ▼                                   │
│                    ┌─────────────────┐                          │
│                    │  Input Parser   │                          │
│                    │  (Normalize to  │                          │
│                    │   text/JSON)    │                          │
│                    └────────┬────────┘                          │
│                             ▼                                   │
│                    Structured Input                             │
│                    to Brain Layer                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Brain Layer (Reasoning + Planning)

The Brain is the **core of the AI Agent** — it's where reasoning, planning, and decision-making happen. In n8n, this is the **AI Agent node** combined with an **LLM model**.

#### The Reasoning Loop

The brain doesn't just run once. It operates in an **iterative reasoning loop** (often called the ReAct pattern — Reasoning + Acting):

```
┌─────────────────── BRAIN LAYER (ReAct Loop) ──────────────────┐
│                                                               │
│  ┌──────────┐                                                 │
│  │  Input   │                                                 │
│  │ (from    │                                                 │
│  │ Percept.)│                                                 │
│  └────┬─────┘                                                 │
│       ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              REASONING ENGINE (LLM)                     │  │
│  │                                                         │  │
│  │  Step 1: THINK                                          │  │
│  │  "The user wants to know their order status.            │  │
│  │   I should use the Order Lookup tool."                  │  │
│  │                                                         │  │
│  │  Step 2: ACT                                            │  │
│  │  → Call tool: order_lookup(order_id="4521")             │  │
│  │                                                         │  │
│  │  Step 3: OBSERVE                                        │  │
│  │  ← Tool returns: { status: "shipped", eta: "Apr 3" }    │  │
│  │                                                         │  │
│  │  Step 4: THINK (again)                                  │  │
│  │  "I have the information. I can respond now."           │  │
│  │                                                         │  │
│  │  Step 5: RESPOND                                        │  │
│  │  → "Your order #4521 has been shipped and will         │  │
│  │     arrive by April 3rd."                               │  │
│  │                                                         │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  Components:                                                  │
│  ┌───────────┐  ┌────────────┐  ┌────────────┐                │
│  │  System   │  │  Chat      │  │  Tool      │                │
│  │  Prompt   │  │  History   │  │  Descrip-  │                │
│  │  (persona │  │  (from     │  │  tions     │                │
│  │  + rules) │  │  memory)   │  │  (what's   │                │
│  │           │  │            │  │  available)│                │
│  └───────────┘  └────────────┘  └────────────┘                │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

#### Key Brain Components in n8n

1. **System Prompt** — Defines the agent's personality, instructions, rules, and constraints. Written in the AI Agent node's "System Message" field.

2. **LLM Model** — The actual language model powering reasoning. Supported models include:
   - OpenAI (GPT-4o, GPT-4, GPT-3.5)
   - Google Gemini
   - Anthropic Claude
   - Ollama (local/self-hosted models)
   - Azure OpenAI
   - Any OpenAI-compatible API

3. **Tool Descriptions** — The LLM reads these descriptions to decide which tool to call. Better descriptions lead to better tool selection.

### 3.3 Action Layer (Actuators / Tools)

The Action layer is where the agent **does things** in the real world. In n8n, actions are represented by **Tool sub-nodes** connected to the AI Agent.

#### Available Tool Types in n8n

| Tool Category | Examples | Use Case |
|---|---|---|
| **Search & Research** | SerpAPI, Wikipedia, Brave Search | Finding information online |
| **Data & Code** | Calculator, Code (JS/Python), SQL | Computing, data processing |
| **Communication** | Gmail, Slack, Telegram | Sending messages/emails |
| **Storage & CRM** | Google Sheets, Airtable, Notion, HubSpot | Reading/writing data |
| **HTTP / API** | HTTP Request Tool | Calling any external API |
| **Workflow** | Call n8n Workflow Tool | Invoking other n8n workflows as tools |
| **Vector Store** | Pinecone, Qdrant, Supabase | RAG / knowledge base queries |
| **MCP Servers** | Any MCP-compatible server | Connecting to external AI tool ecosystems |

#### Full Architecture — All Three Layers Combined

```
┌────────────────────────────────────────────────────────────────────┐
│                    n8n AI AGENT — FULL ARCHITECTURE                │
│                                                                    │
│  ┌──────────── PERCEPTION ────────────┐                            │
│  │  Chat Trigger / Webhook / Email    │                            │
│  │  Telegram / Slack / Schedule       │                            │
│  └──────────────┬─────────────────────┘                            │
│                 ▼                                                  │
│  ┌──────────── BRAIN ─────────────────────────────────────────┐    │
│  │                                                            │    │
│  │  ┌─────────────────────────────────────────────────────┐   │    │
│  │  │              AI AGENT NODE                          │   │    │
│  │  │                                                     │   │    │
│  │  │  System Prompt ──▶ LLM (GPT-4o / Gemini / Claude)  │   │    │
│  │  │                        │                            │   │    │
│  │  │                   ┌────┴────┐                       │   │    │
│  │  │                   │ ReAct   │                       │   │    │
│  │  │                   │ Loop    │◀── Memory (context)  │   │    │
│  │  │                   │ Think → │                       │   │    │
│  │  │                   │ Act →   │                       │   │    │
│  │  │                   │ Observe │                       │   │    │
│  │  │                   └────┬────┘                       │   │    │
│  │  └────────────────────────┼────────────────────────────┘   │    │
│  └───────────────────────────┼────────────────────────────────┘    │
│                              ▼                                     │
│  ┌──────────── ACTION ────────────────────────────────────────┐    │
│  │                                                            │    │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐    │    │
│  │  │ HTTP   │ │ Google │ │ Slack  │ │ Code   │ │ Vector │    │    │
│  │  │Request │ │ Sheets │ │  Post  │ │  Exec  │ │ Store  │    │    │
│  │  │ Tool   │ │ Tool   │ │  Tool  │ │  Tool  │ │ Query  │    │    │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘    │    │
│  │                                                            │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 4. Types of AI Agents in n8n

AI agent theory defines several classical agent types. Let's examine each one and see how they map to n8n's implementation.

### 4.1 Simple Reflex Agent

#### Concept
A simple reflex agent acts only on the **current input** — it has no memory and no model of the world. It follows condition-action rules: "IF [percept] THEN [action]."

```
        ┌──────────────┐
        │  Environment │
        └──────┬───────┘
               │ percept
               ▼
        ┌──────────────┐
        │  Condition-  │
        │  Action      │     No memory.
        │  Rules       │     No world model.
        │              │     Pure reaction.
        └──────┬───────┘
               │ action
               ▼
        ┌──────────────┐
        │  Environment │
        └──────────────┘
```

#### n8n Implementation
In n8n, this maps to a **basic LLM chain without memory or tools** — the AI simply receives a prompt and responds. Alternatively, an n8n workflow with IF/Switch nodes and no AI Agent node is the purest form of a simple reflex agent.

**Example — Auto-Reply Bot:**
```
Email Trigger ──▶ IF contains "invoice"
                       │
                  ┌────┼────┐
                  ▼         ▼
              Reply       Reply
           "Received"  "General
            + forward   acknowledgment"
            to finance
```

**When to use in n8n:** Simple routing, auto-tagging, basic classification where rules are clear-cut.

---

### 4.2 Model-Based Reflex Agent

#### Concept
This agent maintains an **internal state** — a model of the world that it updates based on observations. It can handle situations where the current input alone isn't enough to make a decision.

```
        ┌──────────────┐
        │  Environment │
        └──────┬───────┘
               │ percept
               ▼
        ┌──────────────┐
        │  Internal    │
        │  State       │◀── Updated after
        │  (World      │    each observation
        │   Model)     │
        └──────┬───────┘
               │ action based on
               │ state + percept
               ▼
        ┌──────────────┐
        │  Environment │
        └──────────────┘
```

#### n8n Implementation
In n8n, this is an **AI Agent with Memory**. The memory node gives the agent an internal state — it can reference earlier messages in the conversation to make better decisions.

**Example — Context-Aware Support Bot:**
```
Chat Trigger ──▶ AI Agent ──▶ Response
                    │
               ┌────┴────┐
               ▼         ▼
            OpenAI    Window Buffer
            GPT-4o    Memory
            (LLM)     (Internal State)
```

The agent remembers the user said "I'm having trouble with billing" three messages ago, so when they now say "Can you check?", it knows to look up billing info — not shipping info.

**When to use in n8n:** Multi-turn chatbots, support agents, any scenario where the conversation history matters.

---

### 4.3 Goal-Based Agent

#### Concept
A goal-based agent doesn't just react — it has an explicit **goal** and plans a sequence of actions to achieve it. It considers future consequences of its actions.

```
        ┌──────────────┐
        │  Environment │
        └──────┬───────┘
               │ percept
               ▼
        ┌──────────────┐      ┌──────────┐
        │  Planning    │◀───▶│  GOAL    │
        │  Engine      │      │          │
        │  (What       │      │ "Resolve │
        │   sequence   │      │  the     │
        │   of actions │      │  ticket" │
        │   achieves   │      │          │
        │   the goal?) │      └──────────┘
        └──────┬───────┘
               │ planned action sequence
               ▼
        ┌──────────────┐
        │  Environment │
        └──────────────┘
```

#### n8n Implementation
In n8n, this is an **AI Agent with multiple tools and a directive system prompt** that defines a clear objective. The agent plans which tools to call and in what order.

**Example — Research Agent:**
```
Chat Trigger ──▶ AI Agent ──▶ Summary Report
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       GPT-4o   SerpAPI   Wikipedia
       (LLM)   (Search)   (Lookup)
                    │
                    ▼
              Window Buffer
              Memory
```

**System Prompt (Goal definition):**
> "You are a research agent. Your goal is to produce a comprehensive, fact-checked summary on any topic the user asks about. Always search at least 2 sources before responding. Cross-reference findings. Cite your sources."

The agent will autonomously call SerpAPI, then Wikipedia, compare results, and synthesize a coherent report.

**When to use in n8n:** Research bots, data gathering agents, task completion workflows where the outcome is clearly defined.

---

### 4.4 Utility-Based Agent

#### Concept
A utility-based agent goes beyond goals — it assigns a **utility score** (a measure of "goodness") to different possible outcomes and picks the one that maximizes expected value. This is critical when there are trade-offs or conflicting objectives.

```
        ┌──────────────┐
        │  Environment │
        └──────┬───────┘
               │ percept
               ▼
        ┌──────────────┐      ┌──────────────┐
        │  Utility     │◀───▶│  Utility     │
        │  Calculator  │      │  Function    │
        │              │      │              │
        │  "Which      │      │  Score =     │
        │   action     │      │  f(speed,    │
        │   produces   │      │    cost,     │
        │   the best   │      │    quality)  │
        │   outcome?"  │      │              │
        └──────┬───────┘      └──────────────┘
               │ optimal action
               ▼
        ┌──────────────┐
        │  Environment │
        └──────────────┘
```

#### n8n Implementation
In n8n, utility-based behavior is achieved through **advanced system prompts** that instruct the agent to weigh trade-offs, plus **conditional logic** in the workflow itself. You can also implement an **LLM Routing Agent** pattern — where a primary agent decides which specialized sub-agent to delegate to based on the nature of the request.

**Example — Multi-Model Router Agent:**
```
Chat Trigger ──▶ Router Agent (GPT-4o-mini)
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
          Simple     Complex    Creative
          Query      Analysis   Writing
              │         │         │
              ▼         ▼         ▼
          GPT-3.5   GPT-4o    Claude
          (cheap)   (smart)   (creative)
```

The router agent evaluates: "Is this query simple enough for a cheap model, or does it need a powerful one?" It optimizes for the **utility function** of cost vs. quality.

**When to use in n8n:** Cost optimization, multi-model routing, workflows with competing priorities (speed vs. accuracy vs. cost).

---

### 4.5 Learning Agent

#### Concept
A learning agent improves its performance over time based on feedback. It has four conceptual components:

```
┌─────────────────────────────────────────────────────────┐
│                    LEARNING AGENT                       │
│                                                         │
│   ┌────────────┐         ┌────────────┐                 │
│   │  Critic    │◀──────▶│  Learning  │                 │
│   │  (feedback │         │  Element   │                 │
│   │  evaluator)│         │  (improves │                 │
│   └────────────┘         │  behavior) │                 │
│                          └─────┬──────┘                 │
│                               │                         │
│                               ▼                         │
│   ┌────────────┐         ┌────────────┐                 │
│   │  Problem   │◀──────▶│ Performance│                 │
│   │  Generator │         │  Element   │                 │
│   │  (explores │         │  (selects  │                 │
│   │  new       │         │  actions)  │                 │
│   │  scenarios)│         │            │                 │
│   └────────────┘         └────────────┘                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### n8n Implementation
True learning agents require mechanisms beyond a single workflow execution. In n8n, you can build learning behavior through:

1. **RAG (Retrieval-Augmented Generation):** The agent queries a vector store of past interactions, solutions, and feedback to inform current decisions.

2. **Feedback loops:** User ratings or human-in-the-loop approvals are stored and used to refine future prompts.

3. **n8n Evaluations:** n8n provides built-in evaluation tools to monitor agent performance, catch regressions, and iterate on prompts based on real data.

**Example — Self-Improving Support Agent:**
```
Chat Trigger ──▶ AI Agent ──▶ Response ──▶ User Rating
                    │                          │
               ┌────┴────┐                     │
               ▼         ▼                     ▼
            GPT-4o    Vector Store      Store feedback
            (LLM)    (Past solutions    in Vector Store
                      + feedback)       for next time
```

Over time, the agent retrieves similar past queries and their outcomes, learning from what worked and what didn't.

**When to use in n8n:** Long-term knowledge bases, evolving support systems, agents that need to get better over time.

---

### Comparison Matrix — All Agent Types in n8n

| Agent Type | Memory | Planning | Learning | n8n Complexity | Best For |
|---|---|---|---|---|---|
| Simple Reflex | None | None | None | Low | Auto-replies, routing |
| Model-Based | Short-term | None | None | Medium | Multi-turn chat |
| Goal-Based | Short-term | Yes | None | Medium-High | Research, task completion |
| Utility-Based | Short-term | Yes | None | High | Cost optimization, routing |
| Learning | Long-term | Yes | Yes | Very High | Evolving knowledge bases |

---

## 5. Memory in n8n AI Agents

Memory is what transforms a stateless LLM call into a contextual, conversational agent. Without memory, every message is treated as a brand-new conversation. n8n provides several memory options spanning **short-term**, **long-term**, and **external knowledge** categories.

### 5.1 Memory Architecture Overview

```
┌────────────────────── MEMORY IN n8n AI AGENTS ──────────────────────┐
│                                                                     │
│  ┌────────────────── SHORT-TERM MEMORY ──────────────────┐          │
│  │                                                       │          │
│  │  Simple Memory              Window Buffer Memory      │          │
│  │  (In-process RAM)           (Last N messages)         │          │
│  │                                                       │          │
│  │  • Per-execution only       • Sliding window          │          │
│  │  • Lost on restart          • Configurable size       │          │
│  │  • Great for dev/testing    • Balances context & cost │          │
│  │                                                       │          │
│  └───────────────────────────────────────────────────────┘          │
│                                                                     │
│  ┌────────────────── LONG-TERM MEMORY ───────────────────┐          │
│  │                                                       │          │
│  │  Postgres Chat Memory       Redis Chat Memory         │          │
│  │  (Database-backed)          (Cache-backed)            │          │
│  │                                                       │          │
│  │  • Persists across          • Ultra-fast read/write   │          │
│  │    restarts                 • Great for real-time     │          │
│  │  • Per-user sessions        • Also persists across    │          │
│  │  • Production-ready           restarts                │          │
│  │                                                       │          │
│  │  MongoDB Chat Memory        Motorhead Memory          │          │
│  │  (Document store)           (Managed service)         │          │
│  │                                                       │          │
│  │  • Flexible schema          • Cloud-hosted            │          │
│  │  • Complex data             • Built-in management     │          │
│  │                                                       │          │
│  └───────────────────────────────────────────────────────┘          │
│                                                                     │
│  ┌────────────────── EXTERNAL KNOWLEDGE (RAG) ───────────┐          │
│  │                                                       │          │
│  │  Vector Store + Embeddings                            │          │
│  │  (Pinecone, Qdrant, Supabase, In-Memory)              │          │
│  │                                                       │          │
│  │  • Searchable knowledge base                          │          │
│  │  • Documents, FAQs, past conversations                │          │
│  │  • Not "memory" per se, but extends agent knowledge   │          │
│  │                                                       │          │
│  └───────────────────────────────────────────────────────┘          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2 Short-Term Memory

Short-term memory retains context **within a single conversation session** but does not survive server restarts or deployments.

#### Simple Memory (formerly Window Buffer Memory)

This is n8n's built-in, zero-configuration memory option. It stores recent messages in the process's RAM.

**Configuration in n8n:**
```
AI Agent Node
    └── Memory (sub-node connector)
            └── Simple Memory
                    • Context Window Length: 10 (messages)
                    • Session ID: {{ $json.sessionId }}
```

**Key Properties:**

| Property | Value |
|---|---|
| Persistence | In-process RAM only |
| Survives restart? | No |
| Session isolation | Via Session ID field |
| Default window size | Last 5 message pairs |
| Best for | Development, testing, simple bots |

**Important:** The Simple Memory node was previously called "Window Buffer Memory." If you're working with older templates, you may need to remove and re-add this node to use the latest version.

#### How the Context Window Works

```
Message History (Window Size = 3):

  Turn 1: User: "Hi, I need help with billing"        ← Dropped
  Turn 2: Agent: "Sure! What's your account?"         ← Dropped
  Turn 3: User: "Account #12345"                      ← Kept ✓
  Turn 4: Agent: "I see your account. What's wrong?"  ← Kept ✓
  Turn 5: User: "I was double-charged"                ← Kept ✓
  ─────────────────────────────────────────────
  Current: User: "Can you refund it?"                  ← Current ✓

  → Agent sees turns 3, 4, 5 + current.
  → Turns 1 and 2 are outside the window and forgotten.
```

### 5.3 Long-Term Memory

Long-term memory **persists across n8n restarts** by storing conversation history in an external database. This is essential for production deployments.

#### Postgres Chat Memory

The most widely recommended option for production n8n agents.

**Setup Steps:**

1. Set up a PostgreSQL database (local or cloud-hosted)
2. Add Postgres credentials in n8n (Settings → Credentials)
3. Replace Simple Memory with a Postgres Chat Memory node
4. Configure the Session ID to uniquely identify each user/conversation

**Configuration:**
```
AI Agent Node
    └── Memory (sub-node connector)
            └── Postgres Chat Memory
                    • Connection: [Your Postgres Credentials]
                    • Session ID: {{ $json.userId }}_{{ $json.threadId }}
                    • Table Name: chat_history (auto-created)
                    • Context Window Length: 20
```

#### Redis Chat Memory

Ideal for high-throughput, real-time applications where speed is critical.

**When to choose Redis over Postgres:**
- Sub-millisecond read latency is required
- You're already running Redis in your stack
- The agent handles a very high volume of concurrent sessions

**Configuration:**
```
AI Agent Node
    └── Memory (sub-node connector)
            └── Redis Chat Memory
                    • Connection: [Your Redis Credentials]
                    • Session ID: {{ $json.sessionId }}
                    • Session TTL: 3600 (seconds — auto-expire old sessions)
                    • Context Window Length: 15
```

#### Memory Type Comparison

| Feature | Simple Memory | Postgres | Redis | MongoDB |
|---|---|---|---|---|
| Setup complexity | None | Medium | Medium | Medium |
| Persistence | In-RAM only | Full | Full | Full |
| Read speed | Instant | Fast | Very fast | Fast |
| Scalability | Single process | High | Very high | High |
| Session management | Basic | Full | Full + TTL | Full |
| Cost | Free | DB hosting | DB hosting | DB hosting |
| Best for | Dev/testing | Most production | Real-time apps | Complex data |

### 5.4 External Knowledge (RAG — Retrieval-Augmented Generation)

RAG is not "memory" in the traditional sense — it doesn't store conversation history. Instead, it gives the agent access to a **searchable knowledge base** of documents, FAQs, manuals, or any other reference material.

**How RAG Works in n8n:**

```
User Question: "What's your refund policy?"
         │
         ▼
    AI Agent Node
         │
    ┌────┴────────────────┐
    ▼                     ▼
  LLM                 Vector Store
  (Reasoning)         Tool (RAG)
                          │
                     ┌────┴────┐
                     ▼         ▼
                  Embed     Search
                  Query     Similar
                            Documents
                          │
                          ▼
                  Top 3 relevant
                  document chunks
                          │
                          ▼
              Inject into LLM context
                          │
                          ▼
              Agent responds with
              accurate, sourced answer
```

**n8n RAG Setup:**
1. **Ingest documents:** Use a separate workflow to split documents into chunks, generate embeddings, and store them in a vector database.
2. **Add Vector Store Tool:** Connect a Vector Store tool (Pinecone, Qdrant, Supabase, or In-Memory) to the AI Agent.
3. **Agent queries automatically:** When the user asks a question, the agent decides whether to search the knowledge base and injects relevant context into its reasoning.

### 5.5 Session Management Best Practices

Proper session management is critical for multi-user agents. Here are the key principles:

**1. Always use dynamic Session IDs:**
```
✅  Session ID: {{ $json.userId }}_{{ $now.toFormat('yyyy-MM-dd') }}
❌  Session ID: "default"   ← All users share memory!
```

**2. Implement session cleanup:**
- Set TTLs (time-to-live) on Redis sessions
- Run periodic cleanup workflows for Postgres
- Prevent unbounded memory growth

**3. Choose appropriate context window sizes:**

| Use Case | Recommended Window |
|---|---|
| Quick Q&A bot | 3–5 messages |
| Support conversations | 10–15 messages |
| Complex research tasks | 20–30 messages |
| Long-running analysis | 30+ (watch for token limits) |

**4. Migration path:**
```
Development:   Simple Memory    (zero config)
        ↓
Staging:       Postgres Memory  (test persistence)
        ↓
Production:    Postgres/Redis   (full persistence + monitoring)
```

### 5.6 Memory Architecture Decision Tree

```
Start
  │
  ├── Is this a prototype or dev environment?
  │     └── YES → Use Simple Memory
  │
  ├── Does the agent need to remember across restarts?
  │     └── YES → Use Postgres or Redis Chat Memory
  │           │
  │           ├── Is ultra-low latency critical?
  │           │     └── YES → Redis
  │           │     └── NO  → Postgres (simpler, more reliable)
  │           │
  │           └── Do you need complex data structures?
  │                 └── YES → MongoDB
  │
  └── Does the agent need knowledge beyond chat history?
        └── YES → Add RAG (Vector Store Tool)
              │
              └── This works ALONGSIDE any memory type above
```

---

## Quick Reference Card

```
┌──────────────────────────────────────────────────────────┐
│              n8n AI AGENT — QUICK REFERENCE              │
│                                                          │
│  AGENT = Trigger + Brain + Tools + Memory                │
│                                                          │
│  TRIGGER:  Chat Trigger, Webhook, Email, Cron            │
│  BRAIN:    AI Agent Node + LLM (GPT-4o, Gemini, etc.)    │
│  TOOLS:    HTTP Request, Code, Search, Sheets, Slack...  │
│  MEMORY:   Simple | Postgres | Redis | MongoDB           │
│                                                          │
│  AGENT vs AUTOMATION:                                    │
│  • Automation = fixed paths, deterministic               │
│  • Agent = dynamic decisions, LLM-driven                 │
│  • Best approach = hybrid (both together)                │
│                                                          │
│  ARCHITECTURE:                                           │
│  • Perception → Brain (ReAct loop) → Action              │
│                                                          │
│  MEMORY RULE OF THUMB:                                   │
│  • Dev → Simple Memory                                   │
│  • Prod → Postgres (most cases) or Redis (real-time)     │
│  • Knowledge → Add RAG with Vector Store                 │
│                                                          │
│  KEY VERSION NOTE:                                       │
│  Since n8n v1.82.0, all AI Agent nodes work as           │
│  "Tools Agent" (the recommended type). Legacy agent      │
│  types have been consolidated.                           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

