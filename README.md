# LLM-RESEARCHES

A personal, evolving log of my journey researching Large Language Models (LLMs) — guided study, paper summaries, and day-by-day research notes.

## 📓 About this repo

I started with zero background in LLMs. This repo tr<div align="center">

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
Results → Conclusion → My Own Analysis**, so the reasoning behind each step 
is traceable, not just the conclusions.

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
| **Day 1** | LLM Hallucination & Structured Knowledge Integration | Injecting Knowledge Graphs + self-verification loops to fix hallucination at the architecture level |
| **Day 2** | KG-Adapter (PEFT) | Learning KG knowledge *inside* the model via lightweight adapters, instead of retrieving it externally |
| **Day 3** | Structured Reasoning | Extending KG-infused fine-tuning with hallucination-focused datasets and a verification loss term |

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

**Evaluation**
- TruthfulQA, HaluEval, FEVER
- Multi-Hop Benchmarks: HotpotQA, ComplexWebQuestions, GrailQA

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
├── DOCUMENTATION.md        ← full day-by-day research log
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
</div>acks what I'm learning, the papers I study, and the research direction I'm building toward, mentored by Dinesh Sir.

Topics covered so far:

* **LLM Hallucination** — architectural root causes: implicit knowledge storage, probabilistic generation, open-loop architecture
* **Knowledge Graphs (KG) & Neuro-Symbolic AI** — fusing neural language understanding with symbolic factual grounding
* **Self-Verification & Closed-Loop Correction** — Generate → Validate → Refine pipelines (NIT Durgapur 4-phase approach)
* **Meta-Knowledge Guided (MKG) Framework** — Meta-Executor, Meta-Assessor, Meta-Insight Processor; dual-memory (STM/LTM) architecture
* **Multi-Source Knowledge Retrieval** — BGE semantic embeddings, ColBERT, BM25, fused via Reciprocal Rank Fusion
* **KG-Adapter & Parameter-Efficient Fine-Tuning (PEFT)** — injecting KG knowledge into frozen LLM layers via lightweight trainable adapters
* **Graph Neural Networks (GNN/R-GCN)** — node-centered and relation-centered knowledge encoding
* **Structured Reasoning** — Knowledge Graph-infused fine-tuning for multi-hop, logically-connected reasoning
* **Hallucination-Focused Evaluation** — TruthfulQA, HaluEval, FEVER-style fact verification
* **Multi-Hop Reasoning Benchmarks** — HotpotQA, ComplexWebQuestions, GrailQA
* **Bayesian Trust Scoring** — handling noisy, outdated, or conflicting KG data

## 🎯 Current Research Focus

**Structured Reasoning to Reduce Hallucination** — extending existing 
knowledge graph fine-tuning approaches with hallucination-specific datasets 
and a fact-verification objective, to directly measure and reduce 
hallucination rate rather than only task accuracy.

## 📂 Structure

* [`DOCUMENTATION.md`](DOCUMENTATION.md) — day-by-day research log
* `slides/` — presentation decks for each research topic
* `notes/` — detailed write-ups and proposals

## 🚀 Follow the journey

Start here: [DOCUMENTATION.md](https://github.com/souvik-077/LLM-RESEARCHES/blob/main/DOCUMENTATION.md)
