---
layout: doc
title: AI实战基础知识点梳理
date: 2025-01-15
category: llm
tags: [llm, AI, 实战, 基础, 知识体系, 项目实践]
excerpt: 系统梳理AI大模型与实际应用基础知识点，配套典型实战项目与技能要点。
permalink: /docs/llm/2025/ai-foundation-summary/
---

### 1 AI大模型基本原理及API使用
1. AIGC发展：从GPT1到GPT4
 - 什么是AI
 - 分析式AI与生成式AI
 - GPT是如何训练出来的
 - AIGC的表现与优势
 - AIGC的通用能力应用
 - CASE：上手车险反欺诈（基于ChatGLM）
1. 大模型API使用
 - 全球AI发展现状
 - CASE-情感分析-Qwen（掌握DashScope调用大模型）
 - CASE-天气Function-Qwen（了解Function Call）
 - CASE-表格提取-Qwen（了解多模态大模型）
 - CASE-运维事件处置-Qwen"	"- 理解AIGC与大模型的基本原理
- 理解分析式AI与生成式AI的区别
- 熟悉大模型API的调用方法
- 学会使用DashScope等平台调用大模型（如Qwen），完成实际任务（情感分析、表格提取等）
### 2 DeepSeek使用与Prompt工程	"1. DeepSeek使用
 - DeepSeek的创新
 - CASE：小球碰撞试验（Cursor + DeepSeek-R1）
 - DeepSeek私有化部署选择
 - Ollama部署DeepSeek-R1
 - API调用DeepSeek
1. Prompt工程：设计与优化
 - Prompt原理
 - 提示词策略差异
 - 提示词关键原则
 - 提示词编写框架（重要性排序）
 - 提示词编写技巧（限制格式、区分、小样本学习、CoT、角色扮演等）
2. Prompt工程实战
 - CASE：使用提示词完成任务
 - CASE：JSON格式返回
 - CASE：使用CoT分步骤推理
 - CASE：使用Prompt调优Prompt"	"- 了解DeepSeek的创新技术，熟悉其API调用、本地部署（Ollama）及私有化方案
- 理解Prompt工程的核心原理与优化策略
- 学习提示词的设计逻辑、关键原则及主流框架（如CoT、角色扮演等），掌握不同场景下的Prompt优化技巧。
- 通过案例实战（如JSON格式生成、分步骤推理、Prompt自优化），培养解决复杂问题的Prompt工程思维。"


### 3 Cursor编程-从入门到精通	"1. Cursor编程
 - 什么是Cursor Rules
 - Cursor的主要功能
1. Cursor编程实战
 - CASE：多张Excel报表处理
 - CASE：疫情实时监控大屏
2. Trae与CodeBuddy使用
 - Trae使用
 - CodeBuddy使用
 - CASE：多张Excel报表处理"	"‘- 掌握 Cursor 编程的核心概念与功能（如Cursor Rules，熟悉 Cursor 的主要功能等）
- 通过实际案例（如多张 Excel 报表处理、疫情实时监控大屏）掌握 Cursor编程
- 掌握数据可视化能力，使用Cursor可以搭建实时监控大屏
- 了解 Trae 和 CodeBuddy 等其他AI工具

## 模块2: AI大模型应用核心工具及技术

### 4 Embeddings和向量数据库
 1. CASE：基于内容的推荐
 - 什么是N-Gram
 - 余弦相似度计算
 - 为酒店建立内容推荐系统
 2. Word Embedding
 - 什么是Embedding
 - Word2Vec进行词向量训练
 3. 什么是向量数据库
 - FAISS, Milvus, Pinecone的特点
 - 向量数据库与传统数据库的对比
 4. Faiss工具使用
 - Case: 文本抄袭自动检测分析
 - 使用DeepSeek + Faiss搭建本地知识库检索
 - 掌握Embedding核心原理与应用

- 理解N-Gram、词向量（Word2Vec）和余弦相似度等核心概念，学会训练和使用词嵌入模型
- 了解掌握FAISS、Milvus、Pinecone等向量数据库的特点及适用场景
- 通过酒店内容推荐系统、文本抄袭检测等案例，学会基于Embedding和向量数据库构建实际应用
- 学习使用DeepSeek + FAISS构建本地知识库，实现高效语义检索与问答系统。"

### 5 RAG（Retrieval Augmented Generation）技术与应用
 1. 大模型开发的三种范式
 - 提示词工程
 - RAG技术
 - 模型微调
 2. RAG技术
 - 什么是RAG技术？它如何增强大模型的生成能力
 - RAG的核心原理与流程
 - NativeRAG
 - NoteBookLM使用
 3. CASE：DeepSeek +Faiss搭建本地知识库检索
 - PDF文本提取与处理
 - 向量数据库构建
 - 语义搜索与问答链
 - 文本块对应页码信息源
 4. 如何提升RAG质量
 - 数据准备阶段
 - 知识检索阶段
 - 答案生成阶段
- 掌握大模型开发的三大核心范式（提示词工程、RAG技术、模型微调）
- 学习RAG的核心原理（检索-增强-生成），掌握从数据准备、向量检索到答案生成的完整技术链路。
- 通过DeepSeek+FAISS案例，实现PDF文本处理、向量数据库构建及语义搜索问答链的开发。
- 学习如何提升RAG系统的召回和回答能力

### 6RAG高级技巧
1. RAG技术树
 - 模型不同阶段的RAG
 - RAFT方法：基于微调的RAG方法
2. RAG高效召回方法
 - 优化查询扩展
 - 双向改写
 - 索引扩展
 - Small-to-Big 
3. GraphRAG使用
 - GraphRAG 过程
 - 全局搜索
 - 局部搜索
 - GraphRAG使用
4. Qwen-Agent中的RAG
 - 级别一：检索
 - 级别二：分块阅读
 - 级别三：逐步推理
 - RAG评测结果
 - Qwen-Agent使用"	"- 了解RAG前沿技术体系（RAG技术树，包括模型阶段适配、RAFT微调方法等高级技术）
- 掌握查询扩展、双向改写、索引扩展等高效召回方法，提升检索精度与覆盖率
- 学习GraphRAG的全局/局部搜索机制
- 使用Qwen-Agent的三级RAG实现（检索→分块→推理）

### 7 Text2SQL：自助式数据报表开发
1. Text to SQL技术
- LLM模型选择
- 大模型API使用
- Function Call
- 搭建SQL Copilot
- LangChain中的SQL Agent
- 自己编写（LLM + Prompt）
2. 案例: 保险场景SQL Copilot实战"	"’- 学习大模型（LLM）在Text2SQL任务中的应用，掌握模型选型、API调用及Function Call等关键技术。
- 通过LangChain SQL Agent或自研方案（LLM + Prompt），实现自然语言到SQL的自动转换，提升数据查询效率。
- 结合保险行业需求，开发定制化SQL Copilot，完成从自然语言提问到精准SQL生成的端到端实现。"

### 8 LangChain：多任务应用开发
1. LangChain基本概念
 - Models, Prompts, Memory, Indexes, Chains, Agents
 - LangChain中的tools (serpapi, llm-math)
 - LangChain中的Memory
 - LECL构建任务链
2. LangChain实战
 - CASE：动手搭建本地知识智能客服（理解ReAct）
 - CASE：工具链组合设计 （LangChain Agent）
 - CASE：搭建故障诊断Agent（LangChain Agent）
 - CASE：工具链组合设计 （LCEL）
3. AI Agent对比"	"‘- 理解LangChain中的Models、Memory、Chains、Agents等核心组件
- 掌握SerpAPI、LLM-Math等Tools的集成方法，学会用LCEL构建复杂任务工作流。
- 通过知识客服、故障诊断等案例，实现具备记忆、推理和工具调用能力的智能体。
- 对比不同Agent架构优劣，具备根据业务需求选择技术方案的能力。"

### 9	Function Calling与智能Agent开发
1. Function Calling的概念与应用
 - 什么是Function Calling？
 - Function Calling 与 MCP的区别
2. Qwen3 Function Calling使用
 - Qwen3的特点
 - 使用Qwen3完成天气调用 Function Calling
3. 搭建业务助手
 - Qwen-Agent中的 Function Calling
 - 使用Function Calling完成数据库查询
 - 数据表可视化"	"’- 理解Function Calling的概念、原理及其与MCP的区别
- 学习Qwen3的特性，并通过实战案例（如天气查询）掌握其Function Calling的调用与集成方法。
- 基于Qwen-Agent，利用Function Calling实现数据库查询、数据可视化等业务功能，完成端到端开发。"


### 10 MCP与A2A的应用
1. MCP的概念与应用
 - 什么是MCP?
 - MCP 的核心概念 (MCP Host，MCP Client，MCP Server）
 - MCP 的使用场景
 - CASE：旅游攻略MCP
 - CASE：Fetch网页内容抓取
 - CASE：Bing中文搜索
 - CASE：桌面TXT统计器（MCP SDK使用）
2. A2A的概念与应用
 - 什么是Agent2Agent
 - A2A的关键组件
 - A2A的工作流程
 - A2A与MCP的关系
 - CASE：安排篮球活动（多智能体协作）"	"‘- 理解MCP的核心概念（Host/Client/Server）及使用场景，
- 通过旅游攻略生成、网页抓取、中文搜索、桌面TXT统计等案例，实现MCP的落地应用
- 学习Agent2Agent（A2A）的工作流程与关键组件，掌握多智能体协同的任务分配与决策方法
- 理解MCP与A2A的技术差异与互补关系"

### 11 Agent智能体系统的设计与应用
1. 智能体的定义与作用
 - 什么是AI Agent
 - AI Agent 的核心概念 (规划、记忆、工具调用等）
 - AI Agent框架对比（LangChain, LangGraph, Qwen-Agent, Coze, Dify）
 - 什么时候用Agent
 - AI Agent与工作流
2. 智能体分类
 - 反应式 Reactive
 - 深思熟虑 Deliberative
 - 混合式 Hybrid
3. 智能体实战
 - CASE：私募基金运作指引问答助手
 - CASE：智能投研助手
 - CASE：投顾AI助手"	"’- 深入理解Agent的规划、记忆、工具调用等核心模块，掌握主流框架（LangChain/Qwen-Agent等）的技术差异与选型标准。
- 学习反应式、深思熟虑式、混合式Agent的设计模式，能够根据业务需求选择最优架构。
- 通过私募基金问答、智能投研、投顾助手等案例，完成不同类型的Agent实现。"


### 12视觉大模型与多模态理解
1. VLM在行业中的应用
   - Qwen-VL使用
   - Qwen-VL微调
   - 医疗行业中的应用（病历提取）
   - 车险承保中的应用（车辆承保、危险驾驶识别、车辆损失评估、事故要素提取、车辆一致性校验）
2. 视频理解SOTA
   - InternVideo2/2.5 
   -  视频基础模型
   - InternVideo2 预训练
   - 视频多模态注释示例
   - CASE：汽车剐蹭视频理解
3.  MinerU使用
   - PDF 转 Markdown
   - 网页内容提取
   - MinerU的核心技术
   - MinerU的应用场景
   - MinerU使用（在线使用、客户端）
   - MinerU使用（API使用）
   - MinerU私有化部署"	"‘- 学习Qwen-VL等视觉大模型的微调与部署方法，掌握其在医疗、保险等领域的落地实践。
- 理解InternVideo2/2.5等视频基础模型的预训练与多模态注释技术，实现场景下的视频分析（如汽车剐蹭）
- 掌握MinerU的文档转换、内容提取及私有化部署能力，提升多模态数据处理效率"


### 13 Fine-tuning微调艺术
1.高效微调方法概述
   - 参数高效微调(Parameter-Efficient Fine-Tuning, PEFT)
   - 主要方法对比:Adapter, Prefix-tuning,LoRA
2.LoRA的数学原理
   - 低秩分解与矩阵近似
   - 两个低秩矩阵的作用与优化
3.微调数据准备
   - 数据质量与数量要求
   - 不同模型尺寸与场景的数据需求
4.硬件需求与显存计算
   - 微调显存估算方法
   - LORA显存优化与计算示例"	"-掌握高效微调技术
学习LoRA、Adapter等方法，显著降低训练成本,提升大模型微调效率。
- 深入理解LoRA数学原理
  掌握低秩矩阵分解的核心思想，灵活调整参数以适应不同任务需求。
- 优化数据准备策略
  了解数据质量、数量对微调效果的影响，合理规划数据集规模。
- 精准计算硬件需求
  学会估算显存占用，优化GPU资源，降低微调硬件门槛。"


### 14 Fine-tuning实操
1. 模型微调的方法
   - 如何模型微调（以ChatGLM为例）
   - 李飞飞 50美金 复刻R1模型
   - s1: Simple test-time scaling
2. Unsloth：LLM高效微调
   - CASE：qwen2.5-7B微调 (alpaca-cleaned)
   - CASE：训练垂类模型（中文医疗模型）
   - CASE：训练自己的R1模型
   - CASE：YAML配置助手（模型微调）
   - 打造金融垂类大模型（智能客服）"	"‘- 学习从基础微调（如ChatGLM）到高效优化技术（如Unsloth），掌握不同场景下的模型适配策略。
- 掌握低成本微调技术，如何用50美金复刻自己的R1
- 掌握中文医疗模型、金融客服模型等垂类大模型的训练与优化，解决行业特定需求"


## 模块3: Coze和Dify工作原理和应用技术
### 15 Coze工作原理与应用实例
1. Coze工作原理
 - Agent与Copilot的区别
 - 插件使用
 - 工作流使用
 - RAG知识库
2. Coze应用实例
 - CASE：AI新闻Agent
 - CASE：创建搜索新闻工作流
 - CASE：weather_news工作流（基于意图识别）
 - CASE：抖音文案提取&二创
 - CASE：LLM联网搜索
 - CASE：搭建古诗词Agent"	"- 理解Coze的核心机制：掌握Agent与Copilot的区别，熟悉插件、工作流及RAG知识库的应用逻辑。

- 掌握Coze实战技能：通过案例学习，独立完成新闻Agent、天气新闻工作流等场景的搭建与优化。
- 提升AI自动化效率：学会利用Coze实现联网搜索、文案提取与二创等任务，减少人工干预。

### 16 Agent进阶实战与插件开发
1. Agent进阶实战
 - 批处理
 - Coze应用
 - 数据表使用
 - 多Agents模式
 - 多工作流复杂应用
2. Agent应用实例
 - CASE：古诗词绘画（批处理）
 - CASE：智能投顾助手（风险评测与推荐）
 - CASE：客户分层营销助手
 - CASE：智能客服Agent
 - CASE：市场舆情监测Agent"	"- 掌握Agent高阶功能应用：深入理解批处理、数据表操作、多Agents协作及复杂工作流设计，提升自动化任务处理能力。

- 熟练开发Agent插件与集成：学习插件开发逻辑，实现与外部系统的数据交互，扩展Bot功能边界。
- 实战复杂场景解决方案：通过金融、营销、客服等真实案例，掌握多行业场景的AI应用设计与优化技巧。

### 17 Dify本地化部署和应用
1. Dify本地化部署
 - Dify开发平台
 - Docker Compose部署
 - 克隆Dify 代码仓库
 - 启动 Dify 服务
 - 访问 Dify
 - 如何使用 Dify
2. Dify应用实战
 - CASE：LLM联网搜索
 - CASE：搭建古诗词WorkFlow
 - CASE：智能客服ChatFlow
 - CASE：智能文档分析助手(MinerU+Dify)
3. 如何应用Agent API（Coze, Dify）
 - Coze API使用
 - Cozepy工具
 - Dify API使用
"	"- 掌握Dify本地化部署：学习通过Docker Compose部署Dify，完成代码克隆、服务启动及平台访问全流程。

- 熟悉Dify核心功能：理解Dify开发平台的操作逻辑，掌握WorkFlow、ChatFlow等工具的应用方法。
- 实战AI应用开发：通过联网搜索、古诗词生成、智能客服等案例，掌握基于Dify的AI解决方案搭建。
- 整合API扩展能力：学会调用Coze与Dify的API，实现跨平台自动化与功能扩展。


## 模块4: 进阶知识：机器学习与AI大赛

### 18 分析式AI基础
1. 分析式AI基础
 - 分析式AI与生成式AI
 - 十大经典机器学习算法
 - 常用分类算法
 - 贝叶斯分类器
 - 决策树与随机森林
 - SVM支持向量机
 - 逻辑回归
2. 项目实战
 - CASE：二手车价格预测
 - 分类与回归的关系
 - 数据探索
 - 特征选择
 - 模型训练与预测"	"- 理解分析式AI核心概念：掌握分析式AI与生成式AI的区别，认识分析式AI在预测和决策中的独特价值。

- 学习经典机器学习算法：深入理解十大经典机器学习算法，重点掌握分类、回归等核心算法的原理与应用场景。
- 掌握数据建模全流程：从数据探索、特征选择到模型训练与预测，完整学习分析式AI项目的开发流程。
- AI大赛实战能力：通过二手车价格预测等比赛实战，提升算法建模能力。


### 19	不同领域的AI算法	"1. 金融行业的应用场景
 - 银行不同部门的应用场景
 - 客户流失分析与预警
 - 因客定价
 - 长尾客群营销
 - 贷款商机挖掘
 - 商圈生意机会地图
 - 基于评分卡的风控模型开发
 - 期货套利模型
 - 资金流入流出预测
1. 制造行业的应用场景
 - 缺陷检测
2. 快消行业的应用场景
 - 供应链补货
3. AI大赛：二手车价格预测（进阶）"	"- 掌握行业AI应用场景：理解AI在金融、制造、快消等不同领域的典型应用场景及业务价值。

- 提升跨行业解决方案能力：通过缺陷检测、供应链优化等案例，培养将AI技术迁移至不同行业的能力。

- 实战进阶：通过二手车价格预测大赛项目，强化特征工程的实战经验。"									
### 20	机器学习神器	"
1. 预测全家桶
 - Project A：员工离职预测
 - Project B：男女声音识别
 - 分类算法：LR , SVM
 - 树模型：GBDT, XGBoost, LightGBM, CatBoost
2. 机器学习神器
 - 什么是集成学习
 - GBDT原理
 - XGBoost
 - LightGBM
 - CatBoost
 - AI大赛：二手车价格预测
 - 如何防止模型过拟合"	"- 掌握核心机器学习算法：系统学习分类算法（LR、SVM）和树模型（GBDT、XGBoost、LightGBM、CatBoost）的原理与应用。

- 实战项目驱动学习：通过员工离职预测、声音识别等案例，掌握从数据预处理到模型训练的全流程。

- 模型调优能力：通过二手车价格预测大赛，学习防止过拟合的策略，提升模型泛化能力。"									
### 21	时间序列模型	"时间序列分析
什么是时间序列预测
AR、MA、ARMA、ARIMA模型
使用ARMA/ARIMA对沪市指数进行预测
Project A：对沪市指数走势进行预测
Project B：资金流入流出预测"	"- 掌握主流的时间序列算法
- 会使用时间序列算法工具"									
### 22	时间序列AI大赛	"FaceBook时序分析工具prophet
CASE：页面流量预测
CASE：交通流量预测
基于Transformer的时序预测
Informer与FEDformer
AI大赛：资金流入流出预测"	- 参加AI大赛,使用时间序列算法									
	模块5: 深度学习应用与实战											
### 23	神经网络基础与Tensorflow实战	"1. 神经网络基础
 - 神经网络结构
 - 激活函数
 - 损失函数
 - 反向传播
 - 梯度下降
 - 优化方法（SGD、Adam）
 - 使用numpy搭建神经网络
2. TensorFlow实战
 - TensorFlow中的计算图与会话管理
 - 分布式训练与模型并行
 - TensorFlow Serving部署与推理
 - 使用Keras构建简单神经网络
 - 使用Keras进行二手车价格预测"	"- 掌握神经网络核心原理：理解神经网络的基本结构、激活函数、损失函数及优化方法，能够手动实现简单神经网络。

- 熟练使用TensorFlow框架：掌握TensorFlow计算图、会话管理及分布式训练，学会用Keras快速构建和训练模型。

- 培养工程化思维：学习模型部署（TensorFlow Serving）和分布式训练技巧，提升工业级AI项目的落地能力。"									
### 24	Pytorch与视觉检测	"1. PyTorch的基本概念
 - PyTorch的张量与自动求导机制
 - PyTorch的动态图与静态图
2. 构建与优化深度学习模型
 - 如何使用PyTorch构建神经网络
 - 常见的优化技巧与调参方法
3. PyTorch的分布式训练
 - 在多个GPU上进行训练
 - 使用PyTorch Lightning简化模型训练
4. 视觉检测模型 YOLO
 - 从Yolov1到Yolov12
 - ultralytics：基于pytorch的视觉检测工具
 - Project：钢铁表面缺陷检测"	"- 掌握PyTorch核心机制：理解PyTorch张量计算、自动求导及动态图特性，熟练构建和调试深度学习模型。

- 掌握模型优化与分布式训练：掌握神经网络构建、调参技巧及多GPU训练方法，能用PyTorch Lightning提升开发效率。

- 视觉检测实战：了解YOLO算法，掌握使用Ultralytics工具，完成工业级缺陷检测项目。"									
模块6: AI大模型应用落地实战												
### 25	"项目实战：企业知识库
（企业RAG大赛冠军项目）"	"1. 企业RAG大赛：搭建RAG知识库
 - RAG冠军方案（多路由+动态知识库）
 - RAG比赛任务说明
 - 基础RAG系统流程
 - 解析模块、Docling优化、表格序列化
 - 内容提取（ingestion）
 - 检索（Retrieval）
 - LLM 重排序 (LLM reranking)
 - 父页面检索
 - 整合后的检索器
 - 增强 (Augmentation)
 - 生成 (Generation)
 - 思维链、结构化输出、思维链+结构化输出
 - 指令细化 (Instruction Refinement)
 - 提示词创建、Prompt.py 实现
 - RAG系统调参
2. 搭建自己的RAG系统
 - 选择适合的LLM和Embedding模型 
 - MinerU使用
 - 更新中文知识库、设置相关的问题清单
 - 针对开放式的问题，进行Prompt设置
 - 搭建前端页面，比如使用 streamlit"	"- 掌握RAG核心技术：深入理解RAG（检索增强生成）系统的工作原理，包括数据预处理、检索优化、生成增强等核心模块。

- 实战冠军方案复现：学习RAG竞赛优胜方案，掌握多路由、动态知识库等高级技巧，并能优化基础RAG流程。

- 搭建完整RAG系统：从零构建企业级知识库，涵盖模型选型、数据处理、Prompt优化到前端部署的全流程。


### 26 项目实战：交互式BI报表（AI量化交易助手）
1. ChatBI功能设计
 - 自然语言查询：
  用户输入“查询2024年全年贵州茅台的收盘价走势”
 - 对比分析：
  用户输入“对比2024年中芯国际和贵州茅台的涨跌幅”
 - 趋势分析：
  用户输入“预测贵州茅台未来7天的收盘价”
 - 异常点分析
  用户输入“检测贵州茅台近一年超买超卖点”
 - 新闻查询
  用户输入“贵州茅台最近的热点新闻”
2. 数据可视化与交互优化
 - 柱状图：对比分析多个维度的表现
 - 折线图：对历史数据进行趋势呈现
 - 交互设计：支持用户自定义时间范围（如最近1个月、3个月、1年）
3. 智能分析功能
 - 趋势预测：ARIMA模型（未来N天价格预测）
 - 异常检测：布林带分析（超买/超卖点识别）
 - 周期性分析：Prophet模型（趋势分解）
4. 知识库管理
- few shot：撰写few shot示例，提供大模型处理逻辑的准确性"	"- 掌握交互式BI系统开发：学习如何构建支持自然语言查询的BI系统，实现从数据采集到可视化分析的全流程开发。
- 熟练应用智能分析模型：掌握ARIMA、布林带、Prophet等模型在智能分析中的实际应用。
- 优化数据交互体验：设计高效的数据查询与可视化方案，提升用户与系统的交互体验。
- 提升问题解决能力：通过实战项目，培养从需求分析到技术实现的完整问题解决能力。

### 27 项目实战：AI智慧运营助手（百万客群经营）	"1. 智能客户洞察系统
 - 多模态客户画像
 - 动态标签生成：利用大模型生成客户动态标签（如“高净值但风险敏感型”等）。
 - 智能分析细化：基于智能分析模型（分类、聚类等）进一步细化客户画像，为后续精准营销提供更丰富的客户特征维度。
1. 企业知识库引擎
 - RAG增强知识库：营销话术库构建
 - 大模型解释：利用SHAP分析，对个体客户的经营决策进行解释，增强客户经理执行的说服力
2. 智能营销Agent（多Agent协作架构）
 - 分析Agent：价值判断，调用决策模型（如逻辑回归、决策树、LightGBM等）判断客户价值。
 - 推荐Agent：个性化方案生成，结合知识库（包括SHAP分析结果等）生成个性化方案
 - 话术Agent：沟通脚本生成，实时生成沟通脚本，融入关联分析（Apriori）得出的产品组合推荐话术。
3. 可视化大屏搭建与对话式BI系统
 - 可视化大屏搭建：展示百万经营关键指标
 - 对话式BI系统：客户经理询问具体个体客户信息，通过多Agent协作，完成分析和土建"	"- 掌握AI驱动的客户洞察技术：学习多模态客户画像构建与动态标签生成，利用大模型和智能分析模型（分类、聚类等）细化客户特征，提升精准营销能力。

- 构建企业级知识库与决策解释系统：通过RAG增强知识库，结合SHAP分析，实现营销话术优化与客户经营决策的可解释性，增强执行说服力。
- 设计智能营销Agent协作架构：掌握多Agent（分析、推荐、话术）协同工作流，结合决策模型（如LightGBM）和关联分析（Apriori），实现自动化客户价值判断与个性化方案生成。
- 实现可视化大屏与智能AI交互：搭建可视化大屏监控关键指标，开发对话式AI系统，支持客户经理通过自然语言交互快速获取客户洞察与营销建议。
所用技术：RAG、Function Calling、向量数据库、Prompt工程、Agent技术运用

### 28 项目实战：AI搜索类应用（知乎直答）
1. AI搜索架构设计
 - 数据存储与管理：选择合适的数据库（关系型与非关系型）
 - 检索引擎：使用Faiss实现高效文本检索与索引
 - 语义检索：基于DeepSeek等模型进行语义匹配

1. 数据准备与索引构建
 - 数据清洗与标准化：处理原始数据以提升搜索质量
 - 数据索引：构建高效索引支持快速检索
 - 向量化处理：使用NLP模型将文本数据转换为向量

2. 搜索模块开发
 - 基于关键词的检索：实现传统关键词匹配
 - 基于语义的检索：通过语义模型提高检索准确性
 - 混合检索：结合关键词与语义匹配，提升搜索效果

3. 生成模块与个性化推荐
 - 个性化推荐：根据用户行为数据提供推荐结果
 - 搜索结果优化：基于大模型（如DeepSeek）生成结果摘要与建议

4. 前端开发与用户体验
 - 界面设计：创建直观、简洁的搜索界面
 - 搜索与展示：优化结果展示，支持筛选与排序
 - 实时反馈：实现用户输入提示和即时搜索响应"	"- 掌握AI搜索应用的核心技术与架构设计
- 学会数据清洗、索引构建及语义检索的开发方法
- 能够实现个性化推荐与优化搜索结果质量
- 掌握搜索界面的设计与优化,为用户提供优质体验

 所用技术:Faiss、向量数据库、Prompt工程、RAG术、Agent技术运用