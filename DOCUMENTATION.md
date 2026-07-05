# LLM Research Log — Souvik

**Mentor:** Dinesh Sir

**Research Area:** Large Language Models (LLMs)

**Personal Focus:** Hallucination Mitigation via Structured Knowledge Integration → Structured Reasoning

---

## Day 0 — Background

I started this research with no prior background in LLMs, beyond general 
awareness of tools like ChatGPT. During JAVA Lab sessions, Dinesh Sir 
explained how large language models function and, in doing so, introduced 
**hallucination** — the tendency of LLMs to generate fluent but factually 
incorrect output — as one of the field's central open problems.

This raised a natural question: how do these models actually work under the 
hood, and what real architectural gaps cause hallucination? Sir outlined the 
scope for research in this space and provided initial resources covering 
**Knowledge Graphs (KG)**, **Meta-Knowledge Graphs (MKG)**, **GraphRAG**, 
**KG-Adapters**, and **Structured Reasoning** as active approaches to this 
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

### My Personal Reflection
This was our very first day on this research. I didn't know anything about 
this topic before — Dinesh Sir told us all the basic things about LLMs, and 
the future scope of this field, what I could work on and what I had to do 
next. Later Sir gave us resources to study. On this first day I learned the 
basics of what LLM, KG, and MKG actually are. At first it looked complex, 
but after going through it properly, I got interested in it, and that's 
when I started focusing more seriously on the research questions.

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

### My Personal Reflection
For Day 2, Sir gave us resources related to KG-Adapter, PEFT, GNN, and a few 
other topics. After going through the research paper, it looked quite 
complex, so I used ChatGPT to help make sense of the harder parts. But 
later, when I discussed it with Dinesh Sir, I realized that ChatGPT can also 
give wrong answers sometimes. That was an important lesson I learned that 
day — not to blindly trust an AI's explanation without cross-checking it.


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

### My Personal Reflection
This day felt different from the other days. Instead of just summarizing a 
paper, I noticed a real gap — it never actually tested hallucination — and 
came up with my own way to fix it. It's still just a proposal, not tested, 
but it feels more like my own work than the earlier days.

---

## Day 4 — Understanding the Benchmark Datasets Used in LLM/KG Research

**Task given:** Analyze and understand the benchmark datasets referenced 
across the papers studied so far — how each is constructed, its working 
principle, its statistics, and its original source — and record the source 
link for each dataset.

This entry documents each dataset independently, in my own understanding, 
with the original paper cited as the source of truth.

---

### 1. HotpotQA
**Task type:** Multi-hop Question Answering
**Source:** Yang et al., *"HotpotQA: A Dataset for Diverse, Explainable 
Multi-hop Question Answering,"* EMNLP 2018 — 
[ACL Anthology](https://aclanthology.org/D18-1259/) | 
[arXiv](https://arxiv.org/abs/1809.09600)

**Construction:** Crowdworkers were shown pairs of related Wikipedia 
articles and asked to write questions that genuinely require information 
from *both* to answer, rather than being answerable from either one alone. 
This produced roughly 113,000 question-answer pairs.

**Working principle:** Each question requires reading and reasoning over 
multiple supporting documents rather than a single passage. What makes it 
distinctive is that it also provides **sentence-level supporting facts** for 
every answer — so a model isn't just scored on the final answer, but can be 
evaluated on *whether it retrieved the correct evidence* to reach it. It 
also contains "comparison" questions (e.g., comparing two entities' 
attributes), testing a reasoning skill distinct from plain fact chaining.

**Known limitation:** Later research (see 2WikiMultiHopQA below) found that 
a meaningful fraction of HotpotQA's "multi-hop" questions can be answered 
via shortcuts — surface-level cues — without a model actually performing 
multi-hop reasoning.

**Why it matters for our research:** It directly tests whether a model can 
chain together separate pieces of information — the core capacity that 
"structured reasoning" approaches (Days 1–3) are trying to strengthen.

---

### 2. 2WikiMultiHopQA
**Task type:** Multi-hop Question Answering with explicit reasoning paths
**Source:** Ho et al., *"Constructing A Multi-hop QA Dataset for 
Comprehensive Evaluation of Reasoning Steps,"* COLING 2020 — 
[ACL Anthology](https://aclanthology.org/2020.coling-main.580/) | 
[arXiv](https://arxiv.org/abs/2011.01060)

**Construction:** Built by combining structured data from **Wikidata** with 
unstructured **Wikipedia** text, specifically to correct a flaw the authors 
identified in earlier multi-hop datasets: many "multi-hop" questions could 
secretly be answered without doing multi-hop reasoning at all.

**Working principle:** Every question is paired with an explicit 
**evidence/reasoning path** — the exact chain of facts required to reach the 
answer — so a model's intermediate reasoning steps, not just its final 
answer, can be verified. This closes the shortcut-answering gap left open 
by HotpotQA.

**Why it matters:** It's a stricter, more reliable benchmark for confirming 
that a model is *actually* reasoning step-by-step, rather than pattern 
matching to a shortcut — directly relevant to validating multi-hop claims 
made in Day 1's frameworks.

---

### 3. ComplexWebQuestions (CWQ)
**Task type:** Complex, compositional Question Answering over the web/KB
**Source:** Talmor & Berant, *"The Web as a Knowledge-Base for Answering 
Complex Questions,"* NAACL 2018 — 
[ACL Anthology](https://aclanthology.org/N18-1059/) | 
[arXiv](https://arxiv.org/abs/1803.06643)

**Construction:** Built on top of an existing KBQA dataset by algorithmically 
combining simple questions using logical/compositional operations 
(conjunctions, superlatives, comparisons), then having crowdworkers rephrase 
them into natural, fluent questions.

**Working principle:** The authors' key idea is that a complex question can 
often be **decomposed into a sequence of simpler questions**, each 
answerable independently, with the final answer computed by chaining the 
results together. Their experiments showed this decomposition strategy 
improved precision@1 from 20.8% (answering the question directly) to 27.5% 
— a meaningful gain from breaking the question apart rather than treating 
it as one opaque query.

**Why it matters:** It's a strong benchmark for testing whether structured 
or KG-based methods can break down compositional reasoning the way this 
decomposition strategy does — relevant to evaluating both KG-Adapter (Day 2) 
and structured reasoning approaches (Day 3).

---

### 4. WebQuestionsSP (WebQSP)
**Task type:** Knowledge Base Question Answering (KBQA)
**Source:** Yih et al., *"The Value of Semantic Parse Labeling for 
Knowledge Base Question Answering,"* ACL 2016 — 
[ACL Anthology](https://aclanthology.org/P16-2033/)

**Construction:** Extends the earlier **WebQuestions** dataset (questions 
originally collected via the Google Suggest API and answered by crowdworkers, 
Berant et al. 2013) by having annotators write full **semantic parses** — 
structured SPARQL queries — for each question, grounded against the 
**Freebase** knowledge base. This resulted in 4,737 fully-annotated 
questions, split roughly 3,098 for training and 1,639 for testing.

**Working principle:** Because each question has a gold-standard structured 
query attached, models can be trained and evaluated not just on whether 
they produce the right final answer, but on whether they correctly parse 
natural language into a structured KB query. The original paper found this 
extra annotation effort paid off directly: adding semantic parse labels 
improved a state-of-the-art KBQA system's F1 score by a full 5 points.

**Why it matters:** It's one of the standard benchmarks used to evaluate 
KG-Adapter-style methods (Day 2), since it directly tests grounding language 
into structured KB queries — exactly the mechanism KG-Adapter is designed 
to perform.

---

### 5. GrailQA
**Task type:** Knowledge Base Question Answering — generalization testing
**Source:** Gu et al., *"Beyond I.I.D.: Three Levels of Generalization for 
Question Answering on Knowledge Bases,"* WWW/TheWebConf 2021 — 
[arXiv](https://arxiv.org/abs/2011.07743)

**Construction:** Contains 64,331 questions spanning 3,720 relations, 1,442 
entity types, and 86 domains from Freebase — deliberately built to be far 
broader in domain coverage than prior KBQA datasets.

**Working principle:** The authors argue that standard "train and test on 
similar data" (i.i.d.) evaluation is misleading for KBQA, since real-world 
questions can never be fully anticipated in advance. GrailQA instead 
evaluates models at **three explicit levels of generalization**: i.i.d. 
(similar to training data), compositional (new combinations of known 
elements), and zero-shot (entirely unseen relations/entity types).

**Why it matters:** This is a much harder and more realistic test of 
whether a KG-integrated model (like KG-Adapter, Day 2) genuinely 
generalizes, rather than memorizing surface patterns from its training 
distribution.

---

### 6. MetaQA
**Task type:** Multi-hop KBQA with noisy/paraphrased questions
**Source:** Zhang et al., *"Variational Reasoning for Question Answering 
with Knowledge Graph,"* AAAI 2018 — [arXiv](https://arxiv.org/abs/1709.04071)

**Construction:** Built over a movie-domain knowledge graph (extending the 
WikiMovies dataset), with over 400,000 questions spanning 1-, 2-, and 3-hop 
reasoning, plus paraphrased text variants and, in some released versions, 
spoken/audio question variants — deliberately introducing the kind of noise 
real users produce.

**Working principle:** MetaQA addresses two practical problems 
simultaneously: (1) real user questions contain typos, rephrasing, and 
pronunciation variation, which makes entity matching against the KG hard, 
and (2) many of those same questions require multi-hop reasoning to answer. 
The authors propose a variational deep learning approach that handles noisy 
input while learning multi-hop reasoning at the same time.

**Why it matters:** It highlights a real-world constraint often missed in 
clean academic benchmarks — that a research method also needs to be robust 
to imperfect, noisy user input, not just clean textbook-style questions.

---

### 7. T-REx
**Task type:** Knowledge Graph ↔ Natural Language alignment resource (not QA)
**Source:** Elsahar et al., *"T-REx: A Large Scale Alignment of Natural 
Language with Knowledge Base Triples,"* LREC 2018 — 
[ACL Anthology](https://aclanthology.org/L18-1544/)

**Construction:** Built by automatically aligning Wikipedia abstracts with 
Wikidata knowledge base triples using a combination of rule-based and deep 
learning alignment techniques. The final resource contains exactly 11 
million triples aligned to 3.09 million Wikipedia abstracts (6.2 million 
sentences) — at the time of publication, two orders of magnitude larger 
than any prior alignment dataset.

**Working principle:** Unlike the QA datasets above, T-REx is **training 
infrastructure**: it doesn't test a model's answer, it *teaches* a model the 
correspondence between free-form language and structured facts. This is the 
dataset used in the Day 3 paper (Zhang et al.) as the training source for 
knowledge-graph-infused fine-tuning.

**Why it matters:** It's foundational — this is exactly the kind of 
resource needed to teach a model how free text maps onto structured graph 
facts, which underpins the Structured Reasoning approach in Day 3.

---

### 8. FEVER
**Task type:** Fact Extraction and Verification
**Source:** Thorne et al., *"FEVER: a Large-scale Dataset for Fact 
Extraction and Verification,"* NAACL 2018 — 
[ACL Anthology](https://aclanthology.org/N18-1074/) | 
[arXiv](https://arxiv.org/abs/1803.05355)

**Construction:** Annotators generated over 185,000 claims by altering 
sentences taken from Wikipedia, then a separate, independent group of 
annotators verified each claim against Wikipedia evidence **without** 
knowledge of which sentence the claim originally came from — reducing bias 
in the labeling process.

**Working principle:** Each claim is labeled **Supported**, **Refuted**, or 
**NotEnoughInfo**, with the exact supporting or refuting evidence 
sentence(s) recorded alongside it. Notably, the original paper found this 
task genuinely difficult — even when given the correct evidence, their best 
full pipeline reached only **31.87% accuracy**, and dropped further, to 
50.91%, if evidence was ignored entirely — showing that fact verification 
against text is far from a solved problem.

**Why it matters:** This is the dataset behind the fact-verification loss 
term I'm proposing in Day 3 — it's specifically built to test whether a 
claim is actually supported by evidence, which is precisely the gap in 
hallucination measurement I'm trying to close.

---

### 9. TruthfulQA
**Task type:** Truthfulness / Hallucination benchmark
**Source:** Lin et al., *"TruthfulQA: Measuring How Models Mimic Human 
Falsehoods,"* ACL 2022 — 
[ACL Anthology](https://aclanthology.org/2022.acl-long.229/) | 
[arXiv](https://arxiv.org/abs/2109.07958)

**Construction:** The authors deliberately crafted 817 questions across 38 
categories (health, law, finance, politics, conspiracy theories, etc.), 
specifically targeting topics where a model that simply imitates common 
human text is likely to reproduce a popular **misconception** rather than 
the truth.

**Working principle:** Models are scored on whether their answers are 
truthful, using either human evaluation or a fine-tuned "truth classifier" 
model. The paper's most striking finding: their best model was truthful 
only about 58% of the time, compared to roughly 94% for human performance — 
and, notably, **larger models tended to be *less* truthful**. As a concrete 
example from the paper, the 6-billion-parameter GPT-J model was **17% less 
truthful** than its own much smaller 125-million-parameter counterpart, 
since bigger models are better at fluently reproducing convincing-sounding 
misconceptions absorbed from their training data.

**Why it matters:** This result is central to my own research motivation — 
it's direct empirical evidence that scaling model size alone does not fix 
hallucination, which is exactly why structural/architectural interventions 
(Days 1–3), rather than scale alone, are necessary.

---

### Cross-Dataset Analysis

Looking at all nine datasets together, I noticed they fall into three 
distinct functional categories, which matters for how I plan to use them:

1. **Training/alignment resources** (T-REx) — teach a model the 
   correspondence between text and structured facts; used *during* training, 
   not for evaluation.
2. **Reasoning benchmarks** (HotpotQA, 2WikiMultiHopQA, ComplexWebQuestions, 
   WebQuestionsSP, GrailQA, MetaQA) — test whether a model can correctly 
   retrieve and chain facts, with increasing levels of strictness (from 
   HotpotQA's basic multi-hop check, up to GrailQA's zero-shot 
   generalization test).
3. **Truthfulness/verification benchmarks** (TruthfulQA, FEVER) — test 
   whether a model's output is actually *true*, independent of whether the 
   reasoning steps look correct. This category is the one the original Day 3 
   paper (Zhang et al.) never used, and is the specific gap my research 
   proposal targets.

I also noticed a pattern across the reasoning benchmarks: several of them 
(2WikiMultiHopQA, GrailQA) exist *specifically* because earlier datasets 
(HotpotQA, standard KBQA sets) had hidden weaknesses — shortcut-answerable 
questions, or i.i.d. train/test bias — that made reported results look 
better than the models' true reasoning ability. This tells me benchmark 
design itself is an active, evolving research problem, not a fixed, settled 
foundation.

### Summary Table

| Dataset | Type | Size (verified) | Primary Use in My Research |
|---|---|---|---|
| HotpotQA | Multi-hop QA | 113,000 QA pairs | Multi-hop reasoning evaluation |
| 2WikiMultiHopQA | Multi-hop QA + reasoning path | ~192,600 examples | Verifying genuine reasoning, not shortcuts |
| ComplexWebQuestions | Compositional QA | — | Testing question decomposition (20.8%→27.5% precision@1) |
| WebQuestionsSP | KBQA (Freebase) | 4,737 questions | Semantic parsing benchmark |
| GrailQA | KBQA generalization | 64,331 questions | i.i.d./compositional/zero-shot generalization |
| MetaQA | Multi-hop KBQA (noisy) | 400,000+ questions | Robustness to real-world noisy input |
| T-REx | KG–text alignment | 11M triples / 3.09M abstracts | Training data for KG-infused fine-tuning |
| FEVER | Fact verification | 185,445 claims | Fact-verification loss term (my proposal) |
| TruthfulQA | Truthfulness benchmark | 817 questions, 38 categories | Core hallucination-rate evaluation |

### What I Learned
- Not all "datasets" serve the same purpose — some are **training resources** 
  (T-REx), some are **reasoning benchmarks** (HotpotQA, 2WikiMultiHopQA), and 
  some are **evaluation-only truthfulness tests** (TruthfulQA, FEVER). 
  Conflating these categories would make a research design incoherent.
- Benchmark design itself is an active research problem — GrailQA and 
  2WikiMultiHopQA both exist specifically because earlier datasets had 
  hidden flaws (shortcut answers, i.i.d. assumptions) that made results look 
  better than they really were.
- The TruthfulQA finding that larger models can be *less* truthful is a 
  particularly important piece of evidence for my research direction: it 
  justifies why my proposal focuses on structural fixes rather than assuming 
  scale will eventually solve hallucination on its own.

### My Personal Reflection
Going through nine datasets in one sitting was a lot, but it helped me see 
how they connect — like some datasets were made just to fix problems in 
older ones. There's a lot to know about this, and I'm still learning how 
all the pieces fit together.

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
