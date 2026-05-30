# CyberSec-Agent-Assistant

## 项目名称

**CyberSec-Agent-Assistant**：网络安全知识检索与告警研判 Agent 原型

## 项目定位

本项目是一个基于 LangGraph 实现的轻量级网络安全 AI Agent 原型。面向本地模拟环境，不调用真实扫描器、不访问真实网络目标、不执行攻击行为，仅用于演示 AI Agent 在网络安全辅助分析场景中的任务理解、工具调用、知识检索、告警研判和报告生成能力。

## 项目背景

随着 AI Agent 在网络安全领域的应用不断深入，Agent 在执行安全分析任务时需要调用多种工具、访问安全知识库、分析告警日志并生成处置建议。这一过程对 Agent 的任务编排、工具调用治理、权限控制和操作审计提出了新的要求。

本项目作为创新资助项目“面向网络安全领域 AI 智能体的身份和访问管理研究”的前期基础原型，重点验证 Agent 的基础工程能力，为后续深入研究 Agent IAM 问题提供实践基础。

## 为什么使用 LangGraph

- **显式工作流编排**：StateGraph 允许以节点和边的形式明确定义 Agent 的工作流，便于理解、调试和审计每一步决策过程。
- **状态管理**：通过 TypedDict 定义共享状态，节点间数据传递清晰可控。
- **可扩展性**：节点、条件边、子图等机制支持从小型原型到复杂生产级 Agent 的平滑演进。
- **社区生态**：LangGraph 是 LangChain 生态中的核心框架，有丰富的工具集成和社区支持。

## 系统架构

```
用户输入 (user_query)
    │
    ▼
┌─────────────────────────────────────────────┐
│          LangGraph StateGraph               │
│                                             │
│  receive_user_query ──► classify_task       │
│                               │              │
│                               ▼              │
│                         plan_tool_call       │
│                               │              │
│                               ▼              │
│                      retrieve_knowledge      │
│                               │              │
│                               ▼              │
│                         execute_tool         │
│                               │              │
│                               ▼              │
│                       generate_answer        │
│                               │              │
│                               ▼              │
│                       write_tool_log         │
│                               │              │
│                               ▼              │
│                       final_response         │
└─────────────────────────────────────────────┘
    │
    ▼
最终回答 + 审计日志 (tool_calls.jsonl)
```

### 核心模块

| 模块 | 文件 | 职责 |
|------|------|------|
| 状态定义 | `state.py` | 定义 AgentState TypedDict |
| 工作流编排 | `graph.py` | LangGraph StateGraph 构建与节点实现 |
| 任务规划 | `planner.py` | Mock planner，基于关键词进行任务分类和工具选择 |
| 本地工具 | `tools.py` | 5 个本地模拟工具（CVE 查询、攻击类型检索、日志分析、报告生成、工具列表） |
| 知识检索 | `rag.py` | 基于关键词的本地知识库检索 |
| 报告生成 | `report_generator.py` | 安全事件研判报告生成 |
| 审计日志 | `audit_logger.py` | 工具调用过程 JSONL 日志记录 |
| Demo 运行 | `demo.py` | 5 个演示场景的批量运行 |
| 测试 | `test_demo.py` | 13 个测试用例 |

## 工作流说明

Agent 工作流包含 8 个节点，按以下顺序执行：

1. **receive_user_query**：接收用户输入，初始化会话 ID。
2. **classify_task**：基于关键词判断任务类型（cve_query / alert_analysis / attack_type_lookup / report_generation / general_security_question）。
3. **plan_tool_call**：根据任务类型选择需要调用的工具和参数。
4. **retrieve_knowledge**：从本地知识库检索相关内容。
5. **execute_tool**：执行所选本地模拟工具。
6. **generate_answer**：汇总工具输出和知识库内容，生成结构化回答。
7. **write_tool_log**：将本次工具调用过程写入 JSONL 审计日志。
8. **final_response**：输出最终结果。

## 本地知识库说明

| 文件 | 内容 | 条目数 |
|------|------|--------|
| `knowledge_base/cve_samples.json` | 模拟 CVE 漏洞信息（Log4Shell, EternalBlue, MOVEit, XZ Utils, BlueKeep） | 5 条 |
| `knowledge_base/attack_types.md` | 常见攻击类型说明（SSH 暴力破解、SQL 注入、XSS、命令注入、横向移动、WebShell、凭证窃取、可疑外联、可疑 PowerShell） | 9 类 |
| `knowledge_base/security_logs.json` | 模拟安全日志（SSH 登录失败、SQL 注入请求、PowerShell 编码命令、文件上传、外联连接） | 5 条 |

注意：以上数据仅用于本地演示，不需要访问外部数据源。

## 工具列表说明

| 工具 | 功能 | 输入 | 输出 |
|------|------|------|------|
| `search_cve_info` | 查询 CVE 漏洞信息 | cve_id | CVE 详情（名称、严重等级、影响、缓解措施等） |
| `lookup_attack_type` | 检索攻击类型说明 | keyword | 攻击类型描述、风险等级、处置建议 |
