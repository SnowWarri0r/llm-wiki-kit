# concept md 模板

放在 `wiki/concepts/<slug>.md`。比 paper md 更紧凑。

```markdown
---
name: <slug-kebab>
type: concept
sources: [<paper-slug-1>, <paper-slug-2>]
updated: 2026-MM-DD
---

# <概念名 · 中文副名>

## 一句话
<不超过 30 字，最白话>

## 直觉
<跟用户已有 mental model 对照的类比>
<必给"它替代了什么 / 为什么需要它">

## 怎么做的
```
<伪代码 / 公式 / 流程>
```

<额外解释 trick>

## 数字例子
<硬规则（见 SKILL.md 3.6）：带公式/矩阵/变换/分布/概率/损失的概念页必有此段>
<用具体小数字（2×2 矩阵 / [2,1,0]），把每一步算出来，收尾能自检对上>
<抽象断言后立刻跟一个数字坐实，别只留符号公式>

## 跟 X 的对比 / 对照
<可选 · 横向对比表或 bullet>

## 链接
- [[paper-slug]] · 提出 / 用到的 paper
- [[相关概念]] · 关系说明
```

## 注意

- `sources` 字段要包含<strong>所有</strong>引用它的 paper slug，新 paper 引用旧 concept 时记得回去补
- 不要把整个 paper 的内容塞进 concept md —— concept 是<strong>跨 paper 共享</strong>的颗粒
- 内部链接用 Obsidian 风格 `[[<slug>]]`，不写成 markdown link
