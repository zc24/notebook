# Go Import 规范

> Go 社区对 `import` 分组并没有强制性的官方规范，但已经形成了一套被广泛接受的事实标准。本文从分组方案、组内排版、重命名、副作用导入、点导入到应避免的写法，系统整理 `import` 的写法约定，帮助团队统一风格。

## 目录

- [Go Import 规范](#go-import-规范)
  - [目录](#目录)
  - [一、官方与社区主流分组方案](#一官方与社区主流分组方案)
    - [1. gofmt（官方）](#1-gofmt官方)
    - [2. goimports（官方扩展）](#2-goimports官方扩展)
    - [3. Google Go Style Guide](#3-google-go-style-guide)
    - [4. Uber Go Style Guide（2~3 组）](#4-uber-go-style-guide23-组)
    - [5. 业界主流四组方案（含公司私有库）](#5-业界主流四组方案含公司私有库)
  - [二、组内排版规则](#二组内排版规则)
  - [三、Import 重命名](#三import-重命名)
    - [1. 何时不应重命名](#1-何时不应重命名)
    - [2. 必须重命名的场景](#2-必须重命名的场景)
      - [2.1 名称冲突](#21-名称冲突)
      - [2.2 自动生成的 Protocol Buffer 包](#22-自动生成的-protocol-buffer-包)
    - [3. 推荐重命名的场景](#3-推荐重命名的场景)
    - [4. 与本地变量冲突时的命名习惯](#4-与本地变量冲突时的命名习惯)
  - [四、副作用导入（`import _`）](#四副作用导入import-_)
  - [五、点导入（`import .`）](#五点导入import-)
  - [六、应当避免的写法](#六应当避免的写法)
  - [七、推荐工具链与 CI 集成](#七推荐工具链与-ci-集成)
  - [参考资料](#参考资料)

---

## 一、官方与社区主流分组方案

### 1. gofmt（官方）

`gofmt` 是 Go 官方自带的格式化工具，对 `import` 的处理规则：

- 多个 `import` 必须写在 `import (...)` 块内。
- 同一组内按字符串字典序自动排序。
- 不会主动分组，但会保留你用空行划分的分组（组与组之间空行分隔）。

```go
import (
    "fmt"
    "net/http"

    "github.com/foo/bar"
    "myproject/internal/x"
)
```

### 2. goimports（官方扩展）

[goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports) 是 Go 官方扩展工具，在 `gofmt` 基础上对 `import` 分组做了约定：

- 默认按 **标准库 / 其他**（第三方与本地混在一起）两组划分。
- 通过 `-local` 参数（对应 `golangci-lint` 的 `local-prefixes`）可额外切出第 3 组：本地包。

```go
import (
    "fmt"                    // 第 1 组：标准库
    "github.com/foo/bar"     // 第 2 组：第三方
    "myproject/internal/x"   // 第 3 组：本地（-local 指定的前缀）
)
```

这是官方工具能做到的最大粒度，更细的分组需借助社区工具（如 `gci`）。

### 3. Google Go Style Guide

[Google Go Style Guide](https://google.github.io/styleguide/go/decisions#import-grouping) 要求 `import` 按以下顺序分组，组之间用空行分隔：

1. 标准库包
2. 其他（项目与 vendor）包
3. Protocol Buffer 生成包（如 `fpb "path/to/foo_go_proto"`）
4. 副作用导入（如 `_ "path/to/package"`）

> 后两组并不是每个文件都会出现；当存在时，应单独成组放在最后。

```go
// Good:
package main

import (
    "fmt"
    "hash/adler32"
    "os"

    "github.com/dsnet/compress/flate"
    "golang.org/x/text/encoding"
    "google.golang.org/protobuf/proto"

    foopb "myproj/foo/proto/proto"

    _ "myproj/rpc/protocols/dial"
    _ "myproj/security/auth/authhooks"
)
```

并允许进一步细分（推荐使用 [gci](https://github.com/daixiang0/gci) 实现），例如把组织内部的包单独成组。

### 4. Uber Go Style Guide（2~3 组）

[Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md#import-group-ordering) 推荐 2 组，项目较大时接受 3 组（标准库 / 第三方 / 项目内）。这是 Uber、Google 等大厂以及多数开源项目最常用的写法，可使用 `goimports -local` 或 `gci` 强制执行。

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

### 5. 业界主流四组方案（含公司私有库）

随着 [gci](https://github.com/daixiang0/gci)（Go Code Imports）的流行，**4 分组**已经成为大型 Go 项目的事实标准，适合大型 monorepo 或有明确公司私有库的场景。

| 顺序 | 分组                | 说明                                       |
| :--- | :------------------ | :----------------------------------------- |
| 1    | 标准库              | `fmt`、`context`、`os` …                   |
| 2    | 第三方依赖          | `github.com/...`、`go.uber.org/...`        |
| 3    | 公司 / 框架级共享包 | GoFrame、kratos、go-zero、公司内部基础库等 |
| 4    | 当前项目内部包      | `<module-name>/...`                        |

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

无论分几组，组内排版都应遵循以下规则，由工具自动维护，**不要手动调整**：

1. **字典序**：组内按字符串字典序排序。
2. **空行只在组之间**：仅在不同分组之间插入一个空行，组内不要插入空行。
3. **同名包用别名区分**：避免编译期歧义，重命名应遵循 [Import 重命名](#三import-重命名) 一节。

   ```go
   import (
       crand "crypto/rand"
       "math/rand"
   )
   ```

4. **副作用导入紧跟所属分组或单独成组**：`_ "xxx"` 可紧跟所属分组，也可**单独成组放在最后**以突出其副作用语义；同时**必须加注释说明用途**。详见 [副作用导入](#四副作用导入import-_) 一节。

   ```go
   import (
       "database/sql"

       _ "github.com/go-sql-driver/mysql" // register mysql driver
   )
   ```

---

## 三、Import 重命名

> 通常情况下，**包的导入名不应被更改**。只有在必须避免命名冲突，或重命名能显著提升可读性时，才考虑重命名。
>
> 对所有重命名后的本地名，都必须遵循 [包命名规范](./包.md)：禁止使用下划线和大写字母，并尽量保持一致性——同一个被导入的包，在所有文件中应使用相同的本地名。

### 1. 何时不应重命名

- 包名清晰、无冲突时，**不要为重命名而重命名**。
- 好的包名本就不需要重命名；如果你经常想重命名某个包，先考虑重构包本身。
- 避免出于个人偏好缩短或拉长包名。

### 2. 必须重命名的场景

#### 2.1 名称冲突

为避免与其他导入包发生名称冲突，**必须**对其中一个进行重命名。冲突时，**优先重命名最本地化或项目内特有的那个**，保留更通用、更广为人知的名字不变。

```go
// Good:
import (
    "math/rand"

    crand "crypto/rand"
)
```

#### 2.2 自动生成的 Protocol Buffer 包

由于 proto 库的跨语言特性，其包名通常带有下划线（如 `foo_service_go_proto`），不符合 Go 包命名规范，**必须重命名以去除下划线**。本地名的后缀取决于生成该包所用的规则：

| 生成规则           | 后缀   | 用途                     |
| :----------------- | :----- | :----------------------- |
| `go_proto_library` | `pb`   | Protocol Buffer 消息类型 |
| `go_grpc_library`  | `grpc` | gRPC 服务桩代码          |

```go
// Good:
import (
    foopb   "path/to/package/foo_service_go_proto"
    foogrpc "path/to/package/foo_service_go_grpc"
)
```

本地名同样需要遵循 [包命名规范](./包.md)：**优先使用完整单词**，简短名称也可以，但需避免歧义。如果不确定该如何命名，**默认采用 proto 包名中 `_go` 之前的部分（去除下划线）再加 `pb` 后缀**。

例如路径 `path/to/package/push_queue_service_go_proto`：

1. 取 `_go` 之前的部分 → `push_queue_service`；
2. 去除下划线 → `pushqueueservice`；
3. 加上 `pb` 后缀 → `pushqueueservicepb`。

```go
// Good:
import (
    foosvcpb           "path/to/package/foo_service_go_proto"
    pushqueueservicepb "path/to/package/push_queue_service_go_proto"
)
```

### 3. 推荐重命名的场景

如果某个**非自动生成**的包名不够清晰易懂（如 `util`、`v1`），可以重命名以提升可读性。但请**谨慎使用**：

- 如果代码上下文已经能清楚表达包的用途，则无需重命名。
- 在可能的情况下，**优先重构包本身**，为它选择一个更合适的名字。

```go
// Good:
import (
    core "github.com/kubernetes/api/core/v1"
    meta "github.com/kubernetes/apimachinery/pkg/apis/meta/v1beta1"
)
```

### 4. 与本地变量冲突时的命名习惯

如果某个包的名字（如 `url`、`ssh`）会与你想使用的本地变量名冲突，并且确实需要重命名，**推荐加上 `pkg` 后缀**（如 `urlpkg`）。

```go
// Good:
import (
    urlpkg "net/url"
)

func parse(url string) {
    u, err := urlpkg.Parse(url)
    // ...
}
```

> 注意：本地变量会"遮盖"同名包；只有当变量在作用域内**仍需访问**该包时，才需要重命名。

---

## 四、副作用导入（`import _`）

仅因副作用而导入的包（语法形如 `import _ "package"`）的使用范围有严格限制：

- **只能**在 `main` 包中导入，或在确实需要它们的测试代码中导入。
- **库包中应避免**空白导入，即使该库间接依赖它们也不行。把副作用导入限制在 `main` 包内有助于：
  - 控制依赖关系；
  - 让测试可以使用不同的副作用导入而不会冲突，也不会浪费构建成本。

**典型用途**：

- `time/tzdata`：嵌入时区数据；
- `image/jpeg`：图像处理代码中注册 JPEG 解码器；
- 数据库驱动注册：`_ "github.com/go-sql-driver/mysql"`。

**仅有的两个例外**：

1. 使用空白导入绕过 `nogo` 静态检查工具中对不允许导入项的检查。
2. 在使用 `//go:embed` 编译指令的源文件中，可以直接 `import _ "embed"`。

**强制要求**：

- 副作用导入**必须加注释**说明用途。
- 推荐**单独成组放在最后**，使其副作用语义更显眼。

```go
// Good:
package main

import (
    "database/sql"

    _ "github.com/go-sql-driver/mysql" // register mysql driver
    _ "myproj/rpc/protocols/dial"      // register custom rpc protocols
)
```

> 提示：如果你创建的库包在生产环境中**间接依赖**某个副作用导入，请务必在文档中说明它的预期用法。

---

## 五、点导入（`import .`）

`import .` 是一种语言特性，可以把另一个包导出的标识符**不加限定**地引入当前包。

**约定：不要在生产代码中使用 `import .`**，它会让人难以判断符号来自哪里，严重损害可读性。仅在测试和少数 DSL 场景下被允许。

```go
// Bad:
package foo_test

import (
    "bar/testutil" // also imports "foo"
    . "foo"
)

var myThing = Bar() // Bar 来自 foo 包，但读者无法直观看出
```

```go
// Good:
package foo_test

import (
    "bar/testutil" // also imports "foo"
    "foo"
)

var myThing = foo.Bar()
```

---

## 六、应当避免的写法

- 一行一个 `import`，而不是使用 `import ( ... )` 块。
- 手动插入空行或手写注释分组，打乱工具排序（应交给 `gci` / `goimports` 管理）。
- 在生产代码中使用 dot import（`import . "xxx"`）；测试和 DSL 场景例外。
- 在库包中进行空白导入（`_ "xxx"`），无论是直接还是间接依赖。
- 空白导入不写注释说明用途。
- 使用下划线或大写字母作为重命名后的本地包名（如 `my_pkg`、`MyPkg`）。
- 同一个包在不同文件中使用不同的本地名，破坏一致性。
- 仅出于个人偏好缩短包名（如把 `context` 重命名为 `ctx`）。
- CI 中未通过 `golangci-lint` 的 `formatters` 强制 `gci` / `goimports`。

---

## 七、推荐工具链与 CI 集成

| 工具                                                               | 作用                                                        |
| :----------------------------------------------------------------- | :---------------------------------------------------------- |
| [gofmt](https://pkg.go.dev/cmd/gofmt)                              | 官方格式化工具，仅做基础排序，不主动分组                    |
| [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports)   | 在 `gofmt` 基础上自动增删 import，并支持 `-local` 切第 3 组 |
| [gci](https://github.com/daixiang0/gci)                            | 支持任意自定义分组顺序，是 4 分组方案的事实标准工具         |
| [golangci-lint](https://golangci-lint.run/)                        | 在 CI 中集成上述格式化器，统一团队风格                      |

**`golangci-lint` 配置示例**（v2 配置）：

```yaml
formatters:
  enable:
    - gci
    - gofmt
  settings:
    gci:
      sections:
        - standard
        - default
        - prefix(git.mycompany.com)
        - prefix(github.com/myorg/myproject)
      custom-order: true
```

通过 `golangci-lint fmt` 或在 CI 中执行 `golangci-lint run` 强制执行，从根源上避免 import 顺序与分组的争论。

---

## 参考资料

- [Effective Go](https://go.dev/doc/effective_go)
- [Google Go Style Guide — Decisions](https://google.github.io/styleguide/go/decisions)
- [Google Go Style Guide — Decisions / Imports](https://google.github.io/styleguide/go/decisions#imports)
- [Google Go Style Guide — Best Practices / Protocol Buffer Messages and Stubs](https://google.github.io/styleguide/go/best-practices#protocol-buffer-messages-and-stubs)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports)
- [gci](https://github.com/daixiang0/gci)
- [golangci-lint](https://golangci-lint.run/)
