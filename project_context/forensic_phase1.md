# Phase 1 — Evidence and the Timeline (Plaso)

> In this phase, you build the foundation of the entire system: the timeline. Everything that comes after (triage, reconstruction, the Q&A assistant) works on top of the timeline, so generating it correctly and understanding it thoroughly is the essential step. The guiding idea of this phase is twofold. On one hand, converting the evidence of an incident into an analyzable chronological sequence of events, using the industry-standard DFIR tool (Plaso), and with the forensic rigor you established in Phase 0 (working on a copy, verifying with a hash). On the other hand, and almost as importantly, *confronting scale*: when you see with your own eyes that the timeline contains millions of low-level events, you will gain a visceral understanding of the problem that gives purpose to the entire project, and why AI is needed to tame it.

**Phase objective:** obtain the evidence and generate the timeline, the foundation of the system.  
**Duration:** weeks 1-3.  
**Upon completion, you will have:** the evidence of an incident converted into a super timeline using Plaso, loaded and understood in both its structure and, above all, its scale.

---

## The Big Picture

The workflow of this phase converts the evidence into a timeline and builds an understanding of it:

```
   [ 1. Obtain the evidence ]  ──►  incident disk image (copy + hash)
        │
        ▼
   [ 2. Generate the timeline (Plaso) ]  ──►  log2timeline → .plaso → psort → timeline.csv
        │
        ▼
   [ 3. Understand the structure ]  ──►  the 17 l2tcsv fields, the artifacts
        │
        ▼
   [ 4. Confront scale ]  ──►  millions of low-level events: the problem, live
```

---

## Step 1 — Obtain the Evidence

You start by obtaining a realistic forensic dataset: a disk image from an incident. Fortunately, there are excellent public images designed specifically for practicing DFIR that replicate complete incidents. Some highly recommended ones include: **The Stolen Szechuan Sauce** (from the DFIR Madness portal), a complete and well-documented intrusion scenario; the **NIST** cases (CFReDS, such as the data leakage case); and the **Digital Corpora** repository. Any of these will provide you with realistic evidence to work with.

And here, you apply the forensic rigor you established in Phase 0 for the first time in practice. **Always work on a copy**, never on the original, and **verify integrity with a hash** to demonstrate that the evidence has not been altered (the foundation of the chain of custody):

```bash
# Record the evidence hash (chain of custody)
sha256sum incident.dd > incident.dd.sha256
# Always work on a copy, never on the original
```

Even though public practice images are already copies, establishing this discipline from the very beginning (copying and hashing) demonstrates that you understand evidence handling, which is part of what distinguishes serious forensic work. It is a habit worth internalizing.

---

## Step 2 — Generate the Super Timeline

With the evidence ready, you generate the super timeline using Plaso, the industry-standard tool. The workflow consists of two steps, and it is important to understand what each does. The first, `log2timeline.py`, parses the evidence (analyzing all artifacts: file system, registry, event logs, browser history...) and dumps them into an intermediate storage file, the `.plaso` file. The second, `psort.py`, takes that file and converts it (by sorting and filtering) into a readable timeline in CSV format:

```bash
# 1. Parse the evidence → .plaso storage file (all artifacts)
log2timeline.py --storage-file case.plaso /evidence/incident.dd

# 2. Convert to a CSV timeline in l2tcsv format
psort.py --output-time-zone UTC -o l2tcsv -w timeline.csv case.plaso
```

A couple of details that show you know the tool. The **l2tcsv** format (the classic, well-understood format for analysis) **is no longer the default format** in current versions of Plaso, so you must specify it with `-o l2tcsv`. Additionally, `psort` allows you to **filter** (for example, by date range, using a condition on `date`) and choose the output time zone, which is very useful for narrowing down the scope in large datasets. Be patient: parsing a full disk image generates a massive number of events and can take a while. The result, `timeline.csv`, is the raw material for your entire system.

---

## Step 3 — Understand the Timeline Structure

Before analyzing anything with AI, make sure you understand what is inside the timeline by loading it into your Python project. The l2tcsv format contains **17 fields**, and knowing them is key to understanding what a forensic event actually *is*:

```python
import pandas as pd

df = pd.read_csv("timeline.csv")
print(f"Events in the timeline: {len(df):,}")
print(df.columns.tolist())
# date, time, timezone, MACB, source, sourcetype, type, user, host,
# short, desc, version, filename, inode, notes, format, extra
print(df["source"].value_counts())   # what types of artifacts there are, and how many of each
```

It is worth understanding the key fields. Each row is an event with a date and time, and a **MACB** field indicating the timestamp type (Modified, Accessed, Changed, Birth: when something was modified, accessed, changed, or created). The **source** field categorizes the type of artifact (filesystem, registry, log, web history...), while **sourcetype** provides finer detail. The **desc** field contains the event description, and **filename** points to the involved file. Examining the distribution of `source` (how many events exist for each artifact type) gives you an initial overview of the evidence composition: how much of it consists of file metadata, how much is registry data, and how much represents logs. Understanding this structure is what will later allow you to triage and reason about events with proper criteria.

---

## Step 4 — Confronting Scale

This is conceptually the most important part of this phase, and it is worth pausing here, because it makes the problem that defines the entire project tangible. Look at how many events your timeline has:

```python
print(f"Total events: {len(df):,}")
print(df[["date", "time", "source", "desc"]].head(20))   # some low-level events
```

You will most likely see **hundreds of thousands or millions** of events. And here comes the double revelation. First, the **scale**: no one can manually read millions of rows. Seeing that number with your own eyes helps you understand, in a visceral way, the bottleneck of forensic analysis, and why AI—when used correctly—is so valuable: not to replace the investigator, but to tame this overwhelming volume. Second, the **low-level** nature of the events: if you look at a few rows, you will see entries like "file X was modified" or "registry key Y was updated," which are individually uninformative. An investigator does not want a thousand low-level events; they want to know "a USB drive was plugged in" or "this malicious program was executed"—high-level events that must be reconstructed from this deluge of data.

These two challenges (scale and low-level data) are exactly the problems your system will address in the upcoming phases: triage will tame the scale (Phase 2), and reconstruction will turn low-level data into high-level information (Phase 3). Confronting this problem firsthand by seeing the millions of events is what gives meaning and motivation to everything that follows. It reinforces, with the evidence right in front of you, the approach you established in Phase 0.

---

## Step 5 — Verify

Since this phase produces data (the timeline) rather than complex code, verification is mainly a check to ensure the timeline has been successfully generated and loads properly. Here is a simple check:

```python
def verify_timeline(path="timeline.csv"):
    df = pd.read_csv(path)
    assert len(df) > 0, "The timeline is empty"
    assert "source" in df.columns and "desc" in df.columns, "Expected fields are missing"
    print(f"Timeline OK: {len(df):,} events, {df['source'].nunique()} artifact types")
    return df
```

This check confirms that the timeline exists, contains events, and includes the expected fields, which is exactly what you need to move forward. It verifies that the foundation is solidly laid before you begin building on top of it.

---

## Verification: The Definition of "Done"

This phase is complete when the following criteria are met:

- [ ] You have obtained a realistic incident disk image (from public practice datasets).
- [ ] You have applied forensic rigor: you work on a copy and have verified its integrity with a hash.
- [ ] You have generated the super timeline using Plaso (log2timeline → psort → timeline.csv).
- [ ] You understand the structure of the timeline (the l2tcsv fields, artifact types).
- [ ] You have confronted the scale (millions of events) and the low-level nature of the events.
- [ ] **The key test:** you have the evidence of an incident converted into a timeline, and you understand its structure and its scale.

The key test is having the foundation laid and understood: if you have generated the timeline and understand both its structure (what each event is) and its scale (the overwhelming volume), you have the foundation on which everything else will be built. Just as importantly, you have understood firsthand the problem your system is designed to solve. It is not enough to just have the file; you must understand what it contains and why its scale makes AI necessary.

---

## Deliverables and What's Next

By closing Phase 1, you have the foundation of the project: the evidence of an incident converted into a super timeline using Plaso, generated with forensic rigor, and understood in its structure and scale. You have handled industry-standard DFIR tools, applied evidence integrity discipline, and, above all, confronted firsthand the problem (millions of low-level events) that gives meaning to everything that follows. You have the raw material ready for the AI to work with.

The next step, **Phase 2**, begins to tame that scale: **triage and anomaly detection**. You will apply anomaly detection techniques (reusing your honeypot project) to the timeline to identify unusual events and separate potentially relevant data from the massive background noise, reducing the deluge to a manageable set of clues, each traceable back to the evidence. This is the first attack on the bottleneck of scale that you have just witnessed with your own eyes. You will go from "I have evidence converted into an overwhelming timeline" to being on the verge of "I have a system that highlights what is relevant within that deluge."
