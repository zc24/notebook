# Go Module 命名规范

> Go 工具链本身不强制 `module path` 的格式，但社区与官方文档都建议使用「域名 + 路径」的形式。这样既能保证全局唯一，也能让 `go get` 通过 import path 自动定位仓库。本文整理 module 命名的约定、反例与推荐做法。

## 目录

- [Go Module 命名规范](#go-module-命名规范)
  - [目录](#目录)
  - [1. Go 工具链层面：不强制](#1-go-工具链层面不强制)
  - [2. 官方与社区共识：要加域名](#2-官方与社区共识要加域名)
  - [3. 为什么"裸名字"不推荐](#3-为什么裸名字不推荐)
  - [4. 项目实践建议](#4-项目实践建议)
  - [5. 特殊场景例外](#5-特殊场景例外)

---

## 1. Go 工具链层面：不强制

`go mod init <module-path>` 本身不会校验你写的是不是真实域名，下面这些都能编译通过：

```bash
go mod init myapp
go mod init hello/world
go mod init example.com/foo
```

但是——一旦你想让别人 `go get` 你的代码，或者你自己想引用别的仓库，就必须遵守 Go 的「**import path = 可下载地址**」约定。

---

## 2. 官方与社区共识：要加域名

[Go Modules Reference](https://go.dev/ref/mod) 与 Go Blog 都建议 module path 满足以下条件：

- **以域名开头**（含 `.`），用于全局唯一标识
- **域名 + 路径就是仓库地址**，Go 工具能据此自动 `go get`
- **全小写**，避免大小写敏感性问题

典型格式：

```text
<vcs-host>/<org-or-user>/<repo>[/<subpath>]
```

示例：

```text
github.com/gogf/gf/v2
gitlab.com/your-team/your-repo
gitee.com/your-org/your-repo
git.your-company.com/backend/sdc-spider-jobs
```

---

## 3. 为什么"裸名字"不推荐

如果你写 `module sdc-spider-jobs`，会有这些问题：

| 问题 | 说明 |
| --- | --- |
| 无法被外部 `go get` | Go 找不到 `sdc-spider-jobs` 对应的 VCS 地址 |
| 容易冲突 | 别的项目也可能叫这个名，无法保证唯一 |
| 私有仓库代理失效 | `GOPROXY` / `GOPRIVATE` 通过域名匹配 |
| IDE 跳转/重构出问题 | GoLand、VSCode 解析 import 依赖路径规则 |
| `v2+` 版本规则失效 | `/v2` 后缀依赖完整路径语义 |

---

## 4. 项目实践建议

如果项目结构里有 `internal/`、`go.work`，且是公司内部服务，推荐写成公司 Git 仓库的真实路径：

```go
// go.mod
module git.your-company.com/data-platform/sdc-spider-jobs

go 1.22
```

这样可以获得：

- ✅ `internal/` 包能被同 module 下任意目录引用，但禁止外部引用（Go 的 `internal` 规则）
- ✅ CI 拉私有依赖时，配合 `GOPRIVATE=git.your-company.com` 就能直接走 SSH/Token，不经公共代理
- ✅ 后续做 `v2` 升级时只需把路径改成 `git.your-company.com/data-platform/sdc-spider-jobs/v2`

---

## 5. 特殊场景例外

只有以下几种情况可以不加域名：

- **临时 demo / 学习项目**：从不打算发布或被别人引用
- **纯本地工具**：通过 `replace` 指向本地目录

  ```go
  // go.mod
  module local/tool

  require some/lib v0.0.0

  replace some/lib => ../lib
  ```

- **Monorepo 内的子模块**：由 `go.work` 统一管理，路径只在工作区内用

> 即便如此，仍建议写成域名形式，避免日后拆仓库时大改 `import`。
