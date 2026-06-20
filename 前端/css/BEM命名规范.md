# BEM CSS 命名规范

> 一份可直接落地的 BEM（**B**lock-**E**lement-**M**odifier）CSS 命名规范，整理自 [Yandex BEM 官方方法论](https://en.bem.info/methodology/)、[Get BEM](https://getbem.com/)、[CSS Wizardry — Harry Roberts](https://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/) 等社区主流实践，并结合 [Element Plus](https://element-plus.org/)、[Bootstrap](https://getbootstrap.com/)、[GitHub Primer](https://primer.style/)、[Vue.js 风格指南](https://v2.cn.vuejs.org/v2/style-guide/) 等大型开源项目的落地经验。

## 目录

- [BEM CSS 命名规范](#bem-css-命名规范)
  - [目录](#目录)
  - [一、核心原则](#一核心原则)
  - [二、命名风格选型](#二命名风格选型)
    - [2.1 经典 BEM（Yandex 原版）](#21-经典-bemyandex-原版)
    - [2.2 Two Dashes（CSS Wizardry，主流推荐）](#22-two-dashescss-wizardry主流推荐)
    - [2.3 SUIT CSS（PascalCase 风格）](#23-suit-csspascalcase-风格)
    - [2.4 Namespaced BEM（Element Plus 风格）](#24-namespaced-bemelement-plus-风格)
    - [2.5 选型建议](#25-选型建议)
  - [三、命名格式](#三命名格式)
    - [3.1 通用模板](#31-通用模板)
    - [3.2 分隔符速查表](#32-分隔符速查表)
    - [3.3 命名写法](#33-命名写法)
  - [四、完整示例](#四完整示例)
  - [五、常见场景示例](#五常见场景示例)
  - [六、大厂 / 开源项目对照](#六大厂--开源项目对照)
  - [七、常见反例](#七常见反例)
  - [八、工程化落地建议](#八工程化落地建议)
  - [参考资料](#参考资料)

---

## 一、核心原则

- **块是骨架，元素是零件，修饰符是状态**：每个 UI 单元都拆解成 Block / Element / Modifier 三个角色，分工明确、边界清晰。
- **小写 + kebab-case**：基础名一律小写，多个单词用单中横线 `-` 连接，如 `metric-card`。CSS 类名大小写敏感，但混用大小写会和 HTML / 文件名约定冲突，徒增心智负担。
- **选择器扁平、单一职责**：每个类名一锤定音（特异性 `0,0,1,0`），避免 `.a .b .c` 这类后代选择器嵌套——BEM 的核心目标就是让"选择器复杂度"恒等于 1。
- **Element 不能脱离 Block**：`__header` 必须挂在某个 Block 下，不存在游离的 `.header__title`。
- **Modifier 不能单独使用**：`--blue` 必须与 Block / Element 共存，**保留基础类**，不要"替换"基础类。
- **最多一层 `__` 嵌套**：避免 `.block__elem1__elem2`——DOM 结构上的多层嵌套并不需要在类名上体现，写成 `.card__title-icon` 而不是 `.card__title__icon`。
- **语义优先于外观**：能写 `--primary` / `--danger` 就别写 `--blue` / `--red`，避免主题切换时全量改类名。
- **Block 名 ↔ 组件名一一对应**：在 Vue / React 组件化体系中，`Block` 名直接复用组件名（kebab-case），让"组件边界 = 样式边界"。

---

## 二、命名风格选型

> **⚠️ BEM 不是只有一种写法**：从 Yandex 2009 年的最初版本开始，社区演化出多种"BEM 方言"，分隔符、大小写、嵌套层级各有差异。**先选风格，再谈细节**——团队内必须统一一种，避免半个组件用 `_`、半个组件用 `--`。

### 2.1 经典 BEM（Yandex 原版）

由 Yandex 团队在 2009 年提出，使用**单下划线 `_`** 表示修饰符：

```text
block-name__elem-name_mod-name_mod-val
```

```css
.button { }
.button__icon { }
.button_theme_primary { }       /* key-value 形式的修饰符 */
.button_disabled { }            /* 布尔型修饰符 */
```

> **取舍**：表达力强（支持 `key_value` 形式的修饰符），但分隔符全是下划线、肉眼难以一眼区分 Block / Element / Modifier，工程界已较少采用。除非维护 Yandex 系开源项目（如 [bem-react](https://github.com/bem/bem-react)），新项目不推荐。

### 2.2 Two Dashes（CSS Wizardry，主流推荐）

由 Harry Roberts 在 2013 年的 [MindBEMding](https://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/) 一文中推广，使用**双下划线 `__`** + **双中横线 `--`**，是目前**事实标准**：

```text
block-name__element-name--modifier-name
```

```css
.metric-card { }
.metric-card__header { }
.metric-card--blue { }
.metric-card__value--blue { }
```

> **取舍**：分隔符肉眼可辨（`__` 表"从属"、`--` 表"变体"），与经典 BEM 的语义完全一致，但更易读。**本规范以及绝大多数现代项目（Element Plus、Vue 官方示例、Bootstrap 4+）都采用这一风格**，下文所有示例默认基于 Two Dashes。

### 2.3 SUIT CSS（PascalCase 风格）

由 Twitter / Nicolas Gallagher 提出的 [SUIT CSS](https://suitcss.github.io/) 命名约定，**Block 用 PascalCase**，Element 用 camelCase：

```text
ComponentName-descendentName--modifierName
```

```css
.MetricCard { }
.MetricCard-header { }
.MetricCard--blue { }
.MetricCard.is-active { }       /* 状态类用 is- 前缀 */
```

> **取叶**：与 React 组件名（`<MetricCard />`）保持完全一致，组件即类名；缺点是与传统 CSS / HTML 的 kebab-case 习惯冲突，且与 BEM 主流风格不互通。多用于以 React 为主的小众项目。

### 2.4 Namespaced BEM（Element Plus 风格）

在 Two Dashes 基础上加一个**全局命名空间前缀**，避免在多组件库共存的项目中类名冲突。Element Plus 默认使用 `el-`：

```text
<namespace>-<block>__<element>--<modifier>
```

```css
.el-button { }
.el-button__icon { }
.el-button--primary { }
.el-button--large { }
.el-form-item__label { }
```

> **取舍**：是开源组件库的**事实必选**——没有命名空间，组件库的 `.button` 会和业务方的 `.button` 撞车。业务侧应用一般不需要前缀，**只有开发组件库 / 设计系统时才加 namespace**。

### 2.5 选型建议

| 团队 / 场景 | 推荐风格 | 理由 |
| ------------ | --------- | ------ |
| 普通业务项目（Vue / React） | Two Dashes（[2.2](#22-two-dashescss-wizardry主流推荐)） | 社区共识、工具链最完善、与 SCSS `&__elem` / `&--mod` 语法天然契合 |
| 自研 UI 组件库 / 设计系统 | Namespaced BEM（[2.4](#24-namespaced-bemelement-plus-风格)） | 命名空间隔离，避免和宿主应用冲突 |
| 全 React + CSS Modules | SUIT CSS（[2.3](#23-suit-csspascalcase-风格)）或直接 [*.module.scss](https://github.com/css-modules/css-modules) | 类名作用域已被 CSS Modules 隔离，命名风格让位于"组件名 = 类名" |
| 维护 Yandex 系老项目 | 经典 BEM（[2.1](#21-经典-bemyandex-原版)） | 与既有代码保持一致，避免迁移成本 |

> 本规范第三节起的所有示例**默认采用 Two Dashes 风格**。其它风格只在迁移 / 共存场景下使用，应在仓库 `CONTRIBUTING.md` / `README.md` 中明确声明并通过 stylelint 强制约束。

---

## 三、命名格式

### 3.1 通用模板

```text
<block>[__<element>][--<modifier>]
```

字段说明：

| 字段 | 必填 | 说明 |
| ------ | ------ | ------ |
| `block` | 是 | 独立组件 / 模块名，**kebab-case**，能描述用途，如 `metric-card` |
| `element` | 否 | Block 的内部子部分，前面用 `__` 连接，**不能独立存在** |
| `modifier` | 否 | 状态 / 变体 / 主题，前面用 `--` 连接，**必须与基础类共存** |

最小示例：

```text
metric-card                       /* 仅 Block */
metric-card__header               /* Block + Element */
metric-card--primary              /* Block + Modifier */
metric-card__value--danger        /* Block + Element + Modifier */
```

### 3.2 分隔符速查表

| 分隔符 | 用途 | 示例 |
| ------- | ------ | ------ |
| `-`（单中横线） | 多单词连接，组成 Block / Element / Modifier 的基础名 | `metric-card`、`risk-tag` |
| `__`（双下划线） | 表示"从属"，连接 Block 与 Element | `metric-card__header` |
| `--`（双中横线） | 表示"变体"，连接 Block / Element 与 Modifier | `metric-card--blue`、`risk-tag--high` |
| `_`（单下划线） | **禁用**（仅经典 BEM 使用） | ❌ `metric_card` |
| `.`（后代选择器） | **禁用**用于表达 BEM 层级 | ❌ `.metric-card .header` |

### 3.3 命名写法

- **Block 名要有意义**：能见名知意，如 `metric-card` / `user-profile` / `nav-bar`，而不是 `card1` / `box` / `wrap`。
- **Element 写"是什么"**：`__header` / `__title` / `__footer` 表达"这是 Block 的哪部分"，而不是"它在哪"。
- **Modifier 写"语义"**：优先 `--primary` / `--danger` / `--disabled`，避免 `--blue` / `--red`（除非是设计系统里的中性色板）。
- **布尔型 Modifier 直接用形容词**：`--active` / `--disabled` / `--highlighted`，不用 `--is-active`（`is-` 前缀是 SUIT CSS 的习惯，不要混用）。
- **长度控制**：单个类名建议 **≤ 40 字符**，超长说明 Block 拆得太大或命名太啰嗦。

对比：

| 不规范 | 规范 |
| -------- | ------ |
| `.metricCard__header` | `.metric-card__header` |
| `.metric_card__header` | `.metric-card__header` |
| `.metric-card__body__title` | `.metric-card__body-title` 或 `.metric-card__title` |
| `.metric-card__header-blue` | `.metric-card__header--blue` |
| `.metric-card--is-active` | `.metric-card--active` |
| `.card1` / `.box` | `.metric-card` |

---

## 四、完整示例

以一个"排行榜项"为例，展示 Block / Element / Modifier 三者的协作：

```css
/* Block：排行榜项 */
.rank-item { }

/* Elements：排行榜项的子元素 */
.rank-item__number { }
.rank-item__name { }
.rank-item__progress { }

/* Modifiers：变体 / 状态 */
.rank-item--highlighted { }
.rank-item__number--top { }
```

对应的 HTML 结构：

```html
<li class="rank-item rank-item--highlighted">
  <span class="rank-item__number rank-item__number--top">1</span>
  <span class="rank-item__name">设备 12345</span>
  <div class="rank-item__progress"></div>
</li>
```

> Modifier 类**与基础类同时出现**（`rank-item rank-item--highlighted`），而不是替换基础类。这样基础样式与变体样式解耦，便于 JS 动态切换状态时只 toggle 修饰类。

---

## 五、常见场景示例

卡片类组件：

```text
metric-card                       metric-card--primary
metric-card__header               metric-card__title
metric-card__value                metric-card__value--danger
metric-card__footer
```

表单类组件：

```text
form-item                         form-item--required
form-item__label                  form-item__control
form-item__error                  form-item--invalid
```

按钮 / 标签：

```text
btn                               btn--primary    btn--large
btn__icon                         btn__text
risk-tag                          risk-tag--high  risk-tag--low
```

导航 / 菜单：

```text
nav-bar
nav-bar__item                     nav-bar__item--active
nav-bar__item-icon                nav-bar__item-text
```

列表 / 排行榜：

```text
rank-list
rank-list__item                   rank-list__item--top
rank-list__item-number            rank-list__item-progress
```

---

## 六、大厂 / 开源项目对照

| 项目 / 公司 | 命名风格 | 典型示例 |
| ------------ | --------- | --------- |
| **Yandex（BEM 发源地）** | 经典 BEM（`_` 单下划线） | `button__icon`、`button_theme_primary` |
| **Element Plus / Element UI** | Namespaced + Two Dashes | `el-button--primary`、`el-form-item__label` |
| **Ant Design** | Namespaced + Two Dashes（部分） | `ant-btn`、`ant-btn-primary`（早期未严格 BEM）、`ant-form-item__label` |
| **Bootstrap 4 / 5** | Two Dashes（部分场景） | `card`、`card-header`、`btn-primary`、`btn-outline-danger` |
| **GitHub Primer** | Two Dashes + utility 混用 | `Box`、`Box-header`、`Box--condensed` |
| **SUIT CSS** | PascalCase | `MetricCard-header`、`MetricCard--blue` |
| **Vue.js 官方风格指南** | Two Dashes（推荐 Block = 组件名） | `todo-list`、`todo-list__item` |
| **Inuit CSS / CSS Wizardry** | Two Dashes（首倡者） | `c-card__title--large` |

> **结论**：业界**绝大多数现代项目采用 Two Dashes 风格**（[2.2](#22-two-dashescss-wizardry主流推荐)），开源组件库会再加一层 namespace 前缀（[2.4](#24-namespaced-bemelement-plus-风格)）。新项目直接采用本规范 [3.1](#31-通用模板) 的模板即可。

---

## 七、常见反例

| 反例 | 问题 | 正确写法 |
| ------ | ------ | --------- |
| `.metricCard__header` | Block 用驼峰命名，破坏 kebab-case 一致性 | `.metric-card__header` |
| `.metric_card__header` | Block 内部用 `_`，与 `__` 容易混淆 | `.metric-card__header` |
| `.metric-card__body__title` | 多层 `__` 嵌套 | `.metric-card__body-title` 或 `.metric-card__title` |
| `.metric-card .header` | 用了后代选择器，破坏扁平结构 | `.metric-card__header` |
| `<div class="metric-card--blue">` | 只用 Modifier，丢了基础类 | `<div class="metric-card metric-card--blue">` |
| `.--active` / `.__title` | Modifier / Element 脱离 Block 单独使用 | `.tab--active` / `.card__title` |
| `.card1` / `.box` / `.wrap` | Block 名无语义 | `.metric-card` / `.dialog` / `.toolbar` |
| `.metric-card--is-active` | 混用 SUIT CSS 的 `is-` 前缀 | `.metric-card--active` |
| `.metric-card__header-blue` | 用 `-` 表达 Modifier，语义错位 | `.metric-card__header--blue` |
| `.btn.primary` | 多类名拼接表达变体，无法独立 | `.btn.btn--primary`（HTML 同时挂两个类） |

---

## 八、工程化落地建议

1. **配合 SCSS / Less 嵌套语法**：用 `&__elem` / `&--mod` 在源文件中体现层级，编译后仍是扁平的 BEM 类名，源码与产物各自最优。

   ```scss
   .metric-card {
     &__header { /* .metric-card__header */ }
     &__title  { /* .metric-card__title  */ }
     &--blue   { /* .metric-card--blue   */ }

     &__value {
       &--blue { /* .metric-card__value--blue */ }
     }
   }
   ```

2. **Vue / React 单文件组件**：让 **Block 名 = 组件名**（kebab-case），整个 `<style>` 只围绕一个 Block 展开，做到"组件边界 = 样式边界"。

   ```vue
   <!-- MetricCard.vue -->
   <template>
     <div class="metric-card" :class="{ 'metric-card--primary': primary }">
       <div class="metric-card__header">…</div>
       <div class="metric-card__value">…</div>
     </div>
   </template>
   ```

3. **stylelint 强制约束**：使用 [stylelint](https://stylelint.io/) + [postcss-bem-linter](https://github.com/postcss/postcss-bem-linter) 或正则规则，从工具层面卡掉不符合 BEM 的类名。最小配置示例：

   ```json
   {
     "plugins": ["stylelint-selector-bem-pattern"],
     "rules": {
       "plugin/selector-bem-pattern": {
         "preset": "bem",
         "componentName": "[a-z]+(-[a-z]+)*",
         "componentSelectors": {
           "initial": "^\\.{componentName}(?:__[a-z]+(?:-[a-z]+)*)?(?:--[a-z]+(?:-[a-z]+)*)?$"
         }
       }
     }
   }
   ```

4. **CSS 变量与 BEM 对齐命名**：把设计 token 也按 BEM 风格组织，让变量结构与类名结构对齐，便于做主题与暗黑模式切换。

   ```css
   :root {
     --metric-card-bg: #fff;
     --metric-card-title-color: #1f2937;
     --metric-card--primary-bg: #2563eb;
   }
   ```

5. **CI 中校验类名**：在 `pre-commit` / GitHub Actions 中运行 `stylelint`，PR 级强制；建议同时引入 [stylelint-config-standard-scss](https://github.com/stylelint-scss/stylelint-config-standard-scss) 作为基线。

6. **组件库加 namespace**：发布到 npm 的 UI 组件库务必加 namespace 前缀（如 `el-` / `ant-` / `<your-org>-`），避免业务方应用样式冲突；业务项目则**不必**加前缀。

7. **CONTRIBUTING / README 中明确声明风格**：在仓库根目录的 `CONTRIBUTING.md` 或前端目录的 `README.md` 中写明本项目采用的 BEM 风格（默认 [Two Dashes](#22-two-dashescss-wizardry主流推荐)），新成员入职第一份必读文档。

---

## 参考资料

- [BEM 官方方法论 — Yandex](https://en.bem.info/methodology/)
- [Get BEM — 快速入门](https://getbem.com/)
- [MindBEMding — getting your head round BEM syntax — Harry Roberts / CSS Wizardry](https://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/)
- [CSS Tricks — BEM 101](https://css-tricks.com/bem-101/)
- [SUIT CSS — Style Tools for Component-based UI Development](https://suitcss.github.io/)
- [Element Plus 设计原则](https://element-plus.org/zh-CN/guide/design.html)
- [Vue.js 风格指南 — 单文件组件命名](https://v2.cn.vuejs.org/v2/style-guide/)
- [postcss-bem-linter](https://github.com/postcss/postcss-bem-linter) ／ [stylelint](https://stylelint.io/)
- [Battling BEM — 5 Common Problems and How to Avoid Them](https://www.smashingmagazine.com/2016/06/battling-bem-extended-edition-common-problems-and-how-to-avoid-them/)
