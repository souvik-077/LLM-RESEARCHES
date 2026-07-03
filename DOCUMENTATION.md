# LLM Research Log — Souvik

**Mentor:** Dinesh Sir
**Research Area:** Large Language Models (LLMs)
**Personal Focus:** Hallucination Mitigation via Structured Knowledge Integration → Structural Reasoning

---

## Background

I started this research with no prior background in LLMs, beyond general 
awareness of tools like ChatGPT. During JAVA Lab sessions, Dinesh Sir 
explained how large language models function and, in doing so, introduced 
**hallucination** — the tendency of LLMs to generate fluent but factually 
incorrect output — as one of the field's central open problems.

This raised a natural question: how do these models actually work under the 
hood, and what real architectural gaps cause hallucination? Sir outlined the 
scope for research in this space and provided initial resources covering 
**Knowledge Graphs (KG)**, **Meta-Knowledge Graphs (MKG)**, **GraphRAG**, 
**KG-Adapters**, and **Multi-Hop Reasoning** as active approaches to this 
problem. This log documents my own study path through these topics.

---

## Day 1 — Reducing LLM Hallucinations via Structured Knowledge Integration

### Objective
To understand the root causes of hallucination in LLMs and evaluate how 
Knowledge Graphs and neuro-symbolic AI approaches address them — with a 
specific look at practical frameworks (the NIT Durgapur pipeline and the 
Meta-Knowledge Guided / MKG framework).

### The Problem
Traditional LLMs hallucinate because of four compounding architectural 
issues:
- **Implicit knowledge storage** — facts are distributed across billions of 
  floating-point weights with no way to directly verify them
- **Probabilistic generation** — the model predicts the *most probable* next 
  token, not the most *factual* one
- **Open-loop architecture** — there is no internal verification or 
  fact-checking step after generation

A common misconception is that scaling model size alone solves this. In 
reality, hallucination is a structural limitation that persists even in 
larger models — the fix requires redesigning the architecture to include 
external verification loops, not just adding parameters.

### Methodology

**Knowledge Graphs (KG)** store facts as discrete, queryable triples — 
`(Head Entity) —[Relation]→ (Tail Entity)` — making them traceable and 
verifiable, unlike knowledge implicitly encoded in model weights.

**Neuro-Symbolic AI** combines the language fluency of neural models (LLMs) 
with the factual precision of symbolic systems (Knowledge Graphs), aiming to 
get the strengths of both while offsetting each one's individual weaknesses.

I studied this through two frameworks:

**1. NIT Durgapur — 4-Phase Pipeline**, addressing four research questions:
| Phase | Focus | Outcome |
|---|---|---|
| RQ1 | Ontological Alignment — converting LLM output into KG-compatible triples via adapter layers | 31% reduction in hallucination |
| RQ2 | Self-Verification Loop — a Generate → Validate → Refine cycle | Closed-loop correction (stronger than standard RAG, which only retrieves without correcting) |
| RQ3 | Uncertainty Estimation — verifying only when the model is uncertain | Reduces unnecessary computation |
| RQ4 | Conflict Resolution — resolving disagreements using Bayesian trust scoring | 94% accuracy on multi-hop reasoning tasks |

**2. Meta-Knowledge Guided (MKG) Framework** — introduces the idea of 
*meta-knowledge*: not just facts, but knowledge about **how** to use and 
verify facts (analogous to a student double-checking their work, not just 
knowing the textbook). MKG is built from three cooperating components:
- **Meta-Executor (Me)** — generates initial responses from retrieved facts
- **Meta-Assessor (Ma)** — evaluates outputs for hallucination/consistency 
  on a 1–5 scale
- **Meta-Insight Processor (Mip)** — generates corrective feedback and 
  learns patterns over time

MKG also retrieves knowledge using three complementary methods — BGE 
semantic embeddings, ColBERT late-interaction, and BM25 lexical retrieval — 
fused via Reciprocal Rank Fusion, so that abstract queries, precise phrases, 
and exact keyword matches are all covered. It further separates memory into 
a **Short-Term Memory** (session-specific context) and a **Long-Term 
Memory** (a persistent vector store of reusable meta-knowledge across 
sessions).

An important real-world caveat I noted: knowledge graphs themselves are not 
perfectly clean — they typically contain 5–15% inaccurate, outdated, or 
contradictory data. Blindly trusting external KG data is therefore just as 
risky as blindly trusting the LLM's internal weights. MKG addresses this via 
Bayesian trust scoring, temporal validity stamps, and consensus/voting 
mechanisms for conflicting sources.

### Results
- RQ1's adapter-layer alignment alone reduced hallucination by **31%**
- RQ4's conflict-resolution approach reached **94% accuracy** on multi-hop 
  reasoning tasks
- On the GrailQA benchmark, MKG improved accuracy by **+30% over baseline 
  LLMs**, **+17% over standard RAG**, and **+5.4% over other state-of-the-art 
  methods**

### Conclusion
Hallucination is best understood as an **architectural problem**, not an 
inherent flaw that scale alone can fix. Structured knowledge (via Knowledge 
Graphs) combined with self-verification loops and meta-knowledge learning 
offers a measurable, benchmarked path toward more trustworthy LLMs.

### Possible Future Directions (noted for later study)
- Domain-specific adapters (medical/legal/finance)
- Temporal knowledge decay — auto-expiring outdated facts
- Explainability audit trails for high-stakes domains
- Multi-modal KG integration (text + images)
- Real-time KG updates via streaming pipelines

---

## Day 2 — *(to be added)*

**Task given:** 

### What I studied


### Conclusion


---

## Day 3 — *(to be added)*

**Task given:** 

### What I studied


---

## Current Focus — Structural Reasoning

*(to be filled in once I document my understanding here)*

---

*This log is a work in progress and is updated as research continues.*
