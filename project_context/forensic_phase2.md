# Phase 2 — Triage and Anomaly Detection

> In this phase, you make your first attack on the problem you confronted in Phase 1: scale. You have millions of events, impossible to handle manually, and the goal now is to tame them, highlighting what is relevant and setting aside the massive background noise. The guiding idea of this phase stems from a simple intuition in forensics: investigating is, to a large extent, *looking for the unusual* (the anomalous login, the strange process, the out-of-place file), and for this, anomaly detection is the natural tool—one you already master from your honeypot project. But there are two nuances running through this phase that demonstrate maturity: first, that triage **prioritizes and highlights, it does not dictate** (an anomaly is not the same as something relevant), and second, that every event you highlight must remain **traceable** to the evidence, in keeping with the forensic rigor you established.

**Phase objective:** Tame the scale, highlighting what is relevant from the deluge.  
**Duration:** Weeks 3–4.  
**Upon completion, you will have:** A triage module that, starting from the timeline, highlights potentially relevant events and separates them from the background noise, reducing the deluge to a manageable set of clues, each traceable to the evidence.

---

## The Big Picture

The workflow of this phase converts the deluge into a set of clues:

```
   Timeline (millions of events)
        │
        ▼
   [ 1-2. Convert events into features ]  ──►  TF-IDF of the description + temporal features
        │
        ▼
   [ 3. Detect anomalies and triage ]  ──►  IsolationForest + clustering: the unusual
        │
        ▼
   [ 4. Honest and traceable triage ]  ──►  prioritize (do not dictate), with a link to the source
        │
        ▼
   [ A manageable set of clues, each verifiable ]
```

Add the dependency (the same one you used in the honeypot):

```bash
uv add scikit-learn
```

---

## Step 1 — The Intuition: Looking for the Unusual

Before writing code, it is worth understanding why anomaly detection fits so well here. A forensic investigator's job largely consists of finding what is out of the ordinary: most of the millions of events in the timeline are routine and benign activity (the system operating normally), and the interesting parts (the footprints of the incident) are usually the *unusual* within that sea of normalcy. This is exactly the situation that anomaly detection is designed for, and it is the same intuition you applied in your honeypot, where most attacks were routine and the value lay in detecting the anomalous and grouping the similar.

Therefore, you can profitably reuse the techniques from your honeypot, transferring them to a new domain: detecting statistically unusual events and grouping similar events so that rare clusters stand out. The idea is the same; only the material to which it is applied changes. This reuse is not just efficient: it demonstrates that your skills transfer across problems.

---

## Step 2 — Converting Events into Features

To apply anomaly detection, you first convert the timeline events into numerical features that the model can process. A good portion of the phase's critical thinking resides here, because the features you choose determine what "unusual" means. You reuse the idea from the honeypot (TF-IDF over text) and enrich it with features specific to the timeline. Create `src/triage/features.py`:

```python
import pandas as pd
from sklearn.feature_extraction.text import TfidfVectorizer


def build_features(df: pd.DataFrame):
    # Text features: TF-IDF over the event description (reused from the honeypot)
    tfidf = TfidfVectorizer(max_features=500)
    X_text = tfidf.fit_transform(df["desc"].fillna(""))

    # Temporal features: hour of the day (nighttime activity can be suspicious)
    momento = pd.to_datetime(df["date"] + " " + df["time"], errors="coerce")
    df["hour"] = momento.dt.hour

    # (combine X_text with temporal and categorical features, e.g., artifact type)
    return X_text, df
```

The choice of features demonstrates that you understand the domain. **TF-IDF on the event description** (`desc`) captures which events have unusual textual content, just as it captured unusual commands in the honeypot. **Temporal** features (the hour of the day) capture suspicious patterns in time (activity at odd hours, unusual bursts). And the **artifact type** (`source`) situates the event. Combining these signals is what allows anomaly detection to find what is truly unusual, not just rare in a single dimension.

---

## Step 3 — Detecting Anomalies and Triage

With the features ready, you apply anomaly detection and clustering, reusing the algorithms from the honeypot. Create `src/triage/detect.py`:

```python
from sklearn.cluster import DBSCAN
from sklearn.ensemble import IsolationForest


def triage(X_features, df):
    # Anomalies: statistically unusual events (reused from the honeypot)
    iso = IsolationForest(contamination=0.01, random_state=42)
    df["es_anomalo"] = iso.fit_predict(X_features) == -1   # True if anomalous

    # Group similar events: small, rare clusters can be interesting
    df["cluster"] = DBSCAN(eps=0.5, min_samples=5).fit_predict(X_features)

    return df
```

The combination of signals is what makes the triage rich, just as it did in the honeypot. **IsolationForest** flags outlier events (`contamination` controls what proportion is considered anomalous). **DBSCAN** groups similar events, allowing you to spot rare groups (a small cluster that is distinct from the rest often deserves a look). You can go further and combine more signals (temporal bursts of activity, events touching sensitive areas of the system) into a priority score, to sort events from most to least suspicious. The result is what you were looking for: a way to reduce the deluge to a manageable set of prioritized clues.

---

## Step 4 — Honest and Traceable Triage

Here come the two nuances that demonstrate maturity, representing the difference between serious triage and naive triage. 

First: **an anomaly is not the same as something relevant**. Anomaly detection is unsupervised and imperfect: it highlights what is statistically unusual, which is a *useful signal*, but not an absolute truth. Some anomalies are benign (a rare but innocent event), and some relevant events are not anomalous (a normal-looking login that is actually the attacker). This is why triage is a tool for **prioritization**, not a verdict: it shrinks the haystack and highlights clues for the investigator to examine, but it does not decide what is relevant. This links directly to the project's approach (the AI highlights clues, it does not draw conclusions) and to the honesty regarding unsupervised methods that you already applied in the honeypot.

The second nuance is **traceability**, which embodies forensic rigor. Triage does not *replace* the events: it prioritizes and highlights them, always keeping the link to their origin in the timeline (and, through it, to the evidence). Every clue highlighted by the system must be verifiable against the source:

```python
# Every triaged event retains its original index in the timeline, allowing it to be verified.
# We do not replace the events; we prioritize them without losing provenance.
pistas = df[df["es_anomalo"]].copy()   # keeps the original index and all original fields
```

Maintaining the provenance of each event is not a technical detail: it is what makes a triaged clue forensically defensible, because the investigator can always go from the clue to the original event and verify it. A triage system that loses the link to the source would be useless in this domain.

---

## Step 5 — Tests

Verify the triage logic with synthetic data. Complete `tests/test_triage.py`:

```python
import numpy as np
import pandas as pd

from src.triage.detect import triage


def test_el_triaje_marca_anomalias_y_conserva_la_procedencia():
    # Synthetic data: many normal events and a few outliers
    rng = np.random.default_rng(0)
    X = np.vstack([rng.normal(0, 1, (200, 5)), rng.normal(8, 1, (3, 5))])
    df = pd.DataFrame({"desc": [f"evento {i}" for i in range(203)]})

    resultado = triage(X, df)
    assert "es_anomalo" in resultado.columns        # marks anomalies
    assert resultado["es_anomalo"].sum() > 0         # detects some outliers
    assert len(resultado) == len(df)                 # retains all events (provenance)
    assert list(resultado.index) == list(df.index)   # maintains the original index (traceability)
```

The test verifies that the triage process flags anomalies, detects outliers, and, crucially, retains all events with their original index (traceability). This last point is what guarantees that no clue loses its link to the source. Run it with `make test`.

---

## Verification: The Definition of "Done"

This phase is complete when the following conditions are met:

- [ ] You have converted the timeline events into features (TF-IDF of the description + temporal/categorical).
- [ ] You have applied anomaly detection and clustering (reusing your honeypot) to highlight the unusual.
- [ ] You have prioritized the events, reducing the deluge to a manageable set of clues.
- [ ] You understand that triage prioritizes, it does not dictate (an anomaly does not equal relevance).
- [ ] Each clue retains its link to the source (traceability) so it can be verified.
- [ ] **The key test:** The system reduces the deluge of events to a manageable set of clues, each traceable to the evidence.

The key test has two parts reflecting the two nuances of this phase: that the system *reduces* the deluge to something manageable (taming the scale), and that each clue is *traceable* (verifiable against the source). The fact that both matter is what characterizes serious forensic triage: it is not enough to highlight the unusual; it must be done with the understanding that this is a prioritization, not a verdict, while maintaining provenance so that the investigator can validate it. That combination of utility and rigor is what makes the difference.

---

## Deliverables and What Comes Next

By closing Phase 2, you have made your first attack on scale: a triage module that reduces millions of events to a manageable set of prioritized clues, highlighting the unusual and setting aside the noise. You have reused the techniques from your honeypot on a new domain, and you have done so with the maturity to understand that triage prioritizes (does not dictate) and with the rigor of keeping each clue traceable to the evidence. You have begun to tame the deluge.

The next step, **Phase 3**, attacks the other side of the problem and is the most distinctive AI part: **high-level event reconstruction** with an LLM. Triage highlights relevant low-level events, but the investigator wants to understand high-level facts ("a USB was connected", "this program was executed"). You will use a local LLM (reusing your RAG) to reconstruct those high-level facts from the low-level deluge and summarize what happened, with grounding discipline (no hallucinating) and ensuring that each reconstructed fact is traceable to the underlying events supporting it. You have moved from "highlighting relevant clues" to being on the verge of "reconstructing the history of the incident from them."
