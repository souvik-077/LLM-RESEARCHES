# LLM-RESEARCHES

A personal, evolving log of my journey researching Large Language Models (LLMs) — guided study, paper summaries, and day-by-day research notes.

## 📓 About this repo

I started with zero background in LLMs. This repo tracks what I'm learning, the papers I study, and the research direction I'm building toward, mentored by Dinesh Sir.

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
