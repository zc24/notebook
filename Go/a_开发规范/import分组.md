# Go Import 分组规范

> Go 社区对 `import` 分组并没有强制性的官方规范，但有一套被广泛接受的事实标准。本文从主流到细化整理几种常见分组方式以及组内排版规则，帮助团队统一 `import` 块的风格。

## 目录

- [Go Import 分组规范](#go-import-分组规范)
  - [目录](#目录)
  - [一、官方与社区主流分组方案](#一官方与社区主流分组方案)
    - [1. gofmt（官方）](#1-gofmt官方)
    - [2. goimports（官方扩展）](#2-goimports官方扩展)
    - [3. Google Go Style Guide（2 组）](#3-google-go-style-guide2-组)
    - [4. Uber Go Style Guide（2~3 组）](#4-uber-go-style-guide23-组)
    - [5. 四组方案（含公司私有库，业界主流）](#5-四组方案含公司私有库业界主流)
  - [二、组内排版规则](#二组内排版规则)
  - [三、应当避免的写法](#三应当避免的写法)

---

## 一、官方与社区主流分组方案

### 1. gofmt（官方）

`gofmt` 是 Go 官方自带的格式化工具，对 `import` 的处理规则：

- 多个 `import` 必须写在 `import (...)` 块内
- 同一组内按字符串字典序自动排序
- 不会主动分组，但会保留你用空行划分的分组（组与组之间空行分隔）

```go
import (
    "fmt"
    "net/http"

    "github.com/foo/bar"
    "myproject/internal/x"
)
```

### 2. goimports（官方扩展）

[`goimports`](https://pkg.go.dev/golang.org/x/tools/cmd/goimports) 是 Go 官方扩展工具，在 `gofmt` 基础上对 `import` 分组做了约定：

- 默认按 **标准库 / 其他**（第三方与本地混在一起）两组划分
- 通过 `-local` 参数（对应 `golangci-lint` 的 `local-prefixes`）可额外切出第 3 组：本地包

```go
import (
    "fmt"                    // 第 1 组：标准库
    "github.com/foo/bar"     // 第 2 组：第三方
    "myproject/internal/x"   // 第 3 组：本地（-local 指定的前缀）
)
```

这是官方工具能做到的最大粒度，更细的分组需借助社区工具（如 `gci`）。

### 3. Google Go Style Guide（2 组）

Google Go Style Guide 要求**至少分两组**：标准库与其他。

```go
import (
    // Group 1: standard library
    "fmt"
    "hash/adler32"
    "os"

    // Group 2: everything else
    "github.com/foo/bar"
    "golang.org/x/text/encoding"
    "myproject/users/lib/auth"
)
```

并允许进一步细分（推荐使用 [`gci`](https://github.com/daixiang0/gci) 实现），例如把 protobuf 生成的包、本组织的包各自单独成组。

### 4. Uber Go Style Guide（2~3 组）

Uber Go Style Guide 推荐 2 组，项目较大时接受 3 组（标准库 / 第三方 / 项目内）。这是 Uber、Google 等大厂以及多数开源项目最常用的写法，可使用 `goimports -local` 或 `gci` 强制执行。

```go
import (
    "context"
    "fmt"
    "net/http"

    "github.com/gin-gonic/gin"
    "go.uber.org/zap"
    "google.golang.org/grpc"

    "github.com/myorg/myproject/internal/config"
    "github.com/myorg/myproject/internal/service"
)
```

### 5. 四组方案（含公司私有库，业界主流）

随着 `gci`（Go Code Imports）的流行，**4 分组**已经成为大型 Go 项目的事实标准，适合大型 monorepo 或有明确公司私有库的场景。

| 顺序 | 分组 | 说明 |
| --- | --- | --- |
| 1 | 标准库 | `fmt`、`context`、`os` … |
| 2 | 第三方依赖 | `github.com/...`、`go.uber.org/...` |
| 3 | 公司 / 框架级共享包 | GoFrame、kratos、go-zero、公司内部基础库等 |
| 4 | 当前项目内部包 | `<module-name>/...` |

**实际公开案例**：

- GoFrame 官方项目：标准库 / 第三方 / `github.com/gogf/gf` / 本项目
- kratos（B 站开源）：标准库 / 第三方 / `github.com/go-kratos/kratos` / 本项目
- TiDB / PingCAP：标准库 / 第三方 / `github.com/pingcap/...` / 本项目
- Kubernetes：标准库 / 第三方 / `k8s.io/...` / 本项目（用 importas + 自定义脚本）
- Docker / Moby：标准库 / 第三方 / `github.com/docker/...` / 本项目

**`gci` 配置示例**：

```yaml
gci:
  sections:
    - standard
    - default
    - prefix(公司git域名)
    - prefix(当前项目module)
  custom-order: true
```

**导入示例**：

```go
import (
    // 1. 标准库
    "context"
    "fmt"

    // 2. 第三方开源
    "github.com/gin-gonic/gin"
    "go.uber.org/zap"

    // 3. 公司内部共享库
    "git.mycompany.com/common/logger"
    "git.mycompany.com/common/middleware"

    // 4. 当前项目
    "github.com/myorg/myproject/internal/config"
)
```

> 上面的 `// 1. 标准库` 等注释仅用于文档讲解，实际代码中应交给 `gci` / `goimports` 自动分组，**不要手写注释分组**。

---

## 二、组内排版规则

无论分几组，组内排版都应遵循以下规则，由工具自动维护，不要手动调整：

1. **字典序**：组内按字符串字典序排序。
2. **空行只在组之间**：仅在不同分组之间插入一个空行，组内不要插入空行。
3. **同名包用别名区分**：避免编译期歧义。

   ```go
   import (
       crand "crypto/rand"
       "math/rand"
   )
   ```

4. **副作用导入**：`_ "xxx"` 可紧跟所属分组，也可**单独成组放在最后**以突出其副作用语义；同时**必须加注释说明用途**。

   ```go
   import (
       "database/sql"

       _ "github.com/go-sql-driver/mysql" // register mysql driver
   )
   ```

---

## 三、应当避免的写法

- ❌ 一行一个 `import`，而不是使用 `import ( ... )` 块
- ❌ 手动插入空行或手写注释分组，打乱工具排序（交给 `gci` / `goimports` 管理）
- ❌ 使用 dot import（`import . "xxx"`），测试和 DSL 场景例外
- ❌ 空白导入（`_ "xxx"`）不写注释说明用途
- ❌ CI 中未通过 `golangci-lint` 的 `formatters` 强制 `gci` / `goimports`
