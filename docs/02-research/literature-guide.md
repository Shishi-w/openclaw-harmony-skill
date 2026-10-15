# 论文研究指南

> 本文档教大家"怎么找论文、怎么读论文、怎么写综述"，统一团队的文献工作方法。
> 每位成员的论文阅读笔记放在 `docs/02-research/papers/`

## 1. 研究方向（论文检索关键词地图）

围绕本项目，以下建议主题供参考：

| 主题 | 中文关键词 | 英文关键词 |
| --- | --- | --- |
| 智能体与技能 | 大模型智能体、工具使用、技能库、MCP | LLM agent, tool use, function calling, MCP, skill library |
| 可移植性/虚拟化 | 跨平台运行、虚拟机、中间表示、沙箱 | cross-platform, VM, intermediate representation, sandbox, WASM |
| 鸿蒙与国产OS | 鸿蒙、HarmonyOS、方舟、盘古 | HarmonyOS, ArkTS, ArkCompiler, Pangu |
| 模型工具协议 | 工具调用协议、插件系统、提示词工程 | tool-use protocol, plugin, prompt engineering |
| Skill 规范 | 技能规范、AgentSkills、ClawHub | AgentSkills, skill schema, skill registry |

> 参考：本项目直接相关的规范是 [AgentSkills](https://agentskills.io) 与 OpenClaw/ClawHub 文档。

## 2. 到哪里找论文

- **顶会/期刊**（按领域）：
  - AI/ML：NeurIPS, ICML, ICLR, ACL, EMNLP, AAAI, IJCAI
  - 系统/软件：OSDI, SOSP, ASPLOS, ICSE, FSE, ASE
  - 移动/嵌入式：MobiSys, SenSys, IEEE TMC
- **预印本**：arXiv（用关键词检索，盯 `cs.AI` / `cs.SE` / `cs.MA` 分类）。
- **检索工具**：
  - Google Scholar / Semantic Scholar / DBLP
  - arXiv 站内搜索
  - 必应学术 / 中国知网 CNKI（中文综述）
- **从文献到文献**：读一篇好综述（survey）→ 从它的引用列表反向扩充；再看它的被引（citations）看后续发展。


## 4. 精读方法（SQ3R 简化版）

- **S（Survey）**：先扫标题、摘要、图表、结论。
- **Q（Question）**：读前写下"我想从中得到什么"（例如：它的工具抽象接口怎么设计的？）。
- **R（Read）**：带着问题精读方法与实验。
- **R（Recite）**：合上文档，用自己的话复述核心思想——写进笔记。
- **R（Review）**：与已有笔记关联（哪些支撑我们方向A？哪些启发方向B？）。

> 写笔记不是翻译摘要，而是"用自己的话 + 对接我们的研究问题"。先引用来源（标题/作者/年份/链接），再写自己的理解、评价、可复现性、对项目的启示。

## 5. 文献综述（survey）组织方法
在 `docs/02-research/` 维护 `survey-*.md`，按下述结构持续累积：

1. **引言**：领域背景与本研究动机。
2. **分类框架**：把读过的论文按主题归类（见第 1 节），形成一个表格：论文/年份/主题/核心思想/方法/对项目的启示/可复现性。
3. **每类小结与差距分析**：哪些已解决，哪些是空白（→ 支撑我们的贡献点）。
4. **参考文献**：规范化引用（用 BibTeX 或 Markdown 链接）。

> 综述文档长期演化，每周例会更新一次"新增论文"。

## 6. 复现与溯源

- 论文若给出开源代码，在 `paper-reading-template.md` 里记录 repo 链接，并尝试在本地跑通（视为一次方法复现）。
- 复现结果（成功/失败/坑点）记录到 `docs/02-research/repro/` 下对应论文文件中，作为"方法复现台账"。
- 定期组织"复现分享"，把可复现的成果沉淀为代码骨架（方向 A/B 实现可参考）。

## 7. 论文文件命名约定

```
docs/02-research/papers/
  <年份>-<第一作者>-<主题缩写>.md   例如 2024-smith-functioncalling.md
```

在文件顶部 frontmatter 填写：标题、作者、年份、源（arXiv/会议）、链接、关键词、相关性评分（1-5）。
