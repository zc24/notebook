# BEM CSS 命名规范

> BEM（**B**lock-**E**lement-**M**odifier）是由 Yandex 团队提出的 CSS 命名方法论，通过"块 - 元素 - 修饰符"三段式命名，让样式选择器保持扁平、组件边界清晰、类名自带上下文。被 [Element UI](https://element.eleme.cn/) / [Element Plus](https://element-plus.org/) 等主流组件库广泛采用。

## 目录

- [BEM CSS 命名规范](#bem-css-命名规范)
  - [目录](#目录)
  - [一、核心概念](#一核心概念)
  - [二、分隔符规则](#二分隔符规则)
    - [2.1 单中横线 `-`：多单词连接](#21-单中横线--多单词连接)
    - [2.2 双下划线 `__`：表示从属关系](#22-双下划线-__表示从属关系)
    - [2.3 双中横线 `--`：表示修饰 / 变体](#23-双中横线---表示修饰--变体)
  - [三、完整示例](#三完整示例)
  - [四、核心规则](#四核心规则)
  - [五、常见反例](#五常见反例)
  - [六、为什么用 BEM](#六为什么用-bem)
  - [七、工程化落地建议](#七工程化落地建议)
  - [参考资料](#参考资料)

---

## 一、核心概念

BEM 把每个 UI 单元拆解成三个角色：

| 角色 | 含义 | 分隔符 | 示例 |
|------|------|--------|------|
| **Block** | 独立的组件 / 模块，能脱离上下文复用 | 多单词用 `-` 连接 | `metric-card` |
| **Element** | Block 的内部子部分，**不能独立存在** | `__` 双下划线 | `metric-card__header` |
| **Modifier** | Block 或 Element 的状态 / 变体 | `--` 双中横线 | `metric-card--blue` |

> 一句话记忆：**块是骨架，元素是零件，修饰符是状态**。

---

## 二、分隔符规则

### 2.1 单中横线 `-`：多单词连接

用于把多个英文单词连接成一个完整名词，作为 Block 或 Element 的"基础名"。

```css
.metric-card { }
.risk-tag { }
.nav-bar { }
.detail-card { }
```

### 2.2 双下划线 `__`：表示从属关系

表示该元素是某个 Block 的内部组成部分，**不能脱离 Block 单独存在**。

```css
/* metric-card 是 Block，header / title / value / footer 是它的子元素 */
.metric-card__header { }
.metric-card__title { }
.metric-card__value { }
.metric-card__footer { }
```

### 2.3 双中横线 `--`：表示修饰 / 变体

表示该 Block 或 Element 的某种状态、主题色、尺寸等变体，**必须和基础类一起出现**。

```css
/* Block 级别的 Modifier */
.metric-card--blue { }
.metric-card--cyan { }
.metric-card--purple { }
.metric-card--orange { }

/* Element 级别的 Modifier */
.metric-card__value--blue { }
.risk-tag--high { }
.risk-tag--low { }
.nav-tab--active { }
.rank-number--top { }
```

---

## 三、完整示例

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

> Modifier 类**与基础类同时出现**（`rank-item rank-item--highlighted`），而不是替换基础类。这样基础样式与变体样式解耦，便于动态切换状态。

---

## 四、核心规则

1. **Block 名称要有意义**：能描述组件用途，如 `metric-card` / `user-profile`，而不是 `card1` / `box`。
2. **Element 不能脱离 Block**：`__header` 必须出现在某个 Block 内部，不存在游离的 `.header__title`。
3. **不要多层嵌套 Element**：避免 `.block__elem1__elem2`，**最多一层 `__`**——嵌套较深的子元素直接挂在 Block 下即可（写成 `.card__title-icon` 而不是 `.card__title__icon`）。
4. **Modifier 不能单独使用**：`--blue` 必须与 Block / Element 一起出现，且需要**保留基础类**。
5. **选择器保持扁平**：BEM 的核心目标就是避免 `.a .b .c` 这类深层嵌套——一个类名一锤定音。
6. **语义优先于外观**：能写 `--primary` / `--danger` 就别写 `--blue` / `--red`，方便后续换肤与主题切换。

---

## 五、常见反例

| 反例 | 问题 | 正确写法 |
|------|------|---------|
| `.metricCard__header` | Block 用了驼峰命名 | `.metric-card__header` |
| `.metric_card__header` | Block 内部用 `_`，与 `__` 容易混淆 | `.metric-card__header` |
| `.metric-card__body__title` | 多层 `__` 嵌套 | `.metric-card__body-title` 或 `.metric-card__title` |
| `.metric-card .header` | 用了后代选择器，破坏扁平结构 | `.metric-card__header` |
| `<div class="metric-card--blue">` | 只用 Modifier，丢了基础类 | `<div class="metric-card metric-card--blue">` |
| `.--active` / `.__title` | Modifier / Element 脱离 Block 单独使用 | `.tab--active` / `.card__title` |

---

## 六、为什么用 BEM

- **可读性**：看类名就能理解组件结构和层级关系。
- **低耦合**：每个类名自带上下文，不依赖选择器嵌套；DOM 结构调整不会让样式失效。
- **易维护**：类名空间隔离，修改一个组件不会意外影响其他组件。
- **适合组件化**：Block 天然对应 Vue / React 组件的边界，**组件名 ↔ Block 名**一一对应。
- **优先级稳定**：所有选择器都是单类名（特异性 `0,0,1,0`），覆盖时无需 `!important`。

---

## 七、工程化落地建议

1. **配合 SCSS / Less 嵌套语法**：用 `&__elem` / `&--mod` 在源文件中体现层级，编译后仍是扁平的 BEM 类名。

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

2. **Vue / React 单文件组件**：让 **Block 名 = 组件名**（kebab-case），整个 `<style>` 只围绕一个 Block 展开，组件边界 = 样式边界。
3. **stylelint 强制约束**：启用 [stylelint-selector-bem-pattern](https://github.com/postcss/postcss-bem-linter) 等插件，从工具层面卡掉不符合 BEM 的类名。
4. **与 Utility-first（Tailwind）混用**：BEM 适合"组件级语义类"，Tailwind 适合"原子级布局类"，两者并不冲突——业务组件用 BEM 表达结构，间距 / 排版用 Tailwind 微调。
5. **CSS 变量同步命名**：把 token 也按 BEM 风格组织（如 `--metric-card-bg` / `--metric-card-title-color`），让变量结构与类名结构对齐，便于做主题。

---

## 参考资料

- [BEM 官方方法论（英文）](https://en.bem.info/methodology/)
- [Get BEM — 快速入门](https://getbem.com/)
- [CSS Tricks — BEM 101](https://css-tricks.com/bem-101/)
- [Element Plus 设计原则](https://element-plus.org/zh-CN/guide/design.html)
- [postcss-bem-linter](https://github.com/postcss/postcss-bem-linter) ／ [stylelint](https://stylelint.io/)
