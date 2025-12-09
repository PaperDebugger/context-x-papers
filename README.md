# XtraMCP Researcher Node & Community-Federated Conference (CFC) Curation Engine

> 点击切换语言 · Click to switch language：
> **[English](./README.md)** | **[中文](./README_zh.md)**

### 1. Overview

This repository houses the core intelligence for **semantic literature retrieval** and **topic discovery** with demo [nextaiconf.com](https://nextaiconf.com). It serves as the backend engine for the Researcher Module within the **XtraMCP** framework [[PaperDebugger](#cite-paperdebugger) [Github](https://github.com/PaperDebugger/PaperDebugger), [XtraGPT](#cite-xtragpt) [Github](https://github.com/NuoJohnChen/XtraGPT)], powering the **PaperDebugger** writing assistant [[PaperDebugger](#cite-paperdebugger)]. It also provides the technological infrastructure required to operationalize the **Community-Federated Conference (CFC)** model, which addresses the sustainability crisis in centralized AI conferences [[Position](#cite-position) [Github](https://github.com/NuoJohnChen/AI_Conf_Crisis)].

---

### 2. Data Ingestion: The `paperdb` Pipeline

The foundation of this system lies in the `paperdb/` directory, which constructs the comprehensive knowledge base required for semantic analysis.

#### 2.1 Function

This module scrapes and aggregates arXiv metadata and uses **Specter2** to generate document-level vector embeddings.

#### 2.2 Outputs

It produces two critical artifacts:

- `arxiv_merged.parquet`A structured dataset containing metadata (titles, abstracts, authors, categories) for ~700,000 papers.
- `embeddings.npy`
  A dense vector file containing the pre-computed embeddings corresponding to the papers.

#### 2.3 Integration

These files are loaded directly by `app.py` to initialize the `WorkshopRecommender` class, enabling low-latency retrieval **without** real-time inference over the entire corpus.

---

### 3. Core Functionalities: Context2Papers & Papers2Context

The `app.py` engine utilizes `paperdb` data to implement two bidirectional flows of information:

- **Context2Papers** – Semantic Retrieval & Recommendation
- **Papers2Context** – Topic Discovery & Trend Analysis

---

#### 3.1 Context2Papers: Semantic Retrieval & Recommendation

**Goal:** Retrieve high-precision, relevant academic papers given a specific research context (e.g., workshop description, abstract, or research query).

##### Implementation

1. **Vectorization (Recall Layer)**

   - Use **Specter2** (`allenai/specter2_base`) to convert user queries into vectors compatible with the pre-computed `embeddings.npy`.
   - Perform dense retrieval via **cosine similarity** to recall the top‑k candidate papers.
2. **Re-ranking (Precision Layer)**

   - To compensate for information loss from vector compression, employ a **Large Language Model Re-ranker** (`Qwen/Qwen3-Reranker-8B`).
   - It re-scores candidates by assessing full contextual alignment between the query and each paper, substantially improving relevance.

---

#### 3.2 Papers2Context: Topic Discovery & Trend Analysis

**Goal:** Automatically discover latent themes, research trends, and “contexts” from the large collection of papers in `arxiv_merged.parquet`, optionally filtered by region or time.

##### Implementation

1. **Clustering Pipeline**

   - Use **BERTopic** for dynamic topic modeling.
2. **Dimensionality Reduction**

   - Apply **UMAP** to reduce high-dimensional Specter2 embeddings into a clusterable low-dimensional space.
3. **Density Clustering**

   - Use **HDBSCAN** to identify dense clusters of papers, corresponding to emerging research fronts or local hotspots.
4. **Topic Representation**

   - Use `CountVectorizer` to extract class-based TF-IDF keywords and produce human-readable labels for each discovered context/topic.

---

### 4. Role in XtraMCP & PaperDebugger

This repository functions as the **Researcher Node** within the XtraMCP architecture, decoupling orchestration from reasoning [1].

#### 4.1 Researcher in the XtraMCP Architecture

- Acts as the **execution layer** for the Researcher Module.
  - When the XtraMCP control layer receives a user intent (e.g., “Find related work”), it routes the request to this backend via the **Model Context Protocol (MCP)**.
- Provides a **hallucination-free safeguard**:
  - Instead of fabricating citations (a common failure mode of generic LLMs), this module performs **deterministic retrieval** over the vector database created by `paperdb`, ensuring that every recommendation is grounded in real, verifiable literature.

#### 4.2 Service to PaperDebugger

The user-facing Overleaf extension **PaperDebugger** [3] uses this backend to provide real-time **Deep Research** capabilities:

- It analyzes the user’s current manuscript draft to:
  - retrieve highly relevant literature; and
  - generate **Relevance Insights** that help authors position their work within, and against, the state of the art.

---

### 5. Addressing the AI Conference Crisis: The CFC Model

Centralized AI conference models are becoming unsustainable due to **exponential growth in submissions** and **inefficient knowledge dissemination** [2]. This codebase is the technical backbone of the proposed **Community-Federated Conference (CFC)** solution.

#### 5.1 Mitigating Information Overload

Using **Context2Papers**, the system supports the **Scientific Mission** of efficient knowledge exchange:

- Researchers can cut through thousands of submissions and preprints to quickly find work that is highly relevant to their **specific context**.

#### 5.2 Enabling Federated Regional Hubs

The CFC model decouples **peer review** from **presentation**, organizing in-person interaction through **Federated Regional Hubs** (local gatherings of roughly 500–1500 participants).

- **Papers2Context** is crucial for hub organizers:
  - Filter `arxiv_merged.parquet` by region (e.g., “Singapore”).
  - Run topic discovery to identify dominant local research themes and emerging directions.
  - Curate highly focused, high-value workshops and symposia based on these insights.

This helps restore the **Community Building** function of conferences, enabling deep, locality-aware interactions instead of anonymous, oversized mega-events.

## 6 Reference

<div id="cite-paperdebugger"></div>

```bibtex
@misc{hou2025paperdebugger,
      title={PaperDebugger: A Plugin-Based Multi-Agent System for In-Editor Academic Writing, Review, and Editing}, 
      author={Junyi Hou and Andre Lin Huikai and Nuo Chen and Yiwei Gong and Bingsheng He},
      year={2025},
      eprint={2512.02589},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={[https://arxiv.org/abs/2512.02589](https://arxiv.org/abs/2512.02589)}, 
}
```

<div id="cite-xtragpt"></div>

```bibtex
@misc{chen2025xtragpt,
      title={XtraGPT: Context-Aware and Controllable Academic Paper Revision for Human-AI Collaboration}, 
      author={Nuo Chen, Andre Lin HuiKai, Jiaying Wu, Junyi Hou, Zining Zhang, Qian Wang, Xidong Wang, Bingsheng He},
      year={2025},
      eprint={2505.11336},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={[https://arxiv.org/abs/2505.11336](https://arxiv.org/abs/2505.11336)}, 
}
```

<div id="cite-position"></div>

```bibtex
@misc{chen2025position,
      title={Position: The Current AI Conference Model is Unsustainable! Diagnosing the Crisis of Centralized AI Conference}, 
      author={Nuo Chen and Moming Duan and Andre Huikai Lin and Qian Wang and Jiaying Wu and Bingsheng He},
      year={2025},
      eprint={2508.04586},
      archivePrefix={arXiv},
      primaryClass={cs.CY},
      url={[https://arxiv.org/abs/2508.04586](https://arxiv.org/abs/2508.04586)}, 
}

```
