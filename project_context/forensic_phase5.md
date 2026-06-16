# Phase 5 — Rigorous Evaluation

> In this phase, you subject your system to the moment of truth, and there is an emphasis here that distinguishes it from the evaluation of your other projects: **fidelity to the evidence**. In forensics, a system that *hallucinates* (asserting a fact that the evidence does not support) is catastrophic, because you would be acting on a falsehood that could end up in court. Therefore, although you will evaluate the quality of the triage and reconstruction as you would in any system, the *most critical* evaluation—the one that truly decides whether the system is forensically admissible—is verifying that **it does not invent**. The guiding idea of this phase is precisely that: to measure how well it works, yes, but above all to verify, with zero tolerance, that it is faithful to the evidence. Furthermore, this evaluation validates (or refutes) the project's approach: if the system hallucinates, it confirms exactly why AI must be assistive and why the investigator must validate everything.

**Phase objective:** Evaluate the system honestly, especially its fidelity to the evidence.
**Duration:** Weeks 7-8.
**Upon completion, you will have:** A rigorous evaluation of your system: how well it triages and reconstructs, the verification that it does not invent (hallucination detection), an analysis of its failures, and an honest understanding of its limitations.

---

## The Big Picture

The workflow of this phase evaluates rigorously and, above all, with attention to fidelity:

```
   The System (Phases 2-4)
        │
        ├──► [ 1. Evaluate triage ] ──► recall/precision vs ground truth
        │                               (are relevant events left out?)
        │
        ├──► [ 2. Evaluate the LLM ] ──► correctness + FIDELITY (does it hallucinate? — zero tolerance)
        │
        └──► [ 3-4. Analyze & interpret ] ──► failures and hallucinations, honestly
        │
        ▼
   [ Honest evaluation + verification that it does not invent ]
```

---

---

## Step 1 — Evaluating Triage Against a Ground Truth

First, evaluate the triage: does it highlight the events that are actually relevant to the incident? To answer this, you need a **ground truth**, and here it helps that the public practice images you used are *documented*: you know what the attacker did, so you know which events are actually relevant. With that, you compare what the triage highlighted against what it should have highlighted:

```python
# Ground truth: the actually relevant events of the incident (from the case documentation)
actual_relevant = {...}                 # identifiers of the relevant events
triaged = set(clues["id"])              # what the triage highlighted

hits = triaged & actual_relevant
recall = len(hits) / len(actual_relevant)   # were relevant events left out?
precision = len(hits) / len(triaged)        # how much noise did it highlight?
print(f"Recall: {recall:.2f}  ·  Precision: {precision:.2f}")
```

It is useful to understand what each metric measures and which one matters more here, which demonstrates sound judgment. **Recall** (out of all relevant events, how many did the triage highlight) is especially critical in forensics: leaving out a relevant event is dangerous because it can lead to missing something important (for example, concluding that no exfiltration occurred because the key event was not highlighted). This links back to the epistemic honesty of Phase 4: an imperfect recall is exactly why "no evidence was found" does not equal "it did not happen." **Precision** (out of what was highlighted, how much of it was relevant) matters for usability: too much noise wastes the investigator's time. Knowing these numbers, and being honest about recall in particular, is essential to understanding what to trust and what not to trust about the triage.

---

## Step 2 — Evaluating LLM Outputs: Correctness and Fidelity

This is the central part of the phase, and where you reuse—with higher stakes—what you learned about evaluation in your RAG project. You evaluate the LLM outputs (the reconstructions from Phase 3 and the answers from Phase 4) across two dimensions. The first is **correctness**: are the reconstructed facts and answers correct (do they match the documented ground truth)? The second, and critical, dimension is **fidelity** (*faithfulness*): are the outputs actually supported by the events they cite, or has the LLM added something that the evidence does not support?

This second dimension is what you reuse from the RAG evaluation (the faithfulness metric), but here it carries existential significance: **an assertion not supported by its cited evidence is a hallucination**, and in forensics, that is the worst possible error. This is why you verify, for each LLM assertion, whether the cited events actually support it:

```python
def verify_fidelity(assertion: str, cited_events: list) -> bool:
    """Do the cited events actually support the assertion? (faithfulness, from your RAG).
    An assertion NOT supported by its evidence is a hallucination."""
    # Verify that each assertion is genuinely supported by the events it cites.
    # This can be done manually on a sample, or using an LLM as a judge (RAGAS-style),
    # provided the verification itself is rigorous.
    ...
```

In this project, fidelity is the key metric. A system can provide answers that *sound* correct but are actually unsupported by the evidence they cite; detecting these cases is what separates a reliable forensic assistant from a dangerous one. It is best to use a rigorous methodology (there is already a standardized methodology and dataset for evaluating LLMs in forensic analysis that is worth knowing and following) rather than quick manual checks: fidelity is too important to be evaluated lightly.

---

## Step 3 — Analyzing Failures, with a Focus on Hallucinations

An honest evaluation does not stop at the numbers: it analyzes *when* and *why* the system fails, and here, above all, *when it hallucinates*. Examine the cases where the system makes mistakes and, in particular, the few (ideally zero) instances where it invents, looking for patterns. Does it hallucinate when the evidence is scarce (filling in the gaps with plausible guesswork)? When the question is ambiguous? When there are many similar events and it confuses which one to cite? Each pattern tells you something about the system's limits and how to mitigate risk.

Analyzing hallucinations is the most important part, and it should be treated with the seriousness it deserves: in forensics, you do not aim to reduce hallucinations to an "acceptable" level, but rather to understand exactly when they occur so you can prevent them or, at least, warn about them. Discovering, for example, that the system tends to hallucinate when asked about something for which there is little evidence tells you where the danger lies, and reinforces the importance of the system saying "insufficient evidence" instead of inventing. This analysis turns "my system is correct X% of the time" into "I understand when and why my system fails or invents, and how reliable each part is."

---

## Step 4 — Interpreting with Honesty

The final step is to interpret and communicate all of this with the honesty required by the domain, which has an important implication here. Report the triage performance (being candid about recall: what might be left out). Report the correctness and, above all, the LLM's **fidelity**: if it hallucinates, how often, and in what situations. Frame everything by reminding what the system is and validating, with data, the project's approach. This is where the evaluation comes full circle: if the system hallucinates even occasionally (and LLMs do), it *confirms* why AI in forensics must be strictly **assistive** and why the investigator must validate every output against the evidence. The evaluation does not just measure the system; it supports the project's thesis with evidence.

This honesty, as in your other projects, strengthens the work rather than weakening it—and here, more than ever. A forensic system presented with a rigorous evaluation that acknowledges its limits (what the triage might leave out, when the LLM might hallucinate) and therefore insists on its assistive role conveys far more credibility (and is much more responsible) than one claiming perfect reliability. In a high-consequences domain, acknowledging limitations is not a weakness: it is an ethical requirement.

---

## Step 5 — Testing

Verify the evaluation logic with synthetic data. Complete `tests/test_evaluation.py`:

```python
from src.evaluation import recall_precision, verify_fidelity


def test_triage_metrics():
    relevant = {1, 2, 3, 4}
    triaged = {2, 3, 5}                 # hits 2 and 3; leaves out 1 and 4; highlights 5 (noise)
    recall, precision = recall_precision(triaged, relevant)
    assert recall == 0.5                # highlighted half of the relevant events
    assert round(precision, 2) == 0.67  # of the highlighted events, two-thirds were relevant


def test_fidelity_detects_unsupported_assertions():
    # An assertion supported by its events: faithful. Unsupported: hallucination.
    assert verify_fidelity("a login occurred", cited_events=["login at 14:00"]) is True
    assert verify_fidelity("10 GB were exfiltrated", cited_events=["login at 14:00"]) is False
```

The first test verifies the calculation of recall and precision (including the case of relevant events not highlighted, which is critical in forensics); the second verifies that the fidelity check distinguishes a supported assertion from a hallucination. (Actual fidelity checks are more sophisticated, but the evaluation logic is testable.) Run them with `make test`.

---

## Verification: The "Done" Criterion

The phase is complete when the following are met:

- [ ] You have evaluated the triage (recall and precision) against a ground truth, paying attention to what it might leave out.
- [ ] You have evaluated the LLM outputs for correctness and, above all, fidelity (faithfulness).
- [ ] You have detected and analyzed hallucinations (assertions not supported by the cited evidence).
- [ ] You have analyzed the failures, with special attention to when the system invents.
- [ ] You interpret the results with honesty, and the evaluation validates the assistive role of the AI.
- [ ] **The key proof:** You know how well the system triages and reconstructs, you have verified that it does not invent, and you understand its limitations.

The key proof has three sides, and the middle one is the decisive one in this project: knowing the performance (triage, reconstruction), having **verified that it does not invent** (fidelity, hallucinations), and knowing the limitations. That fidelity is so central is characteristic of forensic evaluation: in other domains, a system that works well most of the time may suffice; in forensics, a system that invents—even rarely—is not trustworthy for drawing conclusions, and knowing this is what allows it to be used with due caution. That is the difference between a system that seems to work and one whose reliability you truly understand.

---

## Deliverables and What Comes Next

By closing Phase 5, you have something essential in such a serious domain: a rigorous and honest evaluation of your system. You know how well it triages and reconstructs, you have verified with zero tolerance whether it hallucinates (the key metric, reusing what you learned to evaluate in RAG), you know its failures and limits, and the evaluation has supported, with data, why AI here must be assistive. You have a system whose reliability you understand, which is exactly what allows it to be used responsibly.

The next step, **Phase 6**, makes all of this usable for the investigator: **the investigator's dashboard**. You will build an interface where the investigator explores the timeline, views the triage and reconstruction, and asks questions, and where, above all, **traceability** is central: every AI output must be verifiable against the source evidence from within the interface itself. This is the implementation, within the tool, of all the rigor you have built: an assistant that the investigator can use and, crucially, verify. You have gone from "I know how reliable my system is" to being on the verge of "the investigator can use it and verify every finding against the evidence."
