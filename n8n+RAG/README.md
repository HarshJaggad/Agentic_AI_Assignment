# Financial Documented Assistance – RAG Chatbot (n8n + Ollama)

A fully local **Retrieval-Augmented Generation (RAG)** assistant built in [n8n](https://n8n.io). It answers beginner-level personal-finance questions using **only** the content of a knowledge-base document, and politely refuses anything outside that scope (stock tips, personal tax calculations, guaranteed returns, etc.).

Built as **Assignment 3** for the MBA (Data Science & Data Analytics) program at SCIT, Symbiosis International (Deemed University).

---

## Features

- **Document-grounded answers**: the agent must retrieve from the knowledge base before answering.
- **Fully local LLM stack**: chat model and embeddings run through Ollama, with no paid API.
- **Conversation memory**: a window buffer keeps chat context across turns.
- **Safety guardrails**: general education only. No personalized advice, stock picks, tax calculations, loan-eligibility decisions or guaranteed returns.
- **Graceful fallback**: if the answer isn't in the document, it replies *"I couldn't find that information in the provided knowledge base."*

---

## Architecture

The workflow has two flows.

### 1. Ingestion flow (run once)

```
Manual Trigger → Google Drive (Download file) → Simple Vector Store (Insert)
                                                   ├── Default Data Loader (binary)
                                                   └── Embeddings Ollama (mxbai-embed-large)
```

1. Click **Execute workflow**.
2. The `.docx` knowledge base is downloaded from Google Drive.
3. The Default Data Loader splits it into chunks (12 items in my run).
4. `mxbai-embed-large` embeds each chunk, and the vectors go into the in-memory vector store under the key `vector_store_key`.

### 2. Chat / query flow

```
Chat Trigger → AI Agent ──┬── Ollama Chat Model (llama3.1)
                          ├── Simple Memory (window buffer)
                          └── Simple Vector Store2 (Retrieve as Tool, limit 4)
                                  └── Embeddings Ollama2 (mxbai-embed-large)
```

1. The user sends a message through the n8n chat.
2. The AI Agent decides to call the vector-store tool with a search query.
3. The query is embedded (1024-dimension vector) and the top 4 matching chunks are retrieved.
4. `llama3.1` writes an answer grounded in those chunks, following the system prompt.

---

## Tech Stack

| Component | Choice |
|---|---|
| Orchestration | n8n |
| Chat model | `llama3.1:latest` via Ollama |
| Embedding model | `mxbai-embed-large:latest` via Ollama |
| Vector store | n8n Simple (In-Memory) Vector Store |
| Memory | n8n Simple Memory (window buffer) |
| Source storage | Google Drive |
| Knowledge base format | `.docx` |

---

## Repository Contents

| File | Description |
|---|---|
| `Assignment_3.json` | Exported n8n workflow. Import it to recreate the project. |
| `Agent_Prompt.txt` | System prompt used by the AI Agent. |
| `Financial_Documented_Assistance_RAG_Knowledge_Base.docx` | Knowledge base document. |
| `images/` | Screenshots of the workflow and test runs. |

---

## Knowledge Base Topics

The document is a beginner-friendly reference covering:

1. Budgeting
2. Emergency savings
3. Saving vs. investing (including diversification)
4. Simple and compound interest (with a $100 worked example)
5. Credit cards and borrowing
6. Financial scams and account safety
7. Key terms glossary
8. Quick-reference Q&A
9. Scope limits and safe responses
10. Instructions for a RAG assistant
11. Suggested RAG test questions

---

## Setup

### Prerequisites

- n8n (self-hosted or desktop) with the AI / LangChain nodes
- [Ollama](https://ollama.com) running locally
- A Google account for Drive access (or swap in another file source)

### Steps

1. **Pull the models**
   ```bash
   ollama pull llama3.1
   ollama pull mxbai-embed-large
   ```
2. **Import the workflow**: in n8n go to *Workflows → Import from file* and select `Assignment_3.json`.
3. **Configure credentials**
   - *Ollama*: set the base URL (default `http://localhost:11434`).
   - *Google Drive OAuth2*: connect your account.
4. **Upload the knowledge base** to your Google Drive, then in the **Download file** node select that file.
5. **Ingest the data**: click **Execute workflow** and confirm the Simple Vector Store node shows items inserted.
6. **Chat**: click **Open chat** and ask a question.

> **Note:** The Simple Vector Store keeps data **in memory only**. If n8n restarts, run the ingestion flow again. For persistence, replace it with Qdrant, Pinecone, Supabase, etc.

---

## Agent System Prompt (summary)

The full prompt is in `Agent_Prompt.txt`. Key rules:

- Retrieve from the vector store tool **before** answering.
- Answer strictly from retrieved content, with no outside knowledge or invented figures.
- Use simple, beginner-friendly language and preserve the document's qualifications.
- If the answer isn't found, say so. Don't guess.
- Decline off-topic questions.
- **Financial safety**: education only, no stock or fund recommendations, no guaranteed returns, no personal tax or loan-eligibility decisions.

---

## Testing

Suggested test questions (also in the knowledge base):

| # | Question | Expected behavior |
|---|---|---|
| 1 | What is a budget, and what should I include when creating one? | Grounded answer |
| 2 | Why is it useful to have emergency savings? | Grounded answer |
| 3 | What is the difference between saving and investing? | Grounded answer |
| 4 | Explain compound interest using the $100 example in this document. | $105 after year 1, $110.25 after year 2 |
| 5 | Does diversification guarantee that I will not lose money? | "No", with qualification |
| 6 | Why can paying only the minimum on a credit card cost more over time? | Grounded answer |
| 7 | What should I review before accepting a loan? | Rate, fees, schedule, total repayment, etc. |
| 8 | How can I respond safely to a suspicious financial message? | Verify through an official channel |
| 9 | Which specific stock should I buy tomorrow? | Refuse / not in knowledge base |
| 10 | Can you calculate my personal tax liability? | Refuse / not in knowledge base |

### Sample result

**Query:** *What is compound interest? Explain it using the $100 example given in the knowledge base.*

**Agent retrieval query:** `compound interest explanation and the $100 example`

**Response:** Compound interest is interest calculated on the original principal and accumulated interest, subject to the account's terms. The $100 example shows that if $100 earns 5% interest compounded annually, it becomes $105 after one year and $110.25 after two years, assuming no fees, taxes, withdrawals, or additional deposits. This is a mathematical example, not a promise of returns.

The agent called the vector-store tool, the top-4 chunks included the "Simple and Compound Interest" section, and the answer matched the document.

---

## Screenshots

| Ingestion run | Execution logs |
|---|---|
| ![Workflow](images/workflow-overview.png) | ![Logs](images/ingestion-logs.png) |

| Chat test | Vector store retrieval |
|---|---|
| ![Chat](images/chat-test.png) | ![Retrieval](images/vector-store-output.png) |

| AI Agent configuration |
|---|
| ![Agent](images/agent-config.png) |

---

## Limitations & Future Work

- In-memory vector store is volatile, so swap in a persistent store for real use.
- `llama3.1` is a small local model. A larger model may follow the "no outside knowledge" rule more reliably.
- Retrieval depends on chunking. Tuning chunk size and overlap or enabling reranking could improve results.
- Add an automated evaluation set (n8n *Evaluations* tab) to score faithfulness.
- Support multiple documents and a refresh pipeline.

---

## Disclaimer

This project is for **educational purposes only**. The assistant provides general financial literacy information, **not** personalized financial, investment, tax, or legal advice.

