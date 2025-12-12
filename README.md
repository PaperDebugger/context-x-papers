# Semantic Literature Retrieval Engine for Academic Research & Conference Curation

> 点击切换语言 · Click to switch language：
> **[English](./README.md)** | **[中文](./README_zh.md)**

This repository provides a semantic literature retrieval and topic discovery engine built on vector embeddings and neural re-ranking. It performs high-precision academic paper search over ~700,000 arXiv papers, enabling researchers and conference organizers to ground research contexts in concrete literature and discover emerging research themes.

The engine powers two interconnected projects:
- **NextAIConf** (see [nextaiconf.com](https://nextaiconf.com)) — A deployed platform implementing the **Community-Federated Conference (CFC)** model, which addresses the sustainability crisis in centralized AI conferences through semantic paper curation and regional research theme discovery [[Position](#cite-position) [Github](https://github.com/NuoJohnChen/AI_Conf_Crisis)].
- **PaperDebugger's XtraMCP Researcher Phase** (see [paperdebugger.com](https://www.paperdebugger.com/) or the source code [here](https://github.com/PaperDebugger/paperdebugger)) — An end-to-end academic writing assistant where this engine serves as the hallucination-free literature grounding component, retrieving verifiable citations based on the user's writing context.
---

## 1. Core Concept

This engine was fundamentally designed for NextAIConf to solve conference curation challenges, with two bidirectional retrieval modes:

### ➡️ Context2Papers (From Idea to Literature)
> *"I have a workshop theme or a paper draft, show me the relevant literature."*

- **Semantic Recommendation Engine**. Takes long-form context (workshop descriptions, abstracts, paragraphs) and retrieves semantically relevant papers using vector similarity, not just keyword matching.
* **Use Case:** Conference organizers finding papers that match workshop themes; researchers discovering citations for specific claims.


### ⬅️ Papers2Context (From Literature to Insights)
> *"I have a pile of papers (e.g., from a specific region or year), tell me what they are about."*

**Topic Discovery Engine**. Clusters filtered paper sets (e.g., "Singapore 2025 authors") to reveal latent research trends and thematic patterns.
* **Use Case:** **Federated Regional Hub** organizers [[Position]](#cite-position) identifying local research strengths (e.g., discovering regional concentration in "Multimodal LLMs") to curate targeted workshops.
---

## 2. Implementation

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

This repository functions as the **Researcher Node** within the [XtraMCP](#cite-paperdebugger) architecture, decoupling orchestration from reasoning [[PaperDebugger]](#cite-paperdebugger). The information retrieved can later be used to complement the **Enhancer Node** within [XtraMCP](#cite-paperdebugger) which is powered by the [XtraGPT](#cite-xtragpt) model suite

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
## 5. References
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