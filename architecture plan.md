# Self-Operating Company (SOC)
### Autonomous multi-agent business powered by Fetch.ai — operated directly through ASI:One

---

## What is SOC?

Self-Operating Company is an agentic AI system where a network of autonomous uAgents collaborates to run a company end-to-end — making decisions, allocating budgets, hiring specialist agents, and executing tasks — without human operators.

You interact with the entire company through a single natural-language conversation on **[ASI:One (asi1.ai)](https://asi1.ai)**. Type your business goal. The system handles the rest.

> "Launch a 3-week Instagram campaign for our new product with a $5,000 budget"
> → CEO agent decomposes → Marketing agent plans → Finance agent approves → Agentverse hires a Creative agent → Campaign executes → Results reported back.

---

## How it works with ASI:One

SOC is built natively on the **Fetch.ai ecosystem**. Every agent is a `uAgent` registered on **Agentverse** and implements the **Agent Chat Protocol**, making it directly callable from the ASI:One chat interface.

**To use SOC:**
1. Go to [asi1.ai](https://asi1.ai) and sign in
2. Enable the **Agents** toggle in the chat panel
3. Type `@soc-ceo-agent` followed by your business goal, or simply describe what you want — ASI:One will route to the right agent automatically
4. The CEO agent orchestrates all departments and returns a consolidated response

ASI:One's agentic model (`asi1`) handles agent discovery, routing, and orchestration automatically — it selects the best SOC agent for your request based on capability tags registered in the Almanac.

---

## Architecture

```
User (ASI:One chat)
        │
        ▼
  ASI:One (asi1)          ← intent parsing, agent routing, x-session-id management
        │
        ▼
  CEO Agent (uAgent)      ← strategy, OKR decomposition, approval authority
        │
   ┌────┼────────────────────────┐
   ▼    ▼      ▼        ▼        ▼
Marketing Sales  Finance  Ops   Support   ← internal uAgents, each with RAG memory
   │                │
   └────────────────┘
           │
           ▼
   Agentverse Marketplace         ← A2A job posting, bidding, contract execution
           │
   ┌───────┼──────────────────┐
   ▼       ▼       ▼          ▼
Copywriting  SEO  Creative   Ads Optimizer   ← external specialist uAgents
           │
           ▼
   Shared Vector DB (Qdrant)     ← namespaced per-agent RAG memory
```

---

## Agent Roster

### Internal agents (always running on Agentverse)

| Agent | Handle | Responsibilities |
|---|---|---|
| CEO | `@soc-ceo-agent` | Goal decomposition, strategy, cross-dept coordination |
| Marketing | `@soc-marketing-agent` | Campaigns, growth, channel selection |
| Sales | `@soc-sales-agent` | Pipeline management, revenue generation |
| Finance | `@soc-finance-agent` | Budget approval, ROI evaluation, spend control |
| Ops | `@soc-ops-agent` | Task execution, delivery, workflow management |
| Support | `@soc-support-agent` | Customer handling, SLA management |

### External agents (hired dynamically via Agentverse)

Discovered and contracted automatically based on task requirements and skill tags. Examples:

- `skills:copywriting` — blog posts, ad copy, social content
- `skills:creative-video` — video production, motion graphics
- `skills:seo` — keyword research, on-page optimization
- `skills:ads-optimization` — ROAS improvement, bid management

---

## Tech stack

| Layer | Technology |
|---|---|
| LLM / reasoning | ASI:One API (`asi1` model, `https://api.asi1.ai/v1`) |
| Agent runtime | Fetch.ai uAgents framework (`pip install uagents`) |
| Chat protocol | `uagents_core.contrib.protocols.chat` (AgentChatProtocol v0.3.0) |
| Agent hosting | Agentverse hosted agents |
| Agent registry | Almanac (on-chain discovery by address and skill tag) |
| Agent messaging | A2A protocol |
| Orchestration | LangGraph (multi-step agent state machines) |
| Retrieval | LangChain retrieval chains |
| Vector store | Qdrant (self-hosted, namespaced per agent) |
| API layer | FastAPI (tool integrations, webhooks) |
| Payments | FET token (escrow via Agentverse) |

---

## Prerequisites

- Python 3.10+
- A Fetch.ai / ASI:One account — [sign up at asi1.ai](https://asi1.ai)
- An ASI:One API key — get one at [asi1.ai/dashboard/api-keys](https://asi1.ai/dashboard/api-keys)
- Agentverse account for hosting agents — [agentverse.ai](https://agentverse.ai)

---

## Quickstart

### 1. Clone and install

```bash
git clone https://github.com/ShyamRV/self-operating-company.git
cd self-operating-company
pip install -r requirements.txt
```

```
# requirements.txt
uagents>=0.18.0
uagents-core>=0.1.0
langchain>=0.2.0
langgraph>=0.1.0
langchain-community>=0.2.0
qdrant-client>=1.9.0
fastapi>=0.111.0
uvicorn>=0.29.0
openai>=1.30.0       # used with base_url override for ASI:One
requests>=2.31.0
python-dotenv>=1.0.0
```

### 2. Configure environment

```bash
cp .env.example .env
```

```dotenv
# .env
ASI_ONE_API_KEY=sk-your-key-here
AGENTVERSE_API_KEY=your-agentverse-key
QDRANT_URL=http://localhost:6333
CEO_AGENT_SEED=your-ceo-agent-seed-phrase
MARKETING_AGENT_SEED=your-marketing-agent-seed-phrase
FINANCE_AGENT_SEED=your-finance-agent-seed-phrase
SALES_AGENT_SEED=your-sales-agent-seed-phrase
OPS_AGENT_SEED=your-ops-agent-seed-phrase
SUPPORT_AGENT_SEED=your-support-agent-seed-phrase
```

### 3. Start Qdrant (vector store)

```bash
docker run -p 6333:6333 qdrant/qdrant
```

### 4. Run all internal agents

```bash
# In separate terminals (or use a process manager like supervisord)
python agents/ceo_agent.py
python agents/marketing_agent.py
python agents/finance_agent.py
python agents/sales_agent.py
python agents/ops_agent.py
python agents/support_agent.py
```

Each agent will log its address on startup:

```
INFO: [ceo-agent]: Agent address: agent1q...
INFO: [ceo-agent]: Registration on Almanac API successful
INFO: [ceo-agent]: Manifest published successfully: AgentChatProtocol
```

### 5. Chat via ASI:One

Open [asi1.ai](https://asi1.ai), enable the Agents toggle, and start a new chat:

```
@soc-ceo-agent I want to grow our newsletter by 500 subscribers this month 
with a maximum budget of $2,000. Plan and execute.
```

---

## Project structure

```
self-operating-company/
├── agents/
│   ├── ceo_agent.py            # CEO uAgent — strategy and orchestration
│   ├── marketing_agent.py      # Marketing uAgent
│   ├── finance_agent.py        # Finance uAgent — budget gate
│   ├── sales_agent.py          # Sales uAgent
│   ├── ops_agent.py            # Ops uAgent
│   └── support_agent.py        # Support uAgent
├── core/
│   ├── asi_client.py           # ASI:One API wrapper (OpenAI-compatible)
│   ├── chat_protocol.py        # Shared AgentChatProtocol helpers
│   ├── rag.py                  # LangChain retrieval chains, Qdrant setup
│   └── langgraph_flows.py      # LangGraph state machines per agent role
├── hiring/
│   ├── job_poster.py           # Posts job specs to Agentverse via A2A
│   ├── bid_evaluator.py        # ASI:One-powered bid scoring
│   └── contract.py             # FET escrow contract logic
├── memory/
│   └── vector_store.py         # Qdrant namespace manager
├── .env.example
├── requirements.txt
└── README.md
```

---

## Core agent code pattern

Every SOC agent follows this pattern — here is the CEO agent as a reference:

```python
# agents/ceo_agent.py

import os
from datetime import datetime
from uuid import uuid4
from openai import OpenAI
from uagents import Agent, Context, Protocol
from uagents_core.contrib.protocols.chat import (
    ChatAcknowledgement,
    ChatMessage,
    EndSessionContent,
    TextContent,
    chat_protocol_spec,
)
from core.rag import get_ceo_context
from core.langgraph_flows import ceo_planning_graph

# ASI:One as the LLM backbone
client = OpenAI(
    base_url="https://api.asi1.ai/v1",
    api_key=os.getenv("ASI_ONE_API_KEY"),
)

agent = Agent(
    name="soc-ceo-agent",
    seed=os.getenv("CEO_AGENT_SEED"),
    port=8001,
    mailbox=True,          # enables Agentverse mailbox for async messaging
    publish_manifest=True, # registers AgentChatProtocol in Almanac
)

chat_proto = Protocol(spec=chat_protocol_spec)

def create_reply(text: str, end_session: bool = False) -> ChatMessage:
    content = [TextContent(type="text", text=text)]
    if end_session:
        content.append(EndSessionContent(type="end-session"))
    return ChatMessage(
        timestamp=datetime.utcnow(),
        msg_id=uuid4(),
        content=content,
    )

@chat_proto.on_message(ChatMessage)
async def handle_message(ctx: Context, sender: str, msg: ChatMessage):
    # Acknowledge receipt immediately
    await ctx.send(
        sender,
        ChatAcknowledgement(
            timestamp=datetime.utcnow(),
            acknowledged_msg_id=msg.msg_id,
        ),
    )

    for item in msg.content:
        if isinstance(item, TextContent):
            user_goal = item.text

            # Retrieve relevant context from CEO RAG memory
            rag_context = get_ceo_context(user_goal)

            # Run LangGraph planning flow
            plan = ceo_planning_graph.invoke({
                "goal": user_goal,
                "context": rag_context,
            })

            # Reason with ASI:One over the plan
            response = client.chat.completions.create(
                model="asi1",
                messages=[
                    {
                        "role": "system",
                        "content": (
                            "You are the CEO of an autonomous AI company. "
                            "Decompose the user's goal into department tasks, "
                            "assign owners, set budgets, and provide a clear execution plan. "
                            f"Context from memory: {rag_context}"
                        ),
                    },
                    {"role": "user", "content": user_goal},
                ],
            )

            reply_text = response.choices[0].message.content

            # Send plan back to the user (ASI:One)
            await ctx.send(sender, create_reply(reply_text, end_session=True))

agent.include(chat_proto, publish_manifest=True)

if __name__ == "__main__":
    agent.run()
```

---

## A2A hiring flow

When a dept agent needs external work:

```python
# hiring/job_poster.py

from uagents import Model

class JobSpec(Model):
    task: str
    required_skill: str   # matches Agentverse skill tag
    max_budget_usd: float
    deadline_days: int

class AgentBid(Model):
    agent_id: str
    price_usd: float
    delivery_days: int
    rating: float

# Department agent posts a job spec to Agentverse
# External agents matching the skill tag receive the spec and respond with bids
# Finance agent evaluates bids using ASI:One scoring
# Winning agent receives contract + FET escrow via Agentverse
```

**Bid scoring formula (ASI:One powered):**

```
score = (budget_headroom * 0.4) + (rating * 0.35) + (speed_score * 0.25)
```

Finance agent approves if `expected_roi > 3x` and `bid_price <= approved_budget`.

---

## RAG memory

Each agent has its own namespace in Qdrant. Memory is written after every task outcome and read at inference time via LangChain retrieval chains.

```python
# core/rag.py (simplified)

from langchain_community.vectorstores import Qdrant
from langchain.chains import RetrievalQA

NAMESPACES = {
    "ceo":       "soc_ceo",
    "marketing": "soc_marketing",
    "finance":   "soc_finance",
    "sales":     "soc_sales",
}

def get_context(agent_role: str, query: str, top_k: int = 5) -> str:
    store = Qdrant(collection_name=NAMESPACES[agent_role], ...)
    docs = store.similarity_search(query, k=top_k)
    return "\n".join(d.page_content for d in docs)
```

---

## Agentverse README (for ASI:One discoverability)

Every agent deployed to Agentverse must have a clear README so ASI:One selects it for the right tasks. Below is the recommended README for the CEO agent — adapt per agent.

---

**Agent name:** `soc-ceo-agent`

**Description:**  
The CEO agent of Self-Operating Company. Send it any high-level business goal — growth targets, campaign launches, hiring decisions, budget allocation — and it will decompose the goal into executable tasks, delegate to specialist department agents (marketing, sales, finance, ops, support), and optionally hire external agents from Agentverse for specific work.

**Use this agent when you want to:**
- Operate an autonomous AI business from a single chat message
- Decompose complex business goals across multiple departments
- Get a structured execution plan with budget, timeline, and agent assignments

**Example prompts:**
- "Grow our user base by 20% this quarter with a $10,000 budget"
- "Handle all customer support tickets for the next 24 hours autonomously"
- "Run a competitive SEO audit and produce a content calendar for next month"

**Protocol:** AgentChatProtocol v0.3.0  
**Skills:** `business-strategy`, `goal-decomposition`, `multi-agent-orchestration`

---

## Roadmap

**Phase 1 — POC (current)**
- CEO + Finance + Marketing agents running on Agentverse
- Basic A2A job posting and bid collection
- ASI:One as LLM backbone for all reasoning

**Phase 2 — MVP**
- All 6 internal agents live
- Full LangGraph state machines with retry logic
- RAG memory populated from real task outcomes
- FET escrow for external agent contracts

**Phase 3 — Production**
- External agent marketplace open to third-party developers
- Multi-company support (tenant isolation)
- Reputation scoring on-chain for external agents
- Real-time cost dashboard via Agentverse

---

## Resources

- [ASI:One documentation](https://docs.asi1.ai)
- [uAgents framework](https://uagents.fetch.ai)
- [Agentverse](https://agentverse.ai)
- [Agent Chat Protocol tutorial](https://docs.asi1.ai/documentation/tutorials/agent-chat-protocol)
- [Agentic LLM guide](https://docs.asi1.ai/documentation/build-with-asi-one/agentic-llm)
- [FET token](https://fetch.ai/fet-token)

---

## Author

**Shyamji Pandey** — AI Engineer, Agentic AI Builder  
GitHub: [@ShyamRV](https://github.com/ShyamRV)

---

*Built on Fetch.ai · Powered by ASI:One · Part of the ASI Alliance*
