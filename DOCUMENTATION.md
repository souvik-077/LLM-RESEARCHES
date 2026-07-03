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

## Day 2 — KG-Adapter: Enabling Knowledge Graph Integration in LLMs through Parameter-Efficient Fine-Tuning (PEFT)

📊 [View full Day 2 presentation slides](slides/kg-adapter.pptx)

### Objective
To understand how Knowledge Graph knowledge can be integrated *into* an 
LLM's internal layers — rather than only supplied externally through 
prompts — using a lightweight, trainable adapter module.

### The Problem
Beyond hallucination, LLMs face several related limitations: limited domain 
expertise, knowledge that goes stale without retraining, disagreement 
between the model's internal memory and external facts, heavy dependence on 
prompt quality, and poor explainability of *why* an answer was generated.

Existing approaches each fall short in a different way:
| Method | Approach | Key Problem |
|---|---|---|
| Traditional LLM | Uses only trained memory | Hallucinations, outdated knowledge |
| Prompt Engineering | Injects KG facts into the prompt | Output quality depends entirely on prompt design |
| RAG | Retrieves documents at inference | Ignores graph structure; no real reasoning over relations |
| KG Prompting | Puts KG triples directly in the prompt | Prompt becomes too long; still prompt-dependent |
| Full Fine-Tuning | Retrains the entire model on KG data | Extremely expensive; risks forgetting prior knowledge |

### Methodology

**Core idea:** instead of placing KG facts inside the prompt text, 
KG-Adapter attaches a small trainable module directly inside the frozen LLM. 
The pipeline is: *Knowledge Graph → Graph Encoder (GNN-based) → KG Adapter 
(lightweight, trainable) → LLM Decoder (frozen) → Generated Response.*

Two complementary encoding strategies make this work:
- **Node-centered encoding** — each entity in the KG (a person, place, 
  concept) is converted into a vector representation that the adapter can 
  use
- **Relation-centered encoding** — the *edges* between entities (e.g. 
  "Harrier → has temperament → Friendly") are encoded separately and 
  combined with node vectors, since knowing an entity alone is incomplete 
  without knowing how it relates to others

This is a direct application of **Parameter-Efficient Fine-Tuning (PEFT)**: 
rather than updating all of a model's weights (expensive, memory-heavy, and 
prone to catastrophic forgetting), only the small adapter module is trained 
while the base LLM stays frozen. In practice, this meant training only 
**28 million parameters out of a 7-billion-parameter model — just 0.4% of 
the total.**

### Experimental Setup
- **Base models tested:** LLaMA (7B), Vicuna (7B), ChatGLM
- **Benchmarks:** WebQSP, CWQ (Complex Web Questions), GrailQA
- **Tasks:** Knowledge Graph QA, entity linking, relation reasoning
- **Metrics:** F1 Score, Hits@1, Accuracy, Exact Match

### Results
KG-Adapter outperformed all baseline methods across the KGQA benchmarks 
while training only ~0.4% of total model parameters — showing that 
*learning* structured knowledge through a graph encoder is more effective 
than simply *retrieving* it via prompts or RAG.

### My Assessment — Limitations
Reading critically, I noted several open issues:
- Performance depends entirely on the **quality of the input KG** — errors 
  or gaps in the graph propagate directly into model output
- The adapter has no mechanism to **correct wrong KG data**; it learns 
  whatever the graph contains, including its errors
- Building a good domain-specific KG is itself **expensive and 
  expert-intensive**
- Adapter performance is still **bounded by the base LLM's own capability**
- **Real-time KG updates are difficult** — dynamic changes require 
  re-training or re-initializing the adapter
- Evaluation so far is mostly limited to **KGQA-style tasks**, not the full 
  diversity of real-world NLP use cases

### My Suggestions for Future Improvement
1. Automatic KG updating as new facts emerge
2. Confidence scoring before the model commits to an answer
3. Multi-KG support — integrating multiple domain graphs at once
4. Adaptive adapter selection per domain or query type
5. Visual explanation of the graph reasoning path behind an answer
6. Self-correction — re-verifying the answer against the KG post-generation
7. Ranking KG facts by source trustworthiness
8. Reasoning-path memory — reusing successful inference paths across queries

### Conclusion
Traditional LLMs rely mostly on learned memory, which drives hallucination 
and factual drift. KG-Adapter offers a more structural fix: injecting 
knowledge directly into model internals via a lightweight, PEFT-trained 
module, achieving strong accuracy gains at a fraction of the training cost 
of full fine-tuning. The key distinction from Day 1's approaches is that 
**KG-Adapter learns knowledge, rather than just retrieving it** — a 
meaningfully different strategy from prompt-based or RAG-based methods.

### What I Personally Took Away
- The distinction between adding knowledge *outside* the model (prompting) 
  versus *inside* it (adapters)
- How Graph Neural Networks (GNNs) are used to turn graph structure into 
  vectors usable by a language model
- Why lightweight fine-tuning (PEFT) matters for making serious LLM research 
  feasible without massive GPU infrastructure
- How to read a research paper critically: identifying not just what it 
  claims, but its limitations and what I would improve


---

## Day 3 — *(to be added)*

**Task given:** 

### What I studied


---

## Current Focus — Structural Reasoning

*(to be filled in once I document my understanding here)*

---

*This log is a work in progress and is updated as research continues.*
