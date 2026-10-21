# StackSage

**A developer help desk that sends easy questions to cheap models and hard ones to strong models.**

| | |
|---|---|
| **Student** | SHRUTHIE LAYAA.K |
| **Register No.** | 411723104045 |
| **Department** | CSE |
| **Institution** | PSVPEC |
| **Programme / Track** | Elite |
| **Project ID** | E-ENT-06 |

---

## 1. The User

A **support engineer at a Hyderabad software services company** who supports **20 client development teams**.

## 2. The Problem

Client teams ask **hundreds of PostgreSQL and Python questions a day**, from "what does this function return?" to tricky upgrade problems.

- Sending every question to a large model is **too slow and costs too much**.
- Sending every question to a small model gets the **hard ones wrong**.

StackSage decides, per question, which model should answer it.

## 3. The Data

| Source | Licence |
|---|---|
| PostgreSQL manuals, versions 14 to 18 | PostgreSQL Licence |
| Python 3 documentation | PSF Licence |

- Source: <https://www.postgresql.org/docs/>
- Size: **10,000+ pages minimum** (the PostgreSQL manual alone is about 3,000 pages per version).
- **Multi-tenant:** each client team is a tenant **pinned to its own PostgreSQL and Python versions**, so answers and citations only come from the versions that team uses.

## 4. Core AI Feature: Cost-Aware Model Routing

A small classifier labels each question **easy** or **hard** using features such as:

- question length
- number of retrieved chunks
- version mentions (for example "15 vs 17")

Routing:

| Label | Model |
|---|---|
| Easy | Small **Groq** model |
| Hard | Stronger **Gemini** model |

**Per-answer cost meter** shows, for every answer:

1. which model answered,
2. why it was routed there,
3. its cost in rupees (INR).

**Evaluation:** cost per query and accuracy are measured in three modes: **always-small**, **always-large** and **routed**.

## 5. The Agent: Pull-Request Review Agent

The agent reviews pull requests that change **SQL migrations** or **Python database code**.

It flags risky changes such as:

- locking migrations
- missing indexes
- unsafe string-built queries (SQL injection risk)

and **cites the matching docs page through StackSage**.

> **Human in the loop:** the agent **only comments**. A human engineer decides whether to merge. It never approves, merges or pushes code.

## 6. Three Sample Questions

1. How do I add a column to a large Postgres 16 table without locking it?
2. What changed in the `MERGE` command between Postgres 15 and 17?
3. What is the difference between a list comprehension and a generator expression in Python?

Rough expectation (to be verified in evaluation): Q3 should route to the small model, while Q1 and Q2 (version-specific, multi-chunk) should route to the strong model.

## 7. Success Metric

> **Cost per query cut by 60% or more versus the always-strong model, with an accuracy loss of under 3 points.**

These are **targets**. Measured results will be added to the [Evaluation](#10-evaluation) section once the test set is run.

## 8. Architecture

```
Client team (tenant, pinned versions)
        |
        v
  Query API  --->  Hybrid retrieval (keyword + vector) over versioned docs
        |                         |
        |                         v
        |                 Retrieved chunks
        v                         |
  Routing classifier  <-----------+
  (length, chunk count, version mentions)
        |
   +----+-----+
   |          |
 easy        hard
   |          |
 Groq       Gemini
 (small)    (strong)
   |          |
   +----+-----+
        v
 Answer + citations + cost meter (model, reason, Rs.)
        |
        v
 PR review agent (comments only) ---> human engineer decides to merge
```

## 9. Tech Stack

*(Update this table if choices change.)*

| Layer | Choice |
|---|---|
| Language | Python 3 |
| API | FastAPI |
| Database / vector store | PostgreSQL + pgvector |
| Search | Hybrid (full-text + vector) |
| Background ingestion | Job queue / worker |
| Small model | Groq |
| Strong model | Gemini |
| Auth | Client login per tenant |
| CI | GitHub Actions |
| Agent tooling | Real tools + MCP + memory |

## 10. Evaluation

A fixed test set of questions (starting with the first 10 and growing) is run in three modes.

| Mode | Accuracy | Avg cost / query (Rs.) | Avg latency |
|---|---|---|---|
| Always-small | _TBD_ | _TBD_ | _TBD_ |
| Always-large | _TBD_ | _TBD_ | _TBD_ |
| **Routed** | _TBD_ | _TBD_ | _TBD_ |

Target: routed cost is at least 60% lower than always-large, and accuracy is within 3 points.

## 11. Safety

- Answers must cite a docs page; no citation means the answer is flagged.
- Retrieval is filtered by the tenant's pinned versions to avoid version mix-ups.
- The PR agent is **read-only plus comment**; merge is always a human decision.
- API keys are stored in environment variables, never committed.

## 12. Getting Started

```bash
git clone <repo-url>
cd stacksage
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # add GROQ_API_KEY and GEMINI_API_KEY
python -m stacksage.ingest  # load docs
uvicorn stacksage.api:app --reload
```

## 13. Milestones

| # | Milestone | Status |
|---|---|---|
| 1 | Repo + README + 10 questions + first data | Not done |
| 2 | LLM calls + live link + CI | Not done |
| 3 | 500 pages in + answers with citations | Not done |
| 4 | Hybrid search + background ingestion + client login | Not done |
| 5 | Test score + safety + hardest problem v1 | Not done |
| 6 | Load test + cost + **Demo Day 1 (Sat 28 Nov)** | Not done |
| 7 | Agent that uses your product | Not done |
| 8 | Real tools + MCP + memory | Not done |
| 9 | Planning + human approval | Not done |
| 10 | Agent tests + safety (light week) | Not done |
| 11 | Background jobs + cost (light week) | Not done |
| 12 | **Final Demo Day (Sat 9 Jan)** | Not done |

## 14. Licences and Attribution

- PostgreSQL documentation: PostgreSQL Licence
- Python 3 documentation: PSF Licence
- This project's code: add your chosen licence (for example MIT).
