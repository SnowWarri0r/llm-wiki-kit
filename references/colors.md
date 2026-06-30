# 调色板与配色记录

## 基础调色板（每篇 paper 都用）

```css
--paper:       #f0e9d6;    /* 主背景 · 米黄 */
--paper-2:     #e8e0c8;    /* 次背景 */
--paper-3:     #faf4e1;    /* 卡片背景 */
--paper-card:  #f7f1de;    /* figure 背景 */
--ink:         #181410;    /* 主文字 · 近黑 */
--ink-2:       #3a3128;    /* 次文字 */
--muted:       #7a6f5d;    /* 灰文字 */
--rule:        #bfb398;    /* 分隔线 */
```

paper / ink 这套不要动 —— 是整个 wiki 的视觉一致性来源。

## 强调色（每篇 paper 选一个做 primary accent）

```css
--brick:        #9b2c2c;   /* 砖红 · 强 · GPT / decoder / 生成 / 原始 attention */
--brick-soft:   #efd6c8;   /* 浅砖 · 配 brick 用 */
--ochre:        #b8841c;   /* 金黄 · 中 · 中间状态 / 强调 */
--ochre-soft:   #f0e0a8;   /* 浅金 · 配 ochre 用 */
--moss:         #4a6b3a;   /* 苔绿 · 中 · BERT / encoder / 理解 */
--moss-soft:    #d8e6ce;   /* 浅绿 */
--deep:         #1f3a5f;   /* 深蓝 · 强 · ResNet / foundation / 基础设施 */
--deep-soft:    #c8d4e2;   /* 浅蓝 */
```

## 已用配色（避免冲突）

| paper | primary | 备注 |
|---|---|---|
| attention-is-all-you-need | brick | 始祖 · 红色突出 |
| resnet | deep | 基础设施 · 蓝 |
| bert | moss | 理解 · 绿 |
| gpt-1 | brick | 跟 attention 同色，因为是同源；通过其他元素区分 |
| fish-speech-s2-pro | 混（moss + brick） | 多模态 |
| interaction-models-tml | 混（brick + moss） | 多角度 |

## 配色决策原则

1. **同方向 / 同思想路线用同色系**：GPT 系都用 brick，BERT 系都用 moss
2. **避开同领域已用色**：做 RoBERTa 就不用 moss（避免跟 BERT 混），可考虑 ochre 区分
3. **2-3 篇相同色系可以**：GPT-1/2/3 都 brick 是 OK 的，靠副元素区分（如 hero title font weight、accent line 不同）
4. **混色谨慎用**：用于跨领域综合 paper（如 fish-speech 同时讲音频 + LLM）

## 使用位置

primary accent 出现在：
- `h1.title .em` 斜体强调部分
- `h2 .num` 节号
- `figcaption .fnum` 图编号
- `.tldr` 左边框
- `.note .l` note 标签色
- `a` 链接色
- `figcaption span:first-child` accent
- 各 figure 内 "成功 / 推荐" 的染色（如 GPT-1 表格 "brick-win" 单元格）

避免用：
- 在 background 大面积铺设（会喧宾夺主，bg 还是用 paper-card）
- 在 muted text 上覆盖（会失去对比度）
