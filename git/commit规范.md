# Git 提交信息规范

> 一份可直接落地的 Git Commit Message 规范，整理自 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/)、[Angular Commit Message Format](https://github.com/angular/angular/blob/main/contributing-docs/commit-message-guidelines.md)、[Vue](https://github.com/vuejs/core/blob/main/.github/commit-convention.md) / [Element Plus](https://github.com/element-plus/element-plus/blob/dev/.github/commit-convention.md) 等大型开源项目的实践，并结合团队工程化（commitlint、自动 changelog、SemVer 自动版本）的需求做了取舍。

## 目录

- [Git 提交信息规范](#git-提交信息规范)
  - [目录](#目录)
  - [一、核心原则](#一核心原则)
  - [二、提交信息格式](#二提交信息格式)
  - [三、type 速查表](#三type-速查表)
    - [3.1 核心 type（必备）](#31-核心-type必备)
    - [3.2 扩展 type（按需采用）](#32-扩展-type按需采用)
  - [四、scope 约定](#四scope-约定)
  - [五、subject 写法](#五subject-写法)
  - [六、body 与 footer](#六body-与-footer)
  - [七、破坏性变更（Breaking Change）](#七破坏性变更breaking-change)
  - [八、回滚提交（Revert）](#八回滚提交revert)
  - [九、清理 / 废弃代码场景](#九清理--废弃代码场景)
  - [十、与 SemVer / Changelog 的关系](#十与-semver--changelog-的关系)
  - [十一、常见反例](#十一常见反例)
  - [十二、大厂规范对照](#十二大厂规范对照)
    - [12.1 Angular 为什么移除了 `chore` / `style`](#121-angular-为什么移除了-chore--style)
  - [十三、工程化落地建议](#十三工程化落地建议)
  - [参考资料](#参考资料)

---

## 一、核心原则

- **一次提交只做一件事**：一个 commit 只表达一个意图，便于 review、回滚和 cherry-pick。
- **机器可读 + 人类可读**：格式可被工具解析（生成 changelog、推断 SemVer），同时也能让 reviewer 一眼看懂"做了什么 / 为什么做"。
- **写"为什么"，而不是"做了什么"**：diff 本身已经能说明"做了什么"，commit message 的价值在于解释"为什么这么做"。
- **统一约定优于个人偏好**：团队内全员遵守同一份规范，由 commitlint 强制校验，避免风格漂移。

---

## 二、提交信息格式

采用 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/) 标准格式：

```text
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

字段说明：

| 字段 | 必填 | 说明 |
|------|------|------|
| `type` | 是 | 本次提交的类别，见下方[速查表](#三type-速查表) |
| `scope` | 否 | 本次提交影响的模块 / 范围，小写名词，使用 `()` 包裹 |
| `subject` | 是 | 一句话概述本次改动，**≤ 50 字符**，祈使句，首字母小写，结尾不加句号 |
| `body` | 否 | 详细描述本次改动的动机、思路、影响；每行 **≤ 72 字符** |
| `footer` | 否 | 关联 issue / 破坏性变更声明 / 共同作者等元信息 |

最小示例：

```text
feat(auth): support OAuth2 login via Google
```

完整示例：

```text
fix(order): correct total amount when coupon is expired

Previously the discount was still applied even when the coupon
had expired, resulting in an incorrect order total. Validate the
coupon's expiration time before applying the discount.

Closes #1234
Refs #1200
```

---

## 三、type 速查表

### 3.1 核心 type（必备）

| type | 中文含义 | 使用场景 | SemVer 影响 |
|------|---------|---------|------------|
| `feat` | 新功能 | 新增一个对用户可见的功能 | minor（1.**X**.0） |
| `fix` | Bug 修复 | 修复一个 bug | patch（1.0.**X**） |
| `docs` | 文档 | 只改 README、注释、API 文档等，**不动代码逻辑** | 不影响 |
| `style` | 代码格式 | 空格、缩进、分号、格式化、删空行等，**不改变代码含义** | 不影响 |
| `refactor` | 重构 | 既不是新功能也不是修 bug 的代码改动（改写实现、提取函数、改命名） | 不影响 |
| `perf` | 性能优化 | 提升性能的改动（算法、SQL、缓存、减少内存分配等） | patch |
| `test` | 测试 | 新增 / 修改测试代码（单测、集成测试、e2e） | 不影响 |
| `build` | 构建 | 影响构建系统或外部依赖的改动（`go.mod`、`package.json`、`Dockerfile`、`Makefile`、webpack 等） | 不影响 |
| `ci` | CI 配置 | CI 配置文件和脚本的改动（GitHub Actions、Jenkins、GitLab CI、Drone 等） | 不影响 |
| `chore` | 杂项 | 不属于以上任何一种的日常维护改动（升级依赖、改 `.gitignore`、改编辑器配置等） | 不影响 |
| `revert` | 回滚 | 回滚某个之前的 commit，正文需带 `Reverts: <hash>` | 视被回滚提交而定 |

> 这 11 个 type 已经能覆盖 95% 的日常场景，新人也容易理解，**建议团队默认开启**。

### 3.2 扩展 type（按需采用）

| type | 使用场景 | 典型出处 |
|------|---------|---------|
| `improvement` / `enhance` | 对已有功能的小幅增强（介于 feat 和 refactor 之间） | 部分内部规范 |
| `hotfix` | 紧急线上修复（一般通过 hotfix 分支走特殊发布流程） | GitFlow |
| `release` | 发布提交，常由工具自动生成 | Angular / release-please |
| `deps` | 仅升级依赖（Dependabot / Renovate 默认前缀） | GitHub Dependabot |
| `security` | 安全漏洞修复 | GitHub Security Advisories |
| `i18n` | 国际化、翻译相关 | Vue / Element Plus |
| `a11y` | 无障碍（accessibility）相关 | 前端项目常见 |
| `ui` / `ux` | 纯 UI / 交互调整 | 部分前端团队 |
| `types` | 仅类型定义改动（TS 项目常见） | TypeScript 生态 |
| `workflow` | 工作流脚本、Git hooks 等 | Vue / Element Plus |
| `dx` | 开发者体验（developer experience）相关改动 | Vue |
| `example` / `examples` | 示例代码改动 | Vue / Element Plus |
| `wip` | Work in progress，未完成代码（一般禁止合入主干，仅用于个人分支） | 内部规范 |

> 扩展 type 不必全部启用，**少而精**比"大而全"更重要。

---

## 四、scope 约定

`scope` 用一对小括号包裹，紧贴 `type`，**中间不留空格**：

```text
feat(user): ...
fix(order/coupon): ...
```

约定：

- **使用小写、英文名词**，多个词用 `-` 或 `/` 连接（`feat(user-profile)` / `fix(order/coupon)`）。
- **使用枚举集合**：scope 应来自一个事先约定好的有限列表（例如 `auth` / `user` / `order` / `payment` / `db` / `infra`），避免每人写法不同。
- **scope ≠ 文件路径**：写"业务模块"而不是"目录名"，长度控制在 1～2 个词。
- **跨多个 scope 的改动**：要么去掉 scope，要么拆成多个 commit；不要写成 `feat(user,order,payment)` 这种长串。

---

## 五、subject 写法

- 使用**祈使句**：`add` / `fix` / `remove`，而不是 `added` / `fixes` / `removing`。
- **首字母小写**，结尾**不加句号**。
- 长度 **≤ 50 字符**，必须能在一行内说清"做了什么"。
- 写"做了什么"，不写"如何做"——细节留给 body。

对比：

| 不规范 | 规范 |
|--------|------|
| `FEAT:add user login` | `feat: add user login` |
| `feat:added user login.` | `feat: add user login` |
| `fix: I fixed the bug of user login when token expired and refactored the code` | `fix(auth): refresh token before expiration` |
| `update code` | `refactor(order): extract coupon validation into separate function` |

---

## 六、body 与 footer

**body**（可选）：

- 与 subject 之间空一行。
- 每行 **≤ 72 字符**，便于 `git log` 在终端中阅读。
- 重点说明**动机（why）** 和**影响（impact）**，而不是逐行解释 diff。
- 可以使用列表（`-` / `*`）罗列多点说明。

**footer**（可选）：

- 与 body 之间空一行。
- 常见用途：
  - 关联 issue：`Closes #1234` / `Fixes #1234` / `Refs #1234`
  - 破坏性变更：`BREAKING CHANGE: ...`（详见下一节）
  - 共同作者：`Co-authored-by: Name <email>`
  - 回滚关联：`Reverts: <commit-hash>`

完整示例：

```text
perf(search): cache normalized query string to reduce CPU usage

The normalization step (lowercasing, trimming, removing punctuation)
was being executed on every keystroke, which became a bottleneck
under heavy autocomplete load. Cache the normalized result keyed by
the raw query string; the cache is bounded to 1024 entries via LRU.

Closes #4567
Co-authored-by: Alice <alice@example.com>
```

---

## 七、破坏性变更（Breaking Change）

破坏性变更（不向后兼容的修改）**必须显式标注**，否则会让下游悄无声息地坏掉。两种等价的写法：

**方式 1：`type` 后加 `!`（推荐，最显眼）**

```text
feat(api)!: change response shape of /users

`data.user` is renamed to `data.profile`.
Clients must migrate before v3.0.0.
```

**方式 2：footer 中写 `BREAKING CHANGE:`**

```text
feat(api): change response shape of /users

BREAKING CHANGE: `data.user` is renamed to `data.profile`.
Clients must migrate before v3.0.0.
```

> 任意一种写法都会被工具识别为破坏性变更，自动触发 **major 版本号**升级（**X**.0.0）。

---

## 八、回滚提交（Revert）

直接使用 `git revert <hash>` 时，git 会自动生成形如 `Revert "xxx"` 的 message。推荐改写为：

```text
revert: feat(auth): support OAuth2 login via Google

This reverts commit 1a2b3c4d5e6f7g8h.

OAuth2 callback fails under proxy mode; revert until the proxy
handling is fixed in #1357.
```

要点：

- subject 用 `revert:` 前缀 + 被回滚提交的原 subject。
- body 必须包含 `This reverts commit <full-hash>.` 一行（git 默认行为）。
- 补充**为什么要回滚**，便于后续 review。

---

## 九、清理 / 废弃代码场景

"清理废弃代码"在团队里经常引起 type 选择上的争议。核心判断只有一条：

> **删完之后，下游使用者（外部调用方）的代码会不会坏？**
>
> - **不会坏** → `refactor`（纯重构，不影响外部行为）
> - **会坏** → 必须用 `feat!` / `refactor!` + `BREAKING CHANGE:` footer

### 9.1 决策树

```text
要删除的是什么？
│
├─ 内部代码（没有人从外部 import / 调用）
│  ├─ 删除未引用的函数、变量、import、注释掉的代码  → refactor
│  ├─ 删除已经无用的测试                            → test
│  ├─ 删除已经无用的依赖（go.mod / package.json）   → build（或 chore(deps)）
│  └─ 删除已经无用的文档段落                        → docs
│
└─ 对外暴露的 API（导出函数、类型、HTTP 接口、CLI 参数、配置项等）
   ├─ 之前已标 Deprecated，现在到点移除   → refactor! / feat! + BREAKING CHANGE
   ├─ 直接移除一个仍在使用的功能 / 接口   → feat! + BREAKING CHANGE
   └─ 仅"标记"为 deprecated、还没真正删   → refactor / feat + DEPRECATED footer
```

### 9.2 典型示例

**删除内部未引用的工具函数**：

```text
refactor(user): remove unused helper functions

`normalizeUsername` and `legacyHashPassword` are no longer
referenced anywhere after the auth module rewrite in #1200.
```

**删除已注释掉的死代码**：

```text
refactor: remove commented-out legacy migration code
```

**删除一个公开 API（破坏性）**：

```text
feat(api)!: remove deprecated /v1/users endpoint

The endpoint was marked as deprecated in v2.3.0 (released
2025-04). All clients should migrate to /v2/users.

BREAKING CHANGE: `/v1/users` is removed. Use `/v2/users`
instead. The response shape is identical except for the
`id` field type (number → string).

Closes #4567
```

**仅"标记"deprecated、暂不删除**（使用 Angular 官方的 `DEPRECATED:` footer）：

```text
refactor(api): deprecate /v1/users endpoint

DEPRECATED: `/v1/users` is deprecated and will be removed
in v3.0.0. Use `/v2/users` instead.
```

**删除已废弃的依赖**：

```text
build(deps): remove unused lodash dependency
```

### 9.3 常见判断误区

| 场景 | 容易误用 | 正确用法 | 原因 |
|------|---------|---------|------|
| 删一个内部 unexported 函数 | `chore` | `refactor` | 是代码改动，且既不加功能也不修 bug，标准定义就是 `refactor` |
| 删一个已被弃用的公开 API | `refactor` | `refactor!` / `feat!` + `BREAKING CHANGE` | 即使是"清理"，对调用方也是破坏性变更，必须显式标注 |
| 删一段注释掉的代码 | `style` | `refactor` | `style` 只用于格式化（空格、缩进），删代码属于结构性改动 |
| 删一个不再使用的 Go 文件 / 整个包 | `chore` | `refactor` | 同上，是代码结构改动 |
| 删 `go.mod` 里不再需要的依赖 | `refactor` | `build` | 影响构建系统 / 外部依赖，归 `build` |

### 9.4 推荐的"两阶段废弃"流程

对于**公开 API / SDK / 框架**，推荐使用 Angular 官方的 `DEPRECATED:` + `BREAKING CHANGE:` 两阶段流程，避免一刀切删除导致下游崩溃：

| 阶段 | commit 形式 | 版本号影响 | 说明 |
|------|------------|-----------|------|
| **阶段一：标记弃用** | `refactor(api): deprecate xxx` + `DEPRECATED:` footer | minor / patch | 旧 API 保留，但运行时打 warning 或 IDE 标注；给下游迁移时间 |
| **阶段二：彻底移除** | `refactor(api)!: remove xxx` + `BREAKING CHANGE:` footer | **major** | 真正删代码，触发主版本号升级 |

两阶段之间至少跨一个 minor 版本，给下游充分的迁移窗口；内部业务代码（无外部消费者）则不需要这套流程，直接 `refactor` 删掉即可。

---

## 十、与 SemVer / Changelog 的关系

Conventional Commits 的最大价值，是可以**根据 commit type 自动推断版本号、自动生成 CHANGELOG**：

| commit 类型 | 触发版本号 | 示例（当前 1.2.3） |
|------------|-----------|-------------------|
| 含 `BREAKING CHANGE` 或 `!` | **major** | 1.2.3 → 2.0.0 |
| `feat` | **minor** | 1.2.3 → 1.3.0 |
| `fix` / `perf` | **patch** | 1.2.3 → 1.2.4 |
| `docs` / `style` / `chore` / `ci` / `build` / `test` / `refactor` | 不影响 | 1.2.3 → 1.2.3 |

配合 [standard-version](https://github.com/conventional-changelog/standard-version) / [release-please](https://github.com/googleapis/release-please) / [semantic-release](https://semantic-release.gitbook.io/) 等工具，可以在每次发布时自动：

1. 根据所有 commit 推断新版本号；
2. 生成 / 更新 `CHANGELOG.md`；
3. 打 git tag；
4. （可选）发布到 npm / GitHub Release。

---

## 十一、常见反例

| 反例 | 问题 | 正确写法 |
|------|------|---------|
| `update` / `修复 bug` / `wip` | 没有任何信息量 | `fix(auth): handle nil token in middleware` |
| `FEAT:add login` | type 大写、冒号后缺空格 | `feat: add login` |
| `feat: Added login.` | 用了过去式 + 首字母大写 + 句号 | `feat: add login` |
| `feat(user,order,payment): big refactor` | 一次提交跨多个 scope，难以 review / 回滚 | 拆成多个 commit |
| `fix: 改了一下 user.go 第 23 行的判断` | 描述实现细节而非意图 | `fix(user): reject empty username on register` |
| `feat: 重构了订单模块，顺便修了 3 个 bug，并升级了依赖` | 一次提交做了多件事 | 拆成 `refactor` + `fix` × 3 + `chore(deps)` |
| `Merge branch 'main' into feature/x`（手写） | 无意义的合并提交污染历史 | 使用 rebase 或保留 git 自动生成的合并提交即可 |

---

## 十二、大厂规范对照

| 项目 / 公司 | type 体系 | 备注 |
|------------|----------|------|
| **Angular（最新版）** | `build` / `ci` / `docs` / `feat` / `fix` / `perf` / `refactor` / `test` | Conventional Commits 的源头；早期含 `chore` / `style`，**新版已移除**，详见 [12.1](#121-angular-为什么移除了-chore--style) |
| **`@commitlint/config-conventional`** | `build` / `chore` / `ci` / `docs` / `feat` / `fix` / `perf` / `refactor` / `revert` / `style` / `test` | 社区使用最广的 commitlint 默认配置，**仍保留 `chore` / `style`**，绝大多数社区项目沿用这一套 |
| **Vue / Element Plus** | `feat` / `fix` / `docs` / `dx` / `style` / `refactor` / `perf` / `test` / `workflow` / `build` / `ci` / `chore` / `release` / `types` / `wip` | 在 Angular 基础上扩展，前端生态最常见的一套 |
| **Google（Eng-Practices / AOSP）** | 不强制 type | 要求 subject ≤ 50 字符；body 写 What / Why；通过 Gerrit code review 流程把关，更看重 issue 关联与 reviewer |
| **Linux 内核** | **子系统前缀**（如 `mm:` `net: tcp:` `drivers/usb:` `Documentation:`） | type 概念弱化，重点是 `Signed-off-by` 与 patch 描述 |
| **Kubernetes / Istio 等 CNCF 项目** | 不强制 type | 改由 PR 标签管控（`/kind feature` `/kind bug` `/kind cleanup` `/kind documentation`），效果类似 |
| **GitHub Dependabot** | 默认使用 `deps` / `chore(deps)` | 自动化依赖升级 PR |

> **结论**：业界主流仍以 **Conventional Commits（Angular 风格）** 为事实标准，新项目直接采用即可，无需自己造一套。

### 12.1 Angular 为什么移除了 `chore` / `style`

Angular 官方仓库（[contributing-docs/commit-message-guidelines.md](https://github.com/angular/angular/blob/main/contributing-docs/commit-message-guidelines.md)）最新版的 type 只保留这 8 个：

```text
build | ci | docs | feat | fix | perf | refactor | test
```

需要注意的是，`chore` 和 `style` **并不是被某一个新 type 替代**，而是按"实际在做什么"拆到了其他已有 type 里。

**`chore` 的去向**：

| 原本归 `chore` 的场景 | 现在归到 |
|---------------------|---------|
| 升级 npm 依赖、改 `package.json` / Bazel / gulp 等构建脚本 | **`build`** |
| 改 GitHub Actions / CI 配置文件、CI 脚本 | **`ci`** |
| 单纯的代码清理、改命名、删死代码、调整目录结构 | **`refactor`** |
| 调整文档站点 / README | **`docs`** |
| 加 / 改测试 | **`test`** |

> 官方表格里 `build` 的描述明确写了："Changes that affect the build system or **external dependencies** (example scopes: gulp, broccoli, **npm**)"——`npm` 依赖升级原本是 `chore(deps)` 的典型场景，Angular 现在直接归到 `build`。

**`style` 的去向**：

- 现代项目都接入了 Prettier / clang-format / ESLint / gofmt 等**自动格式化工具**，commit 阶段就不应该再出现"纯格式化"的提交。
- 真要大规模重新格式化，归到 `refactor`；如果是改了 formatter 配置导致的格式化，归到 `build`。

**那为什么社区项目还在用 `chore` / `style`？**

因为 [@commitlint/config-conventional](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional)（commitlint 的默认配置）**依然保留**了这两个 type，再加上历史包袱和"杂项出口"的实用价值，绝大多数社区项目就一直沿用了。

**给团队的选型建议**：

- **新建项目**：直接用 Angular 最新版的 8 个 type 即可，分类更干净，意图更明确。
- **沿用 `chore`**：保留 `chore` 也没问题，但要在团队内部明确——什么样的改动归 `chore`、什么归 `build`，避免随手乱用导致 changelog 出现噪音。

---

## 十三、工程化落地建议

1. **强制校验**：上 [commitlint](https://commitlint.js.org/) + [Husky](https://typicode.github.io/husky/) / [lefthook](https://github.com/evilmartians/lefthook)，配合 `@commitlint/config-conventional`，在 `commit-msg` 钩子阶段卡掉不规范提交。
2. **辅助输入**：本地可用 [commitizen](https://github.com/commitizen/cz-cli)（`git cz`）交互式生成符合规范的提交信息，降低记忆负担。
3. **自动 changelog & 版本号**：发布流程接入 [standard-version](https://github.com/conventional-changelog/standard-version) / [release-please](https://github.com/googleapis/release-please) / [semantic-release](https://semantic-release.gitbook.io/)，让 commit 直接驱动版本号与 CHANGELOG。
4. **scope 用枚举**：在 `commitlint.config.js` 中通过 `scope-enum` 规则把 scope 限定为约定好的列表，避免随意发挥。
5. **PR 标题同样校验**：使用 [action-semantic-pull-request](https://github.com/amannn/action-semantic-pull-request) 等 GitHub Action，确保 PR 标题也符合规范——squash merge 时 PR 标题会成为最终的 commit message。
6. **保护主干历史**：主干分支使用 **Squash merge** 或 **Rebase merge**，让每个合入主干的 PR 对应一条干净的 commit；禁用 Merge commit 模式，避免无意义的合并提交污染历史。
7. **Go 项目额外约定**：
   - `build` 用于 `go.mod` / `go.sum` / `Dockerfile` / `Makefile` 改动；
   - `chore` 用于工具脚本、`.gitignore`、`.golangci.yml` 等；
   - `refactor` 不允许改变外部行为，接口签名变动应归 `feat!` 或 `fix!`。

---

## 参考资料

- [Conventional Commits（中文）](https://www.conventionalcommits.org/zh-hans/)
- [Angular Commit Message Guidelines](https://github.com/angular/angular/blob/main/contributing-docs/commit-message-guidelines.md)
- [Vue Commit Convention](https://github.com/vuejs/core/blob/main/.github/commit-convention.md)
- [Element Plus Commit Convention](https://github.com/element-plus/element-plus/blob/dev/.github/commit-convention.md)
- [Semantic Versioning 2.0.0（中文）](https://semver.org/lang/zh-CN/)
- [How to Write a Git Commit Message — Chris Beams](https://chris.beams.io/posts/git-commit/)
- [commitlint](https://commitlint.js.org/)
- [standard-version](https://github.com/conventional-changelog/standard-version) ／ [release-please](https://github.com/googleapis/release-please) ／ [semantic-release](https://semantic-release.gitbook.io/)
- [Google Engineering Practices — CL Descriptions](https://google.github.io/eng-practices/review/developer/cl-descriptions.html)
