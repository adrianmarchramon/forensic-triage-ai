# Phase 4 — The Q&A Assistant (RAG over Evidence)

> In this phase, you make the system interactive, and this is where your RAG project pays dividends in the most direct way. The reconstruction in Phase 3 provides an overall view of the incident, but the investigator will have specific questions: "What did user X do at such-and-such time?", "Is there any trace of data exfiltration?", "What happened to this file?". The Q&A assistant answers these questions by retrieving relevant events from the timeline and grounding the response in them, complete with citations. The guiding idea of this phase is that this is **RAG, the heart of your previous project, applied to forensic evidence**: the same retrieval, the same grounding, the same citations, now applied to an incident timeline. With two nuances that show you understand the domain: a *hybrid* retrieval that leverages the structure of the timeline, and a particularly careful epistemic honesty, because in forensics, *absence of evidence is not evidence of absence*.

**Phase objective:** Answer the investigator's questions, grounded in the evidence.  
**Duration:** Weeks 6-7.  
**Upon completion, you will have:** An assistant that answers the investigator's questions about the incident, retrieving relevant events and grounding each response in them, with traceable citations to the evidence and within a strictly assistive framework.

---

## The Big Picture

The workflow of this phase answers questions grounded in the evidence:

```
   Investigator's question
        │
        ▼
   [ Hybrid retrieval ]  ──►  structured filters (user, date) + semantic search
        │
        ▼
   [ Relevant events retrieved ]  ──►  those that answer the question
        │
        ▼
   [ Grounded LLM ]  ──►  response grounded ONLY in those events, citing them
        │
        ▼
   [ Traceable + honest response ]  (cites the evidence; distinguishes "did not happen" from "not found")
```

Add the RAG dependency (the same one from your previous project):

```bash
uv add llama-index-embeddings-ollama
```

---

## Step 1 — RAG over Evidence

It is worth recognizing what you are doing: applying RAG, which you already master, to a new corpus. In your RAG project, you retrieved code snippets relevant to a question and grounded your response in them; here, you retrieve *timeline events* relevant to the investigator's question and ground your response in them. The structure is identical: index the corpus, retrieve what is relevant to the query, and generate a grounded response with citations. Being able to transfer RAG from one domain to another is a direct demonstration that your skills are transferable.

And the central lesson of that project also carries over: **retrieval dominates quality**. The response will only be as good as the events you retrieve; if the retrieval leaves out relevant events, the response will be incorrect or a false "no evidence" finding. In forensics, this is even more delicate than in standard RAG: failing to retrieve a relevant event could lead to concluding, for example, that there was no exfiltration when the evidence did exist but was not retrieved. That is why retrieval is the critical point, and why it must be handled with care, which is what the next step is about.

---

## Step 2 — Hybrid Retrieval: Structured Filters + Semantic Search

Here is the nuance that differentiates this RAG from the one in your previous project, and which demonstrates that you understand the nature of the timeline. Unlike free-text documents, timeline events are **structured** (they have fields: date, user, host, artifact type), and forensic questions often have **structured components** ("What did user X do *on Tuesday*?" implicitly contains a user and a date). Leveraging that structure makes retrieval much more precise than a purely semantic search. That is why you use **hybrid** retrieval: combining structured filters (by user, date, artifact type) with semantic search (finding relevant events by their content). Create `src/assistant/retrieve.py`:

```python
from llama_index.core import Document, VectorStoreIndex
from llama_index.core.vector_stores import MetadataFilter, MetadataFilters
from llama_index.embeddings.ollama import OllamaEmbedding

# Index each event as a document, with its fields as metadata (reused from your RAG)
documents = [
    Document(
        text=row["desc"],
        metadata={"id": idx, "date": row["date"], "user": row["user"], "source": row["source"]},
    )
    for idx, row in df.iterrows()
]
index = VectorStoreIndex.from_documents(documents, embed_model=OllamaEmbedding(model_name="nomic-embed-text"))


def retrieve(question: str, user: str | None = None, top_k: int = 10):
    # Structured filters (if the question has them) + semantic search
    filters = None
    if user:
        filters = MetadataFilters(filters=[MetadataFilter(key="user", value=user)])
    retriever = index.as_retriever(filters=filters, similarity_top_k=top_k)
    return retriever.retrieve(question)
```

The key is the combination: the **metadata** of each event (its date, user, type) allows you to *filter* by the structured components of the question, and the **embeddings** allow you to *search* by content. A question like "What did user X do on Tuesday?" benefits from filtering by user and date before searching semantically, which returns much more precise events than a blind semantic search. Leveraging the structure of the timeline in this way is what makes forensic retrieval effective.

---

## Step 3 — Answering with Grounding and Citations

With the relevant events retrieved, you generate the response with the same grounding and citation discipline as your RAG, now with evidentiary value. The prompt grounds the response only in the retrieved events and cites them:

```python
QA_PROMPT = """You are a forensic analysis assistant. Answer the investigator's question \
based ONLY on the following timeline events.

Retrieved events (each with its identifier):
{events}

Question: {question}

Strict instructions:
- Answer ONLY with what the events support. Do NOT invent or speculate.
- Cite the identifiers of the events you rely on.
- If the events do not answer the question, say so clearly. And distinguish between "the evidence \
shows that it did NOT happen" and "I did NOT find evidence of it" (which may mean it was not \
retrieved or not recorded), because they are not the same.
- Remember: you are assisting the investigation, not issuing a ruling. The investigator interprets.
"""
```

Here again, citations are the realization of traceability: each response points to the events it relies on, allowing the investigator to go to the source and verify it. It is the same citation discipline from your RAG, now with forensic value. And the grounding is strict ("answer ONLY with what the events support; Do NOT invent") because, as with the reconstruction, a hallucinated response in forensics is catastrophic.

---

## Step 4 — The Assistive Framework and Epistemic Honesty

This step captures the two maturity points that distinguish a serious forensic assistant, which you already saw hinted at in the prompt. First, the **assistive framework**: the assistant *responds* to help investigate, but it does not *rule* or draw final conclusions. It surfaces what the evidence says (grounded and cited), and the investigator interprets. If the evidence does not provide an answer, the assistant says so instead of filling the gap with a plausible conjecture. This is the interactive manifestation of the project's overall approach (AI assists, it does not conclude).

The second, and particularly important in forensics, is **epistemic honesty**: the maxim that *absence of evidence is not evidence of absence*. When the assistant does not find evidence of something, it must carefully distinguish between two very different things: "the evidence shows that this did not occur" and "I have not found evidence of this." The latter could be because the event was not recorded, because it was not retrieved (remember that retrieval dominates and is imperfect), or because it truly did not happen, and confusing them would be a grave mistake. An assistant that flatly states "there was no exfiltration" when it actually just means "I found no evidence of exfiltration" would lead to a dangerous conclusion. This subtle but crucial distinction demonstrates that you understand the limits of what a retrieval system can assert, and it is precisely the kind of rigor that the forensic field demands.

---

## Step 5 — Tests

Verify the logic that does not require the model (the construction of documents with metadata, and ensuring the prompt enforces the necessary disciplines). Complete `tests/test_assistant.py`:

```python
from src.assistant.qa import QA_PROMPT, event_to_document


def test_events_are_indexed_with_metadata_for_filtering_and_citing():
    row = {"desc": "login", "date": "01/05/2026", "user": "ana", "source": "LOG"}
    doc = event_to_document(idx=42, row=row)
    assert doc.metadata["id"] == 42          # citeable (traceability)
    assert doc.metadata["user"] == "ana"     # filterable (hybrid retrieval)
    assert doc.metadata["date"] == "01/05/2026"


def test_prompt_enforces_grounding_citations_and_epistemic_honesty():
    prompt = QA_PROMPT.format(events="(events)", question="was there exfiltration?")
    assert "Do NOT invent" in prompt                                 # grounding
    assert "Cite the identifiers" in prompt                          # citations (traceability)
    assert "did NOT find evidence" in prompt                         # epistemic honesty
    assert "not issuing a ruling" in prompt or "interprets" in prompt # assistive framework
```

The tests verify that events are indexed with the necessary metadata (for filtering and citing), and that the prompt enforces grounding, citations, epistemic honesty, and the assistive framework. They ensure that the key disciplines are built in. Run them with `make test`.

---

## Verification: The "Definition of Done"

The phase is complete when the following are met:

- [ ] You have built an assistant that answers questions by retrieving relevant events (RAG over the timeline).
- [ ] You use hybrid retrieval (structured filters + semantic search) that leverages the structure of the timeline.
- [ ] Each response is grounded in the retrieved events and cites them (traceability).
- [ ] The assistant is strictly assistive (helps investigate, does not make rulings).
- [ ] The assistant is epistemically honest (distinguishes "did not happen" from "found no evidence").
- [ ] **The key test:** The investigator can ask questions about the incident and get grounded, traceable answers back to the evidence.

The key test is that of a useful and rigorous assistant: if the investigator can ask a question about the incident and obtain a response grounded in the evidence and traceable back to it, you have built the interactive piece that completes the system. And doing so with hybrid retrieval, citations, an assistive framework, and epistemic honesty is what makes it valid for forensics. It is not enough to simply answer; one must answer by grounding, citing, and being honest about what the evidence (and the retrieval) allows us to assert.

---

## Deliverables and Next Steps

By closing Phase 4, you have the complete system in terms of its AI capabilities: in addition to triaging (Phase 2) and reconstructing the story (Phase 3), you now *answer* the investigator's specific questions, retrieving relevant events and grounding each response in them with citations. You have applied RAG, the heart of your previous project, to forensic evidence, with a hybrid retrieval that leverages the structure of the timeline and with the epistemic honesty that the domain demands. You have a forensic assistant that helps investigate without ever replacing the investigator's judgment.

The next step, **Phase 5**, puts all of this to the ultimate test: **rigorous evaluation**. You have a system that triages, reconstructs, and answers, but does it actually work? And above all, is it faithful to the evidence? You will evaluate whether the triage highlights the truly relevant events, whether the LLM's reconstructions and answers are correct and grounded, and, most critically in forensics, whether the system **hallucinates**. This is the phase that distinguishes a system that seems to work from a reliable one, and where you will apply the evaluation rigor of your previous projects with special care: in forensics, inventing a fact is the worst possible mistake. You have gone from "I have an assistant that answers" to being on the verge of "I know how well it works and, above all, that it does not hallucinate."
