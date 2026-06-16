# Phase 0 — Setup, Fundamentals, and Forensic Rigor

> This phase lays the foundation, and in this project, it has a dual uniqueness that is vital to keep absolutely clear, because this is a high-consequence domain: **the correct approach and forensic rigor come before any code**. There is an easy temptation (to build an "automatic investigator" that examines the evidence and dictates what happened), and the most important decision of the project is not to fall into it: committing, from the start, to a strictly *assistive* AI that tames the scale but does not draw conclusions, with all of its outputs being traceable to the evidence. And there is a framework of rigor that, in forensics, is not optional: evidence integrity, traceability, and humility in the face of hallucinations. If you do not establish these two things now (the approach and the rigor), it is easy to drift toward a system that is useless for forensic purposes, or that mishandles sensitive evidence. This is why this phase is not just about "preparing the environment," but rather "getting the approach and rigor right." Furthermore, it is where you demonstrate from minute one that you understand the weight of the domain.

**Phase Objective:** environment ready, understanding DFIR and timeline analysis, and establishing the forensic rigor framework.
**Duration:** first days of week 1.
**Upon completion, you will have:** a well-structured repository and the SIFT Workstation ready, a solid understanding of digital forensic timeline analysis, and, most importantly, the project approach established (AI tames the scale, does not draw conclusions, everything traceable) and the forensic rigor framework documented.

---

## The Big Picture

This phase is about getting the foundation right across four fronts, and note that no evidence is being processed yet (that begins in Phase 1): here, we prepare and decide.

```
   [ 1. Repository, environment, and SIFT ]  ──►  the engineering base + the forensic environment
   [ 2. Understanding the problem ]          ──►  DFIR, timeline, low- and high-level events, scale
   [ 3. Establishing the approach ]          ──►  AI tames the scale, does NOT draw conclusions; everything traceable
   [ 4. Establishing forensic rigor ]        ──►  evidence integrity, assistive AI, traceability, humility
```

---

## Step 1 — The Repository, Environment, and the SIFT Workstation

You start by setting up the foundation, which consists of two parts. The first is the engineering base you already master and reuse: the repository with the foundational structure, the environment with `uv`, code quality with ruff, and a Makefile.

```bash
uv init ai-forensics
cd ai-forensics
uv add pandas            # to handle the timeline (which Plaso exports as CSV)
uv add --dev pytest ruff
mkdir -p src/{ingest,triage,reconstruct,assistant,evaluation} dashboard evidence docs notebooks tests
```

The second part is specific to this project: preparing the **forensic environment**. Download and prepare the SIFT Workstation, the SANS virtual machine with all pre-installed forensic tools (Plaso, Autopsy, Volatility, etc.). This is the environment where you will generate the timeline from the evidence; afterward, you will process the output (the timeline in CSV format) in your Python project. Having this environment ready from the start saves you the ordeal of installing each forensic tool individually.

And there is a detail of rigor that is best established right away in the `.gitignore`: the `evidence/` folder **never** goes into the repository. Forensic evidence is sensitive (it may contain personal data or information from a real incident), and uploading it would be a serious mistake. Configuring this from the very first commit demonstrates that you understand evidence handling.

As for the AI dependencies (Ollama for the LLM, scikit-learn for anomalies), you will add them in the phases where they belong, so as not to install everything you don't use yet all at once. In this phase, the lightweight base is sufficient.

---

## Step 2 — Understanding the Problem: DFIR, Timeline, and AI

Before touching anything, spend some time thoroughly understanding the domain, because there is a difference here compared to some of your other projects: although you bring the cybersecurity mindset from your honeypot, digital forensics (DFIR) and timeline analysis are largely new territory, so truly understanding them is a top priority. Document your findings in `docs/context.md`, which will also serve as material for the README.

Make sure you are clear on the concepts covered in the fundamentals. What **digital forensics** is (investigating an incident post-incident from the evidence, with the characteristic that the results may be used in court). What **timeline analysis** is (consolidating all timestamped traces into a chronological sequence, the heart of the process). And, above all, the key distinction: **low-level** events (the millions of individual footprints: a modified file, an updated registry key) versus **high-level** events (the facts the investigator wants to understand: "a USB was connected," "a program was installed"), which do not appear as such and must be **reconstructed** from low-level patterns. Internalize also the problem of **scale** (millones of events per disk, impossible to cover by hand) and how AI can help tame it.

Understanding this domain well now is what will allow you, in the following phases, to triage with solid judgment, reconstruct the facts correctly, and know what AI can and cannot contribute. It is the investment that makes everything else possible.

---

## Step 3 — Establishing an Honest Approach

This is one of the two most important steps of the phase, and the intellectual core of the project. It is where you commit, in writing, to the correct approach, in order to avoid drifting towards the temptation of the "automatic investigator." And it is beneficial to make the reasoning very clear, because knowing how to explain it is what demonstrates maturity.

The approach is: **AI tames the scale, but does not draw conclusions, and everything it produces is traceable to the evidence**. And the reason, as you saw in the fundamentals, is as follows.

It would be tempting to design the system as an AI that examines the evidence and dictates what happened, but that is unacceptable in forensics for two reasons. First, findings can go to court or inform serious decisions, and an LLM that **hallucinates** a fact (asserting something not supported by the evidence) is catastrophic, because you would be acting on a falsehood. Second, forensic rigor requires every finding to be **verifiable** against the source evidence, and an AI claim that you cannot trace back to its origin is useless.

That is why the AI remains assistive: it tackles the actual bottleneck (scale, which is what truly makes manual analysis unfeasible) by triaging the flood and reconstructing high-level events, while the investigator **validates** everything and **draws** the conclusions, and each AI output must be **traceable** back to the events that support it. Leave this documented, along with its reasoning. This is not a minor detail: it is the decision that will guide all the others (especially the grounding and traceability discipline of the AI phases), and the one that, when well-explained, demonstrates that you understand what AI is useful for (and what it is not) in such a sensitive domain.

---

## Step 4 — Establishing the Forensic Rigor Framework

The other decisive step of the phase is to document the forensic rigor framework, which in this domain is not an optional precaution but a requirement. Document it in `docs/forensic_soundness.md`. There are four pillars that should be made clear.

The first is **evidence integrity**: always working on *copies*, never on the original; verifying integrity with *hashes* (to prove that the evidence has not been altered); and respecting the chain of custody. This is the baseline for making any finding defensible.

The second is the **assistive** role of the AI: the AI suggests, summarizes, and reconstructs, but does not make rulings; the investigator decides. This is the implementation of the approach you established in the previous step.

The third is **traceability**: every output of the AI must be verifiable against the source evidence. Without traceability, a finding has no value in forensics.

And the fourth is **humility in the face of hallucinations**: recognizing that LLMs can make things up, which in forensics is the worst possible error, and designing the system (with grounding and verification) to mitigate it. It is worth highlighting something that shows you know the field: this rigor is not a precaution that you are adding on your own, but rather what forensic literature itself demands (studies insist that the investigator must always validate what the LLM produces). Documenting it this way, making it clear that you understand the weight of the domain, is one of the strongest signs of project maturity.

---

## Step 5 — Recording Decisions

As in your other projects, record the key decisions in the documentation: the approach (AI tames the scale, does not draw conclusions; everything traceable) and its reasoning, the forensic rigor framework, the chosen tools (Plaso and the SIFT Workstation, local LLM for privacy) and why, and the reuse of your skills (RAG grounding, honeypot anomalies). Having this in writing will serve you well for the README, help you maintain consistency, and show that you have made thoughtful decisions in a domain that demands them.

---

## Verification: The "Done" Criterion

The phase is complete when the following is met:

- [ ] The repository has the structure, the environment with `uv`, the code quality, and the Makefile, and the `evidence/` folder is in the `.gitignore`.
- [ ] The SIFT Workstation is ready.
- [ ] You understand DFIR, timeline analysis, and the low-level/high-level distinction, and you have documented it in `docs/context.md`.
- [ ] The approach is established and documented: AI tames the scale, does not draw conclusions, everything traceable, along with its reasoning.
- [ ] The forensic rigor framework is documented in `docs/forensic_soundness.md` (integrity, assistive AI, traceability, humility).
- [ ] **The key test:** you understand digital forensic timeline analysis and why the AI must be assistive and traceable, and you are clear on the rigor framework.

The key test has two halves that reflect the dual character of this phase: comprehension (understanding digital forensic timeline analysis) and judgment (having established the correct approach and forensic rigor). The fact that the second half is so important is characteristic of this project: in a high-consequence domain, *getting the approach and the rigor right* is just as decisive as understanding the technical aspects, because a system that draws conclusions without traceability, or that mishandles evidence, would be useless for forensic purposes, no matter how well-built it is.

---

## Deliverables and What Comes Next

Upon closing Phase 0, you have the foundations firmly in place: the engineering base and forensic environment ready, a solid understanding of digital forensic timeline analysis, and, above all, the project approach established with solid judgment (AI tames the scale, does not draw conclusions, everything traceable) and the forensic rigor framework documented. You have started the project exactly where a good professional would in such a sensitive domain: by ensuring you understand the problem and addressing it with the required rigor before writing a single line of code.

The next step, **Phase 1**, takes you to the raw material of the analysis: **the evidence and the timeline**. You will obtain a realistic forensic dataset (a disk image of an incident, from publicly available practice sets), generate the super timeline with Plaso (preserving evidence integrity, as you have just established), and understand that timeline: its low-level events, its artifacts, and its overwhelming scale. It is the foundation upon which the AI will work. You have gone from "I understand the domain and have established the approach and rigor" to being on the verge of "I have the evidence of an incident converted into a timeline ready for analysis."
