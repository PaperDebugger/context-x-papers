#  XtraMCP Researcher 节点 & 社区联邦会议 (CFC) 策展引擎

> 点击切换语言 · Click to switch language：
> **[English](./README.md)** | **[中文](./README_zh.md)**

---

### 1. 概览

本仓库承载了语义文献检索与主题发现的核心智能，展示于[nextaiconf.com](https://nextaiconf.com)，是 **XtraMCP** 框架 [[PaperDebugger](#cite-paperdebugger) [Github](https://github.com/PaperDebugger/PaperDebugger), [XtraGPT](#cite-xtragpt) [Github](https://github.com/NuoJohnChen/XtraGPT)] 中 **Researcher 模块** 的后端引擎，为写作助手 **PaperDebugger** [[PaperDebugger](#cite-paperdebugger)] 提供支持。同时，它也是实现 **社区联邦会议（Community-Federated Conference, CFC）** 模型的技术基础，用以应对当前中心化 AI 会议在可持续性方面所面临的危机 [[Position](#cite-position) [Github](https://github.com/NuoJohnChen/AI_Conf_Crisis)]。

## 1. 核心概念：这个仓库在做什么？

从本质上讲，这个引擎在两个相反的方向上促进信息流动，以弥合抽象研究语境与具体学术文献之间的鸿沟：

### ➡️ Context2Papers（从想法到文献）
> *「我有一个 workshop 主题或论文草稿，请给我相关文献。」*

* **做什么：** 充当一个 **语义推荐引擎（Semantic Recommendation Engine）**。它不依赖简单关键词，而是接收一个长文本“语境”（例如 workshop 描述、具体段落或摘要），并从一个约 70 万篇论文的数据库中检索出语义上最相关的学术论文。
* **使用场景：** 帮助研究者为具体论断寻找引用文献，或帮助会议组织者为特定 workshop 主题寻找匹配的论文。

### ⬅️ Papers2Context（从文献到洞见）
> *「我有一堆论文（比如某个地区或某一年份的论文），请告诉我它们都在研究什么。」*

* **做什么：** 充当一个 **主题发现引擎（Topic Discovery Engine）**。它对一个经过过滤的论文集合（如「所有 2025 年新加坡作者发表的论文」）进行自动聚类，从中发现潜在主题、研究趋势以及对应的“语境”。
* **使用场景：** 帮助 **联邦区域枢纽（Federated Regional Hub）** 的组织者 [[Position]](#cite-position) 识别本地研究优势，以策划有针对性的 workshop（例如发现某个区域在「多模态大模型」方面论文密度很高）。

---

## 2. 技术实现

为实现上述功能，`app.py` 引擎依赖一条数据流水线以及若干特定的机器学习模型：

### 数据基础：`paperdb` 流水线
系统依赖 `paperdb/` 目录进行数据摄取。
* **产物：** 通过抓取 arXiv 元数据生成 `arxiv_merged.parquet`（包含标题、摘要、作者信息）和 `embeddings.npy`（预计算向量）。
* **效率：** 推荐引擎通过内存映射（memory-mapped）的方式加载这些文件，从而在无需对整个语料库做实时推理的情况下实现低延迟访问。

### 算法与模型
* **用于 Context2Papers：**
    * **召回（Specter2）：** 使用 `allenai/specter2_base` 将用户查询编码到与引文网络相同的向量空间中，利用 **余弦相似度（Cosine Similarity）** 快速检索候选文献。
    * **精排（LLM 重新排序）：** 为修复向量压缩带来的信息损失，使用 `Qwen/Qwen3-Reranker-8B` 根据与查询的全文语义匹配度对候选论文重新打分排序。

* **用于 Papers2Context：**
    * **聚类（BERTopic）：** 使用 **UMAP**（降维）和 **HDBSCAN**（密度聚类）构建流水线，自动发现自然形成的论文簇。
    * **标签生成：** 通过 **CountVectorizer** 提取基于类别的 TF-IDF 关键词，为这些聚类生成可读性较高的人类语义标签。

---

## 3. 在 XtraMCP 与 PaperDebugger 中的角色

本仓库作为 XtraMCP 架构中的 **Researcher 节点** 存在，实现了编排逻辑与推理能力的解耦 [[XtraGPT]](#cite-xtragpt)[[PaperDebugger]](#cite-paperdebugger)。

* **XtraMCP 的 Researcher：**
    * 作为 **Researcher 模块** 的执行层存在。当 XtraMCP 的控制层接收到用户意图（例如「查找相关工作」）时，会通过 Model Context Protocol（MCP）将请求路由到本后端。
    * **防幻觉机制：** 与可能捏造引用的标准大模型不同，本模块在 `paperdb` 生成的真实世界向量上执行确定性检索，确保每条推荐文献都真实存在且可验证。

* **对 PaperDebugger 的支持：**
    * 面向用户的 Overleaf 扩展 **PaperDebugger** [[PaperDebugger]](#cite-paperdebugger) 依赖本后端提供实时的「深度检索（Deep Research）」能力。
    * 通过将用户当前写作语境发送到本引擎的 **Context2Papers** 接口，系统会检索出相关文献，用于生成「相关性洞见（Relevance Insights）」，帮助作者为自己的工作找到合适定位。

---

## 4. 应对 AI 会议危机（CFC 模型）

当前中心化的 AI 会议模式因提交量指数级增长、知识传播效率低下而难以为继 [[Position]](#cite-position)。本代码库为提出的 **社区联邦会议（Community-Federated Conference, CFC）** 方案提供了关键技术基础。

* **缓解信息过载：** **Context2Papers** 支撑「科学使命（Scientific Mission）」中的高效知识交换，让研究者能够从成千上万的投稿中迅速切入最相关的内容。
* **赋能区域枢纽：** **Papers2Context** 对 **联邦区域枢纽** 组织者至关重要。通过按**地区，时间，主题**过滤数据库并执行主题发现，组织者可以识别本地主导研究主题。这一过程重建了「社区建设（Community Building）」这一支柱，使学术交流从匿名化的超级大会回归到具有真实社群纽带的本地化互动。

---

## 6 参考文献

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
