---
title: '开源 vino-server：基于 OpenVINO 的本地 AI 模型服务端'
layout: post
subtitle: '纯 C++20 实现，零运行时依赖'
tags:
  - C++
  - OpenVINO
  - AI
---

## 什么是 vino-server?

vino-server 是一个基于 Intel OpenVINO 的轻量级本地 AI 模型服务端，使用纯 C++20 编写。它旨在为本地部署的 AI 模型提供简洁、高效的 HTTP API 服务接口，让你能够在普通 PC 上轻松运行大语言模型（LLM）、文本嵌入模型和 RAG 知识库问答。

项目地址：[https://github.com/jadedrip/vino-server](https://github.com/jadedrip/vino-server)

## 核心特性

* **纯 C++20 实现**：单二进制文件，零运行时依赖，无需 Python 环境
* **基于 OpenVINO**：充分利用 Intel CPU、集成显卡（iGPU）和 NPU 硬件加速
* **三大核心功能**：
  * 文本嵌入（Text Embedding）
  * LLM 对话与文本生成
  * RAG 知识库问答
* **API 兼容**：同时支持 OpenAI 兼容格式和 Ollama 兼容格式
* **模型自动发现**：自动扫描并加载指定目录下的模型文件
* **配置热更新**：修改配置文件后无需重启即可生效
* **MIT 开源协议**：自由使用和修改

## 为什么选择 vino-server？

### 轻量高效

相比于 Ollama、llama.cpp 等需要 Python 或复杂运行时的方案，vino-server 只是一个单独的可执行文件。编译后直接运行，不需要安装任何依赖。

### Intel 硬件加速

通过 OpenVINO 后端，vino-server 可以充分调用 Intel 处理器的硬件加速能力：

* **CPU**：利用 AVX-512 等指令集加速推理
* **iGPU（核显）**：Intel Iris Xe 等集成显卡提供额外算力
* **NPU**：Intel Core Ultra 处理器内置的 AI 加速单元

### 双协议兼容

vino-server 同时提供两种 API 格式：

1. **OpenAI 兼容格式**：`/v1/chat/completions`、`/v1/embeddings`
2. **Ollama 兼容格式**：`/api/chat`、`/api/embeddings`

这意味着你可以无缝替换现有应用中的 OpenAI 或 Ollama 后端地址，无需修改客户端代码。

## 快速上手

### 编译

```
git clone https://github.com/jadedrip/vino-server.git
cd vino-server
cmake -B build
cmake --build build -j$(nproc)
```

### 运行

```
./build/bin/vino-server
```

服务默认监听在 `http://localhost:8080`。

### 模型放置

将下载好的 OpenVINO IR 模型（`.xml` + `.bin` 文件）放入模型目录，vino-server 会自动发现并加载它们。

## 使用示例

### 对话补全（OpenAI 格式）

```
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "your-model-name",
    "messages": [
      {"role": "user", "content": "你好，请介绍一下你自己"}
    ]
  }'
```

### 文本嵌入

```
curl http://localhost:8080/v1/embeddings \
  -H "Content-Type: application/json" \
  -d '{
    "model": "your-embedding-model",
    "input": "这是一段示例文本"
  }'
```

### RAG 知识库问答

vino-server 内置了 RAG 功能，可以基于你提供的文档进行知识库问答。将文档放入知识库目录后，即可通过 API 进行检索增强生成。

## 适用场景

* **本地开发测试**：在不依赖云服务的情况下开发和调试 AI 应用
* **隐私敏感场景**：数据不出本机，适合处理敏感信息
* **边缘设备部署**：在资源受限的设备上运行 AI 推理
* **学习与研究**：了解 OpenVINO 推理流程和 LLM 服务架构设计

## 技术栈

* 语言：C++20
* 推理引擎：OpenVINO 2026.3.0.0
* HTTP 服务：内置轻量级 HTTP 服务器
* 构建系统：CMake

## 总结

vino-server 为想要在本地运行 AI 模型的开发者提供了一个简单、高效的解决方案。无论你是想在自己的电脑上跑 LLM、做文本向量化的 RAG 应用，还是需要一个兼容 OpenAI/Ollama API 的本地后端，vino-server 都值得一试。

欢迎 Star 和贡献代码！
