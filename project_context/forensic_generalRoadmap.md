# Roadmap: AI-Assisted Forensics

> A system that assists a forensic investigator in finding the needle in the haystack: it ingests incident evidence, converts it into a timeline with millions of events, uses AI to triage this deluge and highlight what is relevant, reconstructs high-level events from low-level ones, and answers the investigator's questions—all backed by evidence and traceable back to it. It combines digital forensics and incident response (DFIR) with modern AI (LLMs and machine learning) to address a real-world problem: forensic investigations are drowning in data volume, and AI, when used correctly, helps tame it.

**Estimated Duration:** 8-10 weeks (part-time)  
**Level:** Intermediate-Advanced  
**Final Outcome:** An AI-assisted forensic analysis system that, starting from incident evidence, generates a timeline, triages and detects anomalies, reconstructs high-level events, and answers the investigator's questions with well-founded, traceable answers, presented with the rigor required by the forensics field.

> **The defining project approach, from day one:** in forensic analysis, the AI **does not draw conclusions** or replace the investigator's judgment, as that would be unacceptable (findings may end up in court, and an LLM "hallucinating" a fact is catastrophic). What AI brings to the table is **taming the scale**: a single disk generates millions of low-level events (every file modification, every registry entry) that are impossible to review manually. The AI's job is to triage this haystack, highlight the clues, and reconstruct high-level events (e.g., "a USB was connected") from the low-level deluge, allowing the investigator to apply their expertise where it truly matters. There is one non-negotiable condition: **every AI output must be grounded in the evidence and traceable back to it**, because in forensics, you cannot act on a claim you cannot verify. The AI makes the haystack searchable and reconstructs the story; the investigator finds and validates the needle.

> **Note on forensic soundness:** This is a high-consequence domain, so the project is built with **forensic rigor**. This means preserving evidence integrity (working on copies, never on the original; verifying with hashes), keeping the AI in an **assistive** role (suggesting and summarizing, not deciding), requiring every finding to be **verifiable** against the source, and remaining humble about the LLM's limitations (hallucinations are unacceptable here). In fact, the literature in this field insists that investigators must always validate what the LLM produces; this framework is not an afterthought—it is the correct way to build the project.

---

## 1. What You Will Build Exactly

An AI-assisted forensic analysis system with this workflow:

```
   Incident evidence (disk image, artifacts)
        │
        ▼
   [ Plaso / log2timeline ]  ──►  super timeline: millions of low-level events
        │
        ▼
   [ Triage + anomaly detection (ML) ]  ──►  tame the scale: the relevant parts of the deluge
        │
        ▼
   [ Local LLM, with grounding (RAG) ]  ──►  reconstruct high-level events + summarize + answer
        │  (every output TRACEABLE to source events)
        ▼
   [ Investigator dashboard ]  ──►  explore, ask, and verify against evidence
```

The core idea, and what drives the value of this project, is what you already saw in the focus note: **the AI tames the scale, but does not draw conclusions, and everything it produces is traceable to the evidence**. The bottleneck in forensic analysis is the investigator manually reviewing an unmanageable volume of data. The AI targets exactly this bottleneck, triaging the deluge and reconstructing the story, but always as a support tool whose outputs the investigator can (and must) verify against the source. You are not automating forensic judgment; you are automating the tedious, high-scale tasks that make manual review unfeasible, keeping human judgment where it belongs: with the investigator.

---

## 2. The Use Case and Why It Is Impressive

The project addresses a real-world problem in forensic analysis and combines several factors that make it powerful.

**Applies modern AI to a real-world, large-scale problem.** Forensic investigations generate overwhelming data volumes (millions of events per disk), and manual triage is the major bottleneck. A system that uses AI to tame this scale (triaging, highlighting anomalies, reconstructing events) solves a genuine and deeply felt pain point in DFIR. Furthermore, this is a highly active research area right now, with cutting-edge work in 2026 focusing on LLMs and RAG for forensic analysis.

**Demonstrates an uncommon maturity: forensic soundness.** This is a high-consequence domain (findings can go to court), and successfully navigating its demands (evidence integrity, the assistive role of AI, the traceability of each finding, humility regarding LLM hallucinations) demonstrates a level of maturity that distinguishes a true professional. Very few portfolio projects tackle these constraints, and executing them well conveys immense capability and reliability.

**It is a synthesis of your skills.** This project integrates what you already master: it repurposes your **RAG** project (the LLM with grounding on evidence, retrieval), the **anomaly detection** from your honeypot (triaging the event deluge), and the **responsible AI framework** from your medical and misinformation projects (assistive AI, traceability). It demonstrates your ability to combine your skills into a larger system to tackle a new problem.

**It has a memorable narrative.** *"I built an AI assistant that helps a forensic investigator find the needle among millions of events, without ever letting the AI draw conclusions that cannot be backed by evidence"* is a pitch that combines utility, technical sophistication, and maturity. Being able to demonstrate the system reconstructing an incident's timeline, with every finding traceable to the source evidence, is highly compelling.

Additionally, this project connects naturally with your previous ones: it shares the **cybersecurity** theme of the honeypot (analyzing incident activity), repurposes the **grounding and local models** of your RAG project, and applies the **responsible AI framework** from the medical and misinformation projects. It is the piece that proves, in a demanding new domain, that you can integrate your skills with sound judgment.

---

## 3. Technology Stack (Updated to 2026)

| Layer | Tool | Why This One |
|------|-------------|--------------|
| Language | Python 3.11+ with **uv** | Standard; repurposes your foundation |
| Forensic Environment | **SIFT Workstation** | The SANS VM with all forensic tools pre-installed |
| Timeline Generation | **Plaso / log2timeline** | The standard for the "super timeline" (unifies all artifacts) |
| Disk Forensics | **Autopsy / The Sleuth Kit** | The standard for analyzing disk images while preserving evidence |
| Memory Forensics | **Volatility 3** | The standard for analyzing memory dumps (optional/extension) |
| Triage and Anomalies | **scikit-learn** | Anomaly detection and prioritization (repurposed from the honeypot) |
| LLM and Grounding | **Ollama** (local LLM) + RAG | Reconstruction, summarization, and Q&A with grounding (repurposed from your RAG) |
| Visualization | **Streamlit** (or Timesketch) | The investigator dashboard |
| Quality Assurance | pytest, ruff, Docker | Reuses your best practices |

A few notes on these choices, as the "why" demonstrates technical judgment:

**SIFT Workstation as the environment.** This is the SANS virtual machine, pre-installed and pre-configured with over 500 forensic tools (Autopsy, Volatility, The Sleuth Kit, Plaso, RegRipper...). Using it spares you the pain of installing and configuring each tool individually, providing a complete, standard forensic environment from day one.

**Plaso / log2timeline as the backbone.** Timeline analysis is the core of DFIR, and Plaso is the industry standard tool: it parses a wide variety of artifacts (filesystem metadata, event logs, browser history, registry entries, application logs) and consolidates them into a single, chronological timeline. It converts a raw disk image into an analyzable sequence of events and will serve as the input for your AI system. (The typical workflow: `log2timeline` generates a `.plaso` storage file, and `psort` converts it into a CSV timeline.)

**Local LLM with grounding (repurposing your RAG).** The most sophisticated AI component directly builds on what you learned in your RAG project: a local LLM (via Ollama, chosen for cost and privacy—the latter being critical with sensitive evidence) that reasons over the timeline using strict **grounding** practices (no fabricating, basing everything on retrieved evidence). This is not an arbitrary idea forced into the domain; it is exactly the approach that cutting-edge 2026 research is applying to forensic analysis (combining forensic tools, retrieval, and local LLMs). Your RAG project has prepared you well for this.

**Anomaly detection (repurposing your honeypot).** To triage the event deluge and highlight what is relevant, you will reuse the anomaly detection techniques from your honeypot project (using scikit-learn), applying them here to the forensic timeline. This is another demonstration of how your skills build upon one another.

---

## 4. Repository Architecture

```
ai-forensics/
├── src/
│   ├── ingest/             # generate and load the timeline (Plaso)
│   ├── triage/             # anomaly detection and prioritization (ML)
│   ├── reconstruct/        # high-level event reconstruction (LLM)
│   ├── assistant/          # question assistant (RAG over evidence)
│   ├── evaluation/         # evaluation (is it correct? is it grounded?)
│   └── config.py
├── dashboard/app.py        # the investigator dashboard
├── evidence/               # evidence (gitignored — sensitive data)
├── docs/
│   ├── context.md          # DFIR, timeline analysis, core approach
│   └── forensic_soundness.md   # evidence integrity, grounding, assistive framework
├── notebooks/
├── tests/
├── pyproject.toml
├── Makefile
└── README.md
```

The repository structure follows the project workflow: `ingest` (timeline generation), `triage` (ML-driven anomaly detection), `reconstruct` (the LLM reconstructing events), `assistant` (RAG-based Q&A), and `evaluation`, with the `dashboard` as a separate component. Two details stand out. The `evidence/` folder must always be added to `.gitignore`, as forensic evidence is highly sensitive and should never be uploaded to a public repository. Additionally, `docs/forensic_soundness.md` is dedicated to evidence integrity, grounding, and the assistive framework; documenting the forensic soundness of the project is a sign of maturity, showing that you understand the gravity of the domain.

---

## 5. Phase-by-Phase Roadmap

Each phase includes an objective, tasks, deliverables, and a definition of "done".

---

### 🔹 Phase 0 — Setup, Foundations, and Forensic Rigor (Week 1, First Days)

**Objective:** Get the environment ready, understand DFIR and timeline analysis, and establish the forensic soundness framework.

**Tasks:**
- [ ] Create the repo structure, set up the environment using `uv`, configure code quality tools and a Makefile (reusing your previous projects), and prepare the SIFT Workstation.
- [ ] Understand digital forensics and incident response (DFIR), timeline analysis, and the applications of AI in this field.
- [ ] Document the core approach (AI tames the scale but does not draw conclusions; everything is traceable to evidence) and its rationale.
- [ ] Document the forensic soundness framework (evidence integrity, assistive AI, grounding, humility in the face of hallucinations).

**Deliverable:** Environment ready + solid understanding of the problem + documented core approach and forensic soundness framework.  
**Definition of Done:** You understand forensic timeline analysis, why the AI must be assistive and traceable, and you have established a clear framework for forensic soundness.

---

### 🔹 Phase 1 — Evidence and Timeline (Plaso) (Weeks 1-3)

**Objective:** Obtain the evidence and generate the timeline, establishing the system's foundation.

**Tasks:**
- [ ] Acquire a realistic forensic dataset (a disk image from an incident, from publicly available DFIR practice sets).
- [ ] Generate the super timeline using Plaso / log2timeline, strictly preserving evidence integrity.
- [ ] Study and understand the timeline: low-level events, artifacts, and scale (the overwhelming volume of data).

**Deliverable:** A generated and thoroughly understood forensic timeline.  
**Definition of Done:** You have converted incident evidence into a structured timeline and understand both its layout and its sheer scale.

---

### 🔹 Phase 2 — Triage and Anomaly Detection (Weeks 3-4)

**Objective:** Tame the scale by highlighting what is relevant within the deluge of data.

**Tasks:**
- [ ] Apply anomaly detection to the timeline to identify unusual events (reusing concepts from your honeypot).
- [ ] Priorize and triage the events, separating potentially relevant clues from background noise.
- [ ] Present the results in a format that allows the investigator to verify each flagged event against the source.

**Deliverable:** A triage module that highlights relevant events.  
**Definition of Done:** The system condenses the deluge of events into a manageable set of clues, with each clue fully traceable to the evidence.

---

### 🔹 Phase 3 — High-Level Event Reconstruction (The LLM) (Weeks 4-6)

**Objective:** Reconstruct high-level events from low-level activities. **This is the most distinctive AI component of the project.**

**Tasks:**
- [ ] Use a local LLM (Ollama, leveraging your RAG work) to reconstruct high-level events from the timeline.
- [ ] Generate a clear, understandable summary of what occurred, utilizing strict grounding (no fabricating facts).
- [ ] Ensure every reconstructed event is traceable back to the specific low-level events that support it.

**Deliverable:** A module that reconstructs and summarizes the incident's timeline of events with full traceability.  
**Definition of Done:** The system translates the deluge of low-level events into a clear, grounded, and well-supported high-level narrative.

---

### 🔹 Phase 4 — Question Assistant (RAG over Evidence) (Weeks 6-7)

**Objective:** Answer the investigator's questions, ensuring answers are grounded in the evidence.

**Tasks:**
- [ ] Build an assistant that answers questions about the evidence by retrieving relevant events (RAG over the timeline).
- [ ] Base every response on the retrieved events and cite them specifically for full traceability.
- [ ] Maintain the assistive framework: the assistant aids the investigation; it does not issue verdicts.

**Deliverable:** A Q&A assistant for the evidence that features strict grounding.  
**Definition of Done:** The investigator can query the system about the incident and receive answers that are well-founded and traceable back to the evidence.

---

### 🔹 Phase 5 — Rigorous Evaluation (Weeks 7-8)

**Objective:** Honestly evaluate the system, focusing specifically on its fidelity to the evidence.

**Tasks:**
- [ ] Evaluate the triage module (does it highlight truly relevant events?) against a ground truth dataset.
- [ ] Evaluate LLM outputs (Are they correct? Are they grounded? Does it hallucinate?) using a rigorous methodology.
- [ ] Analyze failures, paying close attention to hallucinations (which are unacceptable in a forensics context).

**Deliverable:** A rigorous and honest evaluation of the system.  
**Definition of Done:** You know exactly how well the system triages and reconstructs, you have verified that it does not hallucinate, and you understand its limitations.

---

### 🔹 Phase 6 — Investigator Dashboard (Weeks 8-9)

**Objective:** Present the system in a highly usable and traceable manner.

**Tasks:**
- [ ] Build the dashboard where the investigator can explore the timeline, view the triage and reconstruction, and query the assistant.
- [ ] Make traceability central: every AI output must be verifiable against the source evidence directly from the interface.
- [ ] Respect the assistive framework in the user interface design (AI suggests, human decides).

**Deliverable:** An interactive, traceable dashboard for forensic analysis.  
**Definition of Done:** An investigator can use the system and easily verify any AI-generated finding against the source evidence.

---

### 🔹 Phase 7 — Packaging and Presentation (Weeks 9-10)

**Objective:** Effectively showcase the value of the system.

**Tasks:**
- [ ] Containerize the system (reusing your existing patterns).
- [ ] Write the definitive README: covering the problem, core approach, forensic soundness, design decisions, and results.
- [ ] Record a video demonstrating the system reconstructing an incident, highlighting traceability back to the evidence.
- [ ] Write an article/post about the project and what you learned regarding AI in digital forensics.

**Deliverable:** Containerized system + comprehensive README + demo video + blog post.  
**Definition of Done:** A visitor can understand the project in two minutes, grasp its technical depth (DFIR + AI + forensic soundness), and see how it works.

---

## 6. The Ideal README (Brief Checklist)

The README should include: an engaging title and description; the problem and why it matters; an architecture diagram; a GIF or video of the system reconstructing an incident; the tech stack with badges; installation and usage instructions; a **forensic soundness** section (evidence integrity, assistive AI, traceability, and humility regarding hallucinations); a **design decisions** section (why AI tames the scale instead of drawing conclusions, why Plaso, why grounding); and the **results** (how well it triages and reconstructs, along with verification that it does not fabricate facts). (This section can be developed in greater detail later.)

---

## 7. Common Pitfalls to Avoid

| Pitfall | Why It Is Bad | What to Do Instead |
|-------|-----------------|-----------|
| Letting the AI "draw conclusions" | Unacceptable in forensics; hallucinations are catastrophic | Assistive AI; the investigator decides |
| Outputs untraceable to evidence | A non-verifiable finding is useless in forensics | Grounding and full traceability of everything |
| Working on original evidence | Violates evidence integrity | Work on copies; verify using hashes |
| Ignoring scale | A system that doesn't tame data volume doesn't solve the actual problem | Triage and high-level reconstruction |
| Failing to evaluate hallucinations | In forensics, fabricating a fact is the worst possible error | Evaluate fidelity to the evidence with rigor |
| Exaggerating capabilities | AI in forensics has serious limitations | Maintain honesty regarding its assistive role and limits |

---

## 8. Stretch Goals (Going Further)

- **Memory Forensics:** Incorporate memory dump analysis with Volatility 3 (processes, network connections) to expand the sources of evidence.
- **YARA Detection and MITRE ATT&CK Mapping:** Add signature scanning with YARA and map findings to MITRE ATT&CK techniques (similar to your honeypot project).
- **Multi-Source Correlation:** Correlate evidence from multiple sources (disk, memory, network) into a unified view of the incident.
- **Assisted Forensic Report Generation:** Assist in drafting forensic report templates, ensuring every assertion remains traceable to the source evidence.
- **Standardized Dataset Evaluation:** Use the standardized methodologies and datasets being developed by the community to evaluate LLMs in digital forensics.
- **Anti-Forensics Detection:** Detect indicators of anti-forensic techniques (log wiping, timestamp manipulation/timestomping).

---

## 9. Learning Resources

- **Plaso / log2timeline and SIFT Workstation Documentation:** For timeline analysis, which is the heart of this project.
- **DFIR and Timeline Analysis Resources:** SANS materials and DFIR guides to understand the forensic process and timeline analysis.
- **Autopsy / The Sleuth Kit and Volatility Documentation:** For disk and memory forensics.
- **Literature on LLMs and RAG in Forensics:** Cutting-edge research from 2026 (including surveys on LLMs in digital forensics and systems combining forensic tools with local LLMs) provides the current state of the art, emphasizing the critical importance of validation and traceability.
- **Public Forensic Datasets:** Incident disk images for practice (from DFIR repositories and forensic challenges).
- **Concepts to Master:** Timeline analysis and low/high-level events; evidence integrity and chain of custody; retrieval and grounding (from your RAG work); anomaly detection (from your honeypot); and why AI in forensics must be assistive, traceable, and humble.

---

## 10. Summary of Milestones

| Week | Milestone | Status |
|--------|------|--------|
| 1 | Environment + Foundations + Forensic Rigor | ⬜ |
| 1-3 | Evidence and Timeline (Plaso) | ⬜ |
| 3-4 | Triage and Anomaly Detection | ⬜ |
| 4-6 | High-Level Event Reconstruction (LLM) | ⬜ |
| 6-7 | Question Assistant (RAG) | ⬜ |
| 7-8 | Rigorous Evaluation | ⬜ |
| 8-9 | Investigator Dashboard | ⬜ |
| 9-10 | Packaging, README, Video, Post | ⬜ |

---

**The ultimate goal:** When someone looks at this project, they should think, *"this person applies modern AI to a real-world forensic problem, and they do so with the rigor that the domain demands."* This is not a system intended to replace the investigator: it is an assistant that tames scale (millones of events), reconstructs the narrative of an incident, and answers questions, while always grounding everything in evidence and leaving final judgment to the human. It is the type of project that demonstrates not only technical capability, but sound judgment.
