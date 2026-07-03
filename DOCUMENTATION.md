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

📊 [View full Day 1 presentation slides](slides/LLM_hallucination.pptx)

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

## Day 3 — Structured Reasoning in LLMs: Knowledge Graph-Infused Fine-Tuning, and a Proposal to Reduce Hallucinations

📄 [View full write-up](notes/structured-reasoning.docx)

*Based on: "Knowledge Graph-Infused Fine-Tuning for Structured Reasoning in 
Large Language Models" (Zhang et al.)*

### Objective
To study how fine-tuning an LLM with injected knowledge graph information 
improves structured reasoning, and to identify where this approach could be 
extended specifically toward reducing hallucination — which the original 
paper does not directly measure.

### Problem Addressed by the Paper
The paper identifies two core weaknesses in LLMs: **missing reasoning 
chains** (the model can't logically connect separate facts) and **weak 
entity-level understanding** (it doesn't deeply grasp what an entity is or 
how it relates to others). Its claim is that injecting structured knowledge 
graphs during fine-tuning addresses both.

### Methodology (Original Paper)
- A **Graph Neural Network (GCN/R-GCN)** encodes entities and relations from 
  a KG into vector representations
- A **gating mechanism** dynamically balances reliance on plain-text 
  understanding versus structured graph knowledge
- A **joint loss function** trains on the main task (e.g. QA) alongside an 
  alignment objective keeping text and graph representations consistent
- A **knowledge-aware attention mechanism** lets each word in context attend 
  to related entities in the graph

**Dataset used:** T-REx — a large-scale Wikipedia-derived KG dataset with 
millions of entity-relation-entity triples aligned to natural language 
sentences, filtered into smaller, semantically dense subgraphs for training.

### Results (Original Paper)
The proposed method outperformed three baselines (KGLM, DRAGON, KG-SFT) — 
**86.4% QA accuracy**, **82.1% F1-score**, **29.7 BLEU**. A moderate learning 
rate (1e-4) with high subgraph coverage (90%+) gave the most stable results.

### What I Learned
- A dataset for structured reasoning must contain *organized facts*, not 
  just raw text
- Knowledge graphs let an LLM "check" its reasoning against verified facts, 
  instead of relying purely on learned language patterns
- Fusing text and graph structure isn't trivial — it needs dedicated 
  mechanisms (gating, joint loss); naively combining them doesn't work well
- Learning rate and KG coverage both meaningfully affect stability and 
  accuracy
- **The key gap:** the paper only used one dataset (T-REx) and never tested 
  hallucination directly — it measured task accuracy (QA-Acc, F1, BLEU), not 
  factual reliability. This is the gap I'm proposing to address.

---

### My Research Focus: Structured Reasoning to Reduce Hallucination

Building on this paper, my own research angle redirects the framework 
toward a different goal: not just "does the model perform the task well" 
but **"does the model tell the truth, and can we measure that directly."**

#### Datasets I'm Proposing to Use
| Dataset | Role | Purpose |
|---|---|---|
| Wikidata5M | Knowledge source | Larger, cleaner replacement for T-REx |
| FB15k-237 | Cross-check benchmark | Standard KG embedding benchmark for generalization |
| HotpotQA | Multi-hop reasoning | Tests whether the model can chain multiple facts |
| ComplexWebQuestions | Multi-hop reasoning | Multi-hop QA built specifically over KGs |
| TruthfulQA | Hallucination test | Measures false-but-plausible answers |
| HaluEval | Hallucination test | Benchmarks hallucination across QA, dialogue, summarization |
| FEVER | Fact verification | Checks whether generated claims are evidence-supported |

**Why more than one dataset is needed:** T-REx/Wikidata5M supply the 
structured facts to reason over; HotpotQA/ComplexWebQuestions test multi-hop 
chaining; TruthfulQA/HaluEval directly test truthfulness (missing from the 
original paper, and the core of my motivation); FEVER adds an evidence 
verification layer.

#### Modifications I'm Proposing to the Original Method
1. **Upgrade the knowledge source** — replace T-REx with Wikidata5M for 
   larger, cleaner, more current coverage, keeping T-REx as a comparison 
   baseline
2. **Add a hallucination-specific evaluation stage** — introduce TruthfulQA 
   and HaluEval, since the original paper never measures hallucination 
   directly
3. **Add a fact-verification loss term** — extend the joint loss 
   (`L_total = L_task + λL_align`) with a FEVER-style verification term that 
   penalizes unsupported claims
4. **Test multi-hop reasoning explicitly** — use HotpotQA/ComplexWebQuestions 
   to check whether the knowledge-aware attention mechanism can chain 
   multiple facts, not just retrieve single ones
5. **Report a hallucination rate metric** — alongside QA-Acc, F1, and BLEU, 
   report the percentage of factually unsupported statements as a primary 
   metric, since this most directly reflects my research motivation

### Expected Outcome
By combining a stronger knowledge graph (Wikidata5M), multi-hop reasoning 
benchmarks (HotpotQA), and hallucination-specific evaluation (TruthfulQA, 
HaluEval, FEVER), I expect to show that structured reasoning not only 
improves task accuracy — as the original paper demonstrated — but also 
measurably reduces hallucination rate, the gap the original work left 
unaddressed.

### Conclusion
The original paper gave me a strong technical foundation for how knowledge 
graphs are encoded, fused with language models, and jointly trained. My 
contribution is to redirect this framework toward a specific, measurable 
goal — reducing hallucination — by introducing hallucination-focused 
datasets and an additional verification objective the original work did not 
include.

---

## Current Focus — Structured Reasoning

This is my active research direction, connecting directly back to Day 1 
(Hallucination via Structured Knowledge Integration) and Day 2 (KG-Adapter's 
parameter-efficient KG injection): I am now working toward proposing a 
concrete extension — structured reasoning explicitly evaluated and optimized 
for hallucination reduction, not just task accuracy — as detailed in Day 3 
above.

---

*This log is a work in progress and is updated as research continues.*
