# 现代 CSS 方案：CSS Modules、CSS-in-JS、Tailwind

> 一份覆盖三种主流"组件化 CSS 方案"的对比笔记：**CSS Modules**（局部作用域）、**CSS-in-JS**（运行时 / 零运行时样式生成）、**Tailwind CSS**（原子化 utility-first）。整理自 [CSS Modules 规范](https://github.com/css-modules/css-modules)、[styled-components 文档](https://styled-components.com/)、[Emotion 文档](https://emotion.sh/)、[vanilla-extract 文档](https://vanilla-extract.style/)、[Tailwind CSS v4 文档](https://tailwindcss.com/)、[Next.js App Router 样式指南](https://nextjs.org/docs/app/building-your-application/styling)，结合 GitHub、Vercel、Shopify、Stripe、Linear 等公司的公开实践。
>
> **与 [`BEM命名规范.md`](./BEM命名规范.md) 的关系**：BEM 解决"全局 CSS 怎么命名才不冲突"，本笔记的三种方案则从工程层面绕开了"全局命名"这一前提——要么作用域隔离、要么样式与组件共置、要么彻底放弃语义类名。**先理解 BEM 解决的问题，再看本笔记的方案就一目了然**。

## 目录

- [现代 CSS 方案：CSS Modules、CSS-in-JS、Tailwind](#现代-css-方案css-modulescss-in-jstailwind)
  - [目录](#目录)
  - [一、总览与选型](#一总览与选型)
  - [二、CSS Modules](#二css-modules)
    - [2.1 核心思想](#21-核心思想)
    - [2.2 基础用法](#22-基础用法)
    - [2.3 进阶语法](#23-进阶语法)
    - [2.4 与 BEM / Sass 配合](#24-与-bem--sass-配合)
    - [2.5 优缺点](#25-优缺点)
    - [2.6 工程化建议](#26-工程化建议)
  - [三、CSS-in-JS](#三css-in-js)
    - [3.1 核心思想与流派](#31-核心思想与流派)
    - [3.2 styled-components / Emotion（运行时）](#32-styled-components--emotion运行时)
    - [3.3 vanilla-extract / Linaria / Panda（零运行时）](#33-vanilla-extract--linaria--panda零运行时)
    - [3.4 主题与设计 Token](#34-主题与设计-token)
    - [3.5 RSC / SSR 兼容性陷阱](#35-rsc--ssr-兼容性陷阱)
    - [3.6 优缺点](#36-优缺点)
  - [四、Tailwind CSS](#四tailwind-css)
    - [4.1 核心思想：Utility-First](#41-核心思想utility-first)
    - [4.2 基础用法](#42-基础用法)
    - [4.3 设计 Token 与主题](#43-设计-token-与主题)
    - [4.4 组件抽象：`@apply` vs `cva` vs 组件封装](#44-组件抽象apply-vs-cva-vs-组件封装)
      - [方式 1：`@apply`（CSS 层抽象）](#方式-1applycss-层抽象)
      - [方式 2：组件封装（推荐）](#方式-2组件封装推荐)
      - [方式 3：CVA（class-variance-authority，shadcn/ui 同款）](#方式-3cvaclass-variance-authorityshadcnui-同款)
    - [4.5 Tailwind v4 的变化](#45-tailwind-v4-的变化)
    - [4.6 优缺点](#46-优缺点)
    - [4.7 工程化建议](#47-工程化建议)
  - [五、四方案横向对比](#五四方案横向对比)
  - [六、大厂 / 开源项目对照](#六大厂--开源项目对照)
  - [七、组合实践](#七组合实践)
    - [7.1 Tailwind + CSS Modules（推荐）](#71-tailwind--css-modules推荐)
    - [7.2 Tailwind + CSS Variables（主题系统）](#72-tailwind--css-variables主题系统)
    - [7.3 CSS Modules + BEM（遗留项目维护）](#73-css-modules--bem遗留项目维护)
    - [7.4 组件库用 vanilla-extract，应用层用 Tailwind](#74-组件库用-vanilla-extract应用层用-tailwind)
  - [八、迁移路径建议](#八迁移路径建议)
    - [8.1 从 BEM + Sass 迁移](#81-从-bem--sass-迁移)
    - [8.2 从运行时 CSS-in-JS（styled-components / Emotion）迁移](#82-从运行时-css-in-jsstyled-components--emotion迁移)
    - [8.3 新项目零成本起步](#83-新项目零成本起步)
  - [参考资料](#参考资料)
    - [CSS Modules](#css-modules)
    - [CSS-in-JS](#css-in-js)
    - [Tailwind](#tailwind)
    - [综合 / 对比](#综合--对比)

---

## 一、总览与选型

| 方案 | 诞生时间 | 核心机制 | 一句话定位 |
| ---- | ------- | -------- | --------- |
| **BEM**（参照） | 2009 | 命名约定 | 用类名规范防止全局冲突 |
| **CSS Modules** | 2015 | 构建时类名哈希 | 让 CSS 文件自动局部作用域 |
| **CSS-in-JS** | 2014（JSS）／ 2016（styled-components） | JS 运行时 / 构建时生成 CSS | 把样式当成 JS 模块，和组件一起组合 |
| **Tailwind CSS** | 2017 | 原子化 utility class | 用预设工具类直接在 HTML 里写样式 |

选型速查：

| 场景 | 推荐方案 | 备选 |
| ---- | -------- | ----- |
| 中后台 / 业务项目（Vue / React 主流栈） | **Tailwind** 或 **CSS Modules + Sass** | CSS-in-JS |
| Next.js App Router（Server Components） | **Tailwind** / **CSS Modules** / **vanilla-extract** | ❌ 运行时 CSS-in-JS（styled-components / Emotion 与 RSC 不兼容） |
| 设计系统 / 组件库 | **vanilla-extract** / **Tailwind + cva** | CSS Modules + BEM |
| 高度动态主题（白标、用户自定义皮肤） | **CSS-in-JS** 或 **CSS Variables + Tailwind** | — |
| 纯静态页 / 营销页 | **Tailwind** | 原生 CSS |
| 维护遗留项目（jQuery / 多页应用） | **BEM + Sass** | CSS Modules |

> 没有银弹。**判断标准是"团队 + 框架 + 部署形态"，不是"哪个最新"**。下文展开每个方案的核心机制、优缺点和落地姿势。

---

## 二、CSS Modules

### 2.1 核心思想

[CSS Modules](https://github.com/css-modules/css-modules) 是由 Glen Maddern 在 2015 年提出的规范，**不是新语法，而是构建时的转换约定**：每个 `*.module.css` / `*.module.scss` 文件中的类名在编译时会被加上**唯一哈希后缀**，自动隔离作用域。

```css
/* Card.module.css */
.card { padding: 16px; }
.title { font-weight: 600; }
```

编译产物示意：

```css
.Card_card__1aB2c { padding: 16px; }
.Card_title__3xY9z { font-weight: 600; }
```

- 类名按 `[文件名]_[类名]__[hash]` 模式生成，**保证全局唯一**。
- 组件代码通过 import 拿到映射对象：`styles.card` → `"Card_card__1aB2c"`。

> 一句话：**让"全局 CSS"在你眼中是局部的，在浏览器眼中是全局的**。

### 2.2 基础用法

React：

```tsx
import styles from './Card.module.css';

export function Card({ title }: { title: string }) {
  return (
    <div className={styles.card}>
      <h2 className={styles.title}>{title}</h2>
    </div>
  );
}
```

Vue 3：

```vue
<template>
  <div :class="$style.card">
    <h2 :class="$style.title">{{ title }}</h2>
  </div>
</template>

<style module>
.card { padding: 16px; }
.title { font-weight: 600; }
</style>
```

### 2.3 进阶语法

**组合（`composes`）**——CSS Modules 独有的复用机制，类似继承：

```css
/* Button.module.css */
.base { padding: 8px 16px; border-radius: 4px; }
.primary {
  composes: base;
  background: #2563eb;
  color: #fff;
}
```

`composes` 在编译后变成"同时挂多个类名"，比 Sass 的 `@extend` 更安全（不会因为选择器顺序导致优先级翻车）。

**全局逃逸（`:global`）**——偶尔需要保留全局类名（如第三方库的 hook 类）：

```css
.menu :global(.ant-dropdown-menu-item) {
  padding: 4px 8px;
}

:global(.no-scroll) {
  overflow: hidden;
}
```

**值导出（`:export`）**——把 CSS 变量同步到 JS：

```css
:export {
  primaryColor: #2563eb;
  breakpointMd: 768px;
}
```

```ts
import vars from './tokens.module.css';
console.log(vars.primaryColor); // "#2563eb"
```

### 2.4 与 BEM / Sass 配合

CSS Modules **并不取代 BEM**——在 `.module.scss` 内部，你仍然可以用 BEM 写法组织复杂组件，外部看到的只是哈希后的类名：

```scss
// MetricCard.module.scss
.metricCard {
  &__header { font-weight: 600; }
  &__value { font-size: 24px; }

  &--primary { background: #eff6ff; }
}
```

```tsx
<div className={`${styles.metricCard} ${styles['metricCard--primary']}`}>
  <div className={styles['metricCard__header']}>标题</div>
  <div className={styles['metricCard__value']}>42</div>
</div>
```

> 但在 CSS Modules 下，`__`/`--` 的"防冲突"价值变低——更常见的是直接 camelCase（`metricCardHeader`），通过 `styles.metricCardHeader` 访问更顺手。**约定**：组件内 className 用 camelCase，不再追求 BEM 的 kebab-case。

### 2.5 优缺点

**优点**：

- **零运行时成本**：纯构建时转换，最终产物就是普通 CSS，浏览器无需额外解析。
- **学习曲线低**：写法仍是标准 CSS / Sass，只是文件名后加 `.module`。
- **作用域天然隔离**：再也不用想"这个类名会不会和别人撞"。
- **与所有框架兼容**：React / Vue / Svelte / SolidJS / 原生 webpack 项目都支持。
- **SSR / RSC 友好**：纯 CSS 产物，无运行时依赖，Next.js App Router 官方推荐。

**缺点**：

- **类名拼接繁琐**：多类名组合时需要 `clsx` / `classnames` 辅助。
- **动态样式不便**：无法像 CSS-in-JS 那样根据 props 直接生成样式，要么用 CSS 变量、要么靠 `:global` 逃逸。
- **样式与组件分离**：仍是两份文件，不如 CSS-in-JS 同文件共置。
- **跨组件复用难**：`composes` 不能跨文件随意引用（要 `composes: xxx from './shared.module.css'`，啰嗦）。

### 2.6 工程化建议

1. **配合 `clsx` 处理多类名**：

   ```tsx
   import clsx from 'clsx';
   import styles from './Button.module.css';

   <button className={clsx(styles.base, props.primary && styles.primary, props.disabled && styles.disabled)} />
   ```

2. **配合 [TypeScript 类型生成](https://github.com/mrmckeb/typescript-plugin-css-modules)**：让 `styles.foo` 在编辑器中可补全、可跳转，并对拼错的类名报错。

3. **统一类名风格**：在 `.module.css` 中用 camelCase 命名（`metricCard` 而非 `metric-card`），避免 `styles['metric-card']` 这种丑陋的写法。

4. **设计 token 用 `:root` + CSS 变量**：把全局 token（颜色、间距、字号）放在 `globals.css` 的 `:root` 中，CSS Modules 文件内引用 `var(--color-primary)`，便于主题切换。

5. **CI 检测无用类名**：[stylelint-no-unused-selectors](https://github.com/silverwind/stylelint-no-unused-selectors) 或自定义脚本，避免随项目迭代积累死代码。

---

## 三、CSS-in-JS

### 3.1 核心思想与流派

**CSS-in-JS** 把样式表达成 JavaScript 模块——可以是模板字符串、对象、函数。最早由 Christopher Chedeau（Facebook）2014 年的 ["React: CSS in JS"](https://speakerdeck.com/vjeux/react-css-in-js) 演讲推动。

社区按"样式生成时机"分两大流派：

| 流派 | 代表 | 生成时机 | 运行时开销 |
| ---- | ---- | ------- | --------- |
| **运行时 CSS-in-JS** | styled-components、Emotion、Stitches v1 | 浏览器运行时 | 有（每次渲染计算并注入样式） |
| **零运行时 CSS-in-JS** | vanilla-extract、Linaria、Panda CSS、Compiled | 构建时 | 无（产物是静态 CSS 文件） |

> 2023 年之后，**社区共识明显倾向于零运行时**：[Emotion 维护者公开质疑运行时方案](https://github.com/emotion-js/emotion/issues/2800)、[Next.js App Router 不支持 styled-components](https://nextjs.org/docs/app/building-your-application/styling/css-in-js)、[GitHub 把 Primer 从 styled-components 迁回 CSS Modules](https://github.blog/2024-01-12-how-we-improved-push-processing-on-github/)。**新项目优先选零运行时方案**。

### 3.2 styled-components / Emotion（运行时）

[styled-components](https://styled-components.com/) 的标志性 API——**带样式的组件**：

```tsx
import styled from 'styled-components';

const Card = styled.div<{ $primary?: boolean }>`
  padding: 16px;
  border-radius: 8px;
  background: ${p => p.$primary ? '#2563eb' : '#fff'};
  color: ${p => p.$primary ? '#fff' : '#111'};
`;

<Card $primary>Hello</Card>
```

[Emotion](https://emotion.sh/) 同时支持模板字符串和对象语法，API 更灵活：

```tsx
import { css } from '@emotion/react';

<div css={css`
  padding: 16px;
  color: ${theme.colors.primary};
`} />

// 或对象语法
<div css={{ padding: 16, color: theme.colors.primary }} />
```

**核心差异**：

| 特性 | styled-components | Emotion |
| ---- | ----------------- | ------- |
| API | 主推 `styled.tag` | `styled` + `css` prop 双形态 |
| 性能 | 一般 | 通常更快 |
| 包体积 | ~12kb | ~7kb |
| 维护活跃度 | **缓慢**（2024 后基本停滞） | 较活跃 |
| RSC 兼容 | ❌ | ❌ |

### 3.3 vanilla-extract / Linaria / Panda（零运行时）

零运行时方案在**构建时**把 JS 中写的样式抽出来，产物是普通 CSS 文件，浏览器中没有 JS 样式注入逻辑。

**[vanilla-extract](https://vanilla-extract.style/)**（Seek 团队出品，TS 友好）：

```ts
// Card.css.ts
import { style } from '@vanilla-extract/css';

export const card = style({
  padding: 16,
  borderRadius: 8,
  background: '#fff',
  selectors: {
    '&:hover': { background: '#f9fafb' }
  }
});
```

```tsx
import * as styles from './Card.css';
<div className={styles.card} />
```

特点：**完全类型安全**（写 `padding: 'lage'` 直接 TS 报错）、产物等同 CSS Modules、支持设计 token / variants 抽象（`recipe`、`createTheme`）。

**[Linaria](https://linaria.dev/)**：写法接近 styled-components，但构建时静态提取：

```ts
const Card = styled.div`
  padding: 16px;
  background: ${theme.colors.bg};
`;
```

**[Panda CSS](https://panda-css.com/)**（Chakra UI 团队的下一代方案）：类似 vanilla-extract，但带有 Tailwind 风格的预设 token 系统：

```tsx
<div className={css({ p: 4, bg: 'primary.500', rounded: 'lg' })} />
```

### 3.4 主题与设计 Token

CSS-in-JS 在**主题切换**上的优势最为突出——把 theme 当成 JS 对象传给组件树：

```tsx
import { ThemeProvider } from 'styled-components';

const lightTheme = { colors: { primary: '#2563eb', bg: '#fff' } };
const darkTheme  = { colors: { primary: '#3b82f6', bg: '#0f172a' } };

<ThemeProvider theme={isDark ? darkTheme : lightTheme}>
  <App />
</ThemeProvider>

// 组件内
const Card = styled.div`
  background: ${p => p.theme.colors.bg};
  color: ${p => p.theme.colors.primary};
`;
```

> **现代实践**：即使用 CSS-in-JS，也建议主题用 **CSS 变量**而不是 props（避免运行时重渲染所有组件）：
>
> ```tsx
> const Card = styled.div`background: var(--color-bg);`;
> ```

### 3.5 RSC / SSR 兼容性陷阱

**Next.js App Router（13.4+）默认使用 React Server Components (RSC)**，对运行时 CSS-in-JS 极不友好：

- **styled-components 必须把组件标记为 `"use client"`**，否则报错——意味着**服务端组件无法直接用 styled**，整个组件树会被强行降级为客户端组件，丢掉 RSC 的全部收益。
- **Emotion 同上**：[官方说明](https://nextjs.org/docs/app/building-your-application/styling/css-in-js)中 Next.js 明确写道："如果你使用 RSC，请避免运行时 CSS-in-JS"。

**官方建议**（Next.js 14 / 15）：

1. 首选 **Tailwind CSS** 或 **CSS Modules**（纯构建时方案）。
2. 需要 JS 表达力时选 **vanilla-extract**、**Panda**、**Linaria** 等零运行时方案。
3. 若必须用 styled-components / Emotion，按 [官方 Style Registry 教程](https://nextjs.org/docs/app/building-your-application/styling/css-in-js#styled-components) 注入样式，并接受全树 `use client` 的代价。

### 3.6 优缺点

**优点**：

- **样式与组件共置**：单文件即可读完组件的结构 + 样式 + 行为。
- **动态样式天然顺手**：直接 `props => ...`，比 `clsx` + CSS 变量优雅。
- **TS 类型安全**（尤其 vanilla-extract）：props 拼错直接编译报错。
- **主题切换灵活**：JS 对象级别的主题，逻辑表达力最强。

**缺点**：

- **运行时方案性能差**：每次渲染都要计算字符串、hash、注入 `<style>` 标签，列表场景容易掉帧。
- **SSR / RSC 复杂**：需要 Style Registry，写错就 hydration mismatch；与 React Server Components 几乎不兼容。
- **包体积膨胀**：运行时方案多打 7~12kb 的库代码。
- **DevTools 调试体验差**：类名是 `sc-jXbUNg jvCTkj`，开发者工具中难定位。
- **生态分裂**：styled-components、Emotion、Linaria、vanilla-extract、Panda... 选型本身就是负担。

---

## 四、Tailwind CSS

### 4.1 核心思想：Utility-First

[Tailwind CSS](https://tailwindcss.com/)（Adam Wathan，2017）把 CSS 拆成成百上千个**单一职责的原子类**，开发者直接在 HTML 里组合：

```html
<button class="px-4 py-2 rounded-md bg-blue-600 hover:bg-blue-700 text-white font-medium">
  Submit
</button>
```

每个类名对应一条 CSS 声明：

```css
.px-4 { padding-left: 1rem; padding-right: 1rem; }
.bg-blue-600 { background-color: #2563eb; }
.hover\:bg-blue-700:hover { background-color: #1d4ed8; }
```

**与传统 CSS 的根本差异**：

- 传统：先想"这是什么组件 → 写一个语义类名 → 写样式"。
- Tailwind：先想"我要什么视觉效果 → 直接拼工具类"。

> 一句话："**约束的设计系统 + 写完即可丢的内联样式 = 让你又快又一致**"。

### 4.2 基础用法

**项目接入**（Tailwind v4，2025 后主流版本）：

```bash
npm install tailwindcss @tailwindcss/vite
```

```ts
// vite.config.ts
import tailwindcss from '@tailwindcss/vite';
export default { plugins: [tailwindcss()] };
```

```css
/* app.css */
@import "tailwindcss";
```

**常用类名速记**：

| 维度 | 类名前缀 / 示例 |
| ---- | -------------- |
| 间距 | `p-{n}`（padding）/ `m-{n}`（margin）/ `gap-{n}`/ `space-x-{n}` |
| 尺寸 | `w-{n}` / `h-{n}` / `min-w-{n}` / `max-h-screen` |
| 颜色 | `text-blue-600` / `bg-gray-100` / `border-red-500` |
| 布局 | `flex` / `grid grid-cols-3` / `items-center` / `justify-between` |
| 文字 | `text-sm` / `font-semibold` / `leading-relaxed` / `tracking-wide` |
| 状态 | `hover:` / `focus:` / `active:` / `disabled:` / `aria-checked:` |
| 响应式 | `sm:` / `md:` / `lg:` / `xl:` / `2xl:`（默认移动优先） |
| 暗黑 | `dark:bg-slate-800` |

### 4.3 设计 Token 与主题

Tailwind 的核心价值不是"工具类"，而是**预设好的设计 token 系统**——颜色、间距、字号、阴影全在一套有限选择中，**强制一致性**。

**v4 配置（CSS 优先）**：

```css
@import "tailwindcss";

@theme {
  --color-brand-50: #eff6ff;
  --color-brand-500: #2563eb;
  --color-brand-900: #1e3a8a;

  --font-sans: "Inter", system-ui, sans-serif;

  --spacing: 0.25rem; /* 1 单位 = 4px */
}
```

之后即可直接用：`bg-brand-500`、`text-brand-900`、`font-sans`。

**暗黑模式**（v4 默认基于 CSS 媒体查询，可改用 class）：

```css
@variant dark (&:where(.dark, .dark *));
```

```html
<div class="bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100">
  自动跟随主题切换
</div>
```

### 4.4 组件抽象：`@apply` vs `cva` vs 组件封装

当一组工具类重复出现，三种抽象方式：

#### 方式 1：`@apply`（CSS 层抽象）

```css
.btn-primary {
  @apply px-4 py-2 rounded-md bg-blue-600 text-white font-medium hover:bg-blue-700;
}
```

> 适合**少量、稳定的"基础类"**，但 Tailwind 官方明确不推荐过度使用——会重新陷入 BEM 命名困境，丧失原子化优势。

#### 方式 2：组件封装（推荐）

```tsx
// Button.tsx
export function Button({ children, primary }: { children: ReactNode; primary?: boolean }) {
  return (
    <button className={clsx(
      "px-4 py-2 rounded-md font-medium",
      primary ? "bg-blue-600 hover:bg-blue-700 text-white" : "bg-gray-100 hover:bg-gray-200 text-gray-800"
    )}>
      {children}
    </button>
  );
}
```

#### 方式 3：[CVA](https://cva.style/)（class-variance-authority，shadcn/ui 同款）

```ts
import { cva } from "class-variance-authority";

const button = cva("px-4 py-2 rounded-md font-medium", {
  variants: {
    intent: {
      primary: "bg-blue-600 hover:bg-blue-700 text-white",
      secondary: "bg-gray-100 hover:bg-gray-200 text-gray-800",
      danger: "bg-red-600 hover:bg-red-700 text-white"
    },
    size: {
      sm: "text-sm px-3 py-1",
      lg: "text-lg px-6 py-3"
    }
  },
  defaultVariants: { intent: "primary", size: "sm" }
});

<button className={button({ intent: "danger", size: "lg" })}>删除</button>
```

> **现代最佳实践**：组件库优先用 **Tailwind + CVA**（[shadcn/ui](https://ui.shadcn.com/) 的官方写法），少量基础类用 `@apply`，**避免把 Tailwind 重新写成 BEM**。

### 4.5 Tailwind v4 的变化

v4（2025 年 1 月正式发布）是一次重大升级，对老项目迁移有影响：

| 维度 | v3（2021–2024） | v4（2025+） |
| ---- | --------------- | ----------- |
| 配置文件 | `tailwind.config.js` | **CSS 中用 `@theme`**（仍可选保留 JS 配置） |
| 构建引擎 | PostCSS | **Oxide（Rust 编写）**，构建快 5–10 倍 |
| 入口写法 | `@tailwind base; @tailwind components; @tailwind utilities;` | `@import "tailwindcss";` |
| 浏览器要求 | 较宽松 | **现代浏览器**（依赖 `@property`、`color-mix()`、`:is()` 等） |
| JIT | 默认 | 是唯一模式 |
| 默认颜色空间 | sRGB | **OKLCH**，色阶更线性 |

> **迁移建议**：v3 项目无紧迫需求可暂缓迁移；新项目直接 v4。详见 [Tailwind v4 升级指南](https://tailwindcss.com/docs/upgrade-guide)。

### 4.6 优缺点

**优点**：

- **开发速度极快**：无需切到 CSS 文件、无需想类名、无需 BEM。
- **强制一致性**：所有颜色 / 间距来自预设 token，避免"这里 `padding: 14px`、那里 `padding: 15px`"的视觉漂移。
- **产物极小**：JIT 模式下只输出页面用到的类，生产 CSS 通常 < 20kb（gzip）。
- **响应式 / 暗黑 / 状态原生支持**：`md:hover:bg-blue-700` 一行写完。
- **与 RSC 完全兼容**：纯 CSS 产物，无运行时。
- **生态丰富**：[shadcn/ui](https://ui.shadcn.com/)、[Headless UI](https://headlessui.com/)、[Radix Themes](https://www.radix-ui.com/)、[Tailwind UI](https://tailwindui.com/) 形成完整体系。

**缺点**：

- **HTML 看起来"脏"**：长类名串影响可读性，需要靠 Prettier 插件 + 组件抽象缓解。
- **学习成本**：上千个类名要记，初期开发反而变慢（**通常 1–2 周后反超**）。
- **设计自由度受限**：偏离 token 系统时需要 `[arbitrary-value]` 逃逸：`p-[13px]`、`bg-[#abcdef]`，但这是 anti-pattern。
- **复用机制弱于语义类**：一段视觉效果想用 5 处，要么复制类名串、要么抽组件、要么 `@apply`——都不如 BEM 的 `.metric-card--primary` 直观。
- **设计师协作摩擦**：设计稿用 Figma token，开发用 Tailwind token，两套 token 要同步。

### 4.7 工程化建议

1. **强制 [prettier-plugin-tailwindcss](https://github.com/tailwindlabs/prettier-plugin-tailwindcss)**：自动按官方推荐顺序排序类名，避免 PR 中类名顺序大乱斗。

2. **`clsx` / `cn` 工具**：动态类名拼接的标配。shadcn/ui 风格的 `cn` 函数：

   ```ts
   import { clsx, type ClassValue } from "clsx";
   import { twMerge } from "tailwind-merge";

   export function cn(...inputs: ClassValue[]) {
     return twMerge(clsx(inputs));
   }
   ```

   `twMerge` 解决"覆盖冲突"：`cn("px-4", "px-6")` → `"px-6"`，避免重复类名都被保留。

3. **设计 token 单一来源**：Figma → JSON token → `@theme` CSS 变量，用 [Style Dictionary](https://amzn.github.io/style-dictionary/) / [Tokens Studio](https://tokens.studio/) 同步。

4. **ESLint 规则**：[eslint-plugin-tailwindcss](https://github.com/francoismassart/eslint-plugin-tailwindcss) 检查类名拼写、冲突、顺序。

5. **逃逸值（arbitrary value）当成 anti-pattern**：在 lint 规则中限制 `[xxx]` 写法，逼迫团队扩展 token 而非随手写魔法值。

6. **组件库优先用 [shadcn/ui](https://ui.shadcn.com/)**：源码可粘到项目里，结合 Radix + CVA + Tailwind，已成 2025 年 React 组件库事实标准。

---

## 五、四方案横向对比

| 维度 | BEM + Sass | CSS Modules | CSS-in-JS（运行时） | CSS-in-JS（零运行时） | Tailwind |
| ---- | ---------- | ----------- | ------------------- | --------------------- | -------- |
| **学习成本** | 中（命名约定） | 低 | 中 | 中–高 | 高（前期），低（后期） |
| **运行时开销** | 无 | 无 | 有（中–高） | 无 | 无 |
| **作用域隔离** | 靠命名约定 | 构建时哈希 | JS 运行时 | 构建时哈希 | 无需（无语义类） |
| **动态样式** | CSS 变量 | CSS 变量 | **天然顺手** | TS 类型 + recipe | `clsx` + 条件类 |
| **主题切换** | CSS 变量 | CSS 变量 | **JS 对象 + Provider** | CSS 变量 + createTheme | CSS 变量 + class |
| **TS 类型** | 无 | 可选（插件） | 部分 | **完整** | 通过 CVA |
| **RSC 兼容** | ✅ | ✅ | ❌ | ✅ | ✅ |
| **包体积** | 0 | 0 | +7–12kb | 0（运行时） | +0（按需） |
| **DevTools 调试** | 优 | 良 | 差 | 良 | 差（长类名） |
| **设计系统强制力** | 弱 | 弱 | 中 | 强（TS 约束） | **强**（token 约束） |
| **代表项目** | Element Plus、Bootstrap | Next.js 官方推荐、Vercel | 早期 Stripe、Shopify Polaris | Shopify Hydrogen、Seek | Vercel、Linear、shadcn/ui |

---

## 六、大厂 / 开源项目对照

| 公司 / 项目 | 当前方案 | 备注 |
| ---------- | ------- | ---- |
| **Vercel（Next.js 官方站）** | Tailwind | RSC + Tailwind 是官方主推组合 |
| **GitHub Primer** | CSS Modules（2024 迁移自 styled-components） | [迁移原因](https://github.blog/2024-01-12-how-we-improved-push-processing-on-github/)：性能 + RSC 兼容 |
| **Linear** | Tailwind + CSS-in-JS 混合 | 静态部分 Tailwind，复杂动态组件 CSS-in-JS |
| **Shopify Polaris / Hydrogen** | vanilla-extract（迁移自 Sass） | TS 类型安全 + 零运行时 |
| **Stripe Dashboard** | 内部 CSS-in-JS（早期） + CSS Modules | 性能敏感场景已逐步迁出运行时方案 |
| **Discord** | CSS Modules + Stylus | 老项目维护中 |
| **Notion** | styled-components | 历史遗留，迁移成本高 |
| **shadcn/ui** | Tailwind + CVA + Radix | 2024 年起的事实标准 |
| **Tailwind UI / Tailwind Plus** | 纯 Tailwind | Tailwind 团队官方组件库 |
| **Ant Design** | Less + CSS-in-JS（v5 改为内置 `@ant-design/cssinjs`） | 内置主题切换 |
| **Element Plus** | Sass + BEM（`el-` 命名空间） | 经典方案的代表 |
| **Chakra UI v2** | Emotion | v3 切换到 Panda CSS（零运行时） |
| **MUI（Material UI）** | Emotion（v5+） | 与 RSC 兼容性需配置 Style Registry |

> **趋势**：2024–2025 年 React 生态明显**从运行时 CSS-in-JS 撤退**，迁移目的地分两支——**Tailwind + CVA**（应用层）和 **vanilla-extract / Panda**（组件库层）。

---

## 七、组合实践

各方案并非互斥，**生产项目常常混用**：

### 7.1 Tailwind + CSS Modules（推荐）

- 全局布局、间距、颜色 → Tailwind utility 类
- 复杂动画 / 罕见的 selector（如 `:has()` / 长 keyframes）→ `*.module.css`

```tsx
import styles from './Chart.module.css';

<div className={clsx("p-4 rounded-lg bg-white shadow", styles.chartContainer)}>
  {/* ... */}
</div>
```

### 7.2 Tailwind + CSS Variables（主题系统）

- 设计 token 用 CSS 变量定义在 `:root` / `.dark`
- Tailwind 通过 `@theme` 读取 CSS 变量，实现"运行时主题切换 + 静态产物"

```css
:root {
  --color-bg: #ffffff;
  --color-fg: #0f172a;
}
.dark {
  --color-bg: #0f172a;
  --color-fg: #f8fafc;
}

@theme {
  --color-bg: var(--color-bg);
  --color-fg: var(--color-fg);
}
```

### 7.3 CSS Modules + BEM（遗留项目维护）

- 在 `*.module.scss` 内部用 BEM 写法
- 外部通过 `styles.metricCard__header` 引用，作用域隔离由构建保障

### 7.4 组件库用 vanilla-extract，应用层用 Tailwind

- 组件库需要严格 TS 类型 + 主题系统 + 零运行时 → vanilla-extract / Panda
- 应用层快速搭页面、布局 → Tailwind
- 例：Shopify 的内部组合即如此

---

## 八、迁移路径建议

### 8.1 从 BEM + Sass 迁移

1. → CSS Modules：成本最低，只需 `.scss` 改名 `.module.scss`，类名访问改成 `styles.xxx`。
2. → Tailwind：成本较高，建议**按页面 / 模块逐步迁移**，新页面用 Tailwind、旧页面保留 BEM，长期共存。

### 8.2 从运行时 CSS-in-JS（styled-components / Emotion）迁移

1. → CSS Modules：保留样式逻辑，把模板字符串改成 CSS 文件，动态 props 改用 CSS 变量 + className 条件拼接。参考 [GitHub Primer 的迁移帖](https://github.blog/2024-01-12-how-we-improved-push-processing-on-github/)。
2. → Tailwind：适合视觉风格统一的应用，迁移过程同时重构视觉规范。
3. → vanilla-extract / Panda：保留 JS 表达力，去掉运行时开销。

### 8.3 新项目零成本起步

- **简单页面 / 营销站**：Tailwind 直接起步。
- **中后台业务系统**：Tailwind + shadcn/ui + CVA。
- **组件库 / 设计系统**：vanilla-extract / Panda + Tailwind（消费侧）。
- **Next.js App Router 项目**：Tailwind 或 CSS Modules，**不要选 styled-components / Emotion**。

---

## 参考资料

### CSS Modules

- [CSS Modules 规范（GitHub）](https://github.com/css-modules/css-modules)
- [Next.js — CSS Modules 文档](https://nextjs.org/docs/app/building-your-application/styling/css-modules)
- [typescript-plugin-css-modules](https://github.com/mrmckeb/typescript-plugin-css-modules)

### CSS-in-JS

- [Christopher Chedeau — React: CSS in JS（2014）](https://speakerdeck.com/vjeux/react-css-in-js)
- [styled-components 官方文档](https://styled-components.com/)
- [Emotion 官方文档](https://emotion.sh/)
- [vanilla-extract 官方文档](https://vanilla-extract.style/)
- [Linaria 官方文档](https://linaria.dev/)
- [Panda CSS 官方文档](https://panda-css.com/)
- [Next.js — CSS-in-JS 兼容性说明](https://nextjs.org/docs/app/building-your-application/styling/css-in-js)
- [GitHub — How we improved push processing on GitHub（含迁移说明）](https://github.blog/2024-01-12-how-we-improved-push-processing-on-github/)
- [Sam Magura — Why We're Breaking Up with CSS-in-JS](https://dev.to/srmagura/why-were-breaking-up-wiht-css-in-js-4g9b)

### Tailwind

- [Tailwind CSS 官方文档](https://tailwindcss.com/)
- [Tailwind v4 升级指南](https://tailwindcss.com/docs/upgrade-guide)
- [Adam Wathan — CSS Utility Classes and "Separation of Concerns"](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/)
- [shadcn/ui](https://ui.shadcn.com/)
- [class-variance-authority (CVA)](https://cva.style/)
- [tailwind-merge](https://github.com/dcastil/tailwind-merge)
- [prettier-plugin-tailwindcss](https://github.com/tailwindlabs/prettier-plugin-tailwindcss)

### 综合 / 对比

- [Josh W. Comeau — The Styling Landscape Today](https://www.joshwcomeau.com/css/styled-components/)
- [State of CSS — 2024 Survey Results](https://stateofcss.com/)
- [Vercel — Styling in Next.js App Router](https://nextjs.org/docs/app/building-your-application/styling)
