<div align="center">

# 🧠 LLM-RESEARCHES

**A personal, evolving research log on Large Language Models (LLMs)**
*Guided study · Paper analysis · Original research proposals*

![Status](https://img.shields.io/badge/status-active-brightgreen)
![Topic](https://img.shields.io/badge/focus-Structured%20Reasoning-blue)
![Mentor](https://img.shields.io/badge/mentored%20by-Dinesh%20Sir-orange)

</div>

---

## 📓 About This Repo

I started with zero background in LLMs. This repository documents my 
research path — the papers I study, the tasks assigned by my mentor, and 
the original research direction I'm building toward.

Every entry follows the same structure: **Objective → Methodology → 
Results → Conclusion → Personal Reflection**, so the reasoning behind each 
step is traceable, not just the conclusions.

---

## 🎯 Current Research Focus

> **Structured Reasoning to Reduce Hallucination**
> Extending existing knowledge-graph fine-tuning approaches with
> hallucination-specific datasets and a fact-verification objective —
> to directly measure and reduce hallucination rate, not just task accuracy.

---

## 🗺️ Research Roadmap

| Day | Topic | Core Idea |
|---|---|---|
| **Day 0** | Background | How this research started — mentor-guided introduction to LLMs and hallucination |
| **Day 1** | LLM Hallucination & Structured Knowledge Integration | Injecting Knowledge Graphs + self-verification loops to fix hallucination at the architecture level |
| **Day 2** | KG-Adapter (PEFT) | Learning KG knowledge *inside* the model via lightweight adapters, instead of retrieving it externally |
| **Day 3** | Structured Reasoning | Extending KG-infused fine-tuning with hallucination-focused datasets and a verification loss term |
| **Day 4** | Benchmark Datasets | Understanding the 9 datasets behind this research — construction, purpose, and source |

📄 **Full write-ups:** [`DOCUMENTATION.md`](DOCUMENTATION.md)

---

## 🧩 Topics Covered

<table>
<tr><td width="50%" valign="top">

**Core Concepts**
- LLM Hallucination — root causes
- Knowledge Graphs & Neuro-Symbolic AI
- Self-Verification / Closed-Loop Correction
- Graph Neural Networks (GNN / R-GCN)

</td><td width="50%" valign="top">

**Frameworks & Methods**
- Meta-Knowledge Guided (MKG) Framework
- KG-Adapter & Parameter-Efficient Fine-Tuning
- Multi-Source Retrieval (BGE, ColBERT, BM25)
- Bayesian Trust Scoring

</td></tr>
<tr><td width="50%" valign="top">

**Evaluation & Datasets**
- TruthfulQA, FEVER, HaluEval
- Multi-Hop Benchmarks: HotpotQA, 2WikiMultiHopQA, ComplexWebQuestions
- KBQA Benchmarks: WebQuestionsSP, GrailQA, MetaQA
- KG–Text Alignment: T-REx

</td><td width="50%" valign="top">

**My Research Direction**
- Structured Reasoning for hallucination reduction
- Fact-verification loss extension
- Hallucination-rate as a primary metric

</td></tr>
</table>

---

## 📂 Repository Structure

```
LLM-RESEARCHES/
├── README.md              ← you are here
├── DOCUMENTATION.md        ← full day-by-day research log (Day 0-4)
├── slides/                 ← presentation decks
│   ├── LLM_hallucination.pptx
│   └── kg-adapter.pptx
└── notes/                  ← detailed write-ups & proposals
    └── structured-reasoning.docx
```

---

## 🚀 Start Here

👉 **[Read the full research log →](https://github.com/souvik-077/LLM-RESEARCHES/blob/main/DOCUMENTATION.md)**

---

<div align="center">
<sub>Maintained by <a href="https://github.com/souvik-077">souvik-077</a> · Mentored by Dinesh Sir</sub>
</div>
