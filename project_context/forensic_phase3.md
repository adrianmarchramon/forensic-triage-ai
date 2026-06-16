# Phase 3 — High-Level Event Reconstruction

> **This is the most distinctive AI part of the project.** The Phase 2 triage highlighted relevant low-level events, but the investigator does not want a thousand low-level events: they want to understand the *high-level* facts ("a USB was connected", "a program was installed", "someone logged in remotely") and the story of what happened. This is where the LLM comes in as the component that performs this translation: reconstructing high-level facts from low-level event patterns and summarizing the incident in a comprehensible narrative. This is the most valuable contribution of the AI because it turns the flood of data into understanding. However, there are two non-negotiable disciplines that run through this phase, both reused and reinforced from your RAG project: **grounding** (reconstructing only what the evidence supports, without making things up, because a hallucination in forensics is catastrophic) and **traceability** (every high-level fact must be traceable back to the low-level events that prove it).

**Phase objective:** Reconstruct high-level facts from low-level ones.
**Duration:** Weeks 4-6.
**Upon completion you will have:** A module that, using the timeline, reconstructs the high-level facts of the incident and summarizes them into a comprehensible narrative, using grounding discipline and ensuring each fact is traceable to the supporting events.

---

## The Big Picture

The workflow of this phase turns low-level data into a story:

```
   Relevant low-level events (from Phase 2 triage)
        │
        ▼
   [ Local LLM (Ollama), with grounding ]  ──►  reconstruct high-level facts
        │  (each fact CITES the low-level events that support it)
        ▼
   [ Incident summary ]  ──►  a grounded, chronological narrative
        │
        ▼
   [ High-level facts + narrative, traceable to the evidence ]
```

Add the LLM dependency (the same one from your RAG):

```bash
uv add llama-index-llms-ollama
```

---

## Step 1 — The Challenge: From Low-Level to High-Level

It is worth understanding why this is the central analytical challenge of forensics, and why the LLM is the right tool for the job. As you saw in Phase 1, the timeline is full of low-level events ("file X was modified at 14:32", "registry key Y was updated"), which are individually not very informative. However, the facts that actually matter are high-level ("a USB device was connected"), and those facts do not appear as such: they leave a *footprint* of several low-level events combined, and they must be **reconstructed** by recognizing those patterns. This abstraction (from data to meaning) is what turns a pile of events into an understanding of what happened.

And this is where the LLM fits in naturally, showing why it is the right tool rather than an arbitrary choice. Reconstructing high-level facts from patterns and summarizing a narrative is exactly what LLMs excel at: recognizing patterns in text, abstracting, and summarizing. Low-level events are textual (the description field), and the LLM can read them and recognize that "these events together mean a USB was connected." It is the AI component that genuinely contributes something that the triage (which is purely statistical) could not: *meaning*.

---

## Step 2 — Scale Architecture: What the LLM Works On

Before looking at the code, let's address a crucial architectural decision that proves you understand the project's pipeline. You cannot feed the LLM millions of events from the timeline: they will not fit in its context window, nor would it make sense. That is why the LLM works on the **triaged subset** from Phase 2 (the clues that triage already highlighted as relevant) and, if necessary, processes the timeline in **windows** (manageable chunks) to reconstruct the high-level facts of each one.

This naturally connects the two phases and reveals the logic of the pipeline: triage (Phase 2) *reduced* the flood into a manageable set of clues, and reconstruction (Phase 3) works on those clues to extract their meaning. Triage tames the scale so the LLM can do its job; the LLM provides the meaning that triage could not. Understanding that reconstruction relies on triage, and does not operate on millions of raw events, is what makes the system viable and demonstrates architectural maturity.

---

## Step 3 — Reconstructing with Grounding and Traceability

The heart of this phase is the prompt that gives the LLM the low-level events and asks it to reconstruct the high-level ones under the two non-negotiable disciplines. Each event you pass contains an **identifier** so that the LLM can cite which ones support each reconstruction (traceability). Create `src/reconstruct/reconstruct.py`:

```python
from llama_index.llms.ollama import Ollama

llm = Ollama(model="llama3", request_timeout=120.0)   # reused from your RAG

RECONSTRUCT_PROMPT = """You are a forensics analysis assistant. Based on the following LOW-LEVEL \
events from a forensic timeline, reconstruct the HIGH-LEVEL facts they represent \
(for example: "a USB device was connected", "a program was installed", "remote login").

Low-level events (each with its identifier):
{eventos}

Strict instructions:
- Reconstruct ONLY facts that are supported by the events. Do NOT make anything up.
- For each fact, indicate the identifiers of the low-level events that support it.
- If the events are not enough to assert something, say so; do not speculate.

Respond in JSON, as a list of objects containing:
- "fact": the reconstructed high-level event.
- "evidence": the list of identifiers of the low-level events supporting it.
- "confidence": "high" | "medium" | "low".
"""
```

Notice how the prompt embodies both disciplines, reused and reinforced from your RAG project. **Grounding** is explicit and strict ("reconstruct ONLY what the events support; Do NOT make anything up; if they are not enough, say so"), and here it is even more critical than in RAG, because a hallucination in forensics (asserting a fact that the evidence does not support) is the worst possible error. **Traceability** is built-in by requesting the identifiers of the supporting events for each fact: this is, in essence, a *forensic citation*, the same citation discipline you used in RAG, but now with evidentiary value. The investigator will be able to go from "a USB was connected" to the exact low-level events that prove it and verify them. Additionally, the structured JSON output (with the fact, its evidence, and a confidence level) makes the result usable and always communicates certainty.

---

## Step 4 — Summarizing What Happened

With the high-level facts reconstructed, you go one step further: summarizing them into a comprehensible, chronological narrative of what happened during the incident, also with grounding:

```python
SUMMARY_PROMPT = """Based on the following reconstructed high-level facts (each with its \
evidence), draft a clear and chronological summary of what occurred during the incident.

Reconstructed facts:
{hechos}

Strict instructions:
- Ground every statement in the facts; do not add anything that is not supported.
- Maintain chronological order.
- If there are gaps or uncertainty, indicate them explicitly; do not fill them with conjectures.
"""
```

The summary is what turns a list of facts into a story that the investigator can read and understand at a glance: "the attacker gained access here, moved there, and compromised this." However, it maintains discipline: every statement is supported by the reconstructed facts (which in turn are supported by the low-level events), gaps are flagged instead of being filled, and uncertainty is made explicit. This way, the narrative remains traceable back to the evidence and honest about what is not known. This is the difference between a useful, reliable summary and a fabricated story, which would be dangerous in forensics.

---

## Step 5 — Tests

Verify the logic that does not require the model (formatting with identifiers, parsing, and verifying that the prompt enforces the disciplines). Complete `tests/test_reconstruct.py`:

```python
from src.reconstruct.reconstruct import RECONSTRUCT_PROMPT, format_eventos_con_id, parse_json_safe


def test_los_eventos_llevan_identificador():
    eventos = [{"id": "e1", "desc": "USB conectado"}, {"id": "e2", "desc": "driver cargado"}]
    texto = format_eventos_con_id(eventos)
    assert "e1" in texto and "e2" in texto   # each event is citeable (traceability)


def test_el_prompt_impone_grounding_y_trazabilidad():
    prompt = RECONSTRUCT_PROMPT.format(eventos="(eventos)")
    assert "Do NOT" in prompt                       # grounding
    assert "identifiers" in prompt                  # traceability (citations)
    assert "not enough" in prompt or "speculate" in prompt   # honesty with gaps


def test_parseo_degrada_con_seguridad():
    r = parse_json_safe("esto no es json")
    assert isinstance(r, (list, dict))   # does not break on failure
```

The tests verify that each event is citeable (the foundation of traceability), that the prompt enforces grounding and citations, and that parsing fails safely. These are checks to ensure the critical disciplines are incorporated. Run them with `make test`.

---

## Verification: The "Fact" Criterion

The phase is complete when the following conditions are met:

- [ ] You use a local LLM (Ollama, reusing your RAG) that reconstructs high-level facts from low-level ones.
- [ ] The LLM operates on the triaged subset (and in windows if needed), not on millions of events.
- [ ] You apply strict grounding: the LLM reconstructs only what is supported and flags gaps, without making anything up.
- [ ] Each reconstructed fact cites the low-level events that support it (traceability / forensic citations).
- [ ] You generate a comprehensible chronological summary of the incident, which is also grounded.
- [ ] **The key test:** The system converts the flood of low-level events into a comprehensible and grounded high-level narrative.

The key test is the translation of data into understanding: if your system converts the flood of low-level events into a grounded, easy-to-understand high-level narrative (where each fact is traceable to the evidence), you have achieved the most distinctive AI contribution of this project. Doing so with grounding and traceability, rather than letting the LLM fabricate a plausible story, is what makes it forensically valid. A comprehensible narrative is not enough; it must be faithful to the evidence and verifiable.

---

## Deliverables and What Comes Next

By closing Phase 3, you have the most distinctive AI component of the project: a module that converts the flood of low-level events into the high-level facts that the investigator wants to understand, and summarizes them into a comprehensible narrative of the incident. You accomplished this by leveraging triage (which tamed the scale so the LLM could work), reusing the local LLM and the grounding discipline from your RAG, and incorporating traceability (forensic citations) that makes each fact verifiable. You have made the leap from data to understanding without sacrificing rigor.

The next step, **Phase 4**, makes the system interactive: the **question assistant** (RAG over evidence). While the reconstruction provides an overview of the incident, the investigator will have specific questions ("what did user X do at such-and-such time?", "is there any trace of exfiltration?"). You will build an assistant that answers those questions by retrieving relevant events from the timeline and grounding the response in them, citing them: this is RAG, the core of your previous project, applied to forensic evidence. You have gone from "reconstructing the story of the incident" to being on the verge of "answering any investigator question about the evidence with grounded, traceable answers."
