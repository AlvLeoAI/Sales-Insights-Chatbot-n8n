# Sales Insights Chatbot | n8n

---

<aside>

**Goal:**

Let non-technical users ask sales questions in plain English and get data-backed insights drawn from Supabase (PostgreSQL).

**Problem:**

Analysts lose time translating natural-language questions into SQL and formatting results. Stakeholders want quick, readable answers (not tables or code).

**Solution:**

An n8n workflow that converts the user’s question into SQL, executes it on the “Sales Data 2024/2025” tables, and returns a narrative insight with key numbers highlighted.

---

**Step 1: Trigger (Chat UI)**

- **Node:** *When chat message received* — starts the flow when a user submits a question.

**Step 2: Generate SQL from Natural Language**

- **Node:** *SQL Generator* (LLM agent) — converts the question to strict PostgreSQL using rules (quote identifiers, no inline `ORDER BY` in UNION parts, use the existing **“Profit ($)”** column, etc.).
- **Model & Memory:** Powered via OpenRouter with a short-term *Simple Memory* to keep chat context.

**Step 3: Execute SQL**

- **Node:** *Execute a SQL query* — runs the generated query against Postgres/Supabase.

**Step 4: Normalize Output**

- **Node:** *Combine all data* — wraps SQL rows into a single JSON payload (`sql_output`) for the responder agent.

**Step 5: Create the Narrative Answer**

- **Node:** *Response Agent* (LLM) — receives `sql_output` + the original user message and writes a concise, number-rich summary (2–3 short paragraphs) formatted for business users.
- **Model:** OpenRouter Chat Model dedicated to response generation.

---

**Tools Used:**

- n8n (LangChain chat trigger + agents, Postgres node)
- Supabase / PostgreSQL (sales datasets for 2024 & 2025) — queried via the Postgres node.
- OpenRouter LLMs (separate models for SQL generation and narrative response).

**Impact:**

- Stakeholders ask “What was the most expensive product in 2025?” and get a clear answer with exact figures — no SQL knowledge required.
- Consistent guardrails (e.g., always use **Profit ($)** from the table) reduce logic errors and rework.

**🧠 Learnings:**

- Splitting responsibilities (SQL agent → DB → response agent) increases reliability and makes each part easier to debug.
- Normalizing DB rows into a single JSON block simplifies prompt design for the responder agent.
- Lightweight chat memory helps follow-ups (“Now show me profits by region”) while keeping the SQL agent deterministic.
</aside>
