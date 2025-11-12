#  C++ AI应用开发项目 - AI应用服务平台 第二版

> **本项目目前只在[知识星球](https://programmercarl.com/other/kstar.html)答疑并维护**。

在9月份我们发布了[C++AI应用服务平台（第一版）](https://programmercarl.com/other/project_http_ai.html) 这个项目。

当然刚发布，第二版就已经在路上了。

**现在AI应用服务平台第二版（C++），正式发布**！

这次，在自研 [C++ HTTP 框架](https://programmercarl.com/other/project_http.html)上，**把多模型对话、RAG、轻量级 MCP、ASR/TTS、图像识别、消息队列异步化、会话多租户化 全部落地，并用策略模式 + 注册式工厂把“接什么模型、怎么调用、能否用工具”彻底解耦**。

第二版不是简单加功能，而是把AI 应用工程化做深做透。

相对于[第一版](https://programmercarl.com/other/project_http_ai.html)，第二版我们优化了这些内容。

![一图看懂优化点](https://file1.kamacoder.com/i/web/2025-10-14_16-51-35.jpg)

## 第二版核心技术点：

- **多模型适配（Strategy + Factory）**：统一抽象 `AIStrategy`，一键切换 **阿里百炼 / 百炼-RAG / 豆包 /（预留）本地 LLaMA/llama.cpp、GGUF**。
- **轻量级 MCP 思想落地**：通过 **配置化工具注册（`AIToolRegistry`）+ Prompt 协议化** 实现“模型判断→工具调用→二次回答”的 **两段式推理**，对齐 **Model Context Protocol** 的核心理念。
- **RAG 检索增强**：解析→分块→嵌入→ANN 检索（Faiss/Milvus 预留）→可选重排→**带引用回答**，支持 **知识库 ID** 配置化接入。
- **多会话管理**：从 **单用户单会话** 升级为 **单用户多会话**，`unordered_map<userId, map<sessionId, AIHelper>>` 精准隔离上下文。
- **语音链路（ASR/TTS）**：集成 **百度 TTS**（任务创建→轮询→回传 URL），ASR 接口封装预留；支持 **参数化语速/音色**。
- **异步化与可靠性**：**RabbitMQ** 承载持久化写库，前台 **同步写内存、异步入库**，避免主线程阻塞；幂等/重试机制可扩展。
- **全链路可维护**：`AIHelper` 重构，**对话/模型切换/消息入库** 一步到位；**配置驱动（`config.json`）** 管理工具清单与 Prompt 模板。
- **容器化交付**：**独立 v1/v2 Docker 镜像**，MySQL + RabbitMQ 一键拉起；环境一致、上手即跑。

## 本项目视频演示

![image](https://file1.kamacoder.com/i/web/2025-11-07_11-28-19.jpg)

![image](https://file1.kamacoder.com/i/web/2025-11-07_11-28-53.jpg)

![image](https://file1.kamacoder.com/i/web/2025-11-07_11-29-21.jpg)

## 为什么市面上没有C++ AI应用项目？

大家会发现市面上，很少有 C++ AI应用开发的项目教程。

因为C++ 没有成熟的 AI 框架封装（如 LangChain、FastAPI 那种现成的 SDK）。

相对于Java ，**Spring AI 在 Spring Boot 基础上已经封装的 各种AI 应用层框架**。

可以一行配置 即可接入 ChatGPT、Claude、通义、文心等；

还支持 Prompt 模板、工具调用（Function Calling）、RAG、向量检索；内置安全、配置、日志、监控体系；等等


还能无缝集成 Spring Cloud、Spring Security、Redis、MySQL。

换句话说： Spring AI 把“大模型调用”当作一种新的 Bean，让 Java 工程师能像调接口一样玩 AI。

大家看过的 不少 包装了各种高大上的名字的java项目，其实就是在 Spring AI 里的一个配置而已。

而C++ 没有这种生态，以至于，大家在网上 基本找不到 C++ ai应用开发的教程。

因为啥都要自己写，一步一步自己造轮子，难度就上了一个台阶。


## 架构图

![](https://file1.kamacoder.com/i/web/2025-10-15_16-19-50.jpg)

架构图展示了 自研 [C++ HTTP 服务框架](https://programmercarl.com/other/project_http.html) 如何将 AI 模型调用、图像识别、消息队列、数据库存储与多厂商模型 API 进行解耦，实现了高性能、可扩展、可私有化部署的 AI 应用平台。

整个系统从上到下可分为四层：

* 客户端层	用户通过 Web / 命令行 / 其他 SDK 发起请求（例如 AI 聊天、文档问答、图像识别等）
* 业务服务层（C++ 框架核心）	提供对话服务、图像识别服务、用户管理服务，是整个平台的核心逻辑层
* 数据与消息层	负责业务数据的存储、异步任务的转发与缓冲，提升系统稳定性与并发性能
* 推理与第三方平台层	对接多家 AI 大模型（阿里云、百度智能云、火山引擎等）以及本地推理引擎（ONNXRuntime）

## 流程图

![](https://file1.kamacoder.com/i/web/2025-10-15_16-22-16.jpg)

展示了整个系统从 客户端请求 → ChatServer 业务调度 → 多模型调用 → 异步消息入库 的全链路流程

一、总体架构思路

该系统基于[自研的 C++ HTTP 服务框架](https://programmercarl.com/other/project_http.html) 构建，是一个支持：

* 多模型接入（GPT / 通义 / 豆包 / 百炼 / 百川）
* 图像识别（ONNX + OpenCV）
* 语音识别与合成（ASR/TTS）
* 异步消息入库（RabbitMQ）
* 多会话管理
* MCP 工具协议化

的完整 AI 应用服务平台。

系统核心是 ChatServer，它负责：

* 接收客户端请求；
* 调用对应业务 Handler；
* 根据类型分发到不同 AI 模块（聊天、图像识别、语音）；
* 将结果异步入库或交由队列处理。

## 做完这个项目你将收获什么？

这个项目足够稀缺！

当前 99% 的 AI 应用项目都是 Java / Python 实现的，而本项目使用 纯 C++ 构建完整 AI 服务平台。

做完这个项目，你可以学会

* **独立完成 C++ + 大模型 + RAG + 多模态 全链路开发**；
* **理解底层 HTTP、线程池、异步消息、模型推理之间的真实数据流**；
* **把“C++ 系统能力”和“AI 应用能力”结合在一起**。

更具体一些，你会真正理解一个 AI 平台的完整架构：

* 如何在 C++ 框架中封装 多模型策略层（GPT、通义、豆包、百炼）；
* 如何实现类似 MCP（Model Context Protocol） 的上下文管理；
* 如何用 RabbitMQ + 线程池 做异步入库和任务调度；
* 如何接入 语音识别（ASR）+ 语音合成（TTS）；
* 如何集成本地 ONNX 模型推理；
* 如何设计 多会话隔离与上下文管理；
* 如何让一个 AI 服务同时支持 云端模型与本地推理 模式。

## 更详细内容

本项目专栏和代码依然是只分享在[知识星球](https://programmercarl.com/other/kstar.html)里。

详细的 各个文件的讲解：

![](https://file1.kamacoder.com/i/web/2025-10-15_16-50-14.jpg)

流程讲解：

![](https://file1.kamacoder.com/i/web/2025-10-15_16-51-10.jpg)


AI开发的各种细节

![](https://file1.kamacoder.com/i/web/2025-10-15_17-21-26.jpg)

项目难点强调：

![](https://file1.kamacoder.com/i/web/2025-10-15_17-22-38.jpg)


简历写法，做完项目可以直接写到简历上。

![](https://file1.kamacoder.com/i/web/2025-10-15_17-24-19.jpg)

那这个项目面试，都会有哪些问题，如何回答，都列出来了

![](https://file1.kamacoder.com/i/web/2025-10-15_17-25-44.jpg)

## 答疑

本项目在[知识星球](https://programmercarl.com/other/kstar.html)里为 文字专栏形式，大家不用担心，看不懂，星球里每个项目有专属答疑群，任何问题都可以在群里问，都会得到解答：

![](https://file1.kamacoder.com/i/web/2025-09-26_11-30-13.jpg)


## 获取本项目专栏

**本文档仅为星球内部专享，大家可以加入[知识星球](https://programmercarl.com/other/kstar.html)里获取，在星球置顶一**

