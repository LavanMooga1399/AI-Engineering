# RAG_PLAYBOOK.md — Is RAG Dead? How Top Companies Build Retrieval in 2026

> Companion to [`AI_SYSTEM_DESIGN.md`](../ai-system-design/AI_SYSTEM_DESIGN.md) (Section 2, Knowledge). **Read this whenever a task involves giving an LLM access to knowledge it wasn't trained on**: documents, policies, product data, tickets, codebases, search, Q&A bots, "chat with your docs", or any retrieval/RAG pipeline.

```mermaid
flowchart LR
    M0[Model memory alone] --> M1[One-shot RAG]
    M1 --> M2[Agentic retrieval]
    M2 --> M3[Hybrid retrieval]
    M3 --> M4[Knowledge runtime]
```

**RAG is only dead if knowledge never changes.**

### Why people say "RAG is dead": three camps, three different claims

| Camp | Claim | Verdict |
|---|---|---|
| **1. Context windows got huge** | Just paste everything into the prompt | **Weak critique:** accuracy drops in long contexts and cost scales with every token (Section 2) |
| **2. One-shot RAG breaks** | A single lookup fails on vague or multi-step questions | **Real critique:** solved by agentic retrieval (Section 4) |
| **3. Vector databases are dead** | Coding agents read files instead of embedding them | **Partly real:** vectors are only one retrieval method (Section 3) |

Each camp is criticising something different, which is why the argument never gets settled.

**Bottom line:** basic one-shot RAG (one question → one vector lookup → one answer) is fading. Retrieval itself is not; it is evolving into a **knowledge runtime**: retrieval that picks the right method, searches iteratively when needed, and returns answers that are cited, permission-checked, conflict-aware and honest about confidence.

---

## 1. How RAG Works

The model only knows its training data. It hasn't read your handbook, and it doesn't know what changed last week. RAG is the open-book step between the question and the answer: **find the right information first, then answer from it.**

```mermaid
flowchart LR
    subgraph Index["Indexing (ahead of time, and on every change)"]
        direction LR
        D[Source docs<br/>& data] --> CH[Chunk by<br/>structure]
        CH --> MD[Attach metadata:<br/>source · date · owner ·<br/>access rules]
        MD --> ST[(Search indexes:<br/>keyword · vector ·<br/>graph · SQL)]
    end
    subgraph Query["Answering (per question)"]
        direction LR
        Q[Question] --> R[Retrieval<br/>permission-filtered]
        R --> G[Augmented generation<br/>answer only from<br/>retrieved sources]
        G --> A[Answer + citations<br/>+ confidence]
    end
    ST --> R
```

- **Retrieval** is the hard half: finding the right passages within the model's context limit.
- **Generation** is the easier half: the model writes the answer from what was retrieved.
- Almost every RAG technique improves one of these two halves.

---

## 2. Do You Need Retrieval, or Just a Big Context Window?

Large context windows don't remove the need for retrieval, for two reasons:
1. **Accuracy:** models get "lost in the middle" of very long contexts and miss or confuse details.
2. **Cost and latency:** you pay for every token on every call. Sending the whole knowledge base each time is slow and expensive at scale.

```mermaid
flowchart TD
    K[Knowledge the feature needs] --> S{Small, stable, and fits<br/>comfortably in context?<br/>e.g. one short FAQ}
    S -- Yes --> V{Volume low enough that<br/>per-call cost is acceptable?}
    V -- Yes --> CS["Put it in context directly<br/>(use prompt caching if available)<br/>and still run evals"]
    V -- No --> RET[Use retrieval]
    S -- No --> RET
```

Rule: default to **retrieval** for anything large, growing, frequently changing, permissioned or high-volume. Stuffing everything into context is acceptable only for small, stable corpora, and only if evals confirm accuracy.

---

## 3. Match the Retrieval Method to the Data

Vector search was always just one method. "RAG" ≠ "vector database". Pick the method from the data and the questions, and treat anyone selling one answer for every domain with suspicion.

| Method | Best when | Example |
|---|---|---|
| **Keyword / full-text (BM25), grep** | Exact terms matter: names, IDs, codes, error messages, file paths | Codebases, SKUs, log search, legal clause numbers |
| **Vector / semantic search** | Same meaning, different words | "My card keeps getting declined" → doc titled "Payment authorization failures" |
| **Structured queries (SQL / API)** | Answer is in tables or live systems | Order status, account balances, inventory |
| **Knowledge graph** | Questions about relationships across entities | "Which suppliers affect products in region X?" |
| **File navigation (list, open, read)** | Structured, hierarchical content an agent can browse | Repos, docs folders, wikis |
| **Hybrid** (blend of the above + reranking) | Real-world mixed content; the usual production choice | Support KB with product names *and* fuzzy questions |

```mermaid
flowchart TD
    Q[What do users ask about,<br/>and what does the data look like?] --> X{Exact identifiers<br/>dominate?}
    X -- Yes --> KW[Keyword / grep / file navigation]
    X -- No --> SEM{Paraphrased, meaning-based<br/>questions over prose?}
    SEM -- Yes --> VEC[Vector search]
    SEM -- No --> STR{Data lives in tables<br/>or live systems?}
    STR -- Yes --> SQL[SQL / API tools]
    STR -- No --> REL{Questions about<br/>relationships?}
    REL -- Yes --> KG[Knowledge graph]
    REL -- No --> KW
    KW --> MIX{Both exact and<br/>semantic needs?}
    VEC --> MIX
    MIX -- Yes --> HY[Hybrid + reranker]
```

> **Note for the coding agent itself:** to understand this repository, search and read files directly (grep, file listing, opening files). Code is structural and full of exact names, so don't build an embedding index of the codebase for your own navigation.

---

## 4. One-Shot vs Agentic Retrieval

One lookup is enough for simple, single-fact questions over stable content. Multi-part questions ("compare these options", "what changed this quarter", "brief me from several sources") need **agentic retrieval**: the model searches, checks whether the results are sufficient, rewrites the query, and searches again, like a human researcher.

```mermaid
flowchart TD
    Q[Question] --> C{Simple, single fact,<br/>stable content?}
    C -- Yes --> ONE[One-shot: one retrieval,<br/>one answer]
    C -- No --> P[Plan sub-questions]
    P --> SR[Search]
    SR --> EV{Enough evidence<br/>to answer?}
    EV -- No, and budget left --> RW[Rewrite / refine query<br/>or try another method] --> SR
    EV -- No, budget spent --> PART[Answer what is supported;<br/>state what's missing]
    EV -- Yes --> ANS[Synthesise answer<br/>with citations]
```

In 2026, retrieval is no longer a single lookup; it is a reasoning loop: **Retrieve → Read → Decide → Search again (if needed) → Ground the answer.** Agentic techniques include query reformulation, multi-step lookup, and reading results before deciding the next step. These matter most for compare, summarise and investigate questions.

Trade-off: agentic retrieval costs more and is slower per answer. Rules:
- Use it **only when question complexity calls for it**; route simple questions to one-shot.
- Set hard limits: max search iterations, max tokens, and a time budget.
- Log every search step (query, method, results) so loops can be debugged.
- Prefer models that are trained for iterative search and tool use over hand-built chains of tricks.

---

## 5. Indexing and Chunking Rules

- **Chunk by structure,** not fixed character counts: headings, sections, functions, table rows. Keep enough surrounding context (title, section path) in each chunk to make sense alone.
- **Attach metadata** to every chunk: source ID/URL, title, section, last-updated date, version, owner, and **access-control info**.
- **Keep the index fresh:** knowledge never stops changing. Re-index on change (webhooks/events) or on a schedule; remove deleted and superseded documents; record `indexed_at`.
- **Version policies:** when a policy changes, mark the old one superseded rather than leaving both as equally valid. Stale knowledge was behind a real chatbot quoting a policy retired a year earlier.
- **Deduplicate** near-identical documents so they don't crowd out other results.
- Make chunk size, top-k, embedding model and reranker **config values** so they can be tuned against evals.

---

## 6. The Knowledge Runtime: Answers You Can Trust

A good answer is not enough; it must be one you could hand to an auditor. Compare:

| Opaque answer | Knowledge-runtime answer |
|---|---|
| Clean and confident | Same answer, plus: |
| Unknown sources | Every claim tied to an exact source |
| Unknown whether it was allowed to read them | Retrieval respected the user's access rights |
| Contradictions hidden | Conflicting sources flagged, with which one was chosen and why |
| No sense of certainty | Confidence signal: act, or dig deeper |
| No record | Full audit log of what was retrieved and used |

```mermaid
flowchart LR
    Q[Question + user identity] --> AC[Access control:<br/>filter BEFORE retrieval]
    AC --> RT[Retrieve<br/>right method]
    RT --> FR[Freshness check:<br/>drop superseded / stale]
    FR --> CF[Conflict detection<br/>across sources]
    CF --> GEN[Generate grounded answer]
    GEN --> OUT["Answer contract:<br/>claims + citations ·<br/>conflicts · confidence"]
    GEN --> LOG[(Audit log)]
```

### Rules
- **Access control at retrieval time.** Filter by the requesting user's permissions in the search query itself. Never retrieve everything and filter afterwards, and never let documents a user can't see reach the model.
- **Cite every claim.** Each factual statement maps to a source ID and passage. Uncited claims about policies, prices, legal terms or account data are not allowed.
- **Flag conflicts.** If sources disagree, say so, state which one was used (e.g. newer, more authoritative) and which parts of the answer it affects.
- **Signal confidence honestly,** based on evidence quality (number and agreement of sources, recency, retrieval scores), not the model's tone.
- **Say "I don't know"** when sources don't cover the question, and route to a human where the stakes are high.
- **Audit log:** question, user, retrieved source IDs and versions, final answer, citations, confidence, timestamp (redact personal data per privacy rules).
- **Treat retrieved content as data, not instructions.** Documents can contain prompt-injection text; the model must not follow instructions found inside retrieved content.

### Answer contract (structured output)

```json
{
  "answer": "string",
  "claims": [
    { "text": "string", "sources": [{ "id": "doc-123#sec-4", "title": "string", "updated_at": "YYYY-MM-DD" }] }
  ],
  "conflicts": [
    { "description": "string", "sources": ["doc-123", "doc-456"], "resolution": "string", "affects_claims": [0] }
  ],
  "confidence": "high | medium | low",
  "missing_information": ["string"]
}
```

---

## 7. Evaluating Retrieval

Evaluate the two halves separately, then end to end (see [`AI_SYSTEM_DESIGN.md`](../ai-system-design/AI_SYSTEM_DESIGN.md), Section 3):

| What | Metric / check |
|---|---|
| **Retrieval** | Recall@k (is the right passage in the top k?), precision, MRR; per-method comparison (keyword vs vector vs hybrid) |
| **Grounding** | Every claim supported by its cited passage (code check + LLM-as-judge) |
| **Citations** | Cited source actually contains the claim; no invented sources |
| **Conflicts** | Test cases with contradictory docs: flagged and resolved correctly? |
| **Freshness** | Test cases where a policy changed: newest version used? |
| **Permissions** | Test users with restricted access: no leakage, ever (hard fail) |
| **Abstention** | Questions the KB can't answer: says "I don't know"? |
| **Cost & latency** | Tokens and seconds per answer, one-shot vs agentic |

Build the eval set from real user questions, including paraphrases with no shared keywords (e.g. "vacation time" vs "paid time off").

---

## 8. When to Use What

| Approach | Use when | Example |
|---|---|---|
| **Simple one-shot RAG** | Stable, contained knowledge; simple questions | FAQs, product manuals, benefits policy |
| **Agentic retrieval** | Open-ended, multi-step questions, or shifting knowledge | "Compare plans A and B", "What changed this quarter?" |
| **Hybrid retrieval** | Both exact terms and meaning-based matches are needed | Support KB with product names *and* paraphrased problems |
| **Knowledge runtime** | Sensitive or high-trust settings where answers must be inspectable | Legal, finance, HR, healthcare, customer-facing policy answers |

These stack rather than compete: a high-trust support bot is typically hybrid + agentic for hard questions + full knowledge-runtime guardrails.

---

## 9. Retrieval Design (Add to the Feature Spec)

Add this to the AI Design block in [`AI_SYSTEM_DESIGN.md`](../ai-system-design/AI_SYSTEM_DESIGN.md):

```markdown
## Retrieval Design
- **Knowledge sources:** <what, where, size, how often it changes, owner>
- **Context vs retrieval:** <decision + reason (size, cost, accuracy)>
- **Retrieval method(s):** <keyword / vector / SQL / graph / file nav / hybrid + reranker, and why for this data>
- **One-shot or agentic:** <routing rule; iteration, token and time limits>
- **Chunking & metadata:** <strategy; fields attached>
- **Freshness:** <re-index trigger/schedule; how superseded docs are handled>
- **Access control:** <how permissions are enforced at query time>
- **Answer contract:** <citations, conflicts, confidence, abstention behaviour>
- **Audit logging:** <what's logged, where, retention>
- **Evals:** <retrieval metrics, grounding, permissions, freshness, thresholds>
```

---

## The Whole Picture

- **RAG isn't dead; one-shot, vector-only RAG is fading.**
- **Big context windows** don't replace retrieval: accuracy drops and cost scales with every token.
- **Agentic retrieval** loops when questions are complex; route simple ones to one-shot to save cost.
- **Beyond vectors:** match the method to the data; hybrid is the usual production answer.
- **Knowledge runtime:** cited, permissioned, fresh, conflict-aware, confidence-scored and auditable.
- Knowledge never stops changing, so retrieval never stops being needed.
