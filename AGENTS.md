# 开发规则

## 产品范围

- 本项目用于以可复现方式发现和度量数学研究主题。第一阶段语料聚焦代数与表示论，并以 arXiv 的 `math.RT` 为主要入口。
- 不得使用 OpenAI API、付费 LLM API、Google Scholar 爬取，或让 LLM 判定研究主题是否“热门”。后续如加入语言模型功能，必须离线、显式配置，并与产生证据的分析 pipeline 分离。
- 在研究主题发现 MVP 完成前，不实现导师推荐、个性化推荐或自动生成研究报告等后续功能。

## 先调研，后实现

- 新增任何 pipeline 组件前，必须在 `docs/research/` 记录一份针对性的技术调研。调研至少包含官方文档、可行的开源实现（如有）、许可证、维护状态、资源需求和推荐结论。
- 优先复用维护良好的 library，不要重复实现已有的标准算法。若必须自行实现，应记录原因。
- 对影响分析结论的算法，应在设计文档中引用方法论文或一手文档，并在运行 manifest 中记录全部参数。

## 成本、许可证与性能

- “免费运行”是硬性要求。优先采用开放数据、本地开源软件和可自行部署的存储；未经明确批准，不得引入付费 API、SaaS 依赖或需要付费的许可证。
- 性能优先，但不得牺牲正确性、可复现性和可解释性。在替换简单实现前，应使用有代表性的 `math.RT` 数据进行基准测试。
- 优先使用流式/分页采集、受限并发、指数退避重试、可恢复 checkpoint、批处理、Parquet 和稀疏矩阵。不得对持续增长的论文语料物化全量两两稠密相似度矩阵。
- 优先选用 MIT、BSD、Apache-2.0、ISC 和 PostgreSQL License。添加依赖前，审查并记录准确版本的许可证及传递依赖影响；未经明确批准，不得将 GPL/AGPL 组件作为核心运行时依赖。
- 遵守 API rate limit、robots.txt、服务条款和数据许可。每个 source connector 都必须保存政策 URL 和审查日期。

## 默认技术选择

- arXiv 用于数学语料元数据；OpenAlex 用于论文、作者、机构、引用和参考文献的补充。所有来源都是观测值，不得静默覆盖来源特有事实。
- 实体解析优先使用确定性 ID（arXiv ID、DOI、来源 ID）。模糊匹配必须保存候选证据、分数、方法、阈值、决策和审核状态。
- 数学文本规范化使用 `pylatexenc` 加版本化项目规则；不得以未测试的纯正则方式删除 LaTeX。
- 以稀疏 TF-IDF/n-gram 与稀疏 top-k cosine similarity 作为可解释 baseline；bibliographic coupling 从参考文献边构建。
- citation、文本相似度、bibliographic coupling 和 topic 融合图必须分别保存为带版本的派生数据集。
- 在引入 embedding 或 GPL/AGPL 图算法前，先采用确定性且许可证宽松的社区发现 baseline。topic 标签必须来自有分数的术语和代表性论文，不得来自 LLM。

## 可复现性与数据来源追踪

- 保留不可变的原始 source payload（或其内容寻址位置）、source ID、抓取时间、请求参数、source version 和 ingestion run ID。
- 严格分离 raw、staged、canonical 和 derived 数据。不得修改或删除原始来源记录来表达一次规范化或合并决策。
- 每个派生数据集必须有 run manifest，其中包括输入 snapshot ID/hash、代码 commit、配置 hash、依赖版本、随机种子、算法名称/版本、参数、时间戳和输出 hash。
- 对 corpus definition、normalization rules、matching policy、feature configuration、graph configuration、cluster run、topic label 和 metric definition 进行版本化。结果必须可由保存的 manifest 复现。
- 每个展示指标必须同时保存 coverage。缺失引用、机构、摘要或已解析作者 ID 时，绝不能展示为零活跃度。

## 指标与解释

- publication volume、yearly volume、growth、recent activity、active researchers、new entrants、citation velocity、active institutions 和 institution diversity 必须独立保存。未经后续明确设计批准，不得生成单一“热度”分数或排行榜。
- 每个指标必须声明日期口径（首次预印本日期或发表日期）、时间窗口（1/3/5/10 年）、分子、分母、平滑规则和 coverage。不得在同一时间序列混用预印本和发表日期。
- cluster 应表述为在具名语料和配置下由算法发现的论文簇，不能表述为被普遍认可的、绝对的研究领域。

## 质量门槛

- 为解析、规范化、确定性匹配、图边构建、指标公式和 provenance manifest 编写测试。使用小型、版本化 fixture；单元测试不得调用实时外部 API。
- 发布 topic run 前，必须保存 source coverage 检查、去重解析抽样、cluster coherence 审阅、cluster size 分布检查，以及随机种子/参数稳定性检查。
- 在输出和日志中明确展示假设、排除项、失败记录和未解决的匹配。没有证据时，应返回明确的 `unknown`，而不是推断一个值。

## 仓库工作流

- 将 connectors、identity resolution、normalization、features、graphs、clustering、metrics、validation 和 provenance 保持为独立模块，并定义类型化输入/输出契约。
- 配置必须是声明式的并纳入版本控制。不得提交 API key、本地路径、大型原始数据集或其他 secret。
- 改变解释或统计口径的行为时，更新相关设计文档和数据/指标定义；重要的数据源、许可证、schema 或算法决策须新增 ADR。
- 提交前运行与改动文件相关的测试和静态检查；最终报告应列出准确命令及任何环境限制。
