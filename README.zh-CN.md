# 整车智能诊断平台 (VI-Diag)

> **版本**: v2.0 — AI Agent 底座原生架构  
> **状态**: 架构设计阶段  
> **设计文档**: [vehicle-diag-platform-design.md](docs/vehicle-diag-platform-design.md)

---

## 项目概述

VI-Diag 是新一代以 **LLM AI Agent 为底座原生架构**的整车智能诊断平台，覆盖整车全生命周期 — 设计、开发、验证、生产到售后。

平台采用 **AI Agent 作为唯一的中央大脑**，统一接管实时通信路由、业务流程编排、脚本全生命周期管理和知识自我进化。传统微服务降级为"技能执行单元"——只执行、不决策。

### 核心范式

```
传统架构:                      本平台 AI-Native 架构:
 ┌──────────┐                  ┌─────────────────────┐
 │ 微服务集群 │                 │   AI Agent Core     │
 │ ┌──┐┌──┐  │                 │  (中央大脑/调度中心) │
 │ │A ││B │  │                 │                     │
 │ └──┘└──┘  │                 │  • 意图感知         │
 │ ┌──┐┌──┐  │ ← AI是附属模块  │  • 知识检索         │
 │ │C ││AI│  │                 │  • 脚本工厂         │
 │ └──┘└──┘  │                 │  • 任务分发         │
 └──────────┘                  │  • 自我反思         │
                                └──────────┬──────────┘
                                           │ 技能调用
                                ┌──────────▼──────────┐
                                │  技能执行单元        │
                                │ UDS|DoIP|OTA|脚本   │
                                │ 知识引擎|虚拟平台    │
                                └─────────────────────┘
```

### 三大核心原则

| 原则 | 描述 |
|------|------|
| **脚本驱动 (Script-Driven)** | 所有业务逻辑以可调试脚本表达与执行，AI Agent 自动生成/调试/部署/执行脚本 |
| **知识优先 (Knowledge-First)** | 知识库(调优脚本+知识图谱+ODX元数据+历史案例)驱动AI决策，知识不足时才回退到LLM推理 |
| **自主闭环 (Self-Closing Loop)** | AI Agent 感知→检索知识/脚本→有则复用/无则自动创建→调试→部署→反思→优化→入库沉淀 |

---

## 六层 AI 原生架构

```
┌────────────────────────────────────────────────────────────────┐
│  ① AI 决策层 (Agent Intelligence Layer)                        │
│  IntentParser → KnowledgeRetriever (4级管道)                   │
│  → ScriptFactory (自动生成+调试+部署)                           │
│  → Dispatcher → Async Reflection (自我优化)                    │
├────────────────────────────────────────────────────────────────┤
│  ② 技能执行层 (18个微服务降级为Agent Skills)                     │
│  diag | ota | master | script | vplatform | twin | log | ... │
├────────────────────────────────────────────────────────────────┤
│  ③ 协议通信层                                                  │
│  MQTT 5.0 | HTTP/2 | gRPC | WebSocket/WebRTC | Tailscale VPN │
├────────────────────────────────────────────────────────────────┤
│  ④ 车端执行层                                                  │
│  DiagMaster (主控) → DiagSlave (域从控) → DiagEngine (诊断引擎)│
├────────────────────────────────────────────────────────────────┤
│  ⑤ ECU 诊断层                                                  │
│  DoIP节点 | 子网网关 | 基础ECU (CAN/LIN)                       │
├────────────────────────────────────────────────────────────────┤
│  ⑥ 知识数据层                                                  │
│  调优脚本(Milvus) | 知识图谱(Neo4j) | ODX | 案例 | 时序数据    │
└────────────────────────────────────────────────────────────────┘
```

---

## 核心能力

### 🧠 AI Agent Core
- **意图理解**: LLM驱动的意图分类 + 实体提取 + 上下文补全
- **4级知识检索管道**:  
  L1: Milvus向量检索(调优脚本库, 阈值≥0.85)  
  L2: Neo4j图查询(1跳:症状→故障→原因)  
  L3: ODX元数据(DID/DTC/Service定义)  
  L4: 历史案例匹配(相似案例Top-5)
- **脚本工厂**: AI自动生成JS/Python脚本 → 虚拟整车沙箱调试 → 自动修正(≤3轮) → 入库沉淀
- **反思引擎**: 执行后自我反思，优化脚本、更新知识图谱、调整权重
- **知识演进**: Day1: 10%命中率 → Day30: 55% → Day90: 78% → 目标: 90%+

### 🔌 诊断协议栈
- **DoIP** (ISO 13400): UDP车辆发现, TCP诊断通道
- **UDS** (ISO 14229): 完整29项服务支持
- **OBD-II** (ISO 15031): 排放法规监控
- **SOVD** (ISO 23298): 面向服务的车辆诊断
- **CAN/CANFD/LIN**: 完整多总线支持

### 🚗 车端平台 (C++)
- **DiagMaster 主控**: 接收云端指令、管理诊断任务生命周期、调度域从控
- **DiagSlave 从控**: 域级诊断执行节点 — gRPC服务、UDS执行、心跳上报
- **Service Hub**: 车端服务总线 — 服务注册/发现/路由、健康检查、熔断
- **DiagEngine 诊断引擎**: UDS序列执行、脚本解释执行(JS/Python)、诊断树执行、插件热加载
- **下载管理器**: 多源并发下载、断点续传、SHA-256完整性校验
- **日志引擎**: 3级采集(ECU/域控/整车)、DLT标准+JSON双模、MQTT实时上报、预触发采集缓冲

### ☁️ 云端平台 (Java Spring Boot)
- **诊断执行引擎**: UDS指令构造/解析、MQTT网关、会话管理
- **远程诊断**: 云端发起 → 车端执行 → 结果汇聚
- **OTA管理**: 固件打包/签名/灰度发布/回滚
- **车辆数字孪生**: 实时状态同步、3D可视化(WebGL)、故障注入仿真
- **虚拟VHMI**: 实时屏幕镜像、远程触控/按键、录屏回放
- **ODX/OTX工程**: ODX 2.2.0导入/校验(JAXB)、OTX序列执行引擎
- **诊断脚本管理**: 多语言沙箱(JS/Python/Groovy)，版本管理，CI/CD集成
- **云日志中心**: 实时日志流接入、ES全文检索、告警规则引擎、生命周期管理
- **即时软件管理**: Feature Flag热更新、配置快照/回滚、A/B测试策略
- **虚拟整车平台**: Docker vECU容器运行时、vCAN/vLIN/vETH总线模拟、HiL+SiL混合桥接、故障注入、性能压测

### 🤖 AI 诊断 Agent
- 自然语言对话接口
- 多步诊断任务规划(自动生成或遵循脚本)
- 知识图谱推理 + 向量案例匹配
- 自动化结构化诊断报告生成

### 📊 诊断树系统
- 可视化拖拽式诊断树设计器
- 实时诊断树执行引擎(状态机)
- 版本管理、差异对比、发布
- 执行统计与瓶颈分析

### 🔐 安全
- **边界**: WAF + DDoS防护 + IP白名单
- **认证**: OAuth2.0 + JWT + PKI (X.509) + mTLS
- **ECU安全**: Seed-Key (UDS 0x27) + 安全启动链
- **数据**: TLS 1.3传输加密 + AES-256-GCM存储加密 + 动态脱敏
- **合规**: ISO 27001 | GDPR | UN R155/R156

### 🔧 DevOps
- 全生命周期管理: 虚拟设计 → 虚拟集成 → 虚拟验证 → 虚拟验收 → 实际部署
- CI/CD自动化: PR提交自动触发全量诊断回归
- 性能压测: 100+并发虚拟整车实例
- 部署目标: Ubuntu 24 via Docker Compose

---

## 138 项功能需求 (7 大类)

| 类别 | 数量 | 核心能力 |
|------|------|---------|
| FR-1000 通信协议栈 | 9 | DoIP, UDS, OBD, SOVD, CAN, LIN |
| FR-2000 诊断核心 | 9 | 刷写, 路由, Bootloader, DTC, 数据流 |
| FR-3000 智能诊断 | 8 | AI Agent, KG, 诊断树, 推理, 报告 |
| FR-4000 诊断树 | 6 | 编辑器, 引擎, 版本, 分析 |
| FR-5000 云端平台 | 7 | 远程诊断, OTA, 车辆管理, 排放 |
| FR-6000 车端(C++) | 8 | DoIP, UDS, 路由, Bootloader, AUTOSAR |
| FR-7000 AUTOSAR | 5 | DCM, DEM, FIM, DoIP, AP |
| FR-7100 云端主控 | 11 | 云端主控, VHMI, 数字孪生, ODX, OTX, 脚本 |
| FR-7200 车端主控 | 9 | 主控/从控, 下载管理, Service Hub, 诊断引擎 |
| FR-7300 日志收集 | 10 | 多级采集, DLT, 实时上报, 远程拉取 |
| FR-7400 虚拟整车 | 13 | vECU, SiL+HiL, 压测, 生命周期 |
| **FR-7500 AI底座** | **13** | **Agent Core, 知识优先, 脚本工厂, 多Agent, 反思** |

---

## 技术栈

| 层 | 技术 |
|----|------|
| **前端** | React 19 + TypeScript + Vite + Ant Design 5 + Three.js + React Flow |
| **云端后端** | Java 21 + Spring Boot 3.x + Spring Cloud 2024 + WebFlux |
| **车端** | C++ 17/20 + Boost.Asio + gRPC + spdlog + libcurl |
| **AI/ML** | LangChain4j + Spring AI + LLM + Embedding Model + 向量数据库 |
| **消息** | MQTT 5.0 (EMQX) + Apache Kafka + RabbitMQ |
| **数据** | MySQL 8.0 + PostgreSQL 16 + Redis 7 + MongoDB 7 + Neo4j 5 + ClickHouse + Elasticsearch + Milvus + MinIO |
| **容器** | Docker + Docker Compose + Kubernetes |
| **可观测** | Prometheus + Grafana + Loki + OpenTelemetry + Jaeger |
| **组网** | Tailscale Mesh VPN (WireGuard) |

---

## 详细设计覆盖 (20个章节)

| 章 | 服务/模块 | 核心代码 |
|----|----------|---------|
| 12.1 | diag-service | UdsEngine, DiagSessionManager, FlashEngine |
| 12.2 | AI诊断Agent | DiagnosisAgentService, IntentRecognizer, TaskPlanner |
| 12.3 | DoIP协议栈(C++) | DoipServer (UDP+TCP), RoutingEngine |
| 12.4 | 知识图谱 | Cypher 5种查询模式, Milvus向量检索 |
| 12.5 | 部署脚本 | PowerShell deploy.ps1 |
| 12.6 | master-service | TaskOrchestrator, 优先级队列 |
| 12.7 | 主从控(C++) | DiagMaster, DiagSlave, gRPC proto |
| 12.8 | 数字孪生 | TwinSyncEngine, WebSocket推送 |
| 12.9 | ODX/OTX | JAXB解析, OtxExecutionEngine |
| 12.10 | 脚本引擎 | GraalJS沙箱, diag API注入 |
| 12.11 | Service Hub(C++) | ServiceRegistry, DiagEngine |
| 12.12 | 下载管理器(C++) | curl_multi, 断点续传, SHA-256 |
| 12.13 | 数据流 | 20步云端→车端完整链路 |
| 12.14 | 日志引擎(C++) | 3级采集, DLT+spdlog, ES检索 |
| 12.15 | 即时软件 | Feature Flag, 配置回滚 |
| 12.16 | 虚拟整车 | Docker vECU, vCAN总线(C++) |
| 12.17 | 开发生命周期 | 5阶段: 设计→集成→验证→验收→部署 |
| **12.18** | **AI Agent Core** | **AgentCoreEngine, IntentParser, KnowledgeRetriever** |
| **12.19** | **脚本工厂** | **autoGenerateAndDebug, debugInSandbox, autoFixScript** |
| **12.20** | **知识演进** | **命中率: 10% → 55% → 78% → 90%+** |

---

## 开发与部署

### 本地开发
```bash
# 前置条件: JDK 21, Docker, Docker Compose
git clone <repo-url>
cd diag-platform

# 启动基础设施
docker compose -f deploy/docker-compose.yml up -d mysql redis mongodb neo4j

# 编译云端服务
./gradlew :iching-api:bootJar

# 编译车端 (C++)
cd vehicle
cmake -B build && cmake --build build
```

### 部署 (参考环境)
```powershell
# Windows PowerShell 部署脚本
.\deploy\deploy.ps1

# 选项:
.\deploy\deploy.ps1 -SkipTest    # 跳过测试快速编译
.\deploy\deploy.ps1 -SkipBuild   # 跳过编译，使用已有JAR
.\deploy\deploy.ps1 -DryRun      # 干运行（不实际执行）
.\deploy\deploy.ps1 -Version 2.0 # 指定版本
```

**目标**: Ubuntu 24 on Docker Compose (Vagrant-based, IP 10.110.0.5)

---

## 相关文档

- [完整架构设计文档](docs/vehicle-diag-platform-design.md)
- 参考文章: [整车诊断架构全解析——从DoIP到刷写路由](https://mp.weixin.qq.com/s/zd7W5ZbQ3jFQMXfPPNQIDw)

---

## 许可

专有 — 仅供内部使用。

---

## 关键词

`整车诊断` `AI Agent` `DoIP` `UDS` `ODX` `OTX` `AutoSAR` `数字孪生` `云原生` `C++` `Spring Boot` `MQTT` `Tailscale` `虚拟仿真` `HiL` `SiL` `DevOps` `知识图谱` `大语言模型` `LLM`
