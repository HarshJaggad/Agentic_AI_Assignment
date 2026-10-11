# 🤖 Multi-Agent Research & Analysis Workflow (n8n + Ollama + Tavily)

A **sequential two-agent AI workflow** built in [n8n](https://n8n.io). A user asks a question in chat. **Agent 1** researches it on the web, and **Agent 2** turns that research into a clear, structured answer with practical insights.

> 💡 Example question: *"Should a small retail business adopt Generative AI? Analyze its potential benefits, costs, risks, and practical implementation considerations."*

---

## ✨ Features

- 🔗 **Sequential multi-agent pipeline**: each agent has one focused job and hands its output to the next.
- 🌐 **Live web research**: Agent 1 uses the Tavily search tool to gather current facts, examples and sources.
- 🧠 **Insight and decision support**: Agent 2 interprets the research instead of just repeating it.
- 🏠 **Local LLM**: runs on an Ollama chat model with no paid LLM API needed.
- 📋 **Structured output**: answers come back as Direct Answer, Key Findings, Analysis, Recommendations and more.
- 🛡️ **Honest about gaps**: both agents are told not to invent facts, statistics or sources.

---

## 🏗️ Architecture

```
💬 Chat Trigger
      │
      ▼
🔎 Agent 1 – Research ──── 🌐 Tavily Search (tool)
      │                └── 🦙 Ollama Chat Model
      ▼
🧩 Agent 2 – Analysis ──── 🦙 Ollama Chat Model
      │
      ▼
✅ Final answer returned in chat
```

| Step | Node | Role |
|---|---|---|
| 1️⃣ | **When chat message received** | Receives the user's question |
| 2️⃣ | **Agent 1: Research** | Breaks the topic into sub-questions, searches with Tavily and compiles organized findings |
| 3️⃣ | **Agent 2: Analysis** | Reads the findings, interprets them and writes the final response |
| 🔧 | **Ollama Chat Model** | The language model shared by both agents |
| 🔧 | **Search in Tavily** | Web search tool available to Agent 1 |

---

## 🧑‍🔬 The Agents

### 🔎 Agent 1: Research (Knowledge Discovery)
- Interprets the question and splits it into smaller research questions
- Searches the web with Tavily
- Collects facts, examples, industry practices, trends, benefits and challenges
- Compares sources and flags agreements or conflicts
- Keeps source links tied to their findings
- Outputs: `Research Objective`, `Key Findings`, `Detailed Observations`, `Examples and Evidence`, `Opportunities and Challenges`, `Sources and References`, `Information Gaps`
- 🚫 Does **not** give a final answer or recommendation

### 🧩 Agent 2: Analysis (Decision Support)
- Identifies what the user is really asking
- Separates important findings from minor details
- Explains what the findings mean and why they matter
- Surfaces patterns, implications, opportunities and risks
- Outputs (as fits the question): `Direct Answer`, `Key Findings`, `Analysis and Interpretation`, `Practical Implications`, `Recommendations or Next Steps`, `Conclusion`, `Sources and Limitations`
- ✅ Is the **final** agent and replies straight to the user

---

## 🛠️ Tech Stack

| Component | Choice |
|---|---|
| 🔄 Orchestration | n8n (AI Agent nodes) |
| 🦙 LLM | Ollama chat model (local) |
| 🌐 Web search | Tavily |
| 💬 Interface | n8n built-in chat |

---

## 📁 Repository Contents

| File | Description |
|---|---|
| `Assignment_4.json` | Exported n8n workflow (import to recreate) |
| `Agent_1_-_Research.txt` | System prompt for Agent 1 |
| `Agent_2_-_Analysis.txt` | System prompt for Agent 2 |

---

## 🚀 Setup

### ✅ Prerequisites
- n8n with the AI / LangChain nodes
- [Ollama](https://ollama.com) installed and running locally
- A free [Tavily](https://tavily.com) API key

### 📝 Steps

1. **Pull a model in Ollama**
   ```bash
   ollama pull <your-model>   # e.g. llama3.1
   ```
2. **Import the workflow** in n8n: *Workflows → Import from file → `Assignment_4.json`*.
3. **Add credentials**
   - 🦙 *Ollama*: set the base URL (default `http://localhost:11434`)
   - 🌐 *Tavily*: paste your API key
4. **Set the system prompts**: paste `Agent_1_-_Research.txt` into the *System Message* of Agent 1, and `Agent_2_-_Analysis.txt` into Agent 2.
5. **Open the chat** and ask your question 💬

---

## 🧪 Sample Run

| Item | Value |
|---|---|
| ❓ Query | Should a small retail business adopt Generative AI? |
| ⏱️ Total time | About 9 min 40 sec on a local model |
| 🔢 Tokens | About 5,976 |
| 📤 Output | Direct Answer and Key Findings, followed by the remaining sections |

Local models are slower than hosted APIs, so expect longer run times on modest hardware ⏳.

---

## ⚠️ Limitations & Future Ideas

- 🐢 Local inference can be slow for multi-step research.
- 📉 Output quality depends on the chosen Ollama model and the Tavily results.
- 🔁 Both agents currently run once in sequence. A review or fact-check agent could be added.
- 💾 Add persistent memory so follow-up questions keep context.
- 📊 Add evaluation checks (n8n *Evaluations*) to score accuracy and completeness.

---

## 📜 Disclaimer

This project is for **educational purposes**. Generated answers rely on web search results and a language model, so verify important information before acting on it.
