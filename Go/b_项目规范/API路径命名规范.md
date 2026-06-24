# API 路径命名规范

> 后端服务对外暴露的 HTTP API 路径前缀（path prefix）该如何设计？本文结合阿里、腾讯、字节、钉钉、飞书、Stripe、GitHub 等公司的实践，讨论如何用 path prefix 区分**鉴权类型**，并给出推荐方案。

## 目录

- [API 路径命名规范](#api-路径命名规范)
  - [目录](#目录)
  - [一、用 path prefix 区分鉴权类型](#一用-path-prefix-区分鉴权类型)
  - [二、三类常见 API 的命名](#二三类常见-api-的命名)
    - [2.1 正常接口 `/api/v1/xxx`](#21-正常接口-apiv1xxx)
    - [2.2 免鉴权接口：使用 `public`](#22-免鉴权接口使用-public)
    - [2.3 开放平台接口：使用顶级 `openapi/`](#23-开放平台接口使用顶级-openapi)
      - [问题 1：开放平台和内部 API 通常是两套独立工程](#问题-1开放平台和内部-api-通常是两套独立工程)
      - [问题 2：工具链兼容性](#问题-2工具链兼容性)
  - [三、版本号 `v1` 该放在哪](#三版本号-v1-该放在哪)
    - [3.1 全局版本（版本前置）](#31-全局版本版本前置)
    - [3.2 产品级版本（版本后置）](#32-产品级版本版本后置)
    - [3.3 对比与选择](#33-对比与选择)
  - [四、推荐方案](#四推荐方案)
  - [五、附加经验](#五附加经验)
  - [参考资料](#参考资料)

---

## 一、用 path prefix 区分鉴权类型

把鉴权类型写进 path prefix（如 `/api/`、`/openapi/`、`/api/public/`），在大厂里是非常普遍的做法。主要有三个收益：

- **网关侧便于配置鉴权策略**：APISIX / Kong / 自研网关可以直接按 prefix 路由到不同 filter chain（登录态校验、AppKey 校验、限流策略完全不同）。
- **运维侧便于审计和灰度**：免鉴权接口往往是攻击重灾区，独立 prefix 方便单独打 WAF 规则、单独限流。
- **业务侧职责清晰**：开放平台和内部 BFF 的演进节奏完全不同，物理隔离比逻辑隔离更稳妥。

> **Note**：严格按 REST 原则，URI 应该只表达「资源」，不应该表达「鉴权方式」。但在工程实践中，**用 prefix 显式区分鉴权类型的收益远大于 REST 纯洁性的损失**，业内主流是实用主义路线。

---

## 二、三类常见 API 的命名

下面分别给出每类 API 的推荐前缀：

| 鉴权类型 | 推荐路径 | 说明 |
| --- | --- | --- |
| 正常接口（登录态） | `/api/v1/xxx` | 内部 BFF，最常见的写法 |
| 免鉴权接口 | `/api/public/xxx` | 匿名访问，语义直观 |
| 开放平台接口 | `/openapi/v1/xxx` | AppKey + Sign，建议顶级前缀 |

### 2.1 正常接口 `/api/v1/xxx`

最主流的写法，Stripe、GitHub、字节内部 BFF 全是这个风格，几乎是事实标准。

### 2.2 免鉴权接口：使用 `public`

免鉴权前缀的核心要求是**语义直观**——让任何一个新同事看到 path 就知道「这条路径不需要登录」。业内最常用的写法是 `public`，次选 `anon`（anonymous 的缩写）。

> **Warning**：避免使用含义模糊的前缀（内部黑话、未拼写完整的缩写等）。这类前缀既不能表达「免鉴权」语义，也难以被网关 / 审计工具自动识别，半年后维护者要靠注释或文档才能猜出用途。

业内常见的免鉴权前缀：

| 公司 / 项目 | 免鉴权前缀 |
| --- | --- |
| Spring Security 社区 | `/public/**` |
| AWS | `/anonymous/...` |
| 腾讯云控制台 | `/api/v1/public/...` |
| 阿里内部 | `/api/anon/...` 或 `/api/public/...` |
| Kubernetes | `/healthz`、`/readyz`（直接顶级） |

推荐写法：

```text
/api/public/xxx        ← 推荐（v1 可省略，详见三）
/api/anon/xxx          ← 次选
```

### 2.3 开放平台接口：使用顶级 `openapi/`

开放平台接口推荐使用**顶级前缀** `/openapi/v1/...`，而不是嵌在 `/api/open/...` 下面。原因有两个：

#### 问题 1：开放平台和内部 API 通常是两套独立工程

- 内部 API：服务公司自己的前端 / APP，重 BFF 风格。
- 开放平台：服务三方开发者，重 RESTful、重文档、重 SDK 生成、重版本兼容。
- 二者的**演进节奏、SLA、版本策略**都不一样，物理上往往是不同的服务、不同的域名。

业内普遍做法：开放平台**独立顶级 prefix 或独立域名**：

| 公司 | 开放平台路径 |
| --- | --- |
| 钉钉 | `https://api.dingtalk.com/v1.0/...` 或 `/openapi/...` |
| 飞书 | `https://open.feishu.cn/open-apis/...` |
| 微信 | `https://api.weixin.qq.com/...` |
| 京东 | `/openapi/...` 或独立 `api.jd.com` |
| 美团 | `/openapi/...` |
| GitHub | `api.github.com/...`（独立子域） |

#### 问题 2：工具链兼容性

`/openapi/...` 是业界事实标准，文档生成器、SDK 生成器、网关路由都默认能识别这个 prefix；`/api/open/...` 则需要每处手动配置。

> **Tip**：推荐写成顶级前缀，或更彻底地拆到独立子域：
>
> ```text
> /openapi/v1/xxx                              ← 推荐（顶级 prefix）
> https://open.yourcompany.com/api/v1/xxx      ← 更彻底（独立子域）
> ```

---

## 三、版本号 `v1` 该放在哪

`v1` 放在路径的**哪一段**，决定了它管辖的范围——这是两种完全不同的演进策略。

### 3.1 全局版本（版本前置）

版本号紧跟 prefix，放在**路径第二段**，**整套 API 共享一个 `v1`**：

```text
/api/v1/users
/api/v1/orders
/api/v1/public/login
```

升级到 `v2` 时整套 API 同步变更，所有路径一起切。

**代表**：Stripe `/v1/charges`、GitHub `/v3/users`、字节内部 BFF。

### 3.2 产品级版本（版本后置）

版本号放在「**产品 / 大模块**」之后（**路径第三段**），**每个产品独立演进**：

```text
/openapi/contact/v1/users      ← 联系人产品 v1
/openapi/contact/v2/users      ← 联系人独立升级到 v2
/openapi/calendar/v3/events    ← 日历独立到 v3，互不影响
```

联系人升 `v2` 完全不影响日历的 `v3`，三方开发者可以分批迁移。

**代表**：飞书 `/open-apis/contact/v3/users`、`/open-apis/calendar/v4/...`；钉钉旧 OpenAPI `/topapi/v2/user/get`。

### 3.3 对比与选择

| 维度 | 全局版本（前置） | 产品级版本（后置） |
| --- | --- | --- |
| URL 结构 | `prefix/v?/资源` | `prefix/产品/v?/资源` |
| 一次升级影响 | 整套 API | 单个产品 |
| 兼容期 | 短（协同发版） | 长（数月～数年） |
| 适用 | 内部 BFF | 开放平台 |
| 代表 | Stripe / GitHub | 飞书 / 钉钉 |

**核心原则**：

- **内部 BFF 用全局版本**——前后端 / 客户端同组织维护，可以协同发版，整套切换最省心。
- **开放平台用产品级版本**——三方开发者无法被强制升级，老版本要兼容数月～数年，**不能因为单个产品升级而拖累其他产品**。

> **Note**：免鉴权接口（`/api/public/...`）通常是健康检查、登录、注册、验证码这类**几乎不会变版本**的接口，可以**直接省掉 `v1`**：`/api/public/login` 即可。
>
> 另：单产品的开放平台（项目较小、只有一类业务）可以简化为 `/openapi/v1/xxx`；多产品时再演进到 `/openapi/{product}/v1/xxx`。

---

## 四、推荐方案

最终推荐写法（按业务类型选择对应风格）：

```text
/api/v1/xxx              # 内部接口（登录态 / Cookie / JWT）
/api/public/xxx          # 免鉴权接口（健康检查、登录、注册、验证码）
/openapi/v1/xxx          # 开放平台接口（AppKey + Sign）
```

---

## 五、附加经验

- **健康检查别放 `/api/public/` 下**：直接顶级 `/healthz`、`/readyz`、`/metrics`。K8s 探针、Prometheus 抓取都按这个约定，工具链能自动识别。
- **开放平台一定要独立鉴权方案**：AppKey + Timestamp + Nonce + Sign 四件套缺一不可，**不要图省事直接拿内部 JWT 给三方用**。三方不可能像内部前端那样安全持有 token，泄露后只能整租户重置。
- **网关层强制校验 prefix**：
  - `/api/v1/*` 走登录态 filter。
  - `/openapi/v1/*` 走 AppKey filter。
  - `/api/public/*` 走限流但跳过鉴权。
  - **这套规则在网关配置里写死，业务方不能在 Controller 里用 `@PermitAll` / 自定义注解绕过**——这是安全红线。
- **避免「在鉴权路径下放免鉴权接口」**：例如 `/api/v1/users/login` 这种登录接口本身是免鉴权的，但路径在登录态前缀下，容易引发审计争议。要么挪到 `/api/public/login`，要么在网关白名单里单独配置并留好审计记录。
- **path 不应该出现资源 ID 之外的动词**：`/api/v1/users/123/delete` ❌，用 HTTP method `DELETE /api/v1/users/123` ✅。鉴权 prefix 是关于「这条路径怎么进来」的元信息，不是动词，所以放进 path 是合理的。

---

## 参考资料

- [阿里云 API 设计规范](https://help.aliyun.com/zh/api-gateway/)
- [钉钉开放平台 OpenAPI](https://open.dingtalk.com/document/orgapp/api-overview)
- [飞书开放平台 OpenAPI](https://open.feishu.cn/document/home/index)
- [Stripe API Reference](https://stripe.com/docs/api)
- [GitHub REST API](https://docs.github.com/en/rest)
- [Kubernetes API Conventions](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md)
