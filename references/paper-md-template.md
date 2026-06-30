# paper md 模板

放在 `wiki/papers/<slug>.md`。frontmatter 必须填。

```markdown
---
name: <slug-kebab>
type: paper
source: <raw 路径或 arxiv URL>
upstream: <arxiv/openai/github URL>
ingested: 2026-MM-DD
authors: <作者列表 (机构) · 会议 + 年>
---

# <Paper 标题 · 中文副标题>

<一段非空的导言，用人话讲这篇 paper 的历史地位 / 它在 wiki 体系里位置>

## 一句话
**<用 30 字以内说出最核心的洞察>**

## 它要解决的痛点
<痛点 1 · 用 bullet 或段落>
<痛点 2 · 必须说明替代方案为什么不行>
<痛点 3 · trade-off>

## 核心贡献
1. **<标签>**：[[<concept-slug>]] —— 一句话说这个 contribution 干了啥
2. **<标签>**：...
3. ...

## 关键概念
- [[<concept-1>]] · 一行解释关系
- [[<concept-2>]] · 一行解释
- ...

## 我的批注
<3-7 条独立观察，每条 bullet>
- <最反直觉的洞察>
- <一个工程 trick 的解释>
- <历史评价>
- <跟现代工作的差距>

## 跟 wiki 里其他 paper 的关系
- [[<其他 paper>]] · 怎么承接 / 对照
- ...

## 历史定位
- YYYY-MM <前置 paper> · <一句话角色>
- YYYY-MM **<本篇>** · <差异化贡献>
- YYYY-MM <继承 paper> · <如何发展>
- ...
```

## 注意

- frontmatter 中 `upstream` URL 会被 render.py 在 index.html 里渲染成"↗ upstream"链接
- "我的批注"是<strong>必须有</strong>的，体现这是个人 wiki 不是论文摘要
- 不要在 paper md 里写动图 / 复杂图 —— 这些放 bespoke HTML
