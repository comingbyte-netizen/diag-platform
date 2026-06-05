# Vehicle Intelligent Diagnosis Platform (VI-Diag)

> **Version**: v2.0 — AI Agent-Native Architecture  
> **Status**: Architecture Design Phase  
> **Design Doc**: [vehicle-diag-platform-design.md](docs/vehicle-diag-platform-design.md)

---

## Overview

VI-Diag is a next-generation **AI Agent-native** intelligent vehicle diagnosis platform covering the entire vehicle lifecycle — from design, development, verification, production to after-sales service.

The platform adopts **LLM AI Agent as its foundational architecture**, where the AI Agent acts as the sole central brain orchestrating real-time communication routing, business processing orchestration, script lifecycle management, and knowledge evolution. Traditional microservices are downgraded to "Skill Execution Units" — capable executors without autonomous decision-making.

### Core Paradigm

```
Traditional Architecture:           VI-Diag AI-Native Architecture:
 ┌──────────┐                       ┌─────────────────────┐
 │ Microservices │                  │   AI Agent Core     │
 │ ┌──┐┌──┐     │                   │  (Central Brain)    │
 │ │A ││B │     │                   │                     │
 │ └──┘└──┘     │                   │  • Intent Parsing   │
 │ ┌──┐┌──┐     │ ← AI is add-on    │  • Knowledge        │
 │ │C ││AI│     │                   │  • Script Factory   │
 │ └──┘└──┘     │                   │  • Task Dispatch    │
 └──────────┘                       │  • Self-Reflection  │
                                     └──────────┬──────────┘
                                                │ Invocation
                                     ┌──────────▼──────────┐
                                     │  Skill Exec. Units  │
                                     │ UDS|DoIP|OTA|Script │
                                     │  Knowledge|VPlatform│
                                     └─────────────────────┘
```

### Three Core Principles

| Principle | Description |
|-----------|-------------|
| **Script-Driven** | All business logic is expressed and executed as debuggable scripts. AI Agent generates, debugs, deploys, and executes scripts autonomously. |
| **Knowledge-First** | Knowledge base (tuned scripts + knowledge graph + ODX metadata + case history) drives AI reasoning and decisions. LLM fallback occurs only when knowledge is insufficient. |
| **Self-Closing Loop** | AI Agent perceives → retrieves knowledge/scripts → reuses if found, auto-creates if not → debugs → deploys → reflects → optimizes → persists. |

---

## Architecture (6-Layer AI-Native)

```
┌────────────────────────────────────────────────────────────────┐
│  ① AI Decision Layer                                          │
│  IntentParser → KnowledgeRetriever (4-tier pipeline)          │
│  → ScriptFactory (auto-generate + debug + deploy)             │
│  → Dispatcher → Async Reflection (self-optimization)          │
├────────────────────────────────────────────────────────────────┤
│  ② Skill Execution Layer (18 microservices as Agent Skills)   │
│  diag | ota | master | script | vplatform | twin | log | ... │
├────────────────────────────────────────────────────────────────┤
│  ③ Protocol & Communication Layer                              │
│  MQTT 5.0 | HTTP/2 | gRPC | WebSocket/WebRTC | Tailscale VPN │
├────────────────────────────────────────────────────────────────┤
│  ④ Vehicle Execution Layer                                     │
│  DiagMaster (Master) → DiagSlave (Domain Slaves) → DiagEngine │
├────────────────────────────────────────────────────────────────┤
│  ⑤ ECU Diagnostic Layer                                        │
│  DoIP Nodes | Subnet Gateways | Basic ECUs (CAN/LIN)          │
├────────────────────────────────────────────────────────────────┤
│  ⑥ Knowledge & Data Layer                                      │
│  Tuned Scripts(Milvus) | KG(Neo4j) | ODX | Cases | TimeSeries │
└────────────────────────────────────────────────────────────────┘
```

---

## Key Capabilities

### 🧠 AI Agent Core
- **Intent Understanding**: LLM-powered intent classification + entity extraction + context completion
- **4-Tier Knowledge Retrieval Pipeline**:  
  L1: Milvus vector search (tuned scripts, threshold ≥0.85)  
  L2: Neo4j graph query (1-hop symptom→fault→cause)  
  L3: ODX metadata (DID/DTC/Service definitions)  
  L4: Historical case matching (similar case Top-5)
- **Script Factory**: AI auto-generates JS/Python scripts → virtual vehicle sandbox debugging → auto-fix (≤3 rounds) → persist to knowledge base
- **Reflection Engine**: Post-execution reflection for script optimization, knowledge graph update, and weight adjustment
- **Knowledge Evolution**: Day1: 10% hit rate → Day30: 55% → Day90: 78% → Target: 90%+

### 🔌 Diagnostic Protocol Stack
- **DoIP** (ISO 13400): UDP vehicle discovery, TCP diagnostic channel
- **UDS** (ISO 14229): Full 29 service support
- **OBD-II** (ISO 15031): Emission regulation monitoring
- **SOVD** (ISO 23298): Service-oriented vehicle diagnostics
- **CAN/CANFD/LIN**: Full multi-bus support

### 🚗 Vehicle-Side Platform (C++)
- **DiagMaster**: Vehicle diagnostic master controller — receives cloud commands, manages task lifecycle, coordinates domain slaves
- **DiagSlave**: Domain-level diagnostic execution node — gRPC server, UDS execution, heartbeat monitoring
- **Service Hub**: Vehicle service bus — service registration/discovery/routing, health checks, circuit breaker
- **DiagEngine**: Core diagnostic engine — UDS sequence execution, script interpretation (JS/Python), diagnostic tree execution, plugin hot-loading
- **Download Manager**: Multi-source concurrent download, resume from breakpoint, SHA-256 integrity verification
- **Log Engine**: 3-tier log collection (ECU/Domain/Vehicle), DLT standard + JSON dual-format, real-time MQTT streaming, pre-trigger capture buffer

### ☁️ Cloud Platform (Java Spring Boot)
- **Diagnostic Execution Engine**: UDS command construction/parsing, MQTT gateway, session management
- **Remote Diagnosis**: Cloud-initiated → vehicle-executed → result aggregation
- **OTA Management**: Firmware packaging/signing/gray release/rollback
- **Vehicle Digital Twin**: Real-time state sync, 3D visualization (WebGL), fault injection simulation
- **Virtual VHMI**: Real-time screen mirroring, remote touch/key events, session recording/playback
- **ODX/OTX Engineering**: ODX 2.2.0 import/validation (JAXB), OTX sequence execution engine
- **Diagnostic Script Management**: Multi-language sandbox (JS/Python/Groovy), version management, CI/CD integration
- **Cloud Log Center**: Real-time log stream ingestion, ES full-text search, alert rules engine, lifecycle management
- **Instant Software Management**: Feature flag hot-update, configuration snapshot/rollback, A/B testing strategy
- **Virtual Vehicle Platform**: Docker-based vECU container runtime, vCAN/vLIN/vETH bus simulation, HiL+SiL hybrid bridging, fault injection, performance benchmarking

### 🤖 AI Diagnostic Agent
- Natural language conversation interface
- Multi-step diagnostic task planning (auto-generated or scripted)
- Knowledge graph reasoning + vector case matching
- Automated structured diagnosis report generation

### 📊 Diagnostic Tree System
- Visual drag-and-drop diagnostic tree designer
- Real-time tree execution engine with state machine
- Version management, diff comparison, publishing
- Execution statistics and bottleneck analysis

### 🔐 Security
- **Edge**: WAF + DDoS protection + IP whitelist
- **Auth**: OAuth2.0 + JWT + PKI (X.509) + mTLS
- **ECU Security**: Seed-Key (UDS 0x27) + secure boot chain
- **Data**: TLS 1.3 encryption + AES-256-GCM at-rest encryption + dynamic PII masking
- **Compliance**: ISO 27001 | GDPR | UN R155/R156

### 🔧 DevOps
- Full lifecycle management: Virtual Design → Virtual Integration → Virtual Verification → Virtual Acceptance → Real Deployment
- CI/CD automation: PR triggers automatic full diagnostic regression
- Performance benchmark: 100+ concurrent virtual vehicle instances
- Deploy target: Ubuntu 24 via Docker Compose (compatible with the project's vagrant-based development environment)

---

## 138 Functional Requirements (7 Categories)

| Category | Count | Core Capabilities |
|----------|-------|-------------------|
| FR-1000 Protocol Stack | 9 | DoIP, UDS, OBD, SOVD, CAN, LIN |
| FR-2000 Core Diagnostics | 9 | Flash, Routing, Bootloader, DTC, Data Stream |
| FR-3000 Intelligent Diagnosis | 8 | AI Agent, KG, Diagnostic Tree, Inference, Report |
| FR-4000 Diagnostic Tree | 6 | Editor, Engine, Versioning, Analytics |
| FR-5000 Cloud Platform | 7 | Remote Diag, OTA, Vehicle Mgmt, Emission |
| FR-6000 Vehicle (C++) | 8 | DoIP, UDS, Routing, Bootloader, AUTOSAR |
| FR-7000 AUTOSAR | 5 | DCM, DEM, FIM, DoIP, AP |
| FR-7100 Cloud Control | 11 | Master, VHMI, Digital Twin, ODX, OTX, Scripts |
| FR-7200 Vehicle Control | 9 | Master/Slave, Download Mgr, Service Hub, DiagEngine |
| FR-7300 Log Collection | 10 | Multi-tier, DLT, Real-time, Remote Pull |
| FR-7400 Virtual Platform | 13 | vECU, SiL+HiL, Benchmark, Lifecycle |
| **FR-7500 AI Base** | **13** | **Agent Core, Knowledge-First, Script Factory, Multi-Agent, Reflection** |

---

## Technology Stack

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React 19 + TypeScript + Vite + Ant Design 5 + Three.js + React Flow |
| **Cloud Backend** | Java 21 + Spring Boot 3.x + Spring Cloud 2024 + WebFlux |
| **Vehicle Side** | C++ 17/20 + Boost.Asio + gRPC + spdlog + libcurl |
| **AI/ML** | LangChain4j + Spring AI + LLM + Embedding Model + Vector DB |
| **Message** | MQTT 5.0 (EMQX) + Apache Kafka + RabbitMQ |
| **Data** | MySQL 8.0 + PostgreSQL 16 + Redis 7 + MongoDB 7 + Neo4j 5 + ClickHouse + Elasticsearch + Milvus + MinIO |
| **Container** | Docker + Docker Compose + Kubernetes |
| **Observability** | Prometheus + Grafana + Loki + OpenTelemetry + Jaeger |
| **VPN** | Tailscale Mesh VPN (WireGuard) |

---

## Detailed Design Coverage (20 Chapters)

| Ch | Service/Module | Core Code |
|----|---------------|-----------|
| 12.1 | diag-service | UdsEngine, DiagSessionManager, FlashEngine |
| 12.2 | AI Agent | DiagnosisAgentService, IntentRecognizer, TaskPlanner |
| 12.3 | DoIP (C++) | DoipServer (UDP+TCP), RoutingEngine |
| 12.4 | Knowledge Graph | Cypher 5 patterns, Milvus vector retrieval |
| 12.5 | Deploy Script | PowerShell deploy.ps1 |
| 12.6 | master-service | TaskOrchestrator, priority queue |
| 12.7 | Master/Slave (C++) | DiagMaster, DiagSlave, gRPC proto |
| 12.8 | Digital Twin | TwinSyncEngine, WebSocket push |
| 12.9 | ODX/OTX | JAXB parsing, OtxExecutionEngine |
| 12.10 | Script Engine | GraalJS sandbox, diag API injection |
| 12.11 | Service Hub (C++) | ServiceRegistry, DiagEngine |
| 12.12 | Download Mgr (C++) | curl_multi, resume, SHA-256 |
| 12.13 | Data Flow | 20-step cloud→vehicle chain |
| 12.14 | Log Engine (C++) | 3-tier, DLT+spdlog, ES search |
| 12.15 | Instant Software | Feature Flag, config rollback |
| 12.16 | Virtual Platform | Docker vECU, vCAN bus (C++) |
| 12.17 | Lifecycle | 5-stage: Design→Integration→Validation→Acceptance→Deploy |
| **12.18** | **AI Agent Core** | **AgentCoreEngine, IntentParser, KnowledgeRetriever** |
| **12.19** | **Script Factory** | **autoGenerateAndDebug, debugInSandbox, autoFixScript** |
| **12.20** | **Knowledge Evolution** | **Hit rate: 10% → 55% → 78% → 90%+** |

---

## Development & Deployment

### Local Development
```bash
# Prerequisites: JDK 21, Docker, Docker Compose
git clone <repo-url>
cd diag-platform

# Start infrastructure
docker compose -f deploy/docker-compose.yml up -d mysql redis mongodb neo4j

# Build cloud services
./gradlew :iching-api:bootJar

# Build vehicle-side (C++)
cd vehicle
cmake -B build && cmake --build build
```

### Deployment (Reference Environment)
```powershell
# Windows PowerShell deploy script
.\deploy\deploy.ps1

# Options:
.\deploy\deploy.ps1 -SkipTest    # Skip test, quick build
.\deploy\deploy.ps1 -SkipBuild   # Reuse existing JAR
.\deploy\deploy.ps1 -DryRun      # Dry run only
.\deploy\deploy.ps1 -Version 2.0 # Specify version
```

**Target**: Ubuntu 24 on Docker Compose (Vagrant-based, IP 10.110.0.5)

---

## Related Documents

- [Full Architecture Design Document (CN)](docs/vehicle-diag-platform-design.md)
- Reference: [Vehicle Diagnostic Architecture Analysis — From DoIP to Flashing Routing](https://mp.weixin.qq.com/s/zd7W5ZbQ3jFQMXfPPNQIDw)

---

## License

Proprietary — Internal use only.

---

## Keywords

`Vehicle Diagnostics` `AI Agent` `DoIP` `UDS` `ODX` `OTX` `AutoSAR` `Digital Twin` `Cloud-Native` `C++` `Spring Boot` `MQTT` `Tailscale` `Vehicle-in-the-Loop` `HiL` `SiL` `DevOps` `Knowledge Graph` `LLM`
