# AI Forensics — Foundations and Design

> This document represents the conceptual work prior to writing code. It covers what digital forensics and timeline analysis truly are; the problem of scale and the distinction between low-level and high-level events; the honest approach that defines the project (AI tames the scale, it does not draw conclusions, and everything is traceable to the evidence); the forensic rigor framework required by the domain; exactly what the system does and how; the use case and who it impresses; the technology stack justified piece by piece; and the repository architecture. A solid understanding of these foundations is what separates a project that simply "throws an LLM at some logs" from one that demonstrates a deep understanding of forensics, the role that AI can (and cannot) play in it, and the rigor required by a high-stakes domain.

> **The approach defining the project from the outset:** in forensics, AI **does not draw conclusions** nor does it replace the investigator's judgment, as that would be unacceptable (findings can end up in court, and an LLM hallucinating a fact is catastrophic). What AI brings to the table is **taming the scale**: triaging the deluge of events and reconstructing high-level occurrences from low-level ones, allowing the investigator to apply their expertise where it matters. This comes with a non-negotiable condition: **every AI output must be grounded in the evidence and traceable to it**.

> **Forensic rigor:** this is a high-stakes domain, so the project is built with forensic soundness: preserving evidence integrity (working on copies, verifying with hashes), keeping the AI in an assistive role, requiring every finding to be verifiable against the source, and remaining humble regarding the LLM's limitations. The literature in this field itself insists that the investigator must always validate what the LLM produces; this framework is not an afterthought, but the correct way to approach the project.

---

## Introduction: What This Project Truly Is

To understand this project, we must start with two concepts: digital forensics and timeline analysis.

**Digital forensics** (and, in its broadest sense, DFIR: Digital Forensics and Incident Response) is the process of investigating an incident (a breach, an intrusion, a crime) by examining digital evidence (disk images, memory dumps, logs, network captures) to reconstruct what happened: what occurred, when, who, and how. It is an *a posteriori* investigation: something happened, and it must be reconstructed from the traces left behind. It has a defining characteristic that is vital to keep in mind from the beginning: its results can inform serious decisions or be used in legal proceedings, making rigor a requirement, not a luxury.

**Timeline analysis** is the heart of this process and will serve as the backbone of the system. The concept: almost everything that occurs on a system leaves a timestamped trace (a modified file, an updated registry key, a log entry, a visited page). A timeline tool gathers all these traces from all sources and consolidates them into a single chronological sequence of events, the "super timeline." Reconstructing the chronological order of events is the natural way to understand an incident: seeing everything that happened in sequence.

### The Problem of Scale, and Low-Level vs. High-Level Events

This is where the problem that gives meaning to the entire project arises, and it is important to understand it well. A single disk image generates **millions** of low-level events: every file modification, every registry update, every log line. Reviewing this manually is simply unfeasible, and this is the major bottleneck in forensics: the investigator drowning in an unmanageable volume of data.

There is a crucial distinction within this deluge, which is the key to the value of AI: the difference between **low-level** and **high-level** events. The timeline is filled with low-level events ("file X was modified at 14:32", "registry key Y was updated at 14:33") that say very little individually. However, what the investigator wants to understand are **high-level** events: "a USB was connected", "a program was installed", "someone logged in remotely". These high-level facts do not appear as such in the raw timeline; they must be **reconstructed** from the patterns of low-level events they leave behind (for example, connecting a USB leaves a recognizable footprint of several low-level events grouped together). Moving from the low-level deluge to a high-level narrative is the central analytical challenge of forensics, and this is precisely where AI can assist.

### The Idea That Matters Most: Taming the Scale, Not Drawing Conclusions

There is an idea that is the core of this project—its equivalent to "the lie is in the relationship" in a disinformation project or "RL is for response, not classification" in a defense project: **in forensics, AI does not draw conclusions; it tames the scale, and everything it produces is traceable to the evidence**. Understanding why this distinction matters is what demonstrates maturity.

It would be tempting (and a serious mistake) to present the system as an "automated investigator" that examines evidence and rules on what happened. But this is unacceptable in forensics for two reasons. First, findings can go to trial or inform serious decisions, and an LLM **hallucinating** a fact (asserting something not supported by evidence) is catastrophic, as you would be acting on a falsehood. Second, forensic rigor demands that every finding be **verifiable** against the source evidence, and an AI claim that cannot be traced back to its origin is useless (worse than useless: it is dangerous).

This is why the AI is kept in a strictly **assistive** role: it tackles the real bottleneck (scale, which makes manual analysis genuinely unfeasible) by triaging the deluge and reconstructing high-level events, while the investigator **validates** everything against the evidence and **draws** the conclusions. The human retains judgment; the AI handles the scale. As a non-negotiable condition, every AI output must be grounded in the evidence and **traceable** back to it (you must be able to go from an AI claim directly to the low-level events that support it). You are not automating forensic judgment; you are automating the scaling issue that makes it unfeasible. This distinction is the intellectual and ethical core of the project.

### Who It Impresses and Why

As with other projects, it is useful to be explicit about the target audience. A **security or DFIR** profile will appreciate that you combine real forensic tools with AI to tackle a problem they genuinely feel (scale). An **AI** profile will appreciate the application of RAG and LLMs with strict grounding discipline. A **recruiter** will grasp the memorable narrative and the maturity of the approach. Everyone will value that, in a high-stakes domain, you have built with **forensic soundness**: assistive AI, traceability, and humility in the face of hallucinations.

With this in mind, every decision in this document aims for one conclusion: that anyone looking at the project thinks, *"this person applies modern AI to a real forensic problem, and does so with the rigor demanded by the domain."*

---

## 1. What Exactly You Are Going to Build

The system is a forensic analysis assistant, structured as follows:

```
   Incident evidence (disk image, artifacts)
        │
        ▼
   [ Plaso / log2timeline ]  ──►  super timeline: millions of low-level events
        │
        ▼
   [ Triage + anomaly detection (ML) ]  ──►  taming the scale: extracting the relevant
        │                                    from the deluge
        ▼
   [ Local LLM, with grounding (RAG) ]  ──►  reconstruct high-level + summarize + respond
        │  (each output TRACEABLE to source events)
        ▼
   [ Investigator's dashboard ]  ──►  explore, ask, and verify against evidence
```

The workflow embodies this central idea. Evidence is converted into a timeline (Plaso). **Triage** with anomaly detection tames the scale, highlighting what is relevant from the deluge. The **grounded LLM** reconstructs high-level occurrences from low-level events, summarizes, and answers questions—always grounded in the evidence. Finally, the **dashboard** allows the investigator to explore and, crucially, *verify* each AI output against the source evidence. In essence, what you are building is not an oracle that dictates what happened, but an assistant that tames the scale and reconstructs the story, leaving judgment and validation to the investigator.

---

## 2. The Use Case: The Bottleneck of Forensic Scale

It is worth exploring what makes this project so compelling, as it combines several arguments.

### A Real Problem and One of Scale

Forensics is overwhelmed by data volume: millions of events per disk, and an investigator who cannot review them manually. This bottleneck of scale is the primary practical problem in DFIR, and a system using AI to tame it (triaging, highlighting anomalies, and reconstructing high-level events) addresses something genuine and deeply felt. This is not a toy problem; it is a real-world necessity. Furthermore, it is a highly active area of research in 2026, with cutting-edge work on LLMs and RAG applied to digital forensics, demonstrating that you are working on a highly relevant and current topic.

### The Maturity of Forensic Rigor

This is a high-stakes domain, and handling its demands well is what most distinguishes this project. Forensic soundness (evidence integrity, the assistive role of AI, the traceability of every finding, and humility regarding hallucinations) demonstrates a level of maturity that is highly valued in a professional. It is important to emphasize that this rigor is not a caution you add simply out of prudence, but what the literature in the field itself demands (papers consistently emphasize that investigators must always validate what the LLM generates). Building with this rigor is simply the correct way to execute the project, and doing so conveys a level of capability that few portfolio projects achieve.

### A Synthesis of Your Skills

This is perhaps the most valuable argument for your portfolio. The project does not introduce an entirely new technique; instead, it **combines** what you already master, applied to a new domain. It repurposes the **grounding and local models** from your RAG project (the LLM reconstructing and answering questions based on evidence), the **anomaly detection** from your honeypot (triaging the event deluge), and the **responsible framework** from your medical and disinformation projects (assistive AI and traceability). Much like your disinformation project was the convergence point for several of your workstreams, this is another synthesis applied to a demanding domain. Showing that you know how to *integrate* your capabilities into a larger system—rather than just possessing them in isolation—is one of the strongest signals a portfolio can convey.

### A Memorable Narrative

"I built an AI assistant that helps a forensic investigator find the needle in a haystack of millions of events, without ever allowing the AI to draw conclusions it cannot back up with evidence" is a statement that combines utility, sophistication, and maturity. Being able to *demonstrate* the system reconstructing the narrative of an incident, with every finding traceable to the evidence, is highly compelling.

Thus, this project connects with your previous ones: it shares the **cybersecurity** theme of the honeypot, repurposes the **grounding** from your RAG project, and applies the **responsible framework** of the medical and disinformation projects, all integrated within a new domain.

---

## 3. The Technology Stack, Justified Piece by Piece

As in your other projects, choosing the right tools and knowing why you chose them demonstrates solid engineering judgment. The **philosophy** organizing this stack is to use standard industry tools for the forensic tasks, while repurposing your own tools (from RAG and the honeypot) for the AI components.

### SIFT Workstation: The Forensic Environment

The SIFT Workstation is the SANS virtual machine with over 500 forensic tools pre-installed and configured (including Autopsy, Volatility, The Sleuth Kit, Plaso, RegRipper, and many more). Using it saves you the hassle of installing and configuring each tool individually, providing a complete and industry-standard forensic environment from day one. This is where you will generate the timeline and handle the evidence.

### Plaso / log2timeline: The Backbone

Timeline analysis is the core of DFIR, and Plaso is the standard tool. It parses a massive variety of artifacts (file system metadata, event logs, browser history, registry entries, application logs) and consolidates them into a single chronological timeline—the "super timeline." It is what converts a disk image into an analyzable sequence of events, serving as the input for your AI system. (The typical workflow: `log2timeline` generates a `.plaso` storage file, and `psort` converts it into a CSV timeline that you can process.)

### The Local LLM with Grounding: Repurposing Your RAG

The most sophisticated AI component directly repurposes what you learned in your RAG project: a **local** LLM (via Ollama) reasoning over the timeline with strict **grounding** discipline (no inventing, basing everything on the retrieved evidence). Choosing a local model is doubly important here: for cost, as in RAG, but especially for **privacy**, which is crucial when handling forensic evidence that can be highly sensitive and must not leave for an external service. It is worth noting that this approach is not an arbitrary idea forced onto the domain; it is exactly what leading-edge research in 2026 is applying to digital forensics, using systems that combine forensic tools, retrieval, and local LLMs. Your RAG project has prepared you well for this. Retrieval (RAG) will be key to ensuring the assistant grounds its answers in relevant timeline events, making everything fully traceable.

### Anomaly Detection: Repurposing Your Honeypot

To triage the event deluge and highlight what is relevant, you repurpose the anomaly detection techniques from your honeypot project (using scikit-learn), now applied to the forensic timeline. The concept remains the same (identifying unusual behavior within a sea of normal activity), ported to a new domain—yet another example of how your skills intersect.

### Visualization

To allow the investigator to explore the timeline, view the triage and reconstruction, and verify each finding, you will use a dashboard with Streamlit (which you are already familiar with) or timeline-specific tools like Timesketch. The key, as we will see, is that traceability is central to the interface.

### Stack Summary

| Layer | Tool | What it resolves / addresses |
|------|-------------|--------------|
| Forensic Environment | SIFT Workstation | All forensic tools, ready to use |
| Timeline | Plaso / log2timeline | Unifying artifacts into a single sequence |
| Disk/Memory Forensics | Autopsy / TSK / Volatility | Analyzing evidence (disk, memory) |
| Triage & Anomalies | scikit-learn | Taming the scale (repurposed from the honeypot) |
| LLM & Grounding | Ollama + RAG | Reconstructing, summarizing, and responding (from your RAG) |
| Visualization | Streamlit / Timesketch | The investigator's traceable dashboard |
| Quality | uv, pytest, ruff, Docker | Repurposed from your other projects |

---

## 4. Repository Architecture

As in your other projects, a clean repository structure communicates professionalism and is organized to follow the project workflow:

```
ai-forensics/
├── src/
│   ├── ingest/             # generate and load the timeline (Plaso)
│   ├── triage/             # anomaly detection and prioritization (ML)
│   ├── reconstruct/        # reconstruct high-level events (LLM)
│   ├── assistant/          # the Q&A assistant (RAG over evidence)
│   ├── evaluation/         # evaluation (correct?, grounded?)
│   └── config.py
├── dashboard/app.py        # the investigator's dashboard
├── evidence/               # evidence (gitignored — sensitive data)
├── docs/
│   ├── context.md          # DFIR, timeline analysis, approach
│   └── forensic_soundness.md   # evidence integrity, grounding, assistive framework
├── notebooks/
├── tests/
├── pyproject.toml
├── Makefile
└── README.md
```

The organizing principle of this structure is the **separation of project components**. The subfolders within `src/` correspond directly to the workflow: `ingest` (timeline generation), `triage` (anomaly detection ML), `reconstruct` (LLM-based event reconstruction), `assistant` (RAG-based Q&A), and `evaluation`. The `dashboard` lives in its own directory. Two specific details in this project demonstrate a strong grasp of the domain. First: the `evidence/` folder is **always** included in `.gitignore`, as forensic evidence is highly sensitive and must never be uploaded to a repository. Second: the `docs/forensic_soundness.md` file is dedicated to evidence integrity, grounding, and the assistive framework; documenting forensic soundness shows the maturity of understanding the gravity of this domain. As with your other projects, you also reuse the standard baseline of best practices (centralized configuration, tests, Docker, and a Makefile).

---

## In Summary

Before writing a single line of code, the core objectives are clear: you will build an **AI-assisted forensic analysis assistant** that, starting from incident evidence, generates a timeline, tames its scale via triage and anomaly detection, reconstructs high-level occurrences from low-level events using an LLM, and responds to investigator queries—all while remaining grounded in the evidence and fully traceable. You have grasped the domain (timeline forensics, the problem of scale, the distinction between low-level and high-level events), the most important concept (that AI tames the scale but does not draw conclusions, and everything must be traceable back to the evidence), and the forensic rigor required by the domain (evidence integrity, assistive AI, traceability, humility). Your stack combines industry-standard forensic tools with your existing skills in RAG and anomaly detection. Finally, you have an architecture that mirrors the workflow of the project and respects the sensitivity of forensic evidence.

All of this shares a common thread: every decision is designed so that when someone reviews your work, they conclude that you apply modern AI to a real-world forensic problem, integrating your skills with the rigor required by a high-stakes domain. With these foundations established, you can begin Phase 0 knowing not only what you are going to do, but why the AI must remain assistive and traceable, and the exact level of rigor with which it must be built.
