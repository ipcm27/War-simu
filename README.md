# WarSim — Geopolitical Multi-Agent Conflict Simulator

> A multi-agent system that simulates geopolitical conflict scenarios using Game Theory, LLMs, and graph-based state management.

---

## 🧠 Concept

WarSim models geopolitical actors as autonomous agents that **reason, decide, and react** to one another — not through random probability, but through LLM-powered Game Theory reasoning.

Each agent:
- Has explicit **objectives**, **resources**, and **red lines**
- Uses a **ReAct loop** (Reason → Act → Observe → Reason again)
- Holds a **payoff matrix** to calculate utility before acting
- Remembers previous turns and adapts behavior accordingly

---

## 🌍 Fictional Scenario (default)

| Country | Analogy | Role |
|---|---|---|
| **Noxus** | Western superpower | Aggressor / initiator |
| **Shurima** | Middle Eastern theocracy | Target nation |
| **Zaum** | Eastern industrial giant | Strategic ally of Shurima |


---

## 🏗️ Architecture

```
┌─────────────────────────────────────────┐
│              LangGraph Orchestrator     │
│                                         │
│  ┌──────────┐  ┌──────────┐  ┌───────┐  │
│  │ Shurima  │  │  Noxus   │  │ Zaum  │  |
│  │  Agent   │  │  Agent   │  │ Agent │  │
│  └────┬─────┘  └────┬─────┘  └───┬───┘  │
│       └─────────────┴────────────┘      │
│                    │                    │
│           ┌────────▼────────┐           │
│           │  Shared World   │           │
│           │     State       │           │
│           └────────┬────────┘           │
└────────────────────┼────────────────────┘
                     │
             ┌───────▼───────┐
             │     Neo4j     │
             │  Graph Store  │
             └───────────────┘
```

---

## 🎮 Game Theory Implementation

Each agent has an explicit **utility function**:

```python
utility_noxus = (
    oil_access      * 0.40 +
    trade_with_Noxus * 0.30 +
    regional_influence  * 0.30
)
```

Before acting, the agent reasons about what the **other agents will do** — not just what they did:

```
Thought: "If I retaliate now, will Noxus escalate or back down?
          If I stay silent, will they read it as weakness?"

Action: [cut_semiconductor_exports | diplomatic_statement | proxy_support | ...]

Observation: "Noxus market dropped 8%. Internal pressure increased."

Thought: "Partially effective. Do I increase pressure or hold?"
```

The system converges toward a **Nash Equilibrium** when agents stop changing their strategies — even if that outcome is suboptimal for everyone.

---

## 🔁 ReAct Agent Loop

```python
class GeopoliticalAgent:
    def __init__(self, name, objectives, resources, red_lines, utility_fn):
        self.name = name
        self.objectives = objectives
        self.resources = resources
        self.red_lines = red_lines
        self.utility_fn = utility_fn
        self.memory = []  # turn history

    def decide(self, world_state: dict, available_tools: list) -> AgentAction:
        prompt = f"""
        You are {self.name}.

        Your objectives: {self.objectives}
        Your resources: {self.resources}
        Your red lines (never cross): {self.red_lines}

        Current world state:
        {world_state}

        Your action history:
        {self.memory}

        Available actions:
        {available_tools}

        Reason step by step. What do the other actors likely do next?
        What is your utility for each available action?
        Choose your action and justify it.

        Return JSON:
        {{
          "reasoning": "...",
          "action": "...",
          "target": "...",
          "expected_utility": 0.0,
          "confidence": 0.0
        }}
        """
        response = llm.invoke(prompt)
        self.memory.append(response)
        return response
```

---

## 🗂️ Neo4j Graph Model

World state is persisted as a property graph:

```cypher
// Relationships
(Noxus)-[:ATTACKED {type: "airstrike", turn: 1}]->(Shurima)
(Zaum)-[:DEPENDS_ON {resource: "oil", volume: 0.6}]->(Shurima)
(Zaum)-[:THREATENED {instrument: "economic"}]->(Noxus)
(Shurima)-[:REQUESTED_SUPPORT]->(noxus)

// Query: who becomes vulnerable if Shurima falls?
MATCH (n)-[:DEPENDS_ON]->(k:Country {name: "Shurima"})
RETURN n.name, n.dependency_score
```

This enables queries like:
- *Which actors are most exposed if Shurima's regime collapses?*
- *What is the shortest path of influence between noxus and Noxus?*
- *Which red lines have been crossed this simulation?*

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Agent orchestration | [LangGraph](https://github.com/langchain-ai/langgraph) |
| LLM backbone | Claude (Anthropic API) |
| Graph state store | [Neo4j](https://neo4j.com/) |
| Embeddings / RAG (historical data) | ChromaDB + sentence-transformers |
| Language | Python 3.11+ |

---

## 🗺️ Roadmap

- [ ] **Phase 1** — Text agents with ReAct loop (no tools, pure reasoning)
- [ ] **Phase 2** — Explicit payoff matrix + utility function per agent
- [ ] **Phase 3** — Neo4j integration for world state persistence
- [ ] **Phase 4** — LangGraph orchestration with conditional edges
- [ ] **Phase 5** — Monte Carlo Tree Search for lookahead decision making
- [ ] **Phase 6** — Historical RAG layer (SIPRI data, RAND reports)

---

## 📁 Project Structure (planned)

```
warsim/
├── agents/
│   ├── base_agent.py        # ReAct loop, memory, utility fn
│   ├── Noxus.py
│   ├── Shurima.py
│   └── noxus.py
├── graph/
│   ├── world_state.py       # LangGraph state definition
│   ├── orchestrator.py      # Graph edges and flow
│   └── neo4j_store.py       # Neo4j persistence layer
├── tools/
│   ├── economic.py          # cut_exports, sanctions, etc.
│   ├── diplomatic.py        # statements, alliances, etc.
│   └── military.py          # troop movements, proxies, etc.
├── rag/
│   ├── ingest.py            # chunk + embed historical docs
│   └── retriever.py         # query ChromaDB
├── simulation.py            # main entry point
└── README.md
```

---

## 🚀 Getting Started

```bash
git clone https://github.com/your-username/warsim
cd warsim
pip install -r requirements.txt

# Set your API key
export ANTHROPIC_API_KEY=your_key_here

# Run a simulation
python simulation.py --scenario "Noxus_attacks_Shurima" --turns 10
```

---

## 📚 References & Inspiration

- [RAND Corporation Wargaming](https://www.rand.org/topics/wargaming.html)
- [Nash Equilibrium — Stanford Encyclopedia](https://plato.stanford.edu/entries/game-theory/)
- [ReAct: Synergizing Reasoning and Acting in LLMs](https://arxiv.org/abs/2210.03629)
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [Monte Carlo Tree Search explained](https://en.wikipedia.org/wiki/Monte_Carlo_tree_search)

---

*Built for learning. All scenarios are entirely fictional.*
