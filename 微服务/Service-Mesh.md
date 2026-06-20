# Service Mesh（服务网格）

> 一份面向工程落地的 Service Mesh 学习与选型指南，整理自 [Istio](https://istio.io/)、[Linkerd](https://linkerd.io/)、[Envoy](https://www.envoyproxy.io/)、[Consul Connect](https://developer.hashicorp.com/consul/docs/connect)、[Cilium Service Mesh](https://cilium.io/use-cases/service-mesh/) 等开源项目官方文档，并结合 Google、Lyft、HashiCorp、Buoyant、Solo.io、阿里、字节等公司的工程实践。

## 目录

- [Service Mesh（服务网格）](#service-mesh服务网格)
  - [目录](#目录)
  - [一、核心理念](#一核心理念)
  - [二、为什么需要 Service Mesh](#二为什么需要-service-mesh)
    - [2.1 微服务通信治理的演进](#21-微服务通信治理的演进)
    - [2.2 vs 客户端 SDK（Spring Cloud / Dubbo）](#22-vs-客户端-sdkspring-cloud--dubbo)
    - [2.3 vs ESB（SOA 时代）](#23-vs-esbsoa-时代)
  - [三、整体架构](#三整体架构)
    - [3.1 数据平面（Data Plane）](#31-数据平面data-plane)
    - [3.2 控制平面（Control Plane）](#32-控制平面control-plane)
  - [四、核心能力](#四核心能力)
    - [4.1 流量管理](#41-流量管理)
    - [4.2 安全](#42-安全)
    - [4.3 可观测性](#43-可观测性)
    - [4.4 策略与扩展](#44-策略与扩展)
  - [五、服务间调用完整链路](#五服务间调用完整链路)
    - [5.1 流量劫持原理（透明代理）](#51-流量劫持原理透明代理)
    - [5.2 一次请求的完整旅程](#52-一次请求的完整旅程)
    - [5.3 性能开销与优化](#53-性能开销与优化)
  - [六、主流实现对照](#六主流实现对照)
    - [6.1 Istio（功能最全）](#61-istio功能最全)
    - [6.2 Linkerd（轻量极简）](#62-linkerd轻量极简)
    - [6.3 Consul Connect（多环境通用）](#63-consul-connect多环境通用)
    - [6.4 Cilium Service Mesh（eBPF 派）](#64-cilium-service-meshebpf-派)
    - [6.5 选型矩阵](#65-选型矩阵)
  - [七、Sidecar vs Sidecarless](#七sidecar-vs-sidecarless)
  - [八、适用与不适用场景](#八适用与不适用场景)
  - [九、常见反例与误区](#九常见反例与误区)
  - [十、工程化落地建议](#十工程化落地建议)
  - [参考资料](#参考资料)

---

## 一、核心理念

Service Mesh 是一种用于处理**服务间通信**的基础设施层，将网络通信、安全、可观测性等横切关注点（cross-cutting concerns）从业务代码下沉到独立的代理进程（**Sidecar**），让业务代码专注于业务逻辑本身。

核心特征：

- **基础设施化**：把"服务治理"做成跟操作系统、Kubernetes 一样的"看不见的底座"，业务无感知。
- **语言无关**：能力沉淀在代理（Envoy / linkerd2-proxy），不绑定任何业务语言或框架，多语言技术栈天然受益。
- **配置驱动**：流量规则、安全策略通过声明式 YAML / CRD 下发，运行时动态生效，无需重启业务。
- **统一控制面**：跨集群、跨语言、跨团队的服务治理通过同一个控制平面统一管理。
- **零信任友好**：mTLS 默认开启、基于身份（SPIFFE / SVID）的访问控制，天然契合零信任网络。

> **一句话**：Service Mesh = 把"通信能力"从业务里剥离出来，做成跟网络栈一样的基础设施。

---

## 二、为什么需要 Service Mesh

### 2.1 微服务通信治理的演进

```text
                          治理能力放在哪？

[1] 框架内嵌              业务代码（治理能力很弱）
                         ┌────────────┐
                         │ 业务 + 框架  │   Spring MVC / Express 自带 LB / 超时
                         └────────────┘

[2] 治理型 SDK            业务代码 + 治理 SDK（语言绑定、升级困难）
                         ┌─────────────────────┐
                         │ 业务                  │
                         │ + Hystrix / Ribbon   │   Spring Cloud / Dubbo / gRPC
                         │ + Sleuth / Eureka    │
                         └─────────────────────┘

[3] Sidecar 代理          业务代码 + 独立进程（语言无关、热升级）
                         ┌────────┐    ┌──────────┐
                         │ 业务    │ ←→ │ Sidecar  │   Istio / Linkerd（经典 Mesh）
                         └────────┘    └──────────┘

[4] Sidecarless           业务代码 + 内核 / 节点代理（更省资源）
                         ┌────────┐
                         │ 业务    │
                         └───┬────┘
                             ▼
                       eBPF / Node Proxy           Cilium Mesh / Istio Ambient
```

每一步都在做一件事：**把治理能力下沉，让业务越来越纯**。

### 2.2 vs 客户端 SDK（Spring Cloud / Dubbo）

| 维度 | 客户端 SDK | Service Mesh |
|------|-----------|-------------|
| 侵入性 | 强侵入，依赖框架版本 | 业务零侵入，靠 iptables / eBPF 透明劫持 |
| 多语言支持 | 每种语言重写一套 | 一份代理，所有语言通用 |
| 升级方式 | 改代码 → 重新构建 → 重新部署 | 升级代理 / 控制面配置即可 |
| 治理一致性 | 各语言实现常有差异 | 单一实现，行为一致 |
| 部署成本 | 0（库就是业务的一部分） | 每个 Pod 多一个 Sidecar 容器 |
| 性能 | 进程内调用，零开销 | 多两跳 + 加解密，约 +2–5 ms |
| 适合场景 | 单语言单团队、性能极致 | 多语言、多团队、长期演进 |

> **一句话**：SDK 模式把治理"嵌"进业务，Mesh 把治理"剥"出业务；前者性能更好，后者扩展性更好。

### 2.3 vs ESB（SOA 时代）

| 维度 | ESB（SOA 时代） | Service Mesh（云原生时代） |
|------|----------------|----------------------|
| 架构 | 集中式总线 | 去中心化 Sidecar |
| 单点故障 | 有（ESB 挂全挂） | 无（每个 Sidecar 独立运行） |
| 协议转换 | 核心能力（SOAP/JMS/...） | 不做（服务间统一 HTTP / gRPC） |
| 部署 | 独立中间件，重运维 | 随 Pod 自动注入，K8s 原生 |
| 性能瓶颈 | 所有流量都过 ESB | 流量分散在各 Sidecar，水平扩展 |
| 业务逻辑 | 常被塞进 ESB（编排、路由规则） | 严格不做业务，只做通信 |

> **教训**：ESB 失败的核心原因之一是"业务逻辑沉淀到总线"，导致总线变成新的单体。Service Mesh 必须坚守"只做通信、不碰业务"的边界。

---

## 三、整体架构

```text
┌────────────────────────────────────────────────────────────┐
│                  控制平面（Control Plane）                    │
│              Istio / Linkerd / Consul / Cilium               │
│  配置下发 (xDS) │ 证书签发 (CA) │ 服务发现 │ 策略 │ 遥测聚合     │
└──────────────────────────┬─────────────────────────────────┘
                            │ xDS / gRPC 推送
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
   ┌────────┐         ┌────────┐         ┌────────┐
   │Sidecar │         │Sidecar │         │Sidecar │   ← 数据平面（Data Plane）
   │(Envoy) │         │(Envoy) │         │(Envoy) │
   ├────────┤         ├────────┤         ├────────┤
   │ 服务 A  │         │ 服务 B  │         │ 服务 C  │
   └────────┘         └────────┘         └────────┘
```

经典分层：**数据平面**负责"做"（转发、加密、采集），**控制平面**负责"管"（配置、证书、策略）。两者通过 [xDS 协议](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol) 通信，已成为 CNCF 事实标准。

### 3.1 数据平面（Data Plane）

每个业务 Pod 旁边注入一个代理容器（Sidecar），接管该 Pod 所有进出流量。

| 项目 | 数据面代理 | 实现语言 | 资源占用 |
|------|----------|---------|---------|
| Istio | Envoy | C++ | 约 50–100 MB 内存、约 0.1 core |
| Linkerd | linkerd2-proxy | Rust | 约 10–20 MB 内存、更轻 |
| Consul | Envoy（也支持 built-in proxy） | C++ | 同 Istio |
| Cilium Mesh | eBPF + Envoy（按需） | C / C++ | 节点级，单 Pod 接近 0 |

> **取舍**：Envoy 功能最全（HTTP/1、HTTP/2、gRPC、TCP、WebSocket、Wasm 扩展），但偏重；linkerd2-proxy 用 Rust 重写、只支持必要协议，换来极小内存和极低延迟。

### 3.2 控制平面（Control Plane）

控制面不在请求路径上，只负责**配置 + 策略 + 证书 + 元数据**。Istio 1.5 后将原本的 Pilot / Mixer / Citadel / Galley 合并为单体 **istiod**，运维大幅简化。

| 控制面组件 | 职责 |
|----------|------|
| 配置中心 | 把 `VirtualService` / `DestinationRule` 等 CRD 编译为 xDS 推送给所有 Sidecar |
| 证书 / 身份签发 | 给每个工作负载签发 SPIFFE 身份证书，承载 mTLS |
| 服务发现 | 对接 Kubernetes / Consul / Nacos，给数据面下发 endpoints |
| 策略中心 | 鉴权 / 限流 / 配额规则下发 |
| 遥测聚合 | 收集 Sidecar 上报的 metrics / traces / logs，对接 Prometheus / Jaeger / Loki |

---

## 四、核心能力

### 4.1 流量管理

| 能力 | 说明 | Istio 对应资源 |
|------|------|--------------|
| 负载均衡 | round-robin / least-request / consistent-hash | `DestinationRule.trafficPolicy` |
| 超时 / 重试 | 协议级超时、可重试错误码、退避策略 | `VirtualService.http.retries` |
| 熔断 | 连接池 / 请求队列 / 5xx 阈值 | `DestinationRule.outlierDetection` |
| 灰度 / 金丝雀 | 按 weight / header / cookie 分流 | `VirtualService.http.route.weight` |
| 流量镜像 | 把流量复制到影子环境做对比 | `VirtualService.http.mirror` |
| 故障注入 | 主动注入延迟 / 错误，做混沌测试 | `VirtualService.http.fault` |

### 4.2 安全

- **mTLS 双向加密**：默认对所有服务间通信开启，业务无感知；可通过 `PeerAuthentication` 设置 `STRICT` / `PERMISSIVE`。
- **基于身份的访问控制**：基于 [SPIFFE](https://spiffe.io/) 标准签发 SVID，`AuthorizationPolicy` 可写成 `允许 ns=order 的 service-account 调用 /api/pay`。
- **统一 CA 与证书轮转**：默认每 24h 自动轮转，业务无需关心证书管理。
- **零信任网络**：南北向 + 东西向流量都强制身份认证，不再相信"内网"。

### 4.3 可观测性

Sidecar 处于通信路径上，天然能采集：

- **黄金指标（Metrics）**：QPS、错误率、P50/P95/P99 延迟，自动生成 Prometheus 指标（[Istio Standard Metrics](https://istio.io/latest/docs/reference/config/metrics/)）。
- **分布式追踪（Tracing）**：自动注入 / 透传 [B3](https://github.com/openzipkin/b3-propagation) 或 [W3C Trace Context](https://www.w3.org/TR/trace-context/)，对接 Jaeger / Zipkin / Tempo / SkyWalking。
- **访问日志（Access Log）**：每条请求一行结构化日志，可对接 Loki / ELK。
- **流量拓扑**：基于上述数据，可视化呈现服务调用关系（Kiali）。

> **注意**：Mesh 自动生成的 trace span **只覆盖网络跳数**，业务方法内部的 span 仍需 SDK 自己埋点；二者通过 trace context header 拼接成完整链路。

### 4.4 策略与扩展

- **限流 / 配额**：基于 Envoy local-rate-limit 或对接外部 RLS（[ratelimit](https://github.com/envoyproxy/ratelimit)）。
- **WAF / 鉴权**：通过 [ExtAuthz](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/ext_authz_filter) 接外部鉴权服务。
- **协议扩展**：通过 [Envoy Wasm Filter](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/wasm_filter) / Lua / EnvoyFilter 注入自定义逻辑，无需改 Envoy 源码。

---

## 五、服务间调用完整链路

### 5.1 流量劫持原理（透明代理）

应用代码不需要做任何修改，靠 **iptables**（或 eBPF / TPROXY）规则把进出 Pod 的流量重定向到 Sidecar：

```bash
# 简化后的 Istio 注入规则
# 出站：所有出 Pod 的 TCP 流量 → Envoy 15001
iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-port 15001

# 入站：所有进 Pod 的 TCP 流量 → Envoy 15006
iptables -t nat -A PREROUTING -p tcp -j REDIRECT --to-port 15006
```

| 端口 | 用途 |
|------|------|
| `15001` | Outbound listener（出站流量入口） |
| `15006` | Inbound listener（入站流量入口） |
| `15090` | Envoy 自身的 Prometheus 暴露端口 |
| `15021` | Pilot agent 健康检查 |

> Istio Ambient / Cilium Mesh 用 **eBPF** 替代 iptables：在内核 socket 层重定向，跳过协议栈往返，性能更好。

### 5.2 一次请求的完整旅程

```text
服务A → 调用 → 服务B 的 /api/users：

┌──────────┐    ┌──────────┐         ┌──────────┐    ┌──────────┐
│  服务 A   │───▶│ Sidecar A │───网络──▶│ Sidecar B │───▶│  服务 B   │
│  :8080   │    │  :15001  │         │  :15006  │    │  :8080   │
└──────────┘    └──────────┘         └──────────┘    └──────────┘
   应用代码        出站代理              入站代理         应用代码
```

详细步骤：

```text
1. 服务A 发起请求：GET http://service-b:8080/api/users
   │
   ▼
2. iptables / eBPF 透明劫持 → 转发到本地 Sidecar A (Envoy :15001)
   │  ← 应用完全无感知，认为是直接发出
   ▼
3. Sidecar A（出站）处理：
   - 服务发现：service-b → 一组 Pod IP
   - 负载均衡：从健康实例池中按策略选一个
   - 熔断 / 重试：检查目标健康度、失败重试
   - mTLS：用 SPIFFE 身份发起 TLS 握手并加密
   - 生成 trace span、上报 metrics
   │
   ▼
4. 跨 Pod 网络传输（已加密）
   │
   ▼
5. Sidecar B（入站 :15006）处理：
   - mTLS 解密、校验调用方身份
   - 访问控制：服务A 是否被允许访问 /api/users？
   - 限流 / 配额：是否超出阈值？
   - 记录 metrics / trace
   │
   ▼
6. 转发到本地 服务B :8080，服务B 正常处理请求
   │
   ▼
7. 响应原路返回：服务B → Sidecar B → Sidecar A → 服务A
```

### 5.3 性能开销与优化

| 环节 | 延迟增量 |
|------|---------|
| 一跳经过两个 Sidecar | 约 1–3 ms |
| mTLS 加解密 | 约 0.5–1 ms |
| Sidecar CPU 占用 | 约 0.05–0.2 core / 1k QPS |
| 总额外开销 | 通常 **2–5 ms / 请求** |

直连 vs Mesh：

```text
直连：  服务A ──────────────────────▶ 服务B          约 0.5 ms
Mesh：  服务A → Sidecar A → Sidecar B → 服务B       约 3–5 ms
```

常见优化手段：

- **协议复用**：开启 HTTP/2 / gRPC 多路复用，减少握手成本。
- **mTLS Session 复用**：避免每个连接都重新握手。
- **Sidecar 资源调优**：根据 QPS 给 Envoy 合理的 CPU / 内存 request / limit，避免成为瓶颈。
- **Sidecar 配置裁剪**：通过 `Sidecar` CRD 只下发当前 Pod 真正需要的 xDS 配置，减小内存（大集群效果显著）。
- **Sidecarless**：高 QPS 服务可考虑 Cilium Mesh / Istio Ambient，去掉一跳。

---

## 六、主流实现对照

### 6.1 Istio（功能最全）

| 维度 | 说明 |
|------|------|
| 出品 | Google + IBM + Lyft（Envoy 作者），现归 CNCF |
| 控制面 | `istiod`（融合 Pilot / Citadel / Galley） |
| 数据面 | Envoy |
| 优势 | 功能最全（流量管理、安全、可观测性、Wasm 扩展），生态最大 |
| 劣势 | 复杂度高、资源占用大、CRD 多、学习曲线陡 |
| 新趋势 | **Ambient Mesh**（无 Sidecar 模式，L4 走 ztunnel、L7 走 waypoint） |

### 6.2 Linkerd（轻量极简）

| 维度 | 说明 |
|------|------|
| 出品 | Buoyant，CNCF 毕业项目 |
| 控制面 | linkerd-control-plane（Go） |
| 数据面 | linkerd2-proxy（Rust 实现，只为 Mesh 而生） |
| 优势 | 极轻量、极简单、内存占用小、安全默认值好 |
| 劣势 | 功能比 Istio 少（无 Wasm、协议支持窄），生态较小 |
| 适合 | 中小规模、对资源 / 复杂度敏感的团队 |

### 6.3 Consul Connect（多环境通用）

| 维度 | 说明 |
|------|------|
| 出品 | HashiCorp |
| 控制面 | Consul Server（同时是服务注册中心） |
| 数据面 | Envoy（也支持 built-in proxy） |
| 优势 | 同时跑在 K8s / VM / 多数据中心，适合**混合云**与传统基础设施迁移 |
| 劣势 | K8s 原生体验不如 Istio / Linkerd |
| 适合 | 多环境异构、需要服务注册中心 + Mesh 一体化的团队 |

### 6.4 Cilium Service Mesh（eBPF 派）

| 维度 | 说明 |
|------|------|
| 出品 | Isovalent，CNCF 毕业项目（核心是 CNI） |
| 控制面 | Cilium Operator |
| 数据面 | **eBPF**（L4）+ Envoy（仅 L7 按需起） |
| 优势 | 无 Sidecar / 节点级 L4 转发、内核态处理、性能接近裸金属 |
| 劣势 | L7 能力仍依赖 Envoy；与 CNI 强耦合，需要替换网络插件 |
| 适合 | 已用 Cilium 做 CNI、追求性能与低开销的团队 |

### 6.5 选型矩阵

| 团队 / 场景 | 推荐 | 理由 |
|------------|------|------|
| 大规模微服务、多语言、需要全功能治理 | Istio | 功能最全、社区最大，先上 Sidecar 模式，后期可平滑迁移 Ambient |
| 中小规模、追求轻量与稳定 | Linkerd | 简单、Rust 数据面、默认安全 |
| 混合云 / VM + K8s 共存 | Consul | 跨基础设施一体化注册 + Mesh |
| 已用 Cilium CNI、追求极致性能 | Cilium Service Mesh | eBPF L4、节点级代理、Sidecar 可选 |
| 仅需 mTLS + 简单流量管理 | Linkerd / Cilium | 不要为用不上的功能付出 Istio 的复杂度 |
| 仅需限流 / 鉴权而不想全量引入 Mesh | API Gateway（Kong / APISIX / Envoy Gateway） | 网关足以，不必上 Mesh |

> **避坑**：选型应优先匹配团队规模与运维能力，而不是"功能列表"。Istio 强大但需要专职 SRE 才能驾驭。

---

## 七、Sidecar vs Sidecarless

| 维度 | Sidecar（经典） | Sidecarless（新趋势） |
|------|----------------|--------------------|
| 部署形态 | 每个 Pod 一个代理容器 | 节点级代理 / eBPF 内核态 |
| 代表实现 | Istio（默认）、Linkerd | Cilium Service Mesh、Istio Ambient、gRPC Proxyless（xDS 直连） |
| 资源开销 | 高（每 Pod 约 50 MB+） | 低（节点级共享） |
| 隔离性 | 强（一个 Sidecar 一个工作负载） | 弱（节点级代理是共享组件） |
| 启停 / 升级 | 滚动重启业务 Pod | 节点级独立升级，业务不受影响 |
| L7 能力 | 全量 | Ambient 通过 waypoint 按需启用 |
| 成熟度 | 生产广泛验证 | 仍在演进，部分能力对齐中 |

> **趋势判断**：Sidecar 解决了"有没有"的问题，Sidecarless 解决"够不够轻"的问题。短期内两者并存；超大规模场景（千 Pod 以上）会逐步迁移到 Sidecarless 以节省成本。

---

## 八、适用与不适用场景

适合：

- 微服务数量多（**几十到上百个以上**）
- 多语言技术栈（Java / Go / Node / Python 混合）
- 需要统一的**安全策略**（零信任、mTLS）与**可观测性**
- 需要精细的**流量控制**（灰度、金丝雀、影子流量、A/B 测试）
- 已具备 Kubernetes + 监控告警体系，能承接 Mesh 的运维成本

不适合：

- 服务少（**< 10 个**）、架构简单的项目 —— Mesh 反而带来额外运维负担
- **超低延迟场景**：高频交易、实时游戏、行情推送（每跳 2–5 ms 都不可接受）
- **超深调用链**：单次请求经过 10+ 跳服务，累积延迟会成倍放大
- **资源敏感的边缘 / IoT 环境**：每 Pod 多一个 Envoy（约 50–100 MB 内存）难以承受
- 仅需要"网关 + 限流"：上 API Gateway 即可，不必引入 Mesh

---

## 九、常见反例与误区

| 反例 | 问题 | 正确做法 |
|------|------|---------|
| "微服务一定要上 Mesh" | 治理需求不够时，引入反而成本 > 收益 | 先看场景，按 [八、适用与不适用场景](#八适用与不适用场景) 判断 |
| 把业务逻辑写进 EnvoyFilter / Wasm | 重蹈 ESB 覆辙，业务被困在 Mesh | Mesh 只承担通信，业务一律在业务进程里实现 |
| Mesh 替代所有限流 / 鉴权 | 高级业务策略需要业务上下文，Mesh 拿不到 | 通用策略放 Mesh，业务策略放业务 / 中间件 |
| Sidecar 跟业务共用资源限制 | OOM 时一起挂，丧失隔离意义 | Sidecar 和业务容器独立设置 CPU / 内存 request / limit |
| 直接在生产开启全链路 mTLS STRICT | 老服务、外部依赖会瞬间被切断 | 先 `PERMISSIVE` 灰度，监控握手成功率后再切 `STRICT` |
| Trace 只看 Mesh 自动生成的 span | 业务内部耗时被吞掉 | 业务侧仍需 SDK 埋点 + 透传 trace context |
| 同时上 Spring Cloud + Istio 双套治理 | 双重重试 / 双重熔断，行为不可预测 | 选一方为主，另一方关闭对应能力 |
| 把 Ingress / API Gateway 也丢给 Mesh | 边界流量与东西向耦合 | 边界用专门网关（Envoy Gateway / Kong），Mesh 只做东西向 |

---

## 十、工程化落地建议

1. **先解决"为什么用"**：在动手前明确治理痛点（多语言一致性？mTLS？灰度？可观测性？），列出 Mesh 能解决但 SDK 解不了的至少 3 项，再立项。
2. **从小集群 / 非核心业务起步**：选一个独立的命名空间或 BU，跑通 6–8 周，沉淀监控告警、灰度发布、回滚 SOP，再向核心业务推广。
3. **按命名空间分批接入**：用 `istio-injection=enabled` label 控制注入范围，做到**可灰度、可回退**。
4. **mTLS 渐进开启**：先 `PERMISSIVE` 全量铺开 → 监控握手成功率 → 切 `STRICT`。期间盯紧 `istio_requests_total{response_code="0"}` 这类异常码。
5. **CRD 治理**：所有 `VirtualService` / `DestinationRule` / `AuthorizationPolicy` 都进 Git 仓库，通过 GitOps（Argo CD / Flux）下发，禁止 `kubectl apply` 临时改动。
6. **明确边界**：用一份团队规约写清楚——什么能力归 Mesh，什么归 SDK，什么归 API Gateway。避免"治理责任三不管"。
7. **资源 quota 与容量规划**：每 1000 QPS 大致需要 0.1–0.3 core + 50–100 MB 给 Sidecar，提前算入容量。
8. **全链路压测纳入 Sidecar**：性能基线必须**带 Sidecar** 跑，避免上线后发现额外延迟超预算。
9. **配置裁剪**：用 `Sidecar` CRD 限制每个工作负载下发的 xDS 配置范围（大集群下能把 Envoy 内存从数百 MB 降到几十 MB）。
10. **监控控制面本身**：`istiod` / `linkerd-controller` 也是关键依赖，配置面 OOM / 推送延迟会导致全网治理失效，必须接告警。
11. **保留逃生通道**：保留"一键关闭 Mesh"的预案（取消注入 + 滚动重启），任何重大故障可快速回退到无 Mesh 状态。
12. **关注社区演进**：Istio Ambient、Cilium Mesh、gRPC Proxyless 等 Sidecarless 模式正快速成熟，每年评估一次是否值得迁移。

---

## 参考资料

- [Istio 官方文档](https://istio.io/latest/docs/)
- [Linkerd 官方文档](https://linkerd.io/2/overview/)
- [Envoy Proxy 官方文档](https://www.envoyproxy.io/docs/envoy/latest/)
- [Envoy xDS 协议](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol)
- [Consul Connect — HashiCorp](https://developer.hashicorp.com/consul/docs/connect)
- [Cilium Service Mesh](https://cilium.io/use-cases/service-mesh/)
- [Istio Ambient Mesh](https://istio.io/latest/docs/ambient/overview/)
- [SPIFFE / SPIRE — 零信任身份标准](https://spiffe.io/)
- [What's a Service Mesh? And Why Do I Need One? — William Morgan（Buoyant）](https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/)
- [Pattern: Service Mesh — Phil Calçado](https://philcalcado.com/2017/08/03/pattern_service_mesh.html)
- [The Istio service mesh — Google Cloud](https://cloud.google.com/learn/what-is-istio)
- [gRPC Proxyless Service Mesh](https://grpc.io/blog/grpc-with-xds/)
- [CNCF Service Mesh Landscape](https://landscape.cncf.io/category=service-mesh)
