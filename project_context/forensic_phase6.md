# Phase 6 — The Investigator's Dashboard

> In this phase, you make the system usable for the investigator, bringing all of its capabilities (exploring the timeline, viewing triage, reading the reconstruction, asking questions) together into a single interface. However, this phase has a design principle that distinguishes it from the dashboards of your other projects, and which is the culmination of all the forensic rigor you have built: **traceability is central, not an add-on**. In forensics, every AI output must be verifiable against the source evidence, so the dashboard is not designed just to *show* results, but so that the investigator can *verify* each one of them: going from any AI claim to the supporting events, and from there to the evidence. The guiding idea behind this phase is to build a tool that the investigator can use and, above all, verify, where verification is as easy and natural as querying. It is what turns all the rigor of the previous phases into something tangible.

**Phase objective:** present the system in a usable and traceable way.
**Duration:** weeks 8-9.
**Upon completion, you will have:** a dashboard where the investigator explores the timeline, views triage and reconstruction, and asks questions, and where every AI output can be verified against the source evidence, presented within a strictly assistive framework.

---

## The Big Picture

The dashboard gathers the system's capabilities around traceability:

```
   ┌──────────────────────────────────────────────────────────┐
   │  🔬 Forensic Analysis Assistant                           │
   │  ⓘ Assistive tool. Verify every output. The AI            │
   │     suggests; the investigator decides.                   │
   ├──────────────────────────────────────────────────────────┤
   │  [ Timeline ] [ Triage ] [ Reconstruction ] [ Questions ] │
   ├──────────────────────────────────────────────────────────┤
   │  Reconstructed event: "a USB was connected" (confidence: high)│
   │   ▸ Evidence (supporting events) → [verify]               │
   └──────────────────────────────────────────────────────────┘
```

You are already familiar with Streamlit from your previous projects (`uv add streamlit` if you are starting from scratch).

---

## Step 1 — Traceability as a Core Design Principle

Before writing any code, it is worth establishing the principle that organizes the entire dashboard, because that is what sets it apart. In your other dashboards, the goal was to *present* results in the clearest way possible. Here, the goal is twofold: to present *and to allow verification*, and verification is not secondary—it is central. The reason for this runs throughout the entire project: in forensics, an AI output that you cannot verify against the evidence is useless (it is not defensible), and an investigator must not blindly trust any output, but verify it. Therefore, the dashboard is designed *around* traceability.

In practice, this means that every AI output (a reconstructed event, an answer, a triage clue) must be accompanied by a clear and easy path to its supporting events, and from there to the evidence. The question guiding each design decision is not just "is this result clear?", but "can the investigator easily verify this result?". Making verification a first-class action, as natural as querying, is the physical realization of all the rigor from the previous phases (grounding, citations, faithfulness) within the tool. Without this traceability, all that rigor would remain in the code; with it, it reaches the investigator.

---

## Step 2 — The Dashboard Views

The dashboard gathers the system's capabilities into views (tabs), each corresponding to one of the previous phases. Create `dashboard/app.py`:

```python
import streamlit as st

st.set_page_config(page_title="Forensic Assistant", page_icon="🔬", layout="wide")

st.title("🔬 Forensic Analysis Assistant")
st.info(
    "An **assistive** tool for the investigation. Every AI output must be verified "
    "against the evidence. The AI suggests; the investigator decides."
)

tab_timeline, tab_triaje, tab_reconstruccion, tab_preguntas = st.tabs(
    ["Timeline", "Triage", "Reconstruction", "Questions"]
)

with tab_timeline:
    st.subheader("The complete timeline")
    # explore the raw timeline with search and filters (by date, user, artifact type)

with tab_triaje:
    st.subheader("Highlighted clues")
    # events prioritized by triage, ordered by relevance
```

The views follow the system's workflow. The **timeline** view allows exploring raw events with search and filters (leveraging the structure you worked on in Phase 1). The **triage** view shows the clues highlighted by triage (Phase 2), prioritized. The **reconstruction** view shows the high-level events and the narrative (Phase 3). And the **questions** view is the Q&A assistant (Phase 4). Each view provides access to a system capability, and all of them, as we will see, integrate verification. The disclaimer at the top, visible from the very beginning, frames the entire tool: it is assistive, every output must be verified, the AI suggests, and the investigator decides.

---

## Step 3 — Integrated Verification

This is where the principle of traceability becomes code, and it is the most distinct feature of the dashboard. Every AI output is *expandible* to its supporting events, allowing the investigator to verify it without leaving the interface. In the reconstruction view, for instance, each high-level event expands to display the low-level events that prove it:

```python
with tab_reconstruccion:
    st.subheader("Reconstructed high-level events")
    st.caption("AI Reconstruction — expand each event to verify it against the evidence.")
    for hecho in reconstruccion:
        with st.expander(f"{hecho['hecho']}  ·  confidence: {hecho['confianza']}"):
            st.write("**Evidence: the low-level events supporting it**")
            eventos_fuente = df[df["id"].isin(hecho["evidencia"])]
            st.dataframe(eventos_fuente[["date", "time", "source", "desc"]])
```

And the same applies to the questions view: every answer is accompanied by the events it cites, expandable for verification:

```python
with tab_preguntas:
    pregunta = st.text_input("Question about the incident")
    if pregunta:
        respuesta, eventos_citados = asistente.responder(pregunta)
        st.write(respuesta)
        with st.expander("Cited evidence — expand to verify response"):
            st.dataframe(df[df["id"].isin(eventos_citados)][["date", "time", "source", "desc"]])
```

Notice what this achieves: the investigator never has to trust an AI output blindly. Faced with a reconstructed event ("a USB was connected") or an answer ("user X logged in at 14:00"), a single click reveals the exact supporting events, which can then be examined and verified. Verification becomes a first-class action, integrated into every output. This is the traceability of grounding and citations from the previous phases made tangible and actionable in the tool. It is what transforms AI outputs—which would otherwise be unverifiable assertions (useless in forensics)—into leads that the investigator can validate.

---

## Step 4 — Assistive Framing in the Presentation

Just like in your disinformation project, how you present AI outputs is a crucial decision, and here it embodies the project's overall approach. There are several guidelines. Present AI outputs for what they are: *reconstructions and responses to be verified*, not established facts. The language and framing must make it clear that this is the AI's interpretation, which the investigator must confirm (hence the labels like "AI reconstruction — expand to verify"). Always display the **confidence level** of each output so the investigator knows how much caution to exercise. And make **verification** as visible and easy as possible, because that communicates, through the interaction itself, that one must verify rather than blindly trust.

As a whole, the interface conveys a message consistent with the entire project: AI is a powerful aid to manage scale and find clues, but judgment and responsibility lie with the investigator, who validates every finding against the evidence. This presentation approach—where verification is central and the AI never makes final judgments—is what turns a potentially risky technology (an AI opining on forensic evidence) into a responsible and truly useful tool.

---

## Step 5 — Testing the Dashboard

Launch the dashboard (`uv run streamlit run dashboard/app.py`) and navigate through it as an investigator would. Explore the timeline with its filters, look at the triage clues, read the reconstructed events and narrative, and ask questions. And, above all, *verify*: expand a reconstructed event and check that you can see its supporting events; ask a question and confirm that the answer comes with its cited evidence, ready to be expanded. Verify that the disclaimer is visible, that outputs are presented as reconstructions to be verified (not facts), that confidence is shown, and that verification is easy in every case.

The guiding question for testing, which is the criteria for this phase, is: **can the investigator use the system and verify each finding against the evidence?** If they can explore, triage, reconstruct, and ask questions, and easily trace every AI output back to the supporting evidence to check it, you have succeeded. And since the entire system works end-to-end, this dashboard will be the star material for the Phase 7 video.

---

## Verification: The "Done" Criteria

The phase is complete when the following conditions are met:

- [ ] The dashboard brings together the system's capabilities (timeline, triage, reconstruction, questions) into a single interface.
- [ ] Traceability is central: each AI output is expandable to show its supporting events.
- [ ] The investigator can easily verify any event or response against the evidence directly from the interface.
- [ ] Outputs are presented within an assistive framework (reconstructions to be verified, along with their confidence level), not as established facts.
- [ ] The disclaimer frames the tool (assistive; the AI suggests, the investigator decides).
- [ ] **The key test:** the investigator can use the system and verify each finding against the evidence.

The key test is having a tool that is both usable *and* verifiable: if the investigator can use the system to explore the incident and, at each step, verify the AI's outputs against the evidence, you have built a true forensic tool—one that combines the power of AI with the rigor required by the domain. Having verification be this central is the hallmark of this project: in forensics, a tool that does not allow for verification is useless, no matter how powerful it is. This marriage of utility and verifiability is the culmination of all the rigor you have built.

---

## Deliverables and Next Steps

By completing Phase 6, you have the project in its usable form: a dashboard where the investigator explores the timeline, views triage and reconstruction, and asks questions, and where every AI output can be verified against the evidence directly from the interface. You have materialized all the rigor of the previous phases (grounding, citations, faithfulness) within the tool, making traceability the central design principle, and you have presented the system with the assistive framework that the domain demands. You have a forensic tool that is both powerful and verifiable.

The next and final step, **Phase 7**, showcases the value of the entire project: **packaging and presentation**. You will containerize the system, write the definitive README (covering the approach, forensic rigor, design decisions, and evaluation results), record a video showing the system reconstruct an incident with traceability to the evidence, and write a post about what you learned. All the work of the eight phases now needs a showcase, and you have excellent material: a forensic assistant that manages scale, reconstructs the story of an incident, and allows every finding to be verified. You have gone from "the investigator can use and verify the system" to being on the verge of "presenting the entire project to clearly demonstrate what I have mastered."
