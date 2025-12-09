# XtraMCP Researcher Node & Community-Federated Conference (CFC) Curation Engine

> 点击切换语言 · Click to switch language：
> **[English](./README.md)** | **[中文](./README_zh.md)**

This repository houses the core intelligence for **semantic literature retrieval** and **topic discovery** with demo [nextaiconf.com](https://nextaiconf.com). It serves as the backend engine for the Researcher Module within the **XtraMCP** framework [[PaperDebugger](#cite-paperdebugger) [Github](https://github.com/PaperDebugger/PaperDebugger), [XtraGPT](#cite-xtragpt) [Github](https://github.com/NuoJohnChen/XtraGPT)], powering the **PaperDebugger** writing assistant [[PaperDebugger](#cite-paperdebugger)]. It also provides the technological infrastructure required to operationalize the **Community-Federated Conference (CFC)** model, which addresses the sustainability crisis in centralized AI conferences [[Position](#cite-position) [Github](https://github.com/NuoJohnChen/AI_Conf_Crisis)].

---

## 1. Core Concepts: What is this repository doing?

At its heart, this engine facilitates two opposing flows of information to bridge the gap between abstract research contexts and concrete academic literature:

### ➡️ Context2Papers (From Idea to Literature)
> *"I have a workshop theme or a paper draft, show me the relevant literature."*

* **What it does:** It acts as a **Semantic Recommendation Engine**. Instead of relying on simple keywords, it takes a long-form "context" (such as a workshop description, a specific paragraph, or an abstract) and retrieves the most semantically relevant academic papers from a database of ~700,000 entries.
* **Use Case:** Helping a researcher find citations for a specific claim, or helping a conference organizer find papers that fit a specific workshop theme.

### ⬅️ Papers2Context (From Literature to Insights)
> *"I have a pile of papers (e.g., from a specific region or year), tell me what they are about."*

* **What it does:** It acts as a **Topic Discovery Engine**. It takes a filtered set of papers (e.g., "All papers published by Singaporean authors in 2025") and automatically clusters them to discover latent themes, research trends, and "contexts."
* **Use Case:** Helping **Federated Regional Hub** organizers [[Position]](#cite-position) identify local research strengths to curate targeted workshops (e.g., discovering that a region has a high density of "Multimodal LLM" papers).

---

## 2. Technical Implementation

To achieve the concepts above, the `app.py` engine utilizes a data pipeline and specific ML models:

### Data Foundation: The `paperdb` Pipeline
The system relies on the `paperdb/` directory for data ingestion.
* **Artifacts:** It scrapes arXiv metadata to produce `arxiv_merged.parquet` (titles, abstracts, authors) and `embeddings.npy` (pre-computed vectors).
* **Efficiency:** These files are memory-mapped by the recommender to ensure low-latency access without real-time inference on the entire corpus.

### Algorithms & Models
* **For Context2Papers:**
    * **Recall (Specter2):** We use `allenai/specter2_base` to encode user queries into the same vector space as the citation network. **Cosine Similarity** is used to quickly retrieve top candidates.
    * **Precision (LLM Re-ranking):** To fix vector compression loss, we use `Qwen/Qwen3-Reranker-8B` to re-score candidates based on full-text alignment with the query.
* **For Papers2Context:**
    * **Clustering (BERTopic):** We implement a pipeline using **UMAP** (dimensionality reduction) and **HDBSCAN** (density clustering) to find organic groups of papers.
    * **Labeling:** **CountVectorizer** extracts class-based TF-IDF keywords to give human-readable names to these discovered clusters.

---

## 3. Role in XtraMCP & PaperDebugger

This repository functions as the **Researcher Node** within the XtraMCP architecture, decoupling orchestration from reasoning [[XtraGPT]](#cite-xtragpt)[[PaperDebugger]](#cite-paperdebugger).

* **The Researcher of XtraMCP:**
    * It acts as the execution layer for the **Researcher Module**. When the XtraMCP control layer receives a user intent (e.g., "Find related work"), it routes the request to this backend via the Model Context Protocol (MCP).
    * **Hallucination-Free Safeguard:** Unlike standard LLMs that may fabricate citations, this module performs deterministic retrieval over the real-world vectors created by `paperdb`, ensuring every recommendation exists and is verifiable.

* **Service to PaperDebugger:**
    * **PaperDebugger** [[PaperDebugger]](#cite-paperdebugger), the user-facing Overleaf extension, relies on this backend to provide real-time "Deep Research" capabilities.
    * By sending the user's current writing context to this engine's **Context2Papers** endpoint, it retrieves relevant literature to generate "Relevance Insights" and help authors position their work.

---

## 4. Addressing the AI Conference Crisis (CFC Model)

The current centralized AI conference model is unsustainable due to exponential submission growth and inefficient knowledge dissemination [[Position]](#cite-position). This codebase serves as the technical foundation for the proposed **Community-Federated Conference (CFC)** solution.

* **Solving Information Overload:** **Context2Papers** enables the "Scientific Mission" of efficient knowledge exchange, allowing researchers to cut through the noise of thousands of submissions.
* **Enabling Regional Hubs:** **Papers2Context** is critical for organizers of **Federated Regional Hubs**. By filtering the database by **region**, **date**, **categories in AI** and running topic discovery, organizers can identify local research themes. This restores the "Community Building" pillar by fostering meaningful, localized interactions rather than anonymous mega-conferences.

---
## 5 Reference
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
