# Auto-Fix-Agent-Core
解决排查陈旧 Bug、阅读过时的 API 文档以及编写重复的单元测试问题，利用 Cursor 为主阵地，结合开源框架构建了一个“多智能体自动化缺陷修复与代码同步流水线”
# 🤖 Auto-Fix-Agent-Core

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Status](https://img.shields.io/badge/status-Beta-orange)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)

> **Auto-Fix-Agent-Core** 是一个基于多智能体协作（Multi-Agent Collaboration）的自动化代码缺陷修复流水线。通过接入 GitHub Webhook，它能够自动监听 Issue 和异常日志，进行长链推理、全局上下文检索、代码修复以及闭环沙盒测试，最终自动生成 Pull Request。

## ✨ 核心特性 (Key Features)

*   **⚡ 自动化响应**：基于 FastAPI 和 Pydantic 构建的高性能 Webhook 监听服务，秒级响应代码库的新增 Issue 和崩溃日志。
*   **🧠 多模型异构协同**：
    *   **Root Cause Agent (DeepSeek)**：专攻长链推理，深度解析错误堆栈，生成修复假设。
    *   **Retrieval Agent (Gemini)**：利用超大上下文窗口结合 RAG（检索增强生成），精准定位相关代码片段与 API 文档。
    *   **Coder Agent (Claude)**：遵循严格的架构规范生成修复代码与单元测试。
*   **🕸️ 状态图编排与记忆**：底层基于 LangGraph 构建 StateGraph，精确控制不同 Agent 之间的流转逻辑（Tool nodes / LLM nodes），并采用 SQLite 进行图状态的持久化（State Persistence），确保长周期任务不会中断。
*   **🔍 混合检索架构**：集成了 Milvus 向量数据库与 BGE-M3 嵌入模型，对代码库进行语义级切块（Chunking）与混合检索（Dense + Sparse Vector），解决大型单体仓库上下文溢出的痛点。
*   **🔄 闭环验证引擎**：在本地沙盒环境中自动执行 `pytest` / `npm test`，若测试失败，自动提取报错日志作为 Prompt 触发 Agent 重试修正（最大重试 3 次）。

## 🏗️ 架构概览 (Architecture)

```mermaid
graph TD
    A[GitHub Issue / Error Log] -->|FastAPI Webhook| B(DeepSeek: 根因分析 Agent)
    B -->|长链推理 & 假设生成| C(Gemini: RAG 检索 Agent)
    
    subgraph 混合检索系统 (Hybrid Search)
    C <--> |BGE-M3 Embedding| M[(Milvus Vector DB)]
    C <--> |AST / Metadata| S[(Codebase)]
    end
    
    C -->|高维上下文| D(Claude: 代码生成与修复 Agent)
    D -->|生成代码 + 单测| E{本地沙盒验证}
    E -->|失败: 提取报错| D
    E -->|成功: 100% Pass| F[自动创建 GitHub PR]
    
    style B fill:#e6f7ff,stroke:#1890ff
    style C fill:#f6ffed,stroke:#52c41a
    style D fill:#fff0f6,stroke:#eb2f96
    style M fill:#fffbe6,stroke:#faad14
