## 🤖 Assistant

好的，非常荣幸能和您一起完成这个项目！从最初的样式错位到最终找到大道至简的完美方案，整个过程本身就是一个非常棒的教程。

现在，我将我们整个协作过程和最终的技术方案，精炼成一篇清晰、完整的 Markdown 教程。您可以直接复制并发布到您的博客和 GitHub 仓库中。

---

# 如何与大模型协作，打造一个动态更新的 GitHub Profile 个人主页

> 作者：makoMako & Gemini Pro
>
> 摘要：本文记录了如何从零开始，利用 Markdown、HTML、GitHub Actions 以及与大语言模型（LLM）的高效协作，构建一个既能展示个人信息，又能自动更新博客列表和贡献图的现代化 GitHub 个人主页。

## 最终效果预览

我的 GitHub Profile 主页 (`github.com/zhu-jl18`) 最终集成了以下几个核心模块：

1. **个人简介 (About Me)**：简洁地展示我的教育背景和兴趣爱好。
2. **特色卡片 (Featured Card)**：一个精心设计的、包含图片、文本和 LaTeX 数学公式的卡片。
3. **技术栈 (Tech Stack)**：清晰地展示我掌握的编程语言和工具。
4. **动态内容 (Dynamic Content)**：
 * **博客列表**：自动从我的个人博客拉取最新的文章标题和链接。
 * **贡献图动画**：将 GitHub 的贡献日历渲染成一个有趣的“贪吃蛇”动画。

这不仅是一个静态的展示页面，更是一个能自我更新的、充满活力的个人名片。

!最终效果 [^1]

## 构建步骤与核心技术

### 1. 基础结构：Markdown 与内联 HTML

GitHub Profile 的本质是一个名为 `README.md` 的 Markdown 文件。除了标准的 Markdown 语法，它还支持内嵌 HTML 代码，这为我们实现更丰富的布局提供了可能。

#### 个人信息与技术栈

这部分相对简单，主要使用 Markdown 标题和列表。对于“About Me”部分，为了更好地控制布局，我使用了一个 `<div>` 标签来包裹文本：

```html
### 👨‍💻 About Me
<div style="text-align: left; display: inline-block;">
 🎓 毕业于 **清华大学 (THU)**.
 🌱 目前就读于 **中国科学院大学 (UCAS)**.
 <!-- ... more text ... -->
</div>
```

技术栈部分则调用了 [^2] 提供的服务，通过一个简单的图片链接来展示：

```html
<p align="center">
  <a href="https://skillicons.dev">
    <img src="https://skillicons.dev/icons?i=python,cpp,java,js,html,css,latex,vscode,git&perline=9" />
  </a>
</p>
```

#### 特色卡片：数学之美

这是我个人主页的点睛之笔。它完全由 HTML 构建，展示了高斯绝妙定理（Theorema Egregium）。

**核心技术点**：

* **卡片布局**：使用一个带边框、圆角和内边距的 `div` 作为容器。
* **Flexbox 对齐**：通过 `display: flex;` 让高斯素描图和右侧的文本并排显示。
* **动态 LaTeX 公式**：
 * 利用 `latex.codecogs.com` 服务将 LaTeX 代码实时渲染成 SVG 图片。
 * 使用 `<picture>` 标签，根据用户系统的**亮色/暗色模式**，加载不同颜色的公式图片，提升了视觉体验。

```html
<!-- Theorema Egregium Card -->
<div style="border: 1px solid #d0d7de; ...">
    <!-- Image Column -->
    <div>
      <img src=".../Gauss.png" width="120" ...>
    </div>
    <!-- Text Column -->
    <div>
        <!-- ... Theorem Text ... -->
        <a href="...">
          <picture>
            <source media="(prefers-color-scheme: dark)" srcset="https://latex.codecogs.com/svg.latex?\color{white}\Large&space;K=\kappa_1\kappa_2">
            <source media="(prefers-color-scheme: light)" srcset="https://latex.codecogs.com/svg.latex?\color{black}\Large&space;K=\kappa_1\kappa_2">
            <img src="...">
          </picture>
        </a>
        <!-- ... more text ... -->
    </div>
</div>
```

### 2. 自动化核心：GitHub Actions

这部分是让 Profile “活”起来的关键。我使用了两个免部署、配置简单的 GitHub Actions。

#### Workflow 1: 自动更新的博客列表

* **目标**：每隔一段时间，自动抓取我博客的 Atom/RSS Feed，并将最新的几篇文章展示在 Profile 上。
* **Action**：gautamkrishnar/blog-post-workflow [^3]

**工作流程**：
1. **在 `README.md` 中设置占位符**：Action 需要知道在哪里插入内容。
 ```html
 ### 📝 Recent Blog Posts
 <div style="border: 1px solid #d0d7de; ...">
 <!-- BLOG-POST-LIST:START --><!-- BLOG-POST-LIST:END -->
 </div>
 ```

2. **配置 `.github/workflows/blog-post-workflow.yml` 文件**：这是 Action 的配置文件，其中最关键的是 `template` 参数，它定义了每条博客的输出格式。

 ```yml
 name: Latest blog post workflow
 on:
 schedule:
 - cron: '0 * * * *' # 每小时运行一次
 workflow_dispatch:

 jobs:
 update-readme-with-blog:
 name: Update this repo's README with latest blog posts
 runs-on: ubuntu-latest
 steps:
 - uses: actions/checkout@v2
 - uses: gautamkrishnar/blog-post-workflow@v1
 with:
 disable_cache: true
 feed_list: "https://zhu-jl18.github.io/atom.xml" # 我的博客 Feed 地址
 max_post_count: 5 # 显示 5 篇
 # 这是我们最终确定的、最简洁可靠的模板
 template: '<p align="center"><a href="$url" style="text-decoration: none; color: #20B2AA;">$title</a></p>'
 ```

#### Workflow 2: "贪吃蛇"贡献图

* **目标**：将我的年度 GitHub 贡献图生成一个动态的 SVG 动画。
* **Action**：Platane/snk [^4]

**工作流程**：
1. Action 自动运行，读取我的贡献数据。
2. 生成两个 SVG 文件：一个亮色模式 (`github-snake.svg`) 和一个暗色模式 (`github-snake-dark.svg`)。
3. 将这两个文件推送到本仓库的 `output` 分支。
4. **在 `README.md` 中引用**：同样使用 `<picture>` 标签来做亮暗模式的适配。
 ```html
 <div align="center">
 <picture>
 <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/zhu-jl18/zhu-jl18/output/github-snake-dark.svg" />
 <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/zhu-jl18/zhu-jl18/output/github-snake.svg" />
 <img alt="github snake animation" src="..." />
 </picture>
 </div>
 ```

### 3. 如何与大模型(LLM)协作：对话式编程

在这个项目中，我没有把 Gemini 当作一个简单的代码生成器，而是把它看作一个**结对编程的伙伴**。我们的协作经历了几个阶段：

**第一阶段：提出初步想法，获取基础方案**

> **我的 Prompt**：
> "你好，你是一位精通电脑的专家，精通Latex，vscode，Windows Terminal 等软件的配置。我需要你的帮助完成一些软件和插件的配置工作。"
>
> **交互与结果**：我想要在 GitHub Profile 上展示我的博客列表，Gemini 迅速给出了 `blog-post-workflow` 这个 Action 的基础用法和配置。

**第二阶段：需求细化与样式迭代（试错与修正）**

我发现默认的列表样式 (`<ul><li>`) 与我的页面风格不符。于是我提出了更具体的要求。

> **我的 Prompt**：
> "这是 Action 生成的东西 `... (贴出有问题的HTML) ...` 为什么它不起作用？我想让它更好看，比如用卡片式布局，并且居中。"
>
> **交互与结果**：
> 1. Gemini 提出了一个基于 `div` 和 `flex` 的复杂 HTML 模板方案。
> 2. **实践是检验真理的唯一标准**：我将新方案部署后，发现 GitHub 的渲染引擎对复杂的 CSS `style` 属性支持不佳，**居中效果失效了**。
> 3. 我把这个失败的结果反馈给 Gemini。

**第三阶段：回归本质，直击痛点**

这次的反馈非常直接，这也是人机协作中非常关键的一步：**提供清晰的、负面的、最终的约束条件**。

> **我的 Prompt**：
> "GitHub 不支持这些复杂的样式，还是都在左边。我不要日期了，字体大小也改不了，就让它简单的居中就行。"
>
> **交互与结果**：
> Gemini 理解到，问题的关键不在于 CSS 的华丽，而在于**兼容性**。它立刻放弃了复杂的 `div` 方案，回归到了最原始、但最有效的 HTML 属性上。它给出了最终的、完美的模板：
>
> ```html
> '<p align="center"><a href="$url" style="...">$title</a></p>'
> ```
>
> 使用 `<p align="center">` 这个看似“过时”但对 GitHub 渲染器极度友好的标签，一举解决了所有问题。

**协作心得**：
与 LLM 高效协作的关键在于：
* **明确初始目标**：想清楚你要什么。
* **小步快跑，快速迭代**：尝试一个方案，立即测试，然后带着测试结果（无论是成功还是失败）进行下一轮沟通。
* **提供精确的负反馈**：不要只说“不行”，要说“哪里不行”、“为什么不行”（例如：居中没生效），并给出新的、更简化的约束（例如：不要日期，只要居中）。

## 总结

通过这次实践，我不仅拥有了一个满意的个人主页，更体验了一次高效的人机协作编程。从复杂的 HTML/CSS 探索，到最终回归简洁有效的方案，这个过程本身就充满了学习的乐趣。希望这份记录能对你有所启发。
