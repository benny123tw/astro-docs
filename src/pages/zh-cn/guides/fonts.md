---
title: 使用自定义字体
description: 想要为 Astro 生成的网站添加一些自定义的字体？使用 Google Fonts 或者使用本地文件以添加自定义字体。
layout: ~/layouts/MainLayout.astro
setup: |
    import PackageManagerTabs from '~/components/tabs/PackageManagerTabs.astro';
---

Astro 支持所有常见的在网站中添加自定义字体的方法。本指南将向您展示在项目中包含 Web 字体的两种不同选项。

## 使用本地字体文件

如果你想要添加一些自定义字体文件，我们推荐你将它们添加至 [`public/` 目录](/zh-cn/core-concepts/project-structure/#public)。接下来你便可以在 CSS 中使用 `@font-face` 语句注册字体，并使用该 `font-family` 属性设置页面的字体。

### 例子

想象如果你有一个字体文件 `炫酷字体.woff`。

1. 首先将你的自填添加到 `public/fonts/`。

2. 在你的 CSS 中添加一个 `@font-face` 语句。这可能位于你导入的全局 `.css` 文件中，也可能位于你要使用此字体的布局或组件中的 `<style>` 块中。

    ```css
    /* 注册你的字体并告诉浏览器它在哪里 */
    @font-face {
      font-family: 'CoolFont';
      src: url('/fonts/炫酷字体.woff') format('woff');
      font-weight: normal;
      font-style: normal;
      font-display: swap;
    }
    ```

    :::note
    我们将 `/public` 在字体的源链接中，因为所有在 `/public` 目录下的文件都会被添加到网站的根目录中。
    :::

3. 使用 `font-family` 为组件或布局中的元素设置字体样式。在此示例中，自定义字体将应用于 `<h1>` 标题，而不会用于段落 `<p>`。

    ```astro {10-12}
    ---
    // src/pages/example.astro
    ---

    <h1>鸡汤来喽！</h1>

    <p>自定义字体太有趣了！</p>

    <style>
    h1 {
      font-family: 'CoolFont', sans-serif;
    }
    </style>
    ```

## 使用字体资源包

[Fontsource](https://fontsource.org/)（字体资源包）项目简化了使用 Google Fonts 和其他开源字体的过程。当你想用什么字体时，只需安装相应的npm模块并稍作配置即可。

1. 从 [Fontsource 的字体资源目录](https://fontsource.org/fonts)中找到你想用的字体。比如在这个例子中，我们将选择 [Twinkle Star](https://fontsource.org/fonts/twinkle-star)字体。

2. 安装你选择的字体的包。

    <PackageManagerTabs>
      <Fragment slot="npm">
      ```shell
      npm i @fontsource/twinkle-star
      ```
      </Fragment>
      <Fragment slot="pnpm">
      ```shell
      pnpm i @fontsource/twinkle-star
      ```
      </Fragment>
      <Fragment slot="yarn">
      ```shell
      yarn add @fontsource/twinkle-star
      ```
      </Fragment>
    </PackageManagerTabs>

    :::tip
    你可以在 Fontsource 上每个字体页面的“快速安装”部分找到正确的包名称。包名总是以 `@fontsource/` 字体名称开头。
    :::

3. 在要使用字体的布局或组件中导入字体包。通常，需要在通用布局组件中执行此操作，以确保该字体在你的站点中可用。

    导入后 Astro 并添加 `@font-face` 以设置字体所需的必要属性。

    ```astro
    ---
    // src/layouts/BaseLayout.astro
    import '@fontsource/twinkle-star';
    ---
    ```

4. 使用该 `font-family` 字体的 Fontsource 页面上显示的。这将适用于可以在 Astro 项目中编写 CSS 的任何地方。

    ```css
    h1 {
      font-family: "Twinkle Star", cursive;
    }
    ```

## 更多资源

### 使用 Tailwind 添加字体

如果您使用的是 [Tailwind 集成](/zh-cn/guides/integrations-guide/tailwind/)，您可以直接为本地字体添加 `@font-face`，或使用 Fontsource 的 `import` 策略来注册您的字体，然后按照 Tailwind 的文档添加自定义字体。

### 了解 Web 字体如何运作

[MDN的 Web 字体指南](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Web_fonts) 介绍了这个话题。

### 为你的字体生成 CSS

[Font Squirrel 的 Web 字体生成器](https://www.fontsquirrel.com/tools/webfont-generator)会帮你准备字体。
