# Go Modules 版本规范

> 汇总 Go Modules 下的版本号规范、预发布（测试）版本约定，以及如何在 `go.mod` 中引用尚未打 tag 的依赖。
>
> 本文遵循 [Semantic Versioning 2.0.0](https://semver.org/) 与官方 [Go Modules Reference](https://go.dev/ref/mod#versions)。

## 目录

- [Go Modules 版本规范](#go-modules-版本规范)
  - [目录](#目录)
  - [一、版本号规范](#一版本号规范)
  - [二、测试版本命名约定](#二测试版本命名约定)
    - [2.1 标准预发布标识符（推荐）](#21-标准预发布标识符推荐)
    - [2.2 Go 官方常用格式](#22-go-官方常用格式)
    - [2.3 版本优先级（从低到高）](#23-版本优先级从低到高)
  - [三、伪版本（Pseudo-version）](#三伪版本pseudo-version)
    - [3.1 结构](#31-结构)
    - [3.2 三种形态](#32-三种形态)
  - [四、在 go.mod 中使用测试版本](#四在-gomod-中使用测试版本)
    - [4.1 直接指定预发布版本](#41-直接指定预发布版本)
    - [4.2 使用伪版本引用 commit](#42-使用伪版本引用-commit)
    - [4.3 默认行为](#43-默认行为)
  - [五、引用未打 Tag 的依赖](#五引用未打-tag-的依赖)
    - [5.1 引用某个分支的最新 commit（最常用）](#51-引用某个分支的最新-commit最常用)
    - [5.2 引用指定 commit](#52-引用指定-commit)
    - [5.3 引用最新版（含预发布 / 伪版本）](#53-引用最新版含预发布--伪版本)
    - [5.4 完整操作流程](#54-完整操作流程)
  - [六、特殊场景](#六特殊场景)
    - [6.1 私有仓库](#61-私有仓库)
    - [6.2 本地未推送的代码（replace 指令）](#62-本地未推送的代码replace-指令)
    - [6.3 Fork 后替换](#63-fork-后替换)
    - [6.4 子目录中的 module](#64-子目录中的-module)
    - [6.5 Major Version ≥ 2 的特殊规则](#65-major-version--2-的特殊规则)
  - [七、命名规范建议](#七命名规范建议)
  - [八、版本比较](#八版本比较)
  - [九、常见坑与注意事项](#九常见坑与注意事项)
  - [十、快速查询 commit 对应的伪版本](#十快速查询-commit-对应的伪版本)
  - [十一、速查表](#十一速查表)
  - [参考资料](#参考资料)

---

## 一、版本号规范

Go 遵循 [Semantic Versioning 2.0.0](https://semver.org/) 规范，版本格式为：

```plain
v<major>.<minor>.<patch>[-<pre-release>][+<build>]
```

**核心规则**：

- Tag 必须以 `v` 开头（`v1.0.0` ✅、`1.0.0` ❌）。
- 带 `-` 后缀的即为**预发布版本**（pre-release），优先级低于同主版本的正式版本。
- `+` 之后是构建元数据，**不参与版本比较**。

---

## 二、测试版本命名约定

### 2.1 标准预发布标识符（推荐）

| 类型 | 示例 | 说明 |
| --- | --- | --- |
| **Alpha** | `v1.2.0-alpha`、`v1.2.0-alpha.1` | 内部测试，功能不完整 |
| **Beta** | `v1.2.0-beta`、`v1.2.0-beta.2` | 公开测试，功能基本完整 |
| **RC** | `v1.2.0-rc.1`、`v1.2.0-rc.2` | Release Candidate，候选发布版本 |

### 2.2 Go 官方常用格式

```plain
v1.0.0-alpha
v1.0.0-alpha.1
v1.0.0-beta
v1.0.0-beta.2
v1.0.0-rc.1
v1.0.0            ← 正式版
```

### 2.3 版本优先级（从低到高）

```plain
v1.0.0-alpha  <  v1.0.0-alpha.1  <  v1.0.0-beta  <  v1.0.0-beta.2
              <  v1.0.0-rc.1     <  v1.0.0-rc.2  <  v1.0.0
```

SemVer 的比较规则：

- 数字标识符按**数值**比较：`alpha.2 < alpha.10`
- 字母标识符按 **ASCII** 排序：`alpha < beta < rc`
- 有预发布标识的版本**小于**不带标识的同主版本

更完整的比较示例见 [第八节](#八版本比较)。

---

## 三、伪版本（Pseudo-version）

当依赖尚未打 tag，或需要引用特定 commit 时，Go 会自动生成**伪版本号**：

```plain
v0.0.0-20230101120000-abcdef123456
v1.2.3-0.20230101120000-abcdef123456
v1.2.3-pre.0.20230101120000-abcdef123456
```

### 3.1 结构

```plain
<base-version>-<timestamp>-<commit-hash12>
```

| 段 | 说明 |
| --- | --- |
| `base-version` | 基础版本（`v0.0.0` 表示无 tag，否则为最近 tag 的下一版） |
| `timestamp` | UTC 提交时间，格式 `yyyyMMddHHmmss` |
| `commit-hash12` | commit SHA 的前 12 位 |

### 3.2 三种形态

1. **无任何 tag**：`v0.0.0-20230101120000-abcdef123456`
2. **在 tag `v1.2.2` 之后的 commit**：`v1.2.3-0.20230101120000-abcdef123456`
3. **在预发布 tag `v1.2.3-pre` 之后**：`v1.2.3-pre.0.20230101120000-abcdef123456`

---

## 四、在 go.mod 中使用测试版本

### 4.1 直接指定预发布版本

```go
require (
    github.com/some/pkg v1.2.0-beta.1
    github.com/other/pkg v2.0.0-rc.3
)
```

### 4.2 使用伪版本引用 commit

```bash
go get github.com/some/pkg@abcdef123456
go get github.com/some/pkg@master
go get github.com/some/pkg@v1.2.0-beta.1
```

### 4.3 默认行为

- `go get pkg` **不会**自动升级到预发布版本。
- 必须显式指定：`go get pkg@v1.0.0-beta.1`，或当 latest tag 本身就是预发布时使用 `go get pkg@latest`。

---

## 五、引用未打 Tag 的依赖

**结论**：无需打 tag，`go get pkg@<branch|commit>` 即可，Go 会自动生成伪版本号锁定到具体 commit。

### 5.1 引用某个分支的最新 commit（最常用）

```bash
go get github.com/user/repo@main
go get github.com/user/repo@master
go get github.com/user/repo@develop
```

执行后 `go.mod` 会被改写成伪版本：

```plain
require github.com/user/repo v0.0.0-20240315093045-abc123def456
```

### 5.2 引用指定 commit

```bash
go get github.com/user/repo@abc123def456
go get github.com/user/repo@abc123d
```

支持完整 SHA 或短 SHA（最少 7 位）。

### 5.3 引用最新版（含预发布 / 伪版本）

```bash
go get github.com/user/repo@latest
```

> **注意**：`@latest` 优先选已打 tag 的最新**正式版**；若无任何 tag，才会落到默认分支最新 commit 的伪版本。

### 5.4 完整操作流程

```bash
cd your-project
go get github.com/someone/newlib@main
go mod tidy
```

完成后 `go.mod` 示例：

```plain
module myapp
go 1.22
require github.com/someone/newlib v0.0.0-20240315093045-abc123def456
```

`go.sum` 会自动添加对应的 hash 校验。

---

## 六、特殊场景

### 6.1 私有仓库

需配置 `GOPRIVATE`，跳过公共代理和校验：

```bash
export GOPRIVATE=github.com/yourorg/*,gitlab.company.com
go get gitlab.company.com/team/lib@main
```

配合 `~/.netrc` 或 SSH 配置完成鉴权。如只想跳过代理或校验，可单独使用 `GONOPROXY` / `GONOSUMCHECK`。

### 6.2 本地未推送的代码（replace 指令）

引用本地目录，调试时最方便：

```plain
require github.com/user/repo v0.0.0-00010101000000-000000000000
replace github.com/user/repo => ../repo
```

`require` 行的伪版本可以是**占位符**（`v0.0.0-00010101000000-000000000000`），只要有 `replace` 就不会真实拉取。

### 6.3 Fork 后替换

```plain
require github.com/original/repo v1.2.3
replace github.com/original/repo => github.com/yourfork/repo v0.0.0-20240315093045-abc123def456
```

或通过命令行：

```bash
go mod edit -replace github.com/original/repo=github.com/yourfork/repo@main
go mod tidy
```

### 6.4 子目录中的 module

一个仓库里允许存在多个 `go.mod`（典型场景：主仓库 + 独立发版的 CLI / SDK / examples）。**引用**与**发版**两端都要遵循「路径前缀」规则。

**引用**：直接把子目录路径拼到 module 路径后面即可：

```bash
go get github.com/user/repo/subdir@main
go get github.com/user/repo/subdir@v1.2.3
```

**发版**：子模块的 tag 必须以**子目录路径作为前缀**，否则 Go 工具链识别不到。

| Module 路径 | tag 写法 |
| --- | --- |
| `github.com/user/repo`（根模块） | `v1.2.3` |
| `github.com/user/repo/subdir` | `subdir/v1.2.3` |
| `github.com/user/repo/tools/cli` | `tools/cli/v1.2.3` |
| `github.com/user/repo/tools/cli/v2`（v2+） | `tools/cli/v2.0.0` |

完整示例，仓库结构如下：

```plain
repo/
├── go.mod              ← module github.com/user/repo
├── main.go
└── tools/
    └── cli/
        ├── go.mod      ← module github.com/user/repo/tools/cli
        └── main.go
```

发版命令：

```bash
git tag v1.2.3              # 发布根模块
git tag tools/cli/v0.5.0    # 发布子模块（与根模块版本完全独立）
git push origin --tags
```

**几点提醒**：

- 根模块与各子模块的版本号**彼此独立**，不需要保持一致。
- 子模块若升到 `v2+`，module 路径与 tag **都**要带 `/vN`：module 写 `.../tools/cli/v2`，tag 写 `tools/cli/v2.0.0`。
- 如果只打了 `v1.2.3` 没加前缀，`go get .../subdir@v1.2.3` 会报 `unknown revision` 或解析到错误的 commit。
- 验证拉取到的版本：`go list -m -json github.com/user/repo/tools/cli@latest`，看输出中的 `Version` 字段。

### 6.5 Major Version ≥ 2 的特殊规则

Go Modules 对 `v2+` 的包路径有特殊要求，module 路径必须带 `/vN` 后缀：

```plain
module github.com/user/pkg/v2
require github.com/user/pkg/v2 v2.0.0-beta.1
```

预发布版本与伪版本同样遵循 `/vN` 规则。

---

## 七、命名规范建议

**推荐做法**：

- 使用 `-alpha` / `-beta` / `-rc` 三级标识
- 带点号序号：`-beta.1`、`-beta.2`（便于数值比较）
- Tag 必须以 `v` 开头：`v1.0.0-rc.1` ✅，`1.0.0-rc.1` ❌
- 每次预发布迭代递增序号

**避免做法**：

- ❌ `v1.0.0-beta1`（无分隔，按字符串比较可能出问题）
- ❌ `v1.0.0-BETA`（大小写不一致）
- ❌ `v1.0.0-test`、`v1.0.0-dev`（非标准，语义不明）
- ❌ `v1.0.0_beta`（SemVer 不允许下划线）

---

## 八、版本比较

```plain
v1.0.0-alpha      <  v1.0.0-alpha.1
v1.0.0-alpha.1    <  v1.0.0-alpha.beta
v1.0.0-alpha.beta <  v1.0.0-beta
v1.0.0-beta       <  v1.0.0-beta.2
v1.0.0-beta.2     <  v1.0.0-beta.11
v1.0.0-beta.11    <  v1.0.0-rc.1
v1.0.0-rc.1       <  v1.0.0
```

可通过 `golang.org/x/mod/semver` 包以编程方式比较：

```go
import "golang.org/x/mod/semver"

semver.Compare("v1.0.0-beta.1", "v1.0.0-rc.1") // -1
semver.IsValid("v1.0.0-beta.1")                // true
semver.Prerelease("v1.0.0-beta.1")             // "-beta.1"
```

---

## 九、常见坑与注意事项

| 问题 | 说明 / 解决 |
| --- | --- |
| GOPROXY 拉不到 | 默认 `proxy.golang.org` 访问不到私有仓库，配置 `GOPRIVATE` 或 `GONOPROXY` |
| 依赖方强制 SemVer | `v2+` 的 module 路径必须带 `/v2`，伪版本同样受此约束 |
| `go get` 不升级预发布 | 需要显式指定分支或 commit，不会自动跳到预发布 |
| 校验失败（sum mismatch） | 清缓存：`go clean -modcache`，或检查 `GOSUMDB` 配置 |
| 引用分支后想锁定 | 伪版本本身已锁定到具体 commit，CI 中是可重现的 |
| 时间格式必须 UTC | 手写伪版本时必须用 UTC 时间，否则 Go 拒绝 |

---

## 十、快速查询 commit 对应的伪版本

```bash
go list -m -json github.com/user/repo@abc123d
go list -m -json github.com/user/repo@main
```

输出的 `Version` 字段即伪版本号。

---

## 十一、速查表

| 需求 | 命令 |
| --- | --- |
| 用分支最新代码 | `go get pkg@branch` |
| 用指定 commit | `go get pkg@commit-sha` |
| 用最新 tag / 伪版本 | `go get pkg@latest` |
| 使用预发布版本 | `go get pkg@v1.0.0-rc.1` |
| 本地调试 | `replace` 指向本地目录 |
| 私有仓库 | 设置 `GOPRIVATE` |
| 查询伪版本 | `go list -m -json pkg@<ref>` |

---

## 参考资料

- [Go Modules Reference - Versions](https://go.dev/ref/mod#versions)
- [Go Modules Reference - Pseudo-versions](https://go.dev/ref/mod#pseudo-versions)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [golang.org/x/mod/semver](https://pkg.go.dev/golang.org/x/mod/semver)
